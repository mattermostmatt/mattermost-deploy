# Markdown syntax guide

## Headers

# This is a Heading h1
## This is a Heading h2
###### This is a Heading h6

## Emphasis

*This text will be italic*  
_This will also be italic_

**This text will be bold**  
__This will also be bold__

_You **can** combine them_

## Lists

### Unordered

* Item 1
* Item 2
* Item 2a
* Item 2b

### Ordered

1. Item 1
2. Item 2
3. Item 3
    1. Item 3a
    2. Item 3b


Mattermost Deployment Guide



I. Docker "Preview"
II. Full Docker Deployment
III. K8s Deployment




#I. Docker "Preview"


Before getting started with the Mattermost preview container, we need to set up Docker. I've deployed a fresh Amazon Linux EC2 Instance, and have set it up as below:

"sudo yum install docker -y" --*Install Docker*

"sudo systemctl start docker" --*Set Docker to start on system start*

"sudo docker ps" --*Check to make sure no containers are currently running and test the install*

"sudo curl -L https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-compose-plugin-2.6.0-3.el7.x86_64.rpm -o ./compose-plugin.rpm
sudo yum install ./compose-plugin.rpm -y" --*Install docker-compose*

*Now, we'll download and preview the mattermost-preview container, using default configuration:*

docker run --name mattermost-preview -d --publish 8065:8065 --add-host dockerhost:127.0.0.1 mattermost/mattermost-preview

And finally, test to make sure the service is up:

curl localhost:8065

In a browser, enter the public IP of the EC2 instance (configured for access on the port) with the port, to access your preview install.


In Mattermost, 
Continue in browser
Create trial account, this will be for the server itself
Test it out!


II. Full Docker Deployment



sudo yum install docker -y
sudo systemctl start docker
sudo docker ps
sudo curl -L https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-compose-plugin-2.6.0-3.el7.x86_64.rpm -o ./compose-plugin.rpm
sudo yum install ./compose-plugin.rpm -y

git clone https://github.com/mattermost/docker
cd docker


Creates all required folders

mkdir -p ./volumes/app/mattermost/{config,data,logs,plugins,client/plugins,bleve-indexes}

Changes ownership of all folders to the service

sudo chown -R 2000:2000 ./volumes/app/mattermost

Skip NGINX reverse proxy, though it is more secure to not have the public internet hitting mattermost server directly, DDOS, etc

Not configuring SSO (With Gitlab or Okta, etc etc for brevity)

sudo docker compose -f docker-compose.without-nginx.yml up -d

Same deal with configure license, user, etc
curl localhost:8065

Could change env file for settings, redeploy. Also, config json is in ./volumes/app/mattermost/

docker exec -it CONTAINER_ID bash

Logs are in ./mattermost/logs/mattermost.log. Also config in ./mattermost/config/config.json


III. K8s Deployment

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2024-09-12/bin/linux/amd64/kubectl
chmod +x ./kubectl

aws eks update-kubeconfig --region us-east-2 --name mattermost


curl -L https://git.io/get_helm.sh | bash -s -- --version v3.8.2

helm repo add mattermost https://helm.mattermost.com
