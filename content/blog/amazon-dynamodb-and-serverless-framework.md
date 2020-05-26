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
sls dynamodb install
```

> You also need to install Java Runtime version only if it's not installed 
