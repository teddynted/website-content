In this tutorial, I will walk you through steps to creating an AWS Lambda function layer and adding that layer to a function.

AWS Lambda layer helps you with keeping your deployment package as small as **3 MB** thus avoiding deployment package size limit of **250 MB** since your application will grow over time. So your application dependencies will be housed in layer which is separate from your lambda function, a function can use up to 5 layers at time.

> _Layers let you keep your deployment package small, which makes development easier. You can avoid errors that can occur when you install and package dependencies with your function code_ - AWS


### Prerequisites
* AWS account
* AWS CLI(Command Line Interface)
* Node 12.x
* Terminal


Let's start off by installing AWS CLI and configure credentials so that you are able to interact with AWS services through CLI.



```bash
brew install awscli
```


<p class="markdown-paragraph">Head over to the <a class="markdown-link" href="https://console.aws.amazon.com/iam/home?#/users/admin?section=security_credentials" target="_blank">IAM</a> to create an access key, This will be required when configuring AWS CLI.</p>


Configure AWS CLI by running this command:


```bash
aws configure
```


> The AWS CLI will prompt you for four pieces of information (access key, secret access key, AWS Region, and output format)


### Test!


Run this command to see if you have successfully configured you AWS credentials:


```bash
aws s3 ls
```


<p class="markdown-paragraph">You should see a list of buckets if you are authenticated and please create one that is going to house your zipped layer package.</p>


### Let's create our layer!


<p class="markdown-paragraph">Change directories to your development and run the following commands:</p>


```bash
mkdir lambda-packages && cd lambda-packages
touch s3.json
```


<p class="markdown-paragraph">Add these lines to the _**s3.json**_ file:</p>


```json
{
    "S3Bucket": "node-dependencies",
    "S3Key": "axios-package.zip"
}
```

<p class="markdown-paragraph">Create dependencies' directory</p>

```bash
mkdir nodejs
touch command.sh
```


<p class="markdown-paragraph">Add these commands to shell script _**command.sh**_:</p>


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


Run the shell script, it will create a zipped deployment package and upload it to an s3 bucket. 

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

<p class="markdown-paragraph">That's all for now. In the next <a class="markdown-link" href="/blog/working-with-aws-lambda-and-lambda-layers">tutorial</a> I will show you to configure a lambda function so that it pulls axios package from a layer.</p>



