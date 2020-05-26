In this tutorial we are going to integrate AWS' DynamoDB into a Next.JS application we have built in the previous tutorial.
> _`Amazon DynamoDB` is a fully managed NoSQL database service offered by Amazon Web Services (AWS)_

### Prerequisites
* Serverless
* Java Runtime Engine (JRE) version 6.x or newer

Change directories to your development directory and clone this repo: 

```bash
git clone https://github.com/teddynted/nextjs-on-a-lambda-function.git
```

> [`Nextjs app on a Lambda function`](https://teddykekana.com/blog/running-nextjs-app-on-a-lambda-function)

```bash
git clone https://github.com/teddynted/nextjs-on-a-lambda-function.git
cd nextjs-on-a-lambda-function
npm i
```

For the purpose of running our on localhost, we need to install `serverless-dynamodb-local` plugin.
