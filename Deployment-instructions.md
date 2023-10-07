<p align="center">
<img src="https://github.com/kura-labs-org/kuralabs_deployment_1/blob/main/Kuralogo.png">
</p>

## Deployment Instructions:
1. **You MUST clone, branch, make the update to the Jenkinsfile, and merge back into main, before you start your Jenkins build!!!** 
2.  Create a T.2 medium in your public subnet
3. Create a Security Group with ports: 80, 8080, 8000, and 22
4. Install Jenkins and install the following on the T.2 medium:
    - "python3.10-venv", "python3-pip", "ngnix" 
5. Install the following plugin: “Pipeline Keep Running Step”
6. Once you've installed Nginx, edit the configuration file "/etc/nginx/sites-enabled/default" with the information below:
```
###First change the port from 80 to 5000, see below:
server {
listen 5000 default_server;
listen [::]:5000 default_server;

####Now scroll down to where you see “location” and replace it
with the text below:

location / {
proxy_pass http://127.0.0.1:8000;
proxy_set_header Host $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}

```
4. Configure either a Cloudwatch agent or Datadog agent on this server
5. Now update the jenkinsfile with the script below (Use git to update the jenkinsfile): 
```
pipeline {
agent any
stages {
stage ('Build') {
steps {
sh '''#!/bin/bash
python3 -m venv test3
source test3/bin/activate
pip install pip --upgrade
pip install -r requirements.txt
export FLASK_APP=application
'''
}
}
stage ('test') {
steps {
sh '''#!/bin/bash
source test3/bin/activate
py.test --verbose --junit-xml test-reports/results.xml
'''
}
post{
always {
junit 'test-reports/results.xml'
}
}
}
stage ('Clean') {
steps {
sh '''#!/bin/bash
if [[ $(ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2) != 0 ]]
then
ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2 > pid.txt
kill $(cat pid.txt)
exit 0
fi
'''
}
}
stage ('Deploy') {
steps {
keepRunning {
sh '''#!/bin/bash
pip install -r requirements.txt
pip install gunicorn
python3 -m gunicorn -w 4 application:app -b 0.0.0.0 --daemon
'''
}
}
}
}
}
```
6. Create your multi-branch and run it
7. How is the server performing?
8. Can the server handle everything installed on it? if yes, how would a T.2 micro handle in this deployment? 
9. What happens to the CPU when you run another build?
10. Configure an alert or figure out how to configure email notifications on Jenkins
