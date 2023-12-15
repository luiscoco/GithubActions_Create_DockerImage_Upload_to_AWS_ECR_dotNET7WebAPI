# dotnet7WebAPI


## 1. 


## 2. Create a AWS ECR Public repo for storing the Docker image

 
## 3. Create the Github Secrets to store the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY




## 4. 

This is the **main.yml** file for executing the github action workflow to create a .NET 8 Web API Docker image and upload it to AWS ECR 

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ master ]

env:
  ECR_REGISTRY: public.ecr.aws/x6y4g2f4  # Your AWS ECR Registry
  IMAGE_NAME: dotnet8webapirepo  # Replace with your image name
  AWS_REGION: us-east-1  # Replace with your AWS region

jobs:
  build_and_push:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Set up AWS CLI
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      run: |
        aws ecr-public get-login-password --region ${{ env.AWS_REGION }} | docker login --username AWS --password-stdin ${{ env.ECR_REGISTRY }}
    - name: Build, tag, and push image to Amazon ECR
      run: |
        docker build -t ${{ env.ECR_REGISTRY }}/${{ env.IMAGE_NAME }}:latest .
        docker push ${{ env.ECR_REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```

## 5. How deploy to AWS ECS the Docker image stored in my ECR repo 

We create a **ECS Cluster**:

```
aws ecs create-cluster --cluster-name myCluster
```

We create a new **Task Definition**:

```
aws ecs register-task-definition --cli-input-json file://"C:/AWS Task Definition/task-definition.json"
```

We get my AWS Account default **VPC** name:

```
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[].VpcId" --output text
```

For example the VPC name output is: **vpc-07d51d92f73354c0b**

With that VPC name we can retrieve the attached **Subnets** names:

```
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-07d51d92f73354c0b" --query "Subnets[].{ID:SubnetId,Name:Tags[?Key=='Name']|[0].Value}"
```

For example the Subnet output is: **subnet-0df35048d0d30e90f**

We also get the **Security Group** name with this command:

```
aws ec2 describe-security-groups --filters "Name=vpc-id,Values=vpc-07d51d92f73354c0b" "Name=group-name,Values=default" --query "SecurityGroups[].{ID:GroupId,Name:GroupName}" --output text
```

For example the Security Group output is: **sg-051b9197846af4fe0**

We create a new **Service**:

```
aws ecs create-service --cluster myCluster --service-name myDotnetService --task-definition myDotnetApp --launch-type FARGATE --desired-count 1 --network-configuration "awsvpcConfiguration={subnets=[subnet-0df35048d0d30e90f],securityGroups=[sg-051b9197846af4fe0],assignPublicIp=ENABLED}"
```

We describe the c

```
aws ecs describe-services --cluster myCluster --services myDotnetService
```
