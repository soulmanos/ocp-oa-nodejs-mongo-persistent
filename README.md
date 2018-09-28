## Using OpenShift-Applier to create a new Project and Deploy an Application

This repository has been updated to use the openshift-ansible master branch from 3.9.0. Updated to make use of the params_from_vars module. The master branch of openshift applier also requires ansible >= 2.5

* Clone this repo

`git clone https://github.com/soulmanos/ocp-oa-nodejs-mongo-persistent.git`  

* Run the ansible-playbook

 ansible-playbook -i inventory apply.yml -e "project_name=smokey-test target=bootstrap,application"

* Confirm Application reponds with A DB connection established

`curl http://nodejs-mongo-persistent-smoke-test.apps.$GUID.example.opentlc.com | grep -A4 'DB Connection Info'`

* Manual steps can be reproduced from [openshift-applier-example.md](openshift-applier-example.md)
