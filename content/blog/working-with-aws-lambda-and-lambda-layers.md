
We are going to configure a lambda function to use a function layer we have created in a previous tutorial.

> A **functional layer** gives you an ability to share code across multiple lambda functions whereas **dependency layer** is specific to a particular lambda function.

### Create a Lambda function

Create a new service based on the below command.

```zsh
serverless create --template aws-nodejs --path aws-lambda-axios --name aws-lambda-and-lambda-layers
```
This example will generate scaffolding for a service with AWS as a provider and nodejs as runtime. The scaffolding will be generated in the current working directory.

Post running the command your files should look like what's on the image below:

![alt text](https://nextjs-portfolio.s3.amazonaws.com/aws-lambda-layers.jpg "AWS Lambda Layers")

Open **handler.js**, overwrite existing code and paste the following:

```javascript
'use strict';

const axios = require('axios');

module.exports.apiCall = async () => {
    try {
        const { data } = await axios.get('https://jsonplaceholder.typicode.com/todos/1');
        return {
            statusCode: 200,
            body: JSON.stringify(data)
        }
    } catch (e) {
        console.log('error occured:', e);
        return {
            statusCode: 400,
            body: JSON.stringify(e)
        }
    }
};

```

The code above makes a call to a jsonplaceholder api using axios package that will be referenced from our functional layer created in a previous tutorial.

Add a layer to the our lambda function, edit **serverless.yml** by adding the content below:

```yaml
layers:
        - arn:aws:lambda:us-east-1:<account-id>:layer:axios-package-layer:1
```

Final yaml file should look like:

```yaml
service: aws-lambda-and-lambda-layers

provider:
  name: aws
  runtime: nodejs12.x

functions:
  apiCall:
    handler: handler.apiCall
    events:
      - http:
          path: apiCall
          method: get
    layers:
        - arn:aws:lambda:us-east-1:<account-id>:layer:axios-package-layer:1
```

> **NB** replace _account-id_ with your account's id.

## It's time to deploy

In your working directory run _sls deploy_ command:

```bash
sls deploy
```
You should see the below output if all goes well:

![alt text](https://nextjs-portfolio.s3.amazonaws.com/aws-sls-deploy-lambda.png "AWS Lambda Layers")

Let's test out function, run this command:

```bash
curl https://<api-id>.execute-api.us-east-1.amazonaws.com/dev/apiCall
```

> **NB** replace _api-id_ with your aws api gateway id, show in the sls deploy output under endpoints.

Upon running the curl command you should get a json output below:

```json
{"userId":1,"id":1,"title":"delectus aut autem","completed":false}
```
