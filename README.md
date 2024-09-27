# MATTERMOST DEPLOYMENT GUIDE

## Preparation
    *install docker*
    
    sudo yum install docker -y
    sudo systemctl start docker
    
    *install docker-compose
    
    sudo curl -L https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-compose-plugin-2.6.0-3.el7.x86_64.rpm -o ./compose-plugin.rpm
    sudo yum install ./compose-plugin.rpm -y"


### Bare Metal Install




### Docker "Preview"

To start, we'll try out the pre-configured Mattermost docker container. Once docker is configured successfully, this is relatively straightforward.

    sudo docker run --name mattermost-preview -d --publish 8065:8065 --add-host dockerhost:127.0.0.1 mattermost/mattermost-preview

The container should now be running. You can check the name, container ID, and the status by running the following command:

    sudo docker ps

Now, check the web server:

    curl localhost:8065

Finally, paste the public IP of the EC2 instance (or simply localhost if running locally)--with the port--in the browser to continue configuration"

    http://localhost:8065


Once the service loads, step through the configuration, create a trial account with a user and email, and get started!


### Full Docker Deployment

Let's go for the gold this time and install the full production release. Clone the repo:

    git clone https://github.com/mattermost/docker
    cd docker

Next, we'll create all the required folders for our deployment:

    mkdir -p ./volumes/app/mattermost/{config,data,logs,plugins,client/plugins,bleve-indexes}

Change ownership of all folders to the service:

    sudo chown -R 2000:2000 ./volumes/app/mattermost

For sake of brevity, I'm skipping the NGINX install and SSO. Please note, it is more secure to not have the public internet hitting mattermost server directly, DDOS, etc

Let's deploy:

    sudo docker compose -f docker-compose.without-nginx.yml up -d

Once finished, test and make sure the service is up:

    curl localhost:8065

Considerations:

1. Before deployment, you could change env file for settings. Or do this and redeploy. Config json is in ./volumes/app/mattermost/
2. For live config changes, you can jump into the container (docker exec -it CONTAINER_ID bash). Logs are in ./mattermost/logs/mattermost.log. Also config in ./mattermost/config/config.json

    
Overall, using the repo was pretty straightforward to install--as it creates both the mattermost container and the postgres container. I did some experimenting with a baremetal install as well, which was similar but obviously creating a standalone sql instance.



### Extra credit: K8s Deployment

My Kubernetes is rusty. Still, I wanted to try deploying the mattermost operator. I was able to get it deployed, but couldn't get it to come up. Still, the steps are below:

## Preparation
    Create a K8 cluster in EKS
    
    *Install kubectl*
    
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2024-09-12/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    
    *Install helm*

    curl -L https://git.io/get_helm.sh | bash -s -- --version v3.8.2

    *Connect kubectl to your EKS cluster and generate config*

    aws eks update-kubeconfig --region us-east-2 --name mattermost

Create mattermost-license-secret.yml

    apiVersion: v1
    kind: Secret
    metadata:
      name: mattermost-license
    type: Opaque
    stringData:
      license: "foo"
    











helm repo add mattermost https://helm.mattermost.com
