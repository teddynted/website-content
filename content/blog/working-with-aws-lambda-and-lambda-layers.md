
We are going to configure a lambda function to use a function layer we have created in the previous tutorial.

> A **functional layer** gives you an ability to share code across multiple lambda functions whereas **dependency layer** specific to a particular lambda function.

### Create a Lambda function

Create a new service based on the below command.

```bash
serverless create --template aws-nodejs --path aws-lambda-axios --name aws-lambda-and-lambda-layers
```
This example will generate scaffolding for a service with AWS as a provider and nodejs as runtime. The scaffolding will be generated in the current working directory.



