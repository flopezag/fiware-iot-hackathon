# <a name="top"></a>FIWARE IoT Hackathon content
[![License badge](https://img.shields.io/badge/license-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Ansible paybooks to deploy and configure server and the installation and configuration
of docker and docker-compose over ubuntu server and install Orion, IoT Agent - UL2.0 with
MongoDB as database in a FIWARE Lab [FIWARE Lab](https://cloud.lab.fiware.org) cloud.

The OpenStack dynamic inventory code presented here is taken from the repository 
https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/openstack.py

## How to start it

- Create virtualenv and activate it:
```
virtualenv -p python2.7 $NAME_VIRTUAL_ENV
source $NAME_VIRTUAL_ENV/bin/activate
```
- Install the pre-requisites:
```
pip install -r requirements.txt
```
- Put the correct path to your '/env/bin/python' into the localhost_inventory file.
- Download you OpenStack RC file from the OpenStack dashboard (it's available under 
"info" option on the top left of the FIWARE Lab Cloud Portal). Please, do not forget 
to fill in with your password.
```
export OS_REGION_NAME=xxxxxx
export OS_USERNAME=xxxxx
export OS_PASSWORD=xxxxxx
export OS_AUTH_URL=http://130.206.84.8:4730/v3/
export OS_TENANT_NAME=xxxxxxx
```
- You have to add the following to your .openrc file
```
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_IDENTITY_API_VERSION=3
```
- [OPTIONAL] I suggest to add the following line to it.
```
export PS1='(`basename \"$VIRTUAL_ENV`)[\u@FIWARE Lab \W(keystone_user)]\$ '
```
- Source your OpenStack RC file: `source <path to rc file>`. This will load information 
about you OpenStack Setup into your environment.
- Create the security group for spark. Since spark will start some services on random 
ports this will allow all tcp traffic within the security group:
```
openstack security group create <YOUR SEC. GROUP NAME> --description "internal security group for IoT Hackathon"
openstack security group rule create iotweek --protocol tcp --dst-port 22
openstack security group rule create iotweek --protocol tcp --dst-port 1026
openstack security group rule create iotweek --protocol tcp --dst-port 4041
openstack security group rule create iotweek --protocol tcp --dst-port 7896
openstack security group rule create iotweek --protocol tcp --dst-port 27017
```
- Create a keypair to be used in your instances:
```
openstack keypair create <NAME OF THE KEY PAIR> > <YOUR SSH KEY.pem>
```
- Change permissions to the ssh key file:
```
chmod 400 <YOUR SSH KEY.pem>
```
- Edit the setup variables to fit your setup. Open `vars/main.yml` and setup the 
variables as explained there.
- One all the variables are in place you should now be able to create your instances:
```
ansible-playbook -i localhost_inventory --private-key=<YOUR SSH KEY> create_instance_playbook.yml
```
- Then install tempest+ralli on the node (I have noticed that sometimes it takes a while 
for the ssh-server on the nodes to start, therefore if you get an initial ssh-error, 
wait a few minutes and try again).
```
ansible-playbook -i openstack_inventory.py --user=ubuntu --private-key=<YOUR SSH KEY.pem>  deploy_iotplatform_playbook.yml
```
- Once this has finished successfully your IoT Platform instance server is ready to use. You can access
your instance through the execution of the command:
```
ssh -i <YOUR SSH KEY.pem>` ubuntu@<IP of your instance>
```
And now just activate the docker-compose executing:
```
sudo docker-compose up
```
In the directory /home/ubuntu in which is located the file docker-compose.yml.

Now just enjoy FIWARE.

```

## License

These scripts are licensed under Apache License 2.0.
