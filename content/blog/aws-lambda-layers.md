<p>In this tutorial, I will walk you through steps to creating an AWS Lambda function layer and adding that layer to a function</p>


### Prerequisites
* AWS account
* AWS CLI(Command Line Interface)
* Node 12
* Terminal


<p>Let's start off by installing AWS CLI and configure credentials so that you are able to interact with AWS services through CLI.</p>



```bash
brew install awscli
```


<p>Let's Head over to the <a href="https://console.aws.amazon.com/iam/home?#/users/admin?section=security_credentials" target="_blank">IAM</a> to create an access key, This will be required when configuring AWS CLI.</p>


<p>Configure AWS CLI by running this command:</p>


```bash
aws configure
```


> The AWS CLI will prompt you for four pieces of information (access key, secret access key, AWS Region, and output format - _json_)


### Test!


Run this command to see if you have successfully configured you AWS credentials:


```bash
aws s3 ls
```


You should see a list of buckets if you are authenticated and please create one that is going to house your zipped layer package.


### Let's create our layer!


<p>Change directories to your development and run the following commands:</p>


```bash
mkdir lambda-packages && cd lambda-packages
touch s3.json
```


Add these lines to the _**s3.json**_ file:


```json
{
    "S3Bucket": "your-s3-bucket",
    "S3Key": "package-name.zip"
}
```

<p>Create dependencies' directory</p>

```bash
mkdir nodejs && cd nodejs
npm init -y
npm i axios
cd ..
touch command.sh
```


Add these commands to shell script _**command.sh**_:



```bash wrap
rm -rf package-name.zip
zip -r package-name nodejs
echo "Delete object from s3 ..."
aws s3 rm s3://your-s3-bucket-name/package-name.zip
echo "Uploading to s3 ..."
aws s3 cp package-name.zip s3://your-s3-bucket-name/
echo "Creating a layer ..."
aws lambda publish-layer-version --layer-name "your-layer-name" --description "Description of your layer" --content "file://s3.json" --license-info "MIT" --compatible-runtimes "nodejs12.x"
```



Run the shell script, it will create a zipped package and upload it to an s3 bucket. 

```cmd
./command.sh
```

The output will look like the response below:


```bash
Delete object from s3 ...
delete: s3://your-s3-bucket-name/package-name.zip
Uploading to s3 ...
upload: ./batch-one.zip to s3://your-s3-bucket-name/package-name.zip
Creating a layer ...
{
    "LayerVersionArn": "arn:aws:lambda:us-east-1:account-id:layer:your-layer-name:1",
    "Description": "Description of your layer", 
    "CreatedDate": "2020-05-19T19:25:32.028+0000", 
    "LayerArn": "arn:aws:lambda:us-east-1:account-id:layer:your-layer-name", 
    "Content": {
        "CodeSize": 37841523, 
        "CodeSha256": "6JmAdzkHvcDfmHgzEcbZz3voIlkI9ExijkLI3vsJkR8=", 
        "Location": "...."
    }, 
    "Version": 13, 
    "CompatibleRuntimes": [
        "nodejs12.x"
    ], 
    "LicenseInfo": "MIT"
}
```


Copy _**LayerVersionArn**_ attribute value into your yaml serverless file of the lambda function.



```yaml
functions:
  hello:
    package:
      exclude:
        - 'functions/node_modules/**'
    handler: handler.hello
    events:
      - http: ANY /
      - http: ANY /{proxy+}
      - cors: true
    layers:
      - arn:aws:lambda:us-east-1:account-id:layer:your-layer-name:1
```



### That's all! Thanks for stopping by.
