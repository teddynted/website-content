
mkdir lambda-packages && cd lambda-packages
touch s3.json

```json
{
    "S3Bucket": "your-s3-bucket",
    "S3Key": "package-name.zip"
}
```

mkdir nodejs && cd nodejs
npm i axios
cd ..
touch command.sh
