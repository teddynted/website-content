### So what's Next.js?

Next.js it's a server-side rendering ReactJS framework. SSR it's a process of rendering a javascript client-side website into a static html and css on the server. Poor SEO is major disadvantage of a traditional ReactJS applications, that's where SSR comes in.

### Benefits of SSR

* Fast rendering websites.
* Consistent SEO performance.

### Why SEO?

* It Increases traffic to your website.
* It's a cost-effective marketing strategy.
* It increases sales and leads.

### Prerequisites

* Node 12
* AWS CLI
* AWS account

<p class="markdown-paragraph">In this tutorial I will build a Next.js app and deploy to a lambda function using Bitbucket pipelines. Let's set it off by creating a <a class="markdown-link" href="/blog/aws-lambda-layers">lambda layer</a>, run theses commands in your development folder:</p>

```bash
mkdir nextjs-aws-on-lambda-layer && cd nextjs-aws-on-lambda-layer 
mkdir nodejs
cd ..
touch command.sh
```

Add these commands in a bash script:

![alt text](https://nextjs-portfolio.s3.amazonaws.com/layer-shell-script.png "AWS Lambda Layers")

When you run this bash script it will do the following:

* Deletes an existing deployment package.
* Changes directory to the nodejs.
* Install packages.
* Create a zipped deployment package.
* Deploy that package to an S3 bucket
* And lastly, a layer gets created.

Define your s3 bucket and deployment package name inside `s3.json` file:

```bash
touch s3.json
```

![alt text](https://nextjs-portfolio.s3.amazonaws.com/s3-bucket-json.png "AWS S3 bucket")

Head over to a terminal to execute our bash script:

```bash
./command.sh
```

Output:

![alt text](https://nextjs-portfolio.s3.amazonaws.com/shell-script-output.png "Shell script")

We will add `LayerVersionArn` value to our nextjs lambda function in the next few steps.

### Next.js Lambda function

In this section will create a lambda function that will depend on our existing lambda layer. Change directories to your development and run these commands:

```bash
serverless create \
--template aws-nodejs \
--path nextjs-on-a-lambda-function \
--name nextjs-aws-lambda
```
```bash
cd nextjs-on-a-lambda-function
mv handler.js index.js
mkdir pages && cd pages
touch index.js
cd ..
npm init -y
npm i copy-webpack-plugin@5.0.4 serverless-apigw-binary@0.4.4 serverless-offline@4.1.4 serverless-webpack@5.3.1 webpack-node-externals@1.7.2 --save-dev
touch binaryMimeTypes.js bitbucket-pipelines.yml webpack.config.js server.js
```
Add content to `pages/index.js`:

```javascript
export default () => (
    <div>Next.js on AWS lambda!</div>
)
```

Add wildcard binary mime type suport for AWS API Gateway `binaryMimeTypes.js`:
```javascript
module.exports = [
    '*/*'
]
```

#### Bitbucket

Add `bitbucket-pipelines.yml`:

```yaml
image: lambci/lambda:build-nodejs12.x

pipelines:
  branches:      
     dev:
        - step:
           name: Deploy to TEST
           deployment: test
           services:
             - docker
           caches:
             - node
             - docker
           script:
             - export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
             - export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
             - git remote set-url origin ${BITBUCKET_GIT_HTTP_ORIGIN}
             - npm i serverless -g
             - rm -rf .next
             - npm install
             - npm run build
             - sls deploy -v --stage dev
             - git push
```

Ensure that you create a repository for this project in bitbucket and connect it to Bitbucket this project:

<p align="center">
  <img src="https://nextjs-portfolio.s3.amazonaws.com/new-bitbucket-repository.png">
</p>

* In your Bitbucket respository navigate to `repository settings > settings` and enable pipelines.
* Set repository variables by adding your AWS `ACCESS_KEY_ID` and `SECRET_ACCESS_KEY` in `repository settings > Repository variables`.
* Create an ssh key on your local and add it to `repository settings > access keys`.

Head over to your working directory to initiate git and connect this project:

```bash
git init
git checkout -b dev
git remote add origin https://<username>@bitbucket.org/<username>/nextjs-on-a-lambda-function.git
```

Ensure that your branching model is setup like the image below:

<p align="center">
  <img src="https://nextjs-portfolio.s3.amazonaws.com/branching-model.png" alt="Branching model">
</p>

#### Webpack

Let's add webpack functionality to bundle our lambda function.

* `copy-webpack-plugin` - Copies individual files or entire directories, which already exist, to the build directory. 
* `webpack-node-externals` - Allows you to define externals - modules that should not be bundled, e.g node_modules

```javascript
const slsw = require("serverless-webpack");
const nodeExternals = require("webpack-node-externals");
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  entry: slsw.lib.entries,
  target: "node",
  // Since 'aws-sdk' is not compatible with webpack,
  // we exclude all node dependencies
  externals: [nodeExternals()],
  // Run babel on all .js files and skip those in node_modules
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: "babel-loader",
        include: __dirname,
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new CopyWebpackPlugin([
      { copyPermissions: true, from: '.next/**' },
      { copyPermissions: true, from: 'pages/**' },
      { copyPermissions: true, from: 'package.json' }
    ])
  ]
};
```

#### Server

Set up a custom express server for Next.js

```javascript
const next = require('next')
const express = require("express");
const { PORT, NODE_ENV, LAMBDA } = process.env;
const dev = NODE_ENV !== 'production'
const port = parseInt(PORT, 10) || 5000;
const app = next({ dev })
const handle = app.getRequestHandler()

const createServer = () => {
    const server = express();
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

And Edit `index.js` by adding these lines:

```javascript
'use strict';

const { app, server } = require("./server");
const awsServerlessExpress = require("aws-serverless-express"); 
const binaryMimeTypes = require("./binaryMimeTypes");

exports.handler = (event, context, callback) => {  
    app.prepare().then(() => { 
        return awsServerlessExpress.proxy(
            awsServerlessExpress.createServer(server, null, binaryMimeTypes), 
            event, 
            context
        );   
    }).catch(ex => {
        console.error(ex.stack)
        process.exit(1)
    });
};
```

> _`aws-serverless-express` gives us an ability to run serverless applications and REST APIs using your existing Node.js application framework, on top of AWS Lambda and Amazon API Gateway_

#### Serverless

Edit `serverless.yml` yaml file:

```yaml
service: nextjs

plugins:
  - serverless-webpack
  - serverless-offline
  - serverless-apigw-binary

provider:
  name: aws
  runtime: nodejs12.x

  environment:
    NODE_ENV: production
    LAMBDA: true
    STAGE: ${self:provider.stage}

functions:
  index:
    handler: index.handler
    events:
      - http: ANY /
      - http: ANY /{proxy+}
      - cors: true
    layers:
        - arn:aws:lambda:us-east-1:<account-id>:layer:dependencies-layer:1

custom:
  webpack:
    webpackConfig: 'webpack.config.js'
    keepOutputDirectory: false
    includeModules: false
    packager: 'npm'
    excludeFiles:
      - .serverless
      - .webpack 
      - .dynamodb

  apigwBinary:
    types: #list of mime-types
      - '*/*'

package:
  individually: true
```
> _Add an arn to a layer in our `serverless.yml` from the output we got when we deployed our Lambda layer._

### Ready to deploy?

I hope you didn't get bored along the way.

Before comitting and deploying code changes, edit `package.json`:

```json
{
  "name": "nextjs-on-a-lambda-function",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "copy-webpack-plugin": "^5.0.4",
    "serverless-apigw-binary": "^0.4.4",
    "serverless-offline": "^4.1.4",
    "serverless-webpack": "^5.3.1",
    "webpack-node-externals": "^1.7.2"
  },
  "dependencies": {
    "aws-serverless-express": "^3.3.6",
    "express": "^4.17.1",
    "next": "^8.0.0",
    "react": "^16.13.1",
    "react-dom": "^16.13.1"
  }
}
```

Let's add and commit files so that we can deploy our build bundle to `AWS S3 bucket` and push the whole code to `Bitbucket` through pipelines.

```bash
git add .
git commit -m 'commit message' .
git push --set-upstream origin dev
```
Git push will trigger a continous deployment process:

<p align="center">
  <img src="https://nextjs-portfolio.s3.amazonaws.com/pipelines.gif" alt="Branching model">
</p>

Copy api end-point url from your bitbucket pipeline build and open it in your browser:

> It should look something like `https://<api-gateway>.execute-api.us-east-1.amazonaws.com/dev`

<p align="center">
  <img src="https://nextjs-portfolio.s3.amazonaws.com/aws-api-endpoint.gif" alt="Branching model">
</p>

You might need to test your app on localhost before deploying it to AWS, that's where `serverless-offline` comes into play. 

```bash
rm -rf .next
npm run build
sls offline start
```

Let's test on our `http://localhost:3000`

> _This `Serverless` plugin emulates `AWS` and `API Gateway` on your local machine to speed up your development cycles. To do so, it starts an HTTP server that handles the request's lifecycle like APIG does and invokes your handlers._

#### Github

<p class="markdown-paragraph">This project is available on <a class="markdown-link" target="_blank" href="https://github.com/teddynted/nextjs-on-a-lambda-function">github</a> for further reference.</p>
