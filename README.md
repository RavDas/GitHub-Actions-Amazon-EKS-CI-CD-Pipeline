# GitHub Actions with Amazon-EKS CICD Pipeline

![image](https://github.com/user-attachments/assets/6135c7ae-e07b-4f94-9228-db75cf82c7c4)

### Github Actions

From Continuous Integration (CI) and Continuous Deployment (CD) to code quality assurance and security scanning, GitHub Actions brings automation to every aspect of the development process. With custom workflows, enhanced collaboration, and release management, this tool empowers developers to be more efficient, reliable, and productive. Discover how GitHub Actions is not just a concept but a transformative solution in the daily lives of developers.


### Step 1: Launch an EC2 Instance

To launch an AWS EC2 instance with Ubuntu 22.04 using the AWS Management Console, sign in to your AWS account, access the EC2 dashboard, and click “Launch Instances”  to select “Ubuntu 22.04” as the AMI, and in choose “t2.medium” as the instance type. 

Configure the instance details, storage, tags, and security group settings according to your requirements. 

Review the settings, create or select a key pair for secure access, and launch the instance. Once launched, you can connect to it via SSH using the associated key pair.

Here you need to have the created key-pair(.pem file) in the file directory of the your local machine that you launch your console inorder to ssh into EC2 instance which is attached with the same key-pair(.pem file).

![connect](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/04da2651-0421-480f-ac9a-6f2407894691)

![command](https://github.com/user-attachments/assets/1c9f7d39-8332-482a-a80c-8af558be8cc5)

**OR**

ssh into the EC2 instance using GUI of the MobaXtreme terminal like below (Here you can simply attach the key-pair in "Use private key" section).

![image](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/c97cc211-5e64-4ada-bb3c-13f1c639163b)

### Create an IAM ROLE

Navigate to AWS Console

Search “IAM"

Click “Roles”

Click “Create role”

![image](https://github.com/user-attachments/assets/836a3ad8-7727-4320-a5f0-28815d121b63)

1. Select trusted entity
  
Choose “AWS service” as "Trusted entity type" section

![image](https://github.com/user-attachments/assets/f3930ef3-7352-4f69-8512-d2a95b8ec9f6)

Choose EC2 in "Service or use case” under "Use case" section

![image](https://github.com/user-attachments/assets/20bfdea7-316e-41af-ab12-af0e2387abfb)

2. Add permissions policies

Administrator Access (or) EC2 full access

AmazonS3FullAccess and EKS Full access

Click the “Role name” field and type a name - "GithubActions"

Click “Create role” (JUST SAMPLE IMAGE BELOW ONE)

![image](https://github.com/user-attachments/assets/f7681170-94ce-4d9f-905d-f1b367aacc7f)

Goto “EC2” dashboard.

Go to the instance created and add this role to the EC2 instance.

Select instance –> Actions –> Security –> Modify IAM role

Add a newly created Role and click on Update IAM role.

![image](https://github.com/user-attachments/assets/790ec7c8-2719-4f3d-ba98-af38afcce53e)



### Step2

#### A: Install Docker and Run SonarQube Container

Connect to your EC2 instance using Putty, Mobaxtreme or Git bash and install docker on it.

```
sudo apt-get update
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

Pull the SonarQube Docker image and run it.

```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

![image](https://github.com/user-attachments/assets/79390a6c-e55b-4123-aa7e-07b6eb191e88)

After the docker installation, we will create a SonarQube container (Remember to add 9000 ports in the security group).


Now copy the IP address of the EC2 instance

```
<ec2-public-ip:9000>
```
![image](https://github.com/user-attachments/assets/71de344e-d399-487b-a7f7-f084743e8c4a)

Provide Login and password

```
login admin
password admin
```


![image](https://github.com/user-attachments/assets/d640efb7-8a4f-4081-92e0-539839d5d58d)

Update your SonarQube password & This is the SonarQube dashboard

![image](https://github.com/user-attachments/assets/6eccb8cc-0f59-4372-8165-325e50c073cf)


### B: Integrating SonarQube with GitHub Actions

Integrating SonarQube with GitHub Actions allows you to automatically analyze your code for quality and security as part of your continuous integration pipeline.

We already have SonarQube up and running.

On SonarQube Dashboard click on Manually

![image](https://github.com/user-attachments/assets/29ddc1ba-6264-413a-9566-8119c586e963)

Next, provide a name for your project and provide a Branch name and click on setup

![image](https://github.com/user-attachments/assets/dbcb4731-3640-4e66-8fb4-337f2783f132)


On the next page click on "With GitHub Actions"

This will Generate an overview of the Project and provide some instructions to integrate

![image](https://github.com/user-attachments/assets/acc8c024-9682-433a-9ce9-63c2982369d3)


Let’s Open your GitHub and select your Repository -In my case it is "GitHub-Actions-Amazon-EKS-CI-CD-Pipeline".

Click on Settings

![1 2](https://github.com/user-attachments/assets/d1386235-d33b-4f82-bf9f-cb4240b6d60a)


Search for "Secrets and variables" and click on and again click on "Actions"

![image](https://github.com/user-attachments/assets/7e2f9e70-7571-4577-b6be-9078f43b52de)

It will open a page like this click on "New repository secret"

![image](https://github.com/user-attachments/assets/f05c1583-8787-4063-b198-46790f10bf6a)

Now go back to Your SonarQube Dashboard

Copy SONAR_TOKEN and click on "Generate a Token"

![image](https://github.com/user-attachments/assets/2f2baefd-56c5-488c-b229-fa7b286aabf4)

Click on Generate

![image](https://github.com/user-attachments/assets/b7047e01-6684-4acb-89ef-6552ed2db560)

Let’s copy the Token and add it to GitHub secrets

![image](https://github.com/user-attachments/assets/5e701687-8374-4568-82aa-9c95c6cf579c)


Now go back to GitHub and Paste the copied name for the secret and token

Name: SONAR_TOKEN

Secret: Paste Your Token and click on "Add secret"

![image](https://github.com/user-attachments/assets/0ba8404a-713e-46a1-b4bc-c16d0e48c77c)


Now again go back to the SonarQube Dashboard

Copy the Name and Value

![image](https://github.com/user-attachments/assets/a790f50b-6803-4449-b114-0fb3f46fd0ba)


Go to GitHub now and paste-like this and click on add secret by clicking "New repository secret" again.(Here we do that to allow SonarQube to allow our Github repository code to scan and analyze)

![image](https://github.com/user-attachments/assets/2c5415d8-40e3-476c-ac2f-66f596eb8de1)


Our SonarQube secrets are added and you can see

![image](https://github.com/user-attachments/assets/98ae9015-d7d4-44d2-b781-94698a676083)

Go to SonarQube Dashboard and click on continue

![image](https://github.com/user-attachments/assets/178731c6-08ab-4dde-af92-5b9f4640a35d)


Now create your Workflow for your Project. In my case, the project is built using React Js. That’s why I am selecting "Other".

![image](https://github.com/user-attachments/assets/717b69fd-8b30-4c6c-a3bc-abf1f6bcf278)


Now it Generates and workflow for my Project

![image](https://github.com/user-attachments/assets/f8a0bf76-b72b-4397-8a71-d369ecd7cb72)


Go back to GitHub and add the add file "sonar-project.properties" and copy the following code you get under the file name. You can create a file inside Github itself using Add file -> Create new file and commit changes to the repository of create a file in your local machine repository and push it your remote Github repository.

![image](https://github.com/user-attachments/assets/12c9c6d1-48af-40c2-97f6-5fa1a434ab35)

Add the file with content to the Github repository.

![1 3](https://github.com/user-attachments/assets/bd638a14-809b-4b94-869e-04b49a79e2de)


Now add our workflow of our project 

Again go to your GitHub repository and add the add file "sonar.yml"( you can use any name instead of build.yml) inside ".github/workflows/". You can create a file inside Github itself using Add file -> Create new file and commit changes to the repository of create a file in your local machine repository and push it your remote Github repository.

![image](https://github.com/user-attachments/assets/6062bcd4-c5ef-4d76-bfd4-74dbb81b753d)

Copy content and add it to the file

```
name: Build,Analyze,scan

on:
  push:
    branches:
      - main


jobs:
  build-analyze-scan:
    name: Build
    runs-on: [self-hosted]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis

      - name: Build and analyze with SonarQube
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

![1 4](https://github.com/user-attachments/assets/897960c8-7a88-4658-8edb-5bd7d65689c5)

Now workflow is created.

Start again GitHub actions runner from instance

```
cd actions-runner
./run.sh
```







