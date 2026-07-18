#GitHub Actions:
------------------- 
#step1 

paymentAction
│
├── src
├── pom.xml
├── Dockerfile
├── README.md
└── .github
      └── workflows
      
#create dir
mkdir -p .github/workflows

#GitHub Action
.github/workflows/build.yml

#git operation
git add .
git commit -m "Added GitHub Action"
git push origin main

# check git Action in Browser

# There is no Jenkins.Jenkins is in built in git Action

# Deployment to Aws cloud
ssh -i piet.pem ubuntu@54.184.142.1
install docker in Ec2 machine

sudo groupadd docker
sudo usermod -aG docker ubuntu

Update Ec2 IP if ec2 machin has stopped. after start it has new IP


#GitHub action Secrets
Settings → Secrets and variables → Actions

DOCKER_USERNAME=cnsnoida
DOCKER_PASSWORD=xxxxxx  (personal token)

EC2_HOST=54.xx.xx.xx
EC2_USER=ec2-user

EC2_SSH_KEY= .pem file
