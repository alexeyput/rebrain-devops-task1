# Rebrain DevOps practicum
## Task 4 - GIT 04
![Task](https://github.com/alexeyput/rebrain-devops-task1/blob/master/img/git04-task.png?raw=true)

## Client's request
> Create full pipeline to deploying an infrastructure as well as applications in docker containers   
> Management servers must be deployed with minimal manual intervention

#### Languages used to create applications:
- Python
- Java
#### Tools and services used in project:
- Jenkins as an automation tool
- Terraform for infrastructure provisioning
- Ansible for configuring web server and deploying docker containers
- Docker as a container management service
- Maven to build java application
- GitLab as a source code repository
- Nexus as a docker images repository
- Snyk as a source code security checker
- Amazon Web Services as a cloud infrastructure provider

## How it works
1. Developer initiates push event to Git repository
2. Push event triggers playbook to start
3. Jenkins server downloads project files from the repository
4. Responsible person is notified through Telegram bot that the pipeline has been started
5. Jenkins invokes Terraform to deploy infrastructure
6. Python app docker image is being built and uploaded to Nexus repository
7. Java artefact is being created 
8. Java app docker image is being built and uploaded to Nexus repository
9. Ansible installs and configures all the necessary packets and tools on Ec2 instance
10. Pull docker images from Nexus repository and deploy docker containers on Ec2 instance
11. Application's availability are tests.
12. Python app available at address: **<web-server-IP/FQDN>:8090**  
    Java app available at address: **<web-server-IP/FQDN>:8091**

##  Prerequisites
1. Deploy management station:  

_Initialize terraform providers_
```
   terraform plan
```
_To create plan and simulate infrastructure creation_
```
   terraform plan
```
_To perform infrastructure creation_
```
   terraform apply [-auto-approve]
```
_To delete created infrastructure._    
```
   terraform destroy [-auto-approve]
```   
**NOTE. S3 Bucket will not be deleted in case of having stored objects. Please check and delete manually if it necessary.**
2. Install or update the following Jenkins plugins (more information on [Jenkins User Documentation](https://www.jenkins.io/doc/)):
   - HTTP Request Plugin
   - Credentials
   - GitLab
   - Telegram Bot Plugin
   - Snyk Security
   - Email Extension Plugin
4. Create the following security entities in Jenkins:

|  ID               | Kind                          | Description                                        | 
|-------------------|-------------------------------|----------------------------------------------------| 
| gitlab            | Username with password        | Username/password for Gitlab repository            |
| nexus-credentials | Username with password        | Username/password for Nexus repository             | 
| aws-key-pair      | SSH Username with private key | Username and private key to access to Ec2 instance |
| Secret_Access_Key | Secret text                   | AWS Secret access key                              |
| Access_Key_ID     | Secret text                   | AWS Access Key ID                                  |
| telegramToken     | Secret text                   | Telegram Bot token                                 |
| telegramChatId    | Secret text                   | Telegram Chat Id for notofications                 |
5. Create Jenkins pipeline and point it to Git repository
6. Copy private keys to your Jenkins server
7. Edit parameters mentioned in "Configuration files and parameters" section
8. Create Gitlab integration or use webhooks to trigger push events. **Note**. Jenkins port must be seen outside.
9. Create Snyk account and configure integration with the repository

##  Project's folders structure
```
.
├── master                    - Management station part
│   ├── config                - List of repos to which it is allowed to connect via HTTP
│   ├── docker                - Docker compose file to deploy infrastructure services
│   └── terraform             - Template to create infrastructure for the management station
└── slave                     - Web server part
    ├── ansible               - Playbook for instance configuration and containers deployment
    ├── config                - List of repos to which it is allowed to connect via HTTP
    ├── docker                - Docker compose file to deploy docker containers (alternative option)
    ├── jenkins               - Jenkins project to automate CI/CD process
    ├── src
    │   ├── java              - Java app source code
    │   │   ├── Dockerfile    - Used to create java app docker image
    │   │   ├── src
    │   │   │   ...
    │   │   │   └── springboot
    │   │   │    └── Application.java   - Here you can change initial start page
    │   └── python
    │       ├── Dockerfile              - Used to create python app docker image
    │       └── templates
    │           └── index.html          - Here you can change initial start page
    └── terraform                       - Template to create infrastructure for web server
```
## Configuration files and parameters
**slave/jenkins/Jenkinsfile:**

| Parameter         | Description                  |
|-------------------|------------------------------|
| reg_url           | Private repository URL       |
| python_image_name | Python app docker image name |
| java_image_name   | Java app docker image name   |

**slave/terraform/terraform.tfvars:**

| Parameter         | Description                          |
|-------------------|--------------------------------------|
| cidr_block        | VPC CIDR block                       |
| public_cidr_block | Public network CIDR block            |
| avail_zone        | AWS Availability Zone                |
| aws_region        | AWS region                           |
| env_prefix        | Environment prefix (dev, prod, etc.) |
| instance_type     | Ec2 instance type                    |
| keyname           | Private key name to access EC2       |

**slave/ansible/group_vars/all.yaml**

| Parameter                    | Description                                 |
|------------------------------|---------------------------------------------|
| ansible_ssh_private_key_file | Private key used to connect to Ec2 instance |
| ansible_user                 | Username used to connect to Ec2 instance    |
