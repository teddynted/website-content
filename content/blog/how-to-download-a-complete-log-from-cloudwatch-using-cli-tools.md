Cloudwatch Logs enables you to see all of yours log data from your systems, applications, and AWS services that you use, in a single, highly scalable service.

> _`CloudWatch Logs` enables you to centralize the logs from all of your systems, applications, and AWS services that you use, in a single, highly scalable service. You can then easily view them, search them for specific error codes or patterns, filter them based on specific fields, or archive them securely for future analysis._

However browsing through Cloudwatch log groups looking for a specific information can be a very tedious task at times, so I will walk you through the process of querying Cloudwatch using a group name and downloading it to your local in a txt format.

### Pre-requisites

* AWS Serverless CLI

<p class="markdown-paragraph">Let's set it off by installing awslogs - you can read their documentation <a class="markdown-link" target="_blank" href="https://github.com/jorgebastida/awslogs">here</a>.</p>

```bash
pip3 install awslogs
```

### Configure the AWS CLI

Install AWS CLI and configure it, if it's already installed skip to the next section. This command will prompt you for your AWS Access Key ID, Secret Access Key, region name and output format.

```bash
brew install awscli
aws configure
```

### Let's get our data

You should be able to download logs and add data to a txt file upon running this command:

```bash
awslogs get my_lambda_group ALL --start='6/22/2020 00:00' --end='6/26/2020 19:00' | tee logs.txt
```

That's all you had to do!
