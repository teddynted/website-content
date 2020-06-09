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
