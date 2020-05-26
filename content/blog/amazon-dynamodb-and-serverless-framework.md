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

Upon running `sls offline start` a table will be seeded with predefined json data. Add a directory called `seeds` and a `posts.json` file:

```bash
mkdir seeds && cd seeds
touch posts.json
```

Add this content to `posts.json`:

```json
[
  {
    "post_title": "Lorem Ipsum",
    "author": "Author Name",
    "post_category": [
      {
        "value": "React",
        "label": "React"
      }
    ],
    "created_at": "2020-05-23T18:54:07.157Z",
    "post_desc": "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book.",
  }
]
```
