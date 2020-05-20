In this tutorial, I will walk you through steps to creating an AWS Lambda function layer and adding that layer to a function


### Prerequisites
* AWS account
* AWS CLI(Command Line Interface)
* Node 12.x
* Terminal


Let's start off by installing AWS CLI and configure credentials so that you are able to interact with AWS services through CLI.



```bash
brew install awscli
```


Let's Head over to the <a href="https://console.aws.amazon.com/iam/home?#/users/admin?section=security_credentials" target="_blank">IAM</a> to create an access key, This will be required when configuring AWS CLI.


Configure AWS CLI by running this command:


```bash
aws configure
```


> The AWS CLI will prompt you for four pieces of information (access key, secret access key, AWS Region, and output format - _json_)


### Test!


Run this command to see if you have successfully configured you AWS credentials:


```bash
aws s3 ls
```


<p>You should see a list of buckets if you are authenticated and please create one that is going to house your zipped layer package.</p>


### Let's create our layer!


<p>Change directories to your development and run the following commands:</p>


```bash
mkdir lambda-packages && cd lambda-packages
touch s3.json
```


<p>Add these lines to the _**s3.json**_ file:</p>


```json
{
    "S3Bucket": "node-dependencies",
    "S3Key": "axios-package.zip"
}
```

<p>Create dependencies' directory</p>

```bash
mkdir nodejs
touch command.sh
```


<p>Add these commands to shell script _**command.sh**_:</p>


```bash wrap
rm -rf axios-package.zip
cd nodejs
npm init -y
npm i axios
rm -rf package-lock.json
cd ..
zip -r axios-package.zip nodejs
echo "Delete object from s3 ..."
aws s3 rm s3://node-dependencies/axios-package.zip
echo "Uploading to s3 ..."
aws s3 cp axios-package.zip s3://node-dependencies/
echo "Creating a layer ..."
aws lambda publish-layer-version --layer-name "axios-package-layer" --description "Axios dependencies" --content "file://s3.json" --license-info "MIT" --compatible-runtimes "nodejs12.x"
```



Run the shell script, it will create a zipped package and upload it to an s3 bucket. 

```cmd
./command.sh
```

The output will look like the response below:


```bash
Delete object from s3 ...
delete: s3://node-dependencies/axios-package.zip
Uploading to s3 ...
upload: ./axios-package.zip to s3://node-dependencies/axios-package.zip
Creating a layer ...
{
    "LayerVersionArn": "arn:aws:lambda:us-east-1:account-id:layer:axios-package-layer:1", 
    "Description": "Axios dependencies", 
    "CreatedDate": "2020-05-20T11:20:35.831+0000", 
    "LayerArn": "arn:aws:lambda:us-east-1:account-id:layer:axios-package-layer", 
    ....
    "Version": 1, 
    "CompatibleRuntimes": [
        "nodejs12.x"
    ], 
    "LicenseInfo": "MIT"
}
```

> Note that I have replaced my actual account id with a _**account-id**_ in the output above, use your AWS account ID.

<p>That's all for now. In the next <a class="markdown-link" href="/blog/working-with-aws-lambda-and-lambda-layers">tutorial</a> I will show you to configure a lambda function so that it pulls axios package from a layer.</p>



