In this tutorial we are going to integrate AWS' DynamoDB into a Next.JS application we have built in the previous tutorial.
> _`Amazon DynamoDB` is a fully managed NoSQL database service offered by Amazon Web Services (AWS)_

### Prerequisites
* Serverless
* Java Runtime Engine (JRE) version 6.x or newer

 <p class="markdown-paragraph">Change directories to your development directory and clone <a class="markdown-link" href="/blog/running-nextjs-app-on-a-lambda-function">Nextjs app on a Lambda function</a> repo.</p>

```bash
git clone https://github.com/teddynted/nextjs-on-a-lambda-function.git
cd nextjs-on-a-lambda-function
npm install
```

For the purpose of running our on localhost, we need to install `serverless-dynamodb-local` plugin.

```
brew cask install java
npm install --save-dev serverless-dynamodb-local
```

> _You also need to install `Java Runtime version` only if it's not installed._

Open `serverless.yml` and add following entry to the plugins array

```bash
plugins:
  - serverless-dynamodb-local
```

Install DynamoDB Local:

```bash
sls dynamodb install
```

## Configuration

Let's configure `DynamoDb` and add a table called `posts-dev` in `serverless.yml`:

### Create the Resource

Add the following to `serverless.yml`, we are basically describing our posts table and defining postId attribute as our primary key. we get `TableName` value from the custom variable `${self:custom.postsTableName}` in the next section.

BillingMode `PAY_PER_REQUEST` tells DynamoDB that we want to pay per request and use the On-Demand Capacity option.

```yaml
resources:
  Resources:
    PostsDynamoDBTable:
      Type: 'AWS::DynamoDB::Table'
      Properties:
        AttributeDefinitions:
          -
            AttributeName: postId
            AttributeType: S
        KeySchema:
          -
            AttributeName: postId
            KeyType: HASH
        BillingMode: PAY_PER_REQUEST
        TableName: ${self:custom.postsTableName}
```

Add the following custom: block to our `serverless.yml`, here we are defining a custom variable `postsTableName` that is used the above snippet and we also adding a basic configuration to our dynamodb.

> _table name should match your enviroment e.g `posts-dev`, `posts-uat` etc._

```yaml
custom:
  postsTableName: 'posts-${self:provider.stage}'
  dynamodb:
    stages:
      - dev
    start:
      migrate: true
      port: ${env:DYNAMODB_PORT, '8000'}
      seed: true
      inMemory: true
      convertEmptyValues: true

    seed:
        dev:
          sources:
            - table: ${self:custom.postsTableName}
              sources: [seeds/posts.json]
```

Add the `iamRoleStatements` property to the provider block in your `serverless.yml`, we are adding permissions to our DynamoDb  table within your account.

```yaml
provider:
  name: aws
  runtime: nodejs12.x
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:Scan
        - 'lambda:InvokeFunction'
      Resource:
        - { "Fn::GetAtt": ["PostsDynamoDBTable", "Arn" ] }
        - '*'
        - "arn:aws:dynamodb:${self:provider.region}:*:table/*"
  environment:
    POSTS_TABLE: ${self:custom.postsTableName}
    NODE_ENV: production
    LAMBDA: true
    STAGE: ${self:provider.stage}
```

The above `environment` variables are made available to our code under `process.env`.

### Seeding DynamoDb

Seeding it's a process of posting sample data into your dynamodb table, I will show you how to seed when deploying to dev environment. Let's do the following to create a sample json data.

```bash
mkdir seeds && cd seeds
touch posts.json
```

Add this content to `posts.json`:

```json
[
    {
        "post_title": "Lorem Ipsum",
        "author": "John Doe",
        "post_category": [
            {
                "value": "React",
                "label": "React"
            }
        ],
        "created_at": "2020-05-23T18:54:07.157Z",
        "post_desc": "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book.",
        "postId": "32f39119-42dd-4423-8601-2a0dae190527"
    }
]
```

> _Upon running `sls offline start` `posts` table will be seeded/populated with predefined json data._

### Query Dynamo DB

Let's get data from our dynamodb post table. Create a server directory in root of your project and add two files:

```javascript
mkdir server && cd server
touch dynamo.js postsModel.js
```

#### Model Posts Table

> _postsModel.js_

```javascript
const POSTS_TABLE = process.env.POSTS_TABLE;
const IS_OFFLINE = process.env.IS_OFFLINE;

const params = {
    TableName: POSTS_TABLE
};

const visible = ["postId", "post_title", "post_body"];

const transform = v => {
    console.log(v);
    return Object.keys(v)
        .filter(v => {
            return IS_OFFLINE ? true : visible.includes(v);
        })
        .reduce((obj, key) => {
            obj[key] = v[key];
            return obj;
        }, {});
};

module.exports = {
    params,
    visible,
    transform
};
```

#### DynamoDb

Set up DynamoDb so that we can perform operations against it.

> _dynamo.js_

```javascript
const AWS = require("aws-sdk");
const { IS_OFFLINE } = process.env;

if( IS_OFFLINE ) {
    AWS.config.update({
        region: "us-east-1",
        endpoint: "http://localhost:8000"
    });
}

const dynamoDb = new AWS.DynamoDB.DocumentClient();
const postsModel = require("./postsModel");

exports.getPosts = () => {
    return new Promise( async (resolve, revoke) => {
        const params = {
            TableName: postsModel.params.TableName,
        }
        dynamoDb.scan(params, (err, data) => {
            if (err) {
                revoke(err.message);
            } else {
                if( data.Items ) {
                    resolve(data.Items);
                } else {
                    resolve([]);
                }
            }
        });  
    });
};
```
##### Server

In the root directory, edit your custom server file to add API end-point that will query DynamoDB and send back data.

> _server.js_

```javascript
const express = require("express");
const next = require("next");
const bodyParser = require("body-parser");
const cors = require("cors");
const { LAMBDA, PORT, NODE_ENV } = process.env;
const dynamo = require("./server/dynamo");

const port = parseInt(PORT, 10) || 3000;
const dev = NODE_ENV !== "production";
const app = next({ dev });
const handle = app.getRequestHandler();

global.fetch = require("node-fetch");

const createServer = () => {
    const server = express();
    server.use(bodyParser.json({ limit: "50mb" }));
    server.use(cors());
    server.get("/posts", async (req, res) => {
        try {
            const data = await dynamo.getPosts();
            res.status(200).json({
                data: data
            });
        } catch (error) {
            res.status(500).json(error);
        }
    });
    server.get("*", (req, res) => handle(req, res));
    return server;
};

const server = createServer();

if (!LAMBDA) {
    app.prepare().then(() => {
        server.listen(port, err => {
            if (err) throw err;
            console.log(`Ready on http://localhost:${port}`);
        });
    });
}

exports.app = app;
exports.server = server;
```

##### Display data

In this section we're going to initiate a call in the front-end to get us data from the back-end and display that data using **Apisauce**

```bash
npm i apisauce --save
```

Let's also upgrade to the latest next package, since our cloned repo is on version 8.0.0:

```bash
npm uninstall next
npm install next
npm install babel-loader
```

Add the code below to this file _pages/index.js_:

```javascript
import React from 'react'
import { create } from 'apisauce'

const api = create({
  baseURL: "http://localhost:3000",
  headers: { 
      Accept: 'application/json',
      'Content-Type': 'application/json' 
  },
  timeout: 60000
});

const fetchData = async () => await api.get('/posts')
  .then(res => ({
    posts: res.data,
  }))
  .catch(() => ({
      posts: [],
    }),
  );

class Index extends React.Component {
  static async getInitialProps(ctx) {
    const res = await fetchData();
    const { posts } = res;
    return { posts }
  }
  render() {
      return <div>{JSON.stringify(this.props.posts)}</div>
  }
}

export default Index
```

##### Let's Run It!

Run this command:

```bash
sls offline start
```

And open http://localhost:3000 in a browser, You should be able to see the same result as the screenshot below:


![alt text](https://nextjs-portfolio.s3.amazonaws.com/nextjs-dynamodb-apisauce.jpg "Next.JS Data DynamoDb Query")

<p class="markdown-paragraph">Complete code can be found on <a class="markdown-link" href="https://github.com/teddynted/amazon-dynamodb-and-serverless-framework" target="_blank">Github</a>.</p>

