# Jenkins packer demo build

```
ARTIFACT=`packer build -machine-readable packer-demo.json |awk -F, '$0 ~/artifact,0,id/ {print $6}'`
AMI_ID=`echo $ARTIFACT | cut -d ':' -f2`
echo 'variable "APP_INSTANCE_AMI" { default = "'${AMI_ID}'" }' > amivar.tf
aws s3 cp amivar.tf s3://terraform-state-a2b62lf/amivar.tf
```

# Jenkins terraform build

```
cd jenkins-packer-demo
aws s3 cp s3://terraform-state-a2b62lf/amivar amivar.tf
touch mykey
touch mykey.pub
terraform apply -auto-approve -var APP_INSTANCE_COUNT=1 -target aws_instance.app-instance
```

What this is doing:

1- Git clone from Github (JENKINS)

2- Use PACKER to build a custom Amazon AMI having nodeJS pre-installed + the cloned app repo built code (JENKINS)

3- Git clone from Github a terraform repo (JENKINS)

4- terraform apply (JENKINS)
|\_> The EC2 instance is launched from our custom AMI, created by Packer (AWS)

5- Save Terraform's state in an S3 bucket (remote state) (AWS)

To save the terraform state remotely, run terraform init and apply. After the bucket is created, used the file backend.tf
and correct the bucket name. That bucket will then be used as a place to store the state of terraform the next time we use it.
