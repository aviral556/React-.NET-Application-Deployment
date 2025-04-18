1.Upload build to S3:
ng build --prod
aws s3 sync dist/voice-assistant/ s3://bucket-name --delete

2.Create CloudFront Distribution:
aws cloudfront create-distribution \
  --origin-domain-name your-bucket-name.s3.amazonaws.com \
  --default-root-object index.html

