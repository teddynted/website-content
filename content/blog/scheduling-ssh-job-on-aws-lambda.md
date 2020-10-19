How to schedule an ssh cron job on AWS Lambda(Node.js) function to upload a csv file to an EC2 instance, The job will run between Monday and Friday at 3am. 

_Cron Jobs are used for scheduling tasks to run on the server. They're most commonly used for automating system maintenance or administration._

```yaml
service: scheduling-ssh-job-using-aws-lambda

frameworkVersion: '2'

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
  sshScheduler:
    handler: handler.sshScheduler
    events:
      - schedule:
          rate: cron(0 3 ? * MON-FRI *)
          enabled: true
```

