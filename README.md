# AWS ECR anonymous proxy

Based on official nginx alpine.

The container will renew the aws token every 6 hours.

Variables required:
```
AWS_KEY
AWS_SECRET
REGION
```
or mount .aws/config file with all the required configurations. (more details below)
## Docker run:
##### Without ssl
This will require either to add insecure registry URL or a load balancer with valid ssl certificates.
Check https://docs.docker.com/registry/insecure/ for more details.
```
docker run -e AWS_SECRET='YOUR_AWS_SECRET' \
-e AWS_KEY='YOUR_AWS_KEY' \
-e REGION='YOUR_AWS_REGION' \
-d catalinpan/aws-ecr-proxy
```
##### With your own certificate
```
docker run -e AWS_SECRET='YOUR_AWS_SECRET' \
-e AWS_KEY='YOUR_AWS_KEY' \
-e REGION='YOUR_AWS_REGION' \
-v `pwd`/YOUR_CERTIFICATE.key:/etc/nginx/ssl/default.key:ro \
-v `pwd`/YOUR_CERTIFICATE.crt:/etc/nginx/ssl/default.crt:ro \
-d catalinpan/aws-ecr-proxy
```
##### With a valid AWS CLI configuration file
The configuration should look like below example.  
```
cat ~/.aws/config
```
```
[default]
# region example eu-west-1
region = REGION     
aws_access_key_id = YOUR_AWS_KEY
aws_secret_access_key = YOUR_AWS_SECRET
```
```
docker run -v ~/.aws:/root/.aws:ro
-v `pwd`/YOUR_CERTIFICATE.key:/etc/nginx/ssl/default.key:ro \
-v `pwd`/YOUR_CERTIFICATE.crt:/etc/nginx/ssl/default.crt:ro \
-d catalinpan/aws-ecr-proxy
```

## SSL
The certificates included are just to get nginx started. Generate your own certificate, get valid ssl certificates or use the container behind a load balancer with valid SSL certificates.

#### Self signed certificates
```
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout default.key -out default.crt
```

## Kubernetes example
The configuration provided will require valid ssl certificates or to be behind a load balancer with valid ssl.
The configuration can be changed to get aws_config and ssl certificates as secrets.

## Health check endpoint

If you are using this ECR proxy behind a load balancer that needs an endpoint to check if it's alive, you can hit /ping
which will return a "200 OK".
