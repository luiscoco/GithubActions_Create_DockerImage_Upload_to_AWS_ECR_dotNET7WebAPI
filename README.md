# GithubActions: How to create .NET 7 Web API Docker image and Upload it to AWS ECS Elastic Container Service

## 1. Create a .NET 7 Web API application in Visual Studio

We run Visual Studio 2022

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/6fe14492-2f95-4534-8d8b-72f441b1087f)

We create a new project 

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/54d18326-747c-45ee-a2d2-14957383cd3b)

We select the projec template "ASP.NET Core Web API"

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/3b1197cb-1d82-40ea-9e9c-2fba59f55b20)

We set the project name and location

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/810afcd7-7ce4-46a0-9656-b2349464ea9b)

We select the project main options: .NET 7 framework, enable Docker, configure HTTPS, use OpenAPI and user controllers

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/992695dc-64da-4e29-9996-6552752da6d1)

## 2. Upload your .NET 7 Web API to a Github repo 

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/2f55c5b8-8bb5-427c-b17b-fb61caf7411b)

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/21543565-2415-49c0-862d-dd74f686d831)

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/5d90dfd4-b90c-41ab-a2a0-eff5516aaa77)

## 3. Create a AWS ECR Public repo for storing the Docker image

Navigate to the AWS ECR service and create a new Public repo

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/3161dc5d-4c1e-4c96-ad2b-7c254a2214b9)

We set the repository name, and the container compatible operating system and architecture

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/e5aa8759-8a27-4deb-b8ae-dcbca4a7f40b)

We press the **Create repository** button

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/184c1aae-d741-43f4-91e6-69bf1b923ff7)

See the new repo in the list

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/387fdd4d-9c94-48cb-903c-3aaed49bcaef)
 
## 4. Create the Github Secrets to store the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY

We start this section in **AWS** creating an access key and a secret key 

We select the **Security Credentials** option in the top right menu

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/f93c0820-4244-41d3-b7d0-cdb95b903fc9)

We click on the **Create access key** button

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/c9955848-528b-4026-83b9-be9a49bd028a)

Select Use case **Command Line Interface CLI**, check "I understand the above recommendation and want to proceed to create an access key." and press Next button

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/fdaf9936-2543-41c0-9fcc-1c7c73adcac6)

We input the description tag and press the **Create access** key button 

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/3c7a2777-ddc5-4726-a89d-1fb378e2eddf)

We click on the **Download .csv file** and and Excel file will be donwloaded to our laptop with the access key and secret key

Go to you **Github repo** and create two secrets for stroing the access key and secret key

Press the **Settings** button

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/e56b4f46-e10f-4e8d-90ef-0b43219c825b)

We select **Secrets and variables** and we store **AWS_ACCESS_KEY_ID** and **AWS_SECRET_ACCESS_KEY** in two github repository secrets

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/42cda001-e27d-4194-b40a-2f548324f9b3)

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/9e3dc39e-d534-45a6-b29d-f401a53f5eb6)

## 5. Create the Github actions workflow

In the application Github repo click on the **Actions** button

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/2afe59ba-5088-4ebd-88b5-1840b0f30416)

We input the **main.yml** file for executing the github action workflow to create a .NET 7 Web API Docker image and upload it to AWS ECR 

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
    # New step to delete existing images
    - name: Delete existing images in ECR repository
      run: |
        aws ecr-public describe-images --repository-name ${{ env.IMAGE_NAME }} --region ${{ env.AWS_REGION }} --output=json | jq -r '.imageDetails[].imageDigest' | xargs -I {} aws ecr-public batch-delete-image --repository-name ${{ env.IMAGE_NAME }} --image-ids imageDigest={} --region ${{ env.AWS_REGION }}
    
    - name: Build, tag, and push image to Amazon ECR
      run: |
        docker build -t ${{ env.ECR_REGISTRY }}/${{ env.IMAGE_NAME }}:latest .
        docker push ${{ env.ECR_REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```

We press "**Commit changes...**" button

## 6. How deploy to AWS ECS the Docker image stored in my ECR repo 

We create a **ECS Cluster**:

```
aws ecs create-cluster --cluster-name myCluster
```

Create a new file task-definition.json and place it in a local folder: **C:/AWS Task Definition/task-definition.json**

This is the **task-definition.json** file source code

```json
{
    "family": "myDotnetApp",
    "executionRoleArn": "arn:aws:iam::550146943653:role/ecsTaskExecutionRole",
    "networkMode": "awsvpc",
    "containerDefinitions": [
        {
            "name": "dotnet8webapi",
            "image": "public.ecr.aws/x6y4g2f4/dotnet8webapirepo:latest",
            "cpu": 1024,
            "memory": 3072,
            "essential": true,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80
                }
            ],
            "environment": [
                {
                    "name": "ASPNETCORE_ENVIRONMENT",
                    "value": "Development"
                }
            ]
        }
    ],
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "1024",
    "memory": "3072"
}
```

We create a new **Task Definition**:

```
aws ecs register-task-definition --cli-input-json file://"C:/AWS Task Definition/task-definition.json"
```

We get my AWS Account default **VPC** name:

```
aws ec2 describe-vpcs ^
  --filters "Name=isDefault,Values=true" ^
  --query "Vpcs[].VpcId" ^
  --output text
```

For example the VPC name output is: **vpc-07d51d92f73354c0b**

With that VPC name we can retrieve the attached **Subnets** names:

```
aws ec2 describe-subnets ^
  --filters "Name=vpc-id,Values=vpc-07d51d92f73354c0b" ^
  --query "Subnets[].{ID:SubnetId,Name:Tags[?Key=='Name']|[0].Value}"
```

For example the Subnet output is: **subnet-0df35048d0d30e90f**

We also get the **Security Group** name with this command:

```
aws ec2 describe-security-groups ^
  --filters "Name=vpc-id,Values=vpc-07d51d92f73354c0b" "Name=group-name,Values=default" ^
  --query "SecurityGroups[].{ID:GroupId,Name:GroupName}" ^
  --output text
```

For example the Security Group output is: **sg-051b9197846af4fe0**

We create a new **Service** with the "aws ecs create-service" command 

Do not forget to set the **subnetId** and the **securityGroupId**

```
aws ecs create-service ^
  --cluster myCluster ^
  --service-name myDotnetService ^
  --task-definition myDotnetApp ^
  --launch-type FARGATE ^
  --desired-count 1 ^
  --network-configuration "awsvpcConfiguration={subnets=[subnet-0df35048d0d30e90f],securityGroups=[sg-051b9197846af4fe0],assignPublicIp=ENABLED}"
```

For accessing the endpotint click on the Public IP address and add the controller name

```
http://IPAddress/controlername
```

