By the end of this tutorial you will have learnt to create a csv file on the fly in your Lambda’s function and upload that file to Ubuntu EC2 instance via ssh and sftp.

### Prerequisites

* AWS Account
* Existing Ubuntu EC2 instance with a key pair
* Configured AWS CLI on your local

### Let's waste no time

First, we’ll start by using the Serverless CLI tool to bootstrap our project and install packages:

```bash
serverless create --template aws-nodejs --path sftp-file-upload-service --name sftp-file-upload
```

#### Npm Packages

```bash
npm init -y
npm install ssh2-sftp-client json2csv
```

Open `handler.js` file and replace the code with the below:

```javascript
'use strict';

const fs = require('fs');

const uploadFileToSftpServer = async (content, remoteFolder) => {
      try {
          const connSettings = {
            host: 'public-ip-address',
            port: 22,
            username: 'ubuntu',
            privateKey: fs.readFileSync('./path/to/your/key.pem'),
          };
          const Client = require('ssh2-sftp-client');
          const { Parser } = require('json2csv');
          const sftp = new Client();
          // Extract fields' headers from a JSON object / file
          const fields = Object.keys(content[0]);
          const json2csvParser = new Parser({fields, quote: ''});
          const csv = json2csvParser.parse(content);
          // Define the name of the file to be uploaded to server
          const file = `colors.csv`;
          const awsTempFolder = `/tmp/${file}`;
          // Create your file in a Lambda temporary folder/directory 
          fs.writeFileSync(awsTempFolder, csv);
          const data = fs.createReadStream(awsTempFolder);
          // Define an absolute path of your remote sftp server
          const remotePath = `${remoteFolder}${file}`;
          // Connect to your server to dump that csv file 
          console.log('Connecting....');
          await sftp.connect(connSettings);
          // Upload the file, if your connection is successful
          console.log('Uploading....');
          await sftp.put(data, remotePath);
          // Remove/delete your file from a Lambda's temp folder/directory
          fs.unlinkSync(awsTempFolder);
          // And lastly disconnect from your remote server
          console.log(`File Uploaded!`);
          return await sftp.end();
      } catch (err) {
          console.log('File upload failed because ', err.message);
          return err.message;
      }
};

module.exports.sftpFileUpload = async (event, context) => {
  // Sample JSON data
  const data = [
    {
      color: 'red',
      value: '#f00'
    },
    {
      color: 'green',
      value: '#0f0'
    },
    {
      color: 'blue',
      value: '#00f'
    },
    {
      color: 'cyan',
      value: '#0ff'
    },
    {
      color: 'magenta',
      value: '#f0f'
    },
    {
      color: 'yellow',
      value: '#ff0'
    },
    {
      color: 'black',
      value: '#000'
    }
  ];
  const res = await uploadFileToSftpServer(data, 'Uploads/');
  const response = {
    statusCode: 200,
    body: res,
  };
  console.log(`Display response${response}`);
  return response;
};
```
The code above passes colors' JSON object and EC2's folder name to `uploadFileToSftpServer` function and it will do following:

* Converts json into csv with column titles and proper line endings using `json2csv` package.
* Temporarily store the csv file in the Lambda's `tmp` folder.
* Using `ssh2-sftp-client` package, SSH into your EC2 instance and upload your csv file from Lambda's a `tmp` folder.
* Upon uploading a file a successfully, will remove the file from Lambda tmp folder to clear storage - _Since it has the capacity of 512 MB_

### Permissions

Let's grant our IAM role the permission to access Elastic Compute Cloud - EC2. Open `serverless.yml` and replace all with the code:

```yaml
service: ssh-ec2-instance

provider:
  name: aws
  runtime: nodejs10.x
  stage: dev
  region: us-east-1
  iamRoleStatements:
  - Effect: Allow
    Action:
      - ec2:CreateNetworkInterface
      - ec2:DescribeNetworkInterfaces
      - ec2:DeleteNetworkInterface
    Resource: '*'

functions:
  sftpFileUpload:
    handler: handler.sftpFileUpload
    events:
      - http:
          path: sftpFileUpload
          method: get
```
### Let's deploy our Lambda function

```bash
sls deploy -v
```

![alt text](https://nextjs-portfolio.s3.amazonaws.com/lambda-deployment.png "Deploy AWS Lambda")

#### And that's all!
