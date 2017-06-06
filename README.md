# <a name="top"></a>FIWARE IoT Hackathon content
[![License badge](https://img.shields.io/badge/license-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Ansible paybooks to deploy and configure server and the installation and configuration
of docker and docker-compose over ubuntu server and install Orion, IoT Agent - UL2.0 and
Comet with MongoDB as database in a FIWARE Lab [FIWARE Lab](https://cloud.lab.fiware.org) 
cloud.

The OpenStack dynamic inventory code presented here is taken from the repository 
https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/openstack.py

If you want to obtain more information about those FIWARE Generic Enablers, please visit 
the following web sites:
- [Orion Context Broker](https://fiware-orion.readthedocs.io)
- [IoT Agent for UL2.0 (HTTP or MQTT transport)](https://github.com/telefonicaid/iotagent-ul)
- [IoT Agent for JSON (MQTT transport)](https://github.com/telefonicaid/iotagent-uljson)
- [IoT Agent for OMA-LWM2M (CoAP transport)](https://github.com/telefonicaid/lightweightm2m-iotagent)
- [Short Term Historic (Comet)](https://fiware-sth-comet.readthedocs.io/en/latest/)


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
- Create the security group for your virtual machine:
```
openstack security group create <YOUR SEC. GROUP NAME> --description "internal security group for IoT Hackathon"
openstack security group rule create iotweek --protocol tcp --dst-port 22
openstack security group rule create iotweek --protocol tcp --dst-port 1026
openstack security group rule create iotweek --protocol tcp --dst-port 4041
openstack security group rule create iotweek --protocol tcp --dst-port 7896
openstack security group rule create iotweek --protocol tcp --dst-port 8666
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
ansible-playbook -i openstack_inventory.py --user=ubuntu --private-key=<YOUR SSH KEY.pem> \
 deploy_iotplatform_playbook.yml
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


## How to play with it

Firstly, be sure that you have installed [curl](https://curl.haxx.se/) program 
and [jq](https://stedolan.github.io/jq/download/).

The first thing that you have to do is register a new service in the IoT Agent

```  
curl -H "Content-type: application/json" -H "Fiware-Service: openiot" -H "Fiware-ServicePath: /" \ 
http://<IP of your FIWARE Lab Instance>:4041/iot/services -d '{
 "services": [
   {
     "apikey":      "4jggokgpepnvsb2uv4s40d59ov",
     "cbroker":     "http://0.0.0.0:1026",
     "entity_type": "thing",
     "resource":    "/iot/d"
   }
 ]
}'
```

You can check now the new created service through the following command:

```
curl -H "Content-type: application/json" -H "Fiware-Service: openiot" -H "Fiware-ServicePath: /" \ 
http://<IP of your FIWARE Lab Instance>:4041/iot/services | jq .
```

Now, you have to register the sensor (devices), which type and which values should receive from it.

```
curl http://<IP of your FIWARE Lab Instance>:4041/iot/devices \
-H "Content-type: application/json" -H "Fiware-Service: openiot" -H "Fiware-ServicePath: /" \
-d '{
 "devices": [
   {
     "device_id":   "my_device_01",
     "entity_name": "my_entity_01",
     "entity_type": "thing",
     "protocol":    "PDI-IoTA-UltraLight",
     "timezone":    "Europe/Madrid",
     "attributes": [
       {
         "object_id": "t",
         "name":      "temperature",
         "type":      "int"
       },
       {
         "object_id": "l",
         "name":      "luminosity",
         "type":      "number"
       }
     ]
   }
 ]
}'
```

To check the information of the device, just execute the sentence:

```
curl -H "Content-type: application/json" -H "Fiware-Service: openiot" -H "Fiware-ServicePath: /" \ 
http://<IP of your FIWARE Lab Instance>:4041/iot/devices | jq .
```

Now, we can subscribe the Short Term Historical data (FIWARE STH-Commet) to store the different context 
information that we send to the Orion Context Broker. Just execute the following sentence:
```
curl -X POST -H "Content-Type: application/json" -H "Accept: application/json" -H "Fiware-Service: openiot" \ 
-H "Fiware-ServicePath: /" -H "Cache-Control: no-cache" -d '{
    "entities": [
        {
            "type": "thing",
            "isPattern": "false",
            "id": "my_entity_01"
        }
    ],
    "attributes": [
        "temperature"
    ],
    "reference": "http://130.206.115.154:8666/notify",
    "duration": "P1M",
    "notifyConditions": [
        {
            "type": "ONCHANGE",
            "condValues": [
                "temperature"
            ]
        }
    ]
}' "http://<IP of your FIWARE Lab Instance>:1026/v1/subscribeContext"
```
 
Now, you can send context information in the Ultralight 2.0 format through HTTP using the following sentence:

```
curl "http://<IP of your FIWARE Lab Instance>:7896/iot/d?k=4jggokgpepnvsb2uv4s40d59ov&i=my_device_01" \ 
-d 't|37#l|1200' -H "Content-type: text/plain"
```

And you can recover the current context information from Orion Context Broker through the following command:

```
curl http://<IP of your FIWARE Lab Instance>:1026/ngsi10/queryContext -H "Content-type: application/json" \ 
-H "Fiware-Service: openiot" -d '{
   "entities": [
       {
           "type": "", 
           "id": "my_entity_01", 
           "isPattern": "false"
       }
   ], 
   "attributes": []
}'
```

Just send 2 more context information with similar senteces like:
```
curl "http://130.206.115.154:7896/iot/d?k=4jggokgpepnvsb2uv4s40d59ov&i=my_device_01" -d 't|38#l|1200' \ 
-H "Content-type: text/plain"

curl "http://130.206.115.154:7896/iot/d?k=4jggokgpepnvsb2uv4s40d59ov&i=my_device_01" -d 't|39#l|1200' \ 
-H "Content-type: text/plain"
```

Now, you can get some aggregated data from temperature executing the following command:
```
curl -X GET \
  'http://130.206.115.154:8666/STH/v1/contextEntities/type/thing/id/my_entity_01/attributes/temperature?aggrMethod=sum&aggrPeriod=day&dateFrom=2015-01-28T00%3A00%3A00&dateTo=2018-01-01T23%3A59%3A59' \
  -H 'accept: application/json' \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' | jq .
```  


## License

These scripts are licensed under Apache License 2.0.
