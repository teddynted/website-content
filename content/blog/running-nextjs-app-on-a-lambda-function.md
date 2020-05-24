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

`bitbucket-pipelines.yml`:

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
             - npm rebuild node-sass
             - npm install
             - npm run build
             - sls deploy -v --stage dev
             - git push
```

Ensure that you create a repository for this project in bitbucket and connect it to Bitbucket this project:

<p align="center">
  <img src="https://nextjs-portfolio.s3.amazonaws.com/new-bitbucket-repository.png">
</p>

In your Bitbucket respository navigate to `repository settings > settings` and enable pipelines:

<p align="center">
  <img src="https://nextjs-portfolio.s3.amazonaws.com/repository-settings-bitbucket.png">
</p>

Set repository variables by adding your AWS `ACCESS_KEY_ID` and `SECRET_ACCESS_KEY`:

![alt text](https://nextjs-portfolio.s3.amazonaws.com/repository-variables-blurred.png "Shell script")


