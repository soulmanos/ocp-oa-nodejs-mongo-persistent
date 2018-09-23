## Using OpenShift-Applier to create a new Project and Deploy an Application

* References

https://www.katacoda.com/logandonley/scenarios/openshift-applier  
https://mcanoy.github.io/2017/10/26/deploying-gitlab-to-openshift-with-the-applier/

* Manual Steps to Deploy a 'smoke-test' app:
```
oc new-project smoke-test
oc new-app nodejs-mongo-persistent
watch oc get pod
oc get route
curl http://nodejs-mongo-persistent-smoke-test.apps.$GUID.example.opentlc.com | grep -A4 'DB Connection Info'
oc delete project smoke-test
```
* Create project directory for openshift-applier  

`mkdir sample-applier && cd $_`

* Create ansible-galaxy requirements file
```
cat <<EOM >requirements.yml
- name: openshift-applier
  scm: git
  src: https://github.com/redhat-cop/openshift-applier
  version: v3.9.0
EOM
```
* Create additional directory structure for ansible-playbook

```
mkdir -p inventory/{group_vars,host_vars} params templates
touch inventory/group_vars/all.yml inventory/host_vars/{application.yml,bootstrap.yml} inventory/hosts
```
* This is the endstate project directory after all steps followed below
```
.
├── apply.yml
├── inventory
│   ├── group_vars
│   │   └── all.yml
│   ├── hosts
│   └── host_vars
│       ├── application.yml
│       └── bootstrap.yml
├── params
│   ├── nodejs-mongo-persistent
│   │   └── build
│   └── projectrequests
│       └── project
├── requirements.yml
├── roles
│   └── openshift-applier
│       └── etc ...
└── templates
    └── app
        └── nodejs-mongo-persistent.yml
```
* Find the name of the application template you want to pull in

`oc get templates -n openshift | grep nodejs-mongo-persistent`  

```
mkdir templates/app
oc export template nodejs-mongo-persistent -n openshift -o yaml > templates/app/nodejs-mongo-persistent.yml
```
* Examine `parameters:` section at the bottom of the `*.yml`  

For this particular template (nodejs-mongo-persistent) all the parameters have default values specified, the ruby template requires parameters
* Create a parameters file - App won't deploy without one - Just set a value to one already set - Get the app to deploy
```
mkdir params/nodejs-mongo-persistent
echo 'NAME=nodejs-mongo-persistent' > params/nodejs-mongo-persistent/build
```
* Create the OpenShift Project/NameSpace where the application will run
```
mkdir params/projectrequests

cat <<EOM >params/projectrequests/project
NAMESPACE=smoke-test
NAMESPACE_DISPLAY_NAME="smoke-test nodejs-mongo-example"
EOM
```
* OpenShift Applier Template Format, populated with the standard OpenShift project request template
```
cat <<EOM >inventory/host_vars/bootstrap.yml
---
ansible_connection: local
openshift_cluster_content:
- object: projects
  content:
  - name: dev
    template: "https://raw.githubusercontent.com/redhat-cop/cluster-lifecycle/master/files/projectrequest/template.yml"
    template_action: create
    params: "{{ playbook_dir }}/params/projectrequests/project"
    tags:
    - projectrequests
    - projectrequests-dev
EOM
```
* To finish up the inventory, we need to update the hosts file to include the host_vars we just created.
```
cat <<EOM >inventory/hosts
[bootstrap]
bootstrap

[application]
application
EOM
```
* Next we need to create the openshift_cluster_content to tell it to create OpenShift objects from the template and parameters
```
cat <<EOM >inventory/host_vars/application.yml
---
ansible_connection: local
openshift_cluster_content:
- object: nodejs-components
  content:
  - name: nodejs-ex
    template: "{{ playbook_dir }}/templates/app/nodejs-mongo-persistent.yml"
    params: "{{ playbook_dir }}/params/nodejs-mongo-persistent/build"
    namespace: "{{ nodejs_namespace }}"
    tags:
    - app
EOM
```
* Create a playbook that'll call the OpenShift Applier
```
cat <<EOM >apply.yml
---
- name: install-required-roles--openshift-applier--with-ansible-galaxy
  hosts: localhost
  tasks:
    - name: install-ansible-roles
      local_action: command ansible-galaxy install -r requirements.yml -p roles/

- name: Deploy {{ target }} 
  hosts: "{{ target }}"
  vars:
    nodejs_namespace: "smoke-test"
  tasks:
    - include_role:
        name: openshift-applier/roles/openshift-applier
EOM
```

* Install the openshift-applier Ansible role via galaxy  

`ansible-galaxy install -r requirements.yml -p roles`

* Deploy the app

`ansible-playbook -i inventory apply.yml -e "target=bootstrap,application"`

### Notes

> On first run it failed - This is because we skipped the params file for the template `nodejs-mongo-persistent`

```
TASK [openshift-applier/roles/openshift-applier : Fail if template but no params is specified] ****************************************************************************************************
fatal: [application]: FAILED! => {"changed": false, "failed": true, "msg": "Template specified, but no params file supplied"}
```

* To use this playbook in another playbook

> `include` statements are processed as they are encountered.

```
- name: Get required roles with ansible galaxy
  local_action: command ansible-galaxy install -r requirements.yml -p roles/

- name: Install some.role
  include_role: 
    name: some.role
```    