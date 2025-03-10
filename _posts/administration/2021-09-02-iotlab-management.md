---
layout: post
title: Using IoT-Lab with the PiSeduce Resource Manager
subtitle: Sensor Reservation and Management
category: Administration
index: 13
---
The PiSeduce Resource Manager is a reservation tool designed for Raspberry Pis.
It manages the user reservations and deploys the environments on the Raspberrys.
However, the resource manager can be connected to the [FIT IoT-Lab
plateform](https://www.iot-lab.info/){:target="_blank"} in order to reserve
different types of sensors.

To connect the resource manager to the IoT-Lab infrastructure, we deploy a
piseduce_agent with a specific configuration. Then, we connect the
piseduce_webui to this agent. As the piseduce_webui interface can manage
multiple agents, the webui can manage Raspberrys, IoT-Lab sensors and g5k
servers at the same time.

To connect the agent to the IoT-Lab infrastructure, we start by getting the
piseduce_agent from the github repository:
```
git clone http://github.com/remyimt/piseduce_agent
```

Then, we modify the configuration file `config_agent.json`. We change the
**auth_token** property to another string (at least 10 characters). This token
is used to secure the connection between the webui and the agent. We will need
it later. Then, we set the **node_type** property to `iot-lab`. The IoT-Lab
testbed consists of multiple sites. One agent can manage only the site defined
by the **iot_site** property. So, if we want to use many IoT-Lab sites, we have
to deploy multiple agents. To deploy multiple agents to the same server, we have
to use a different port (**port_number** property) and a different database file
(**db_url** property).

Here, we configure one agent to manage the *strasbourg* site of the IoT-Lab
testbed. The content of our configuration file (**do not use the auth_token
defined below**) is:
```
{
    "auth_token": [ "iotlabsuperlongtoken1234" ],
    "port_number": 8090,
    "comments": "Type of the nodes managed by this agent (raspberry, g5k or iot-lab)",
    "node_type": "iot-lab",
    "g5k_site": "nancy",
    "iot_site": "strasbourg",
    "key_file": "secret.key",
    "db_url": "sqlite:///test-agent.db",
    "comments": "Do not forget the ending '/' at the end of the env_path",
    "env_path": "/root/environments/"
}
```

The **key_file** property defines the encryption key used to decrypt the
password of the IoT-Lab account. This key file is generated by the webui that
uses it to encrypt passwords. Indeed, users of the resource manager must provide
their IoT-Lab account information (user and password) to reserve sensors on the
IoT-Lab infrastructure. This password is encrypted with the key file before
being stored in the database. So, we have to copy this key from the webui to the
piseduce_agent directory.

Before starting the agent API, we have to create the tables of the SQLite
database:
```
python3 init_database.py config_agent.json
```
Then, we start the agent API with the following command:
```
python3 agent_api config_agent.json
```

On the resource manager interface (with the administration privilege), we open
the agent panel by clicking on the *(admin) Agent* item of the left menu. We
fill in the *Add Agent* form with an appropriate name for the agent (e.g.,
*iot_strasbourg*), the IP, the port and the token previously defined. We click
on the *Add* button and the agent should appear in the *Existing Agents* section
with the *connected* state.

The sensors managed by the agent should appear in the reserve panel. To reserve
these nodes, we must register our IoT-Lab username and our IoT-Lab password. We
open the settings panel by clicking on the *Settings* item of the menu. On the
*IoT-Lab Credentials* tab, we fill in the form to register the IoT-Lab account
information. Now, we can reserve sensors from the reserve panel.

After making the reservation, from the configure panel, we can select a firmware
and/or a monitoring profile. To add new firmwares or new monitoring profiles, we
must connect to the [IoT-Lab
platform](https://www.iot-lab.info/){:target="_blank"}. Finally, we click on the
*Deploy* button to start the experiments.

The agent_api.py script can be run as SystemD services. To
configure the service, we can run the following commnands (as root):
```
cp admin/*.service /etc/systemd/system/
systemctl enable agent_api.service
```
Then, we can start the agent API as described below:
```
service agent_api start
```

The configuration of the IoT-Lab agent is completed. PiSeduce users can now reserve
and manage IoT-Lab sensors from the resource manager.
