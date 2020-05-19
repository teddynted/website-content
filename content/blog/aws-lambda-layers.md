<p>mkdir lambda-packages && cd lambda-packages</p>
<p>touch s3.json</p>

```json
{
    "S3Bucket": "your-s3-bucket",
    "S3Key": "package-name.zip"
}
```

<p>mkdir nodejs && cd nodejs</p>
<p>npm i axios</p>
<p>cd ..</p>
<p>touch command.sh</p>

```bash
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
