## Using OpenShift-Applier to create a new Project and Deploy an Application

* Clone this repo

`git clone https://github.com/soulmanos/ocp-oa-nodejs-mongo-persistent.git`  

* Run the ansible-playbook

`ansible-playbook -i inventory apply.yml -e "target=bootstrap,application"`

* Manual steps can be reproduced from [opensift-applier-example.md](openshift-applier-example.md)