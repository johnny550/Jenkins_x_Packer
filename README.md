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

Build a custom AMI from a public source AMI, add a node Js app in it.

How to use this?

1- Terraform apply of this repository.

2- Update backend.tf with the actual name of the bucket.

3- terraform init to send the tf state.

4- Use the ip given by the output of the TERRAFORM APPLY in step 1. Aim for port 8080. This steps leads you to Jenkins.

5- On Jenkins, add a new account and set everything up

6-a\* Create a 1st job. Git URL: https://github.com/johnny550/packer-demo

b\* Script of job: ./jenkins-terraform.sh

7- Execute the job. It builds the AMI from the given source and adds the Node JS app

8-a\* Create a 2nd job. Git URL: https://github.com/johnny550/Jenkins_x_Packer

b\* Script of job: ./scripts/jenkins-run-terraform.sh

9- Execute the job. It uses the remote state of terraform to know it won't need to recreate existing resources. It sets INSTANCE_COUNT to 1 and will
start a new instance with our custom AMI created in step 7.

10- Take the IP of the app instance (you can find it iin the output of the 2nd job_view Jenkins cuild console output). Aim for port 80. See "HELLO WORLD".
