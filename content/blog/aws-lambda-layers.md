
mkdir lambda-packages && cd lambda-packages
\ntouch s3.json

```json
{
    "S3Bucket": "your-s3-bucket",
    "S3Key": "package-name.zip"
}
```

\nmkdir nodejs && cd nodejs
\nnpm i axios
\ncd ..
\ntouch command.sh
