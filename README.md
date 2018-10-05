## Using OpenShift-Applier to create a new Project and Deploy an Application

Switch to branch Requires-Ansible-2.5 to use the most up-to-date version of this bootstrap

* Clone this repo

`git clone https://github.com/soulmanos/ocp-oa-nodejs-mongo-persistent.git`  

* Run the ansible-playbook

`ansible-playbook -i inventory apply.yml -e "target=bootstrap,application"`

* Confirm Application reponds with A DB connection established

`curl http://nodejs-mongo-persistent-smoke-test.apps.$GUID.example.opentlc.com | grep -A4 'DB Connection Info'`

* Manual steps can be reproduced from [openshift-applier-example.md](openshift-applier-example.md)
