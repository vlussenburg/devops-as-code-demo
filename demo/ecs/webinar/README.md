
## 1.	The code is in this repository: https://github.com/marcoman/devops-as-code-demo - clone this repo to get the templates noted below.
 
## 2.	Run this from the base directory (You may need to --build this).
docker-compose up
NOTE: This brings in XLD/XLR 8.5.2 as the previous version was 8.5.0

## 3.	Per https://github.com/xebialabs/devops-as-code-demo/tree/master/demo/ecs, setup your AWS credentials.
- ../../config/awsconfig2xld.py  > awsconfig.yaml
- xl apply -f awsconfig.yaml
- create a file .xlvals with content:
```
jira_username=user@xebialabs.com
jira_password=Your Jira token, not password.
```
- xl apply -f ../../config/configure-xl-devops-platform.yaml

## 4.	Next, apply the templates in the demo/ecs directory:
```
xl apply -f webinar/xlr-lift-and-shift.yaml
xl apply -f webinar/xld-infrastructure-local-docker.yaml
xl apply -f webinar/xld-infrastructure-aws.yaml
xl apply -f webinar/xld-environments-local-docker.yaml
xl apply -f webinar/xld-environments-aws.yaml
```

NOTE: The environment file for local docker is remarkably empty. Looks like xl generate doesn’t produce the right bits.  This is something I’ll have to work on.


## 5.	Fix-up template references not included in generate:

- XLD: Infrastructure/aws needs verifySSL to be checked (YAML value is true
- XLD: Environments/Local Docker needs to be created - YAML generate doesn't work (need to figure it out, probably very simple).
  -	Really it is to name it as "Local Docker"
  -	Use the "Infrastrcture/docker-local" container.


## 6.	 XLR: template Lift-and-Shift / DEV phase/Create Jira Work Task needs a Title
