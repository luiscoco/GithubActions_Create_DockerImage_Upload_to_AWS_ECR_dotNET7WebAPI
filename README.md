# GithubActions: How to create .NET 7 Web API Docker image and Upload it to AWS ECS Elastic Container Service

## 1. Create a .NET 7 Web API application in Visual Studio


## 2. Upload your .NET 7 Web API to a Github repo 




## 3. Create a AWS ECR Public repo for storing the Docker image

Navigate to the AWS ECR service and create a new Public repo

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/3161dc5d-4c1e-4c96-ad2b-7c254a2214b9)

We set the repository name, and the container compatible operating system and architecture

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/e5aa8759-8a27-4deb-b8ae-dcbca4a7f40b)

We press the **Create repository** button

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/184c1aae-d741-43f4-91e6-69bf1b923ff7)

See the new repo in the list

![image](https://github.com/luiscoco/GithubActions_Create_DockerImage_Upload_to_AWS_ECR_dotNET7WebAPI/assets/32194879/387fdd4d-9c94-48cb-903c-3aaed49bcaef)
 
## 3. Create the Github Secrets to store the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY

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



## 4. Create t

Click in

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
    # New step to delete existing images
    - name: Delete existing images in ECR repository
      run: |
        aws ecr-public describe-images --repository-name ${{ env.IMAGE_NAME }} --region ${{ env.AWS_REGION }} --output=json | jq -r '.imageDetails[].imageDigest' | xargs -I {} aws ecr-public batch-delete-image --repository-name ${{ env.IMAGE_NAME }} --image-ids imageDigest={} --region ${{ env.AWS_REGION }}
    
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
