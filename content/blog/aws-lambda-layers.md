<p>In this tutorial, I will walk you through steps to creating an AWS Lambda function layer and adding that layer to a function</p>

### Prerequisites
* AWS account
* AWS CLI(Command Line Interface)
* Node
* Terminal

<p>Let's start off by installing AWS CLI and configure credentials so that you are able to interact with AWS services through CLI.</p>

```bash
brew install awscli
```

<p>Let's Head over to the <a href="https://console.aws.amazon.com/iam/home?#/users/admin?section=security_credentials" target="_blank">IAM</a> to create access key</p>

<p>Configure AWS CLI by running this command:</p>

```bash
aws configure
```

> The AWS CLI will prompt you for four pieces of information (access key, secret access key, AWS Region, and output format - _json_)

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

```bash
mkdir nodejs && cd nodejs
npm i axios
cd ..
touch command.sh
```

Add these commands to shell script _**command.sh**_:

```bash wrap
rm -rf package-name.zip
cd nodejs
npm install
cd ..
zip -r package-name nodejs
echo "Delete object from s3 ..."
aws s3 rm s3://your-s3-bucket-name/package-name.zip
echo "Uploading to s3 ..."
aws s3 cp package-name.zip s3://your-s3-bucket-name/
echo "Creating a layer ..."
aws lambda publish-layer-version --layer-name "your-layer-name" --description "Description of your layer" --content "file://s3.json" --license-info "MIT" --compatible-runtimes "nodejs12.x"
```
