l# GitHub Actions with Amazon-EKS CICD Pipeline

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


### Step 1.B: Add a self-hosted runner to EC2 instance

Go to GitHub and click on Settings –> Actions –> Runners

![image](https://github.com/user-attachments/assets/f04e5899-0b72-42a5-bf66-10de52a2f141)

Click on "New self-hosted runner"

![image](https://github.com/user-attachments/assets/987bf22a-8f10-4c0c-bcd8-17b1b5a63bce)


Now select Linux and Architecture X64

![2 59](https://github.com/user-attachments/assets/513eb78f-a054-42e6-a245-ab578c19a61a)


Use the below commands to add a self-hosted runner to our EC2 instance

![2 6](https://github.com/user-attachments/assets/c5c7fd86-d28b-4e99-a69b-54ded8967839)


And paste the commands in the terminal of the EC2 instance.

**NOTE: Use your runner command in the EC2 instance**



```
mkdir actions-runner && cd actions-runner
```

![image](https://github.com/user-attachments/assets/bc2f5b8a-9f0d-478a-84b0-c9abf5c0fccd)


The command “mkdir actions-runner && cd actions-runner” is used to create a new directory called “actions-runner” in the current working directory and then immediately change the current working directory (use home as the current directory) to the newly created “actions-runner” directory. This allows you to organize your files and perform subsequent actions within the newly created directory without having to navigate to it separately.

```
curl -o actions-runner-linux-x64-2.317.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.317.0/actions-runner-linux-x64-2.317.0.tar.gz
```

This command downloads a file called “actions-runner-linux-x64-2.310.2.tar.gz” from a specific web address on GitHub and saves it in your current directory.

![image](https://github.com/user-attachments/assets/cd007ca1-d559-4844-96a9-569210025ba9)

Let’s validate the hash installation

```
echo "fb28a1c3715e0a6c5051af0e6eeff9c255009e2eec6fb08bc2708277fbb49f93  actions-runner-linux-x64-2.310.2.tar.gz" | shasum -a 256 -c
```

![image](https://github.com/user-attachments/assets/facaf6de-5bb0-4b77-83f4-04f9f0529d03)

Now extract the installer

```
tar xzf ./actions-runner-linux-x64-2.317.0.tar.gz
```

![image](https://github.com/user-attachments/assets/f683fbe6-70f8-48a3-8435-409b6b9aa5fd)

Let’s configure the runner

```
./config.sh --url https://github.com/RavDas/GitHub-Actions-Amazon-EKS-CI-CD-Pipeline --token A2MXW4323ALGB72GGLH34NLFGI2T4
```

![image](https://github.com/user-attachments/assets/a67b4275-6bc8-4c28-bbd9-7a6efde64d55)

If you provide multiple label names for the runner, use commas for each label.

Created runner in Github available in Settings -> Actions -> Runners

![runner](https://github.com/user-attachments/assets/f226e9ee-62ae-4227-9acb-b96de5e8308b)


Let’s start runner.

```
./run.sh
```

![image](https://github.com/user-attachments/assets/301f9948-653a-4967-b0ef-7e317f9b0d47)

Let’s close Runner for now.

```
ctrl + c  #to close
```


### Step 2.A: Install Docker and Run SonarQube Container

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


### Step 2.C: INSTALLATION OF OTHER TOOLS


1. Install Java 17 - Install Temurin (formerly Adoptium) JDK 17.
3. Install Trivy (Container Vulnerability Scanner).
4. Install Terraform (Build infastructure of the AWS EKS cluster).
5. Install kubectl (Kubernetes command-line tool).
6. Install AWS CLI (Amazon Web Services Command Line Interface).
7. Install Node.js 16 and npm.

Create a script automates the installation of these software tools commonly used for development and deployment.

```
sudo vi tools.sh
```

Copy below  code to the ```tools.sh``` file.

```
# Install Java

#!/bin/bash
sudo apt update -y
sudo touch /etc/apt/keyrings/adoptium.asc
sudo wget -O /etc/apt/keyrings/adoptium.asc https://packages.adoptium.net/artifactory/api/gpg/key/public
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version

# Install Trivy

sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

# Install Terraform

sudo apt install wget -y
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Install kubectl

sudo apt update
sudo apt install curl -y
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Install AWS CLI

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt-get install unzip -y
unzip awscliv2.zip
sudo ./aws/install

# Install Node.js 16 and npm

curl -fsSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/nodesource-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/nodesource-archive-keyring.gpg] https://deb.nodesource.com/node_16.x focal main" | sudo tee /etc/apt/sources.list.d/nodesource.list
sudo apt update
sudo apt install -y nodejs
```

Make the created file executable

```
sudo chmod +x tools.sh
```

Check whether the versions are also installed or not.

```
trivy --version
terraform --version
aws --version
kubectl version
node -v
java --version
```

![image](https://github.com/user-attachments/assets/e0147d11-d586-45fb-bebb-2372aa891b24)

![image](https://github.com/user-attachments/assets/ac6f7dc1-5000-4d79-8481-aec085ce74e2)

![image](https://github.com/user-attachments/assets/269e936f-03fd-4b49-a1bb-2c197fd95c00)


### Step 2.B.1 EKS provision using Terraform

Clone the repo onto your EC2 instance and changes the directory to EKS terraform files.

```
git clone https://github.com/RavDas/GitHub-Actions-Amazon-EKS-CI-CD-Pipeline.git
cd GitHub-Actions-Amazon-EKS-CI-CD-Pipeline
cd Eks-terraform
```

Change the region of the EKS cluster according your choice.

![2 1](https://github.com/user-attachments/assets/7075972d-555b-4b41-9ffb-25e4ac68ba9d)

Change your S3 bucket in the backend file and region.

![2](https://github.com/user-attachments/assets/9b4fb018-733c-4077-a3b8-4c42e96f7134)


Initialize the terraform

Purpose: Initializes a Terraform working directory.
Key Actions: Downloads and installs the providers defined in the configuration files.
           : Prepares the working directory for other commands.

```
terraform init
```

![image](https://github.com/user-attachments/assets/91927164-81ee-4e24-b51c-d0508fccadbf)

Validate the configuration and syntax of files

Purpose: Validates the configuration files.
Key Actions: Checks the syntax and internal consistency of the configuration files.
           : Ensures that the configuration is syntactically valid and internally consistent.
```
terraform validate
```

![image](https://github.com/user-attachments/assets/e3a8620d-90d1-4176-98bb-76baf2aedca1)


Terraform plan and apply

Purpose: Creates an execution plan.
Key Actions: Shows what actions Terraform will take to achieve the desired state.
           : Identifies the changes required to reach the desired state from the current state.

Purpose: Applies the changes required to reach the desired state.
Key Actions: Executes the actions proposed in the execution plan.
           : Updates or creates infrastructure resources as defined in the configuration files.

```
terraform plan
terraform apply
```

![image](https://github.com/user-attachments/assets/e3e8f0b3-0c2c-48f1-b62c-658bc319b81a)

It will take approximately 10 minutes to create the cluster.

![image](https://github.com/user-attachments/assets/236e034b-d2db-4597-b967-992df0b1909d)

![2 2](https://github.com/user-attachments/assets/2b4e36a2-3f55-4b54-b0da-09c439c6c75a)


Under Clusters -> Node group;

we can see a Node group being created with one EC2 instance running in that node group.

![2 3](https://github.com/user-attachments/assets/2030ea5d-e67c-4924-ab8f-5be9afcaf775)

![image](https://github.com/user-attachments/assets/bc1fd67c-8656-4b9d-81f9-3690c9d887d5)


### 2.C: Integrating SonarQube with GitHub Actions

Integrating SonarQube with GitHub Actions allows you to automatically analyze your code for quality and security as part of your continuous integration pipeline.

We already have SonarQube up and running.

On SonarQube Dashboard click on Manually

![image](https://github.com/user-attachments/assets/29ddc1ba-6264-413a-9566-8119c586e963)

Next, provide a name for your project and provide a Branch name and click on setup

![image](https://github.com/user-attachments/assets/dbcb4731-3640-4e66-8fb4-337f2783f132)


On the next page click on "With GitHub Actions"

This will Generate an overview of the Project and provide some instructions to integrate

![image](https://github.com/user-attachments/assets/acc8c024-9682-433a-9ce9-63c2982369d3)


Let’s Open your GitHub and select your Repository - In my case it is "GitHub-Actions-Amazon-EKS-CI-CD-Pipeline".

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


Now create your workflow for your project. In my case, the project is built using React Js. That’s why I am selecting "Other".

![image](https://github.com/user-attachments/assets/717b69fd-8b30-4c6c-a3bc-abf1f6bcf278)


Now it generates a properties file for SonarQube and a workflow for my project.

![image](https://github.com/user-attachments/assets/f8a0bf76-b72b-4397-8a71-d369ecd7cb72)


Go back to GitHub and add the add file "sonar-project.properties" and copy the following code you get under the file name. You can create a file inside Github itself using Add file -> Create new file and commit changes to the repository of create a file in your local machine repository and push it your remote Github repository.

![image](https://github.com/user-attachments/assets/12c9c6d1-48af-40c2-97f6-5fa1a434ab35)

Add the file with content to the Github repository.

![1 3](https://github.com/user-attachments/assets/bd638a14-809b-4b94-869e-04b49a79e2de)


Now add the workflow of our project.

Again go to your GitHub repository and add the add file "runner.yml"( you can use any name instead of build.yml) inside ".github/workflows/". You can create a file inside Github itself using Add file -> Create new file and commit changes to the repository of create a file in your local machine repository and push it your remote Github repository.

![image](https://github.com/user-attachments/assets/6062bcd4-c5ef-4d76-bfd4-74dbb81b753d)

Copy content and add it to the file.

```
name: Build,Analyze,scan

on:
  push:
    branches:
      - main


jobs:
  build-analyze-scan:
    name: Build
    runs-on: [self-hosted]  # we change it to 'self-hosted' instead of 'ubuntu-latest'
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

![4 2](https://github.com/user-attachments/assets/a0c5eb2f-f852-49d5-b6c0-33b835edaaa1)


**NOTE: When you compare the workflow provided by SonarQube, we change runs-on property from 'self-hosted' to 'ubuntu-latest'. The reason is;**


**################################################################################################################**


GitHub Actions runners can be categorized into two main types: GitHub-hosted runners and self-hosted runners. The YAML configuration for workflows in your repository will generally look similar regardless of the runner type, but there are key differences in how you specify the runners.

1. GitHub-Hosted Runners
These runners are managed by GitHub. They are ephemeral and come pre-installed with a variety of tools and software. To use a GitHub-hosted runner, you simply specify the type of operating system you need in your workflow file.

- runs-on: ubuntu-latest; tells GitHub to use the latest version of the Ubuntu runner.
  
2. Self-Hosted Runners

Self-hosted runners are machines that you manage yourself. You can use any operating system and configure the software environment as needed. This allows for greater customization and control, and it can also help in running workflows that require specific hardware or software setups.

To use a self-hosted runner, you need to specify the self-hosted label in your workflow file. Additionally, you might add custom labels to differentiate between different types of self-hosted runners.

- runs-on: self-hosted tells GitHub to use a self-hosted runner.
  
If you have multiple self-hosted runners with different labels, you can specify those labels to ensure the job runs on the appropriate machine.**

#### Key Differences

Here's a table summarizing the key differences between GitHub-hosted runners and self-hosted runners:

| **Aspect**         | **GitHub-Hosted Runners**                       | **Self-Hosted Runners**                        |
|--------------------|-------------------------------------------------|------------------------------------------------|
| **Runner Specification** | `runs-on: ubuntu-latest`, `runs-on: windows-latest`, etc. | `runs-on: self-hosted` or `runs-on: [self-hosted, custom-label]` |
| **Environment Control** | Pre-configured environments managed by GitHub | Full control over the environment, allowing for customization and use of specific hardware |
| **Maintenance**    | Maintenance, updates, and scaling handled by GitHub | Requires you to maintain the runner, including updates, scaling, and ensuring availability |
| **Cost**           | Limited free usage with GitHub's pricing plans; additional usage may incur costs | No additional cost from GitHub, but you bear the cost of maintaining your own infrastructure |


**#################################################################################################################**


Now the workflow is created.

Start again GitHub actions runner from the EC2 instance. (Inorder to continue the workflow in Github, You shoud start the runner in your ssh-ed EC2 terminal as we are using self-hosted mechanism to run the workflow in Github that uses the EC2 instance resources which we created instead of Github resources.)

```
cd actions-runner
./run.sh
```

Go to your Github repository and click on "Actions" now.

![1 5](https://github.com/user-attachments/assets/bf387db6-5cdc-40cc-938a-04cc1dc17a4c)

Now it’s automatically started the workflow.

![image](https://github.com/user-attachments/assets/e21df26a-7a02-41a2-9414-d2915ff0fcb0)

![1 6](https://github.com/user-attachments/assets/b1f3fc42-f04f-4404-b0fc-2923232b5ae7)

Let’s click on Build and see what are the steps involved.

![image](https://github.com/user-attachments/assets/e65ec23d-52d5-45da-a829-189a72e5c0fe)

Click on Run sonarsource and you can do this after the build completion

![image](https://github.com/user-attachments/assets/1326d3a9-2b32-40aa-951e-78af25286fc4)


After Build is completed, you will following outputs in,

a) In EC2 ssh-ed terminal,

![2 65](https://github.com/user-attachments/assets/5ffe3c52-bd37-46db-acbc-4df5fd262cd3)

b) In Github Actions,

![image](https://github.com/user-attachments/assets/731736dd-5bba-45f6-bbd6-6aa31904505b)


Go to the SonarQube dashboard and click on projects and you can see the analysis

![image](https://github.com/user-attachments/assets/8eb502eb-b138-4c76-a66b-852939ba6fc0)

If you want to see the full report, click on issues.


Now add the remaining steps to the workflow(runner.yml) file.

Next, install npm dependencies - Install Node.js dependencies. You can replace this with your specific npm install command.

```
- name: NPM Install
        run: npm install # Add your specific npm install command
```

Next intsall Trivy - Scans the current directory files (denoted by .) and redirects the output to a file named trivyfs.txt.


```
- name: Install Trivy
        run: |
          # Scanning files
          trivy fs . > trivyfs.txt
```

If you add this to the workflow(runner.yml), you will get below output,

a) In EC2 ssh-ed terminal;

![2 65](https://github.com/user-attachments/assets/88c3c268-e5b5-4d1e-90b2-a6beb3791faf)


b) In Github Actions;

![2 7](https://github.com/user-attachments/assets/a6437e3c-3da7-4967-a9eb-af4b74c3f42f)

![2 8](https://github.com/user-attachments/assets/7e80c51b-70c6-4f61-b7f6-0b321dfa9e70)

To view the ```trivyfs.txt``` file,

```
# cd actions-runner/_work/<github_repository_name>/<github_repository_name>/
cd actions-runner/_work/GitHub-Actions-Amazon-EKS-CI-CD-Pipeline/GitHub-Actions-Amazon-EKS-CI-CD-Pipeline/
cat trivyfs.txt
```

![2 9](https://github.com/user-attachments/assets/1e55d89e-41a4-48ee-9fc1-406b25bdacf4)


### Step 3: Build, push and deploy the Docker image

Create a Personal Access token for your Docker hub account.

Go to Docker hub and click on your profile –> Account settings –> Security –> New access token

![image](https://github.com/user-attachments/assets/c0fc53cc-c3d5-4fff-9d07-f356330c34d1)

Provide a name and click on"Generate" to receive a token.

![image](https://github.com/user-attachments/assets/39bc16c7-3a77-46fe-94e8-01f155c18bcb)

Copy the token save it in a safe place, and close.


Now Go to GitHub again and click on "Settings".

![1 2](https://github.com/user-attachments/assets/d1386235-d33b-4f82-bf9f-cb4240b6d60a)

Search for "Secrets and variables" and click on and again click on "Actions".

![image](https://github.com/user-attachments/assets/ab89c5c6-fb73-4808-a46d-67987e2fc74f)

Click on "New Repository secret".

![image](https://github.com/user-attachments/assets/35fa5ab0-419b-42dd-bccd-942a2430abbd)

Add your Dockerhub username with the secret name as;

```
Name - DOCKERHUB_USERNAME
Secret - <ravdas>   #use your docker hub username
```

![image](https://github.com/user-attachments/assets/9b573f88-736c-4dc8-917d-192f8b5ff090)

Click on "Add secret".

Let’s add our token also. Again, click on the new repository secret again.

```
Name - DOCKERHUB_TOKEN
Secret - <generated_token_from_dockerhub>
```

![image](https://github.com/user-attachments/assets/c0f763bf-0793-4cc3-95ad-038ff19c0216)

Paste the token that you generated and click on Add secret.


Now, return back to the workflow (```runner.yml```) in Github and add the following step that builds a Docker image with specific build arguments and tags. It also logs into Docker hub using the provided credentials stored in secrets and pushes the Docker image to the Docker hub.

```
- name: Docker build and push
        run: |
          # Run commands to build and push Docker images
          docker build -t ravdas/tic-tac-toe:latest .  # tag using <dockerhub_username/any_name:latest>
          docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}
          docker push ravdas/tic-tac-toe:latest
        env:
          DOCKER_CLI_ACI: 1
```

If you run this step inside build job, now you will get below output in Github Actions,

![image](https://github.com/user-attachments/assets/f855fceb-ce3d-4bcd-adfb-5059746d423f)

Now go to Docker hub abd check whether the Docker image is pushed to Docker hub.

![3 3](https://github.com/user-attachments/assets/19aba6c7-63b0-4289-800e-7786c5cf9dcd)


To Deploy the Docker image, add the following code to the workflow (```runner.yml```) file.

```
deploy:
    needs: build-analyze-scan
    runs-on: [self-hosted] # Use your self-hosted runner label here
```

Above section defines another job named “deploy.” It specifies that this job depends on the successful completion of the “build-analyze-scan” job. It also runs on a self-hosted runner. You should replace self-hosted with the label of your self-hosted runner.

Then pull the Docker image from Docker Hub, specified by the tag of the image(ravdas/tic-tac-toe:latest), which was built and pushed in the previous “Docker build and push” job.

```
steps:
      - name: Pull the Docker image
        run: docker pull ravdas/tic-tac-toe:latest
```

Below step runs Trivy to scan the Docker image tagged as sevenajay/tic-tac-toe:latest. You should add the Trivy scan command here.

```
    - name: Trivy image scan
            run: trivy image ravdas/tic-tac-toe:latest # Add Trivy scan command here
```

After modifying the workflow, the output in Github Action will be,

![image](https://github.com/user-attachments/assets/468059f6-26f9-4192-9af0-6fc5f1c12634)


Image scan report of the Docker image can be accessed by viewing the ```trivyimage.txt``` file,

```
# cd actions-runner/_work/<github_repository_name>/<github_repository_name>/
cd actions-runner/_work/GitHub-Actions-Amazon-EKS-CI-CD-Pipeline/GitHub-Actions-Amazon-EKS-CI-CD-Pipeline/
cat trivyimage.txt
```

![image](https://github.com/user-attachments/assets/719013d0-1611-4ff7-afbb-9b0184f00389)

Next step runs a Docker container named “game” in detached mode (-d). It maps port 3000 on the host to port 3000 in the container. It uses the Docker image tagged as ravdas/tic-tac-toe:latest. (Make sure to open the port 3000 of the EC2 instance via security group of it to allow inbound traffic)


```
    - name: Run the container
            run: docker run -d --name ticgame -p 3000:3000 ravdas/tic-tac-toe:latest
```

When we modify the workflow with deploy job, the output in Github Action will be,

![image](https://github.com/user-attachments/assets/f8ad2710-2ef4-4ddd-8b92-d1f129aad0a1)


After build stage gets successfull, then only it moves to the deploy stage. if not both stages will fail.

![image](https://github.com/user-attachments/assets/7c668d12-15ea-46a6-87fa-3586ce028850)

![3 0](https://github.com/user-attachments/assets/b7709cf4-49cc-4887-9091-a5692eb24de1)


Now the application has been dployed to the container. We can access the application using,

```
<ec2-ip:3000>
```

![image](https://github.com/user-attachments/assets/b7a5fa4d-f99a-45d2-a2e1-8f75fefdb38b)


### STEP 4: Deploy to EKS


To provide;

* Necessary configuration to connect to and interact with your Amazon EKS cluster.

* Authentication and Authorization: The kubeconfig file contains the necessary authentication credentials and endpoint information required to authenticate your requests and authorize your access to the cluster.

* Seamless Cluster Management: Updating the kubeconfig allows you to use kubectl and other Kubernetes tools seamlessly with your EKS cluster, enabling you to manage resources, deploy applications, and monitor cluster status effectively.

Add the below step to the workflow.

```
      - name: Update kubeconfig
        run: aws eks --region <cluster-region> update-kubeconfig --name <cluster-name>
```

Now add a step to deploys Kubernetes resources defined in the deployment-service.yml file to the Amazon EKS cluster using kubectl apply.

```
      - name: Deploy to EKS
        run: kubectl apply -f deployment-service.yml
```

The output in Github Actions after adding two recent steps to the workflow ,

![deploy 22](https://github.com/user-attachments/assets/f783707f-f8e4-4c54-869b-0010c2ce2e97)

Output from the EC2 terminal about deployemnt, services, replica sets and pods in Kubernets cluster,

![get all](https://github.com/user-attachments/assets/9dbdd804-a043-47b8-ad0c-9f810daf2b13)

Further, it will create a Load Balancer on our AWS account when we apply ```deployment-service.yml```, as we have defined Kubernetes service using Load Balancer.

![lb](https://github.com/user-attachments/assets/86893599-5f69-4be7-a8b5-486e11c87170)



### Step 5: SLACK Notifications


Go to your Slack channel, if you don’t have [create one](https://slack.com/get-started#/createnew).

Go to Slack channel and create a channel for notifications

Click on your name

Select "Settings and Administration"

Click on "Manage apps"

![image](https://github.com/user-attachments/assets/270eaffb-78c2-4328-bd9a-0314154ad452)


It will open a new tab, select "Build".

![image](https://github.com/user-attachments/assets/0d504eb7-17a4-4a8f-89c7-ec74324c10e8)

Click on create an app

![image](https://github.com/user-attachments/assets/6bb0009b-ade2-493b-810b-9b30bb8d53c2)

Select from scratch

![image](https://github.com/user-attachments/assets/34ed5ed3-fb6a-483a-a086-0b8abf81b9b9)

Provide a name for the app and select workspace and create

![image](https://github.com/user-attachments/assets/7d05e2f2-56ec-41e3-9bce-f93cba5e6d7e)

Select "Incoming Webhooks"

![image](https://github.com/user-attachments/assets/b59ac197-7829-4c71-8687-eabbd449a3c5)

Set "Incoming Webhooks" to On

![image](https://github.com/user-attachments/assets/e530df38-c8aa-4be8-b6ce-a021750ecc41)

Click on "Add New Webhook to Workspace".

![image](https://github.com/user-attachments/assets/74799aba-de19-498a-8464-7ec0dceeae1f)

Select Your channel that created for notifications and allow

![image](https://github.com/user-attachments/assets/957d573a-2241-4ca7-911f-2ad15fd7aaac)


It will generate a webhook URL. Copy it

Now come back to GitHub and click on "Settings"

Go to Secrets –> Actions –> New repository secret and add

```
Name - SLACK_WEBHOOK_URL
Secret - <copied_webhook_url>
```

![image](https://github.com/user-attachments/assets/f6f8a26e-fbe6-411e-92e3-abca15a662cc)

Now add the below code to the workflow(```runner.yml```) and commit to start the workflow.

```
      - name: Send a Slack Notification
        if: always()
        uses: act10ns/slack@v1
        with:
          status: ${{ job.status }}
          steps: ${{ toJson(steps) }}
          channel: '#git'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

Above step sends a Slack notification. It uses the act10ns/slack action and is configured to run “always,” which means it runs regardless of the job status. It sends the notification to the specified Slack channel using the webhook URL stored in secrets.



**========================================================================================================**


#### Complete Workflow(runner.yml)

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
      - name: npm install dependency
        run: npm install
      - name: Trivy file scan
        run: trivy fs . > trivyfs.txt
      - name: Docker Build and push
        run: |
          docker build -t tic-tac-toe .
          docker tag tic-tac-toe sevenajay/tic-tac-toe:latest
          docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}
          docker push sevenajay/tic-tac-toe:latest
        env:
          DOCKER_CLI_ACI: 1
      - name: Image scan
        run: trivy image sevenajay/tic-tac-toe:latest > trivyimage.txt
  deploy:
   needs: build-analyze-scan
   runs-on: [self-hosted]
   steps:
      - name: docker pull image
        run: docker pull sevenajay/tic-tac-toe:latest
      - name: Image scan
        run: trivy image sevenajay/tic-tac-toe:latest > trivyimagedeploy.txt
      - name: Deploy to container
        run: docker run -d --name game -p 3000:3000 sevenajay/tic-tac-toe:latest
      - name: Update kubeconfig
        run: aws eks --region ap-south-1 update-kubeconfig --name EKS_CLOUD
      - name: Deploy to kubernetes
        run: kubectl apply -f deployment-service.yml
      - name: Send a Slack Notification
        if: always()
        uses: act10ns/slack@v1
        with:
          status: ${{ job.status }}
          steps: ${{ toJson(steps) }}
          channel: '#githubactions-eks'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Step 6: Output From running the complete workflow,

Summary of Build and deploy jobs pipeline,

![summary](https://github.com/user-attachments/assets/6460362b-f7c4-4fef-a28a-65337bb18748)

Build job workflow,

![build](https://github.com/user-attachments/assets/20d2220d-2a40-458d-98b7-476cf024fca3)

Image scan report,

![scan](https://github.com/user-attachments/assets/854d18dc-a8a7-4321-a715-524510d1ff76)

Deploy job workflow,

![deploy](https://github.com/user-attachments/assets/f8e9b55f-18f6-42be-8ce3-407ab548f025)

![4 1](https://github.com/user-attachments/assets/ecf4cb1e-9894-4e98-a0b5-f2e76f4fa23b)

```./run.sh``` output in the ssh-ed EC2 terminal.

![success](https://github.com/user-attachments/assets/13f5d167-bd4d-4f14-89b0-20c8992f0696)

To view Kubernetes cluster created provide following command on EC2 terminal,

```
kubectl get all
```

![get all](https://github.com/user-attachments/assets/22973d29-106d-4aa9-820a-8e8a4bd667eb)


Now all jobs are completed and the workflow is built without any errors!!

To view the application on the browser, open the port in the security group for the Node group instance which is created when we created the EKS cluster using terraform. (It is the port where the Load Balancer service will introduce to allow external traffic to the cluster. In our case it is 31849)

![node](https://github.com/user-attachments/assets/e4c100f2-f6a2-4831-8d8f-c6a1534f6b8a)

![sg port](https://github.com/user-attachments/assets/c88152ac-71d7-4fca-a36e-b9b523ec8fb9)

Load Balancer created,

![lbs](https://github.com/user-attachments/assets/69ad1d4e-f7a8-44b2-9ec2-a59bce4016ff)


After that copy the external IP and paste it into the browser.

![url lb](https://github.com/user-attachments/assets/28952c73-8028-4459-9c72-de345b7f5e6d)

```
<external_ip/<port>
```

Output,

![image](https://github.com/user-attachments/assets/0ac6e479-59c6-474e-b334-489025d9bc72)



### Step 7: Delete the Workflow and AWS, EKS Cluster resources


After the jobs are completed and successful deployment, we can delete the Kubernetes Cluster resources by running the below code in the workflow(```runner.yml```). It will delete the container and delete the Kubernetes deployment.


##### Destruction workflow

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

      - name: Deploy to container
        run: | 
          docker stop game
          docker rm game

      - name: Update kubeconfig
        run: aws eks --region ap-south-1 update-kubeconfig --name EKS_CLOUD

      - name: Deploy to kubernetes
        run: kubectl delete -f deployment-service.yml

      - name: Send a Slack Notification
        if: always()
        uses: act10ns/slack@v1
        with:
          status: ${{ job.status }}
          steps: ${{ toJson(steps) }}
          channel: '#githubactions-eks'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

```


Slack Notification or runnign the destructive workflow

![image](https://github.com/user-attachments/assets/27a46755-6dc2-4667-9663-10707c5bf414)

No Laod Balancer available now.

![lbss](https://github.com/user-attachments/assets/0aeb14c4-19e9-4e38-a895-4117191f664f)

Running the destructive workflow in Github Actions,

![delete](https://github.com/user-attachments/assets/033311f8-4523-4cc3-82b6-3dcbba0d6d8c)


Stop and remove the self-hosted runner.

![7 1](https://github.com/user-attachments/assets/90aa5764-0d49-483c-ad69-a3004dfb41a9)

![7 2](https://github.com/user-attachments/assets/e34a0194-aad0-4226-9e58-66f45279d293)

Copy the above command and paste in the EC2 ssh-ed terminal.

![7 3](https://github.com/user-attachments/assets/49c6a693-535b-4df5-97d4-bf3794b84123)


Now to delete the EKS cluster using Terraform,

Go to,

```
cd /home/ubuntu/GitHub-Actions-Amazon-EKS-CI-CD-Pipeline/Eks-terraform

terraform destroy --auto-approve
```

It will take 10 minutes to destroy the EKS cluster.

![terra](https://github.com/user-attachments/assets/f054bf56-ea7d-423e-aed5-65551a422be3)

Meanwhile, delete the Docker hub token (Account settings -> Security)

![delete docker token](https://github.com/user-attachments/assets/65dc6aaf-0829-4e6c-9b64-b5da4d39d93e)

Once cluster destroys, delete The EC2 instance and IAM role.

![terminate inst](https://github.com/user-attachments/assets/a5808d58-5124-4d2c-bb0b-5f80e8016c9f)

Delete the secrets from GitHub also.


