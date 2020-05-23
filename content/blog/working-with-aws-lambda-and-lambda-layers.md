
We are going to configure a lambda function to use a function layer we have created in the previous tutorial.

> A **functional layer** gives you an ability to share code across multiple lambda functions whereas **dependency layer** specific to a particular lambda function.

### Create a Lambda function

Create a new service based on the below command.

```bash
serverless create --template aws-nodejs --path aws-lambda-axios --name aws-lambda-and-lambda-layers
```
This example will generate scaffolding for a service with AWS as a provider and nodejs as runtime. The scaffolding will be generated in the current working directory.

Post running the command your files should look like what's on the below image:

![alt text](https://nextjs-portfolio.s3.amazonaws.com/aws-lambda-layers.jpg "AWS Lambda Layers")

Open _handler.js_ and paste the following code:

```javascript
'use strict';

const axios = require('axios');

module.exports.apiCall = async event => {
  try {
      const res = await axios.post('https://randomuser.me/api/', {})
      console.log(res)
      return {
          statusCode: 200,
          body: JSON.stringify(res)
      }
  } catch (e) {
      console.log(e)
      return {
          statusCode: 400,
          body: JSON.stringify(e)
      }
  }
};
```

The code above makes a call to a random user api using axios package that will referenced from our functional layer created in the previous tutorial.




