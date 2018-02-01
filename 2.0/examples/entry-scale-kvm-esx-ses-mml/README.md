
(c) Copyright 2016 Hewlett Packard Enterprise Development LP
(c) Copyright 2018 SUSE LLC

Licensed under the Apache License, Version 2.0 (the "License"); you may
not use this file except in compliance with the License. You may obtain
a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations
under the License.


## Ardana Single region Entry Scale Cloud with KVM & ESX Hypervisor and External SES integration Example ##

The input files in this example deploy a cloud that has the following characteristics:


### Control Planes ###

- Core cluster: Runs Core OpenStack Services, (e.g. keystone, nova api, glance api, neutron api, horizon, heat api). Default configuration is two nodes of role type CONTROLLER-ROLE

- Metering & Monitoring cluster: Runs the OpenStack Services for metering & monitoring (e.g. celiometer, monasca & logging). Default configuration is three nodes of role type MTRMON-ROLE

- Database & Message Queue Cluster: Runs clustered MySQL and RabbitMQ services to support the Ardana cloud infrastructure. Default configuration is three nodes of role type DBMQ-ROLE. Three nodes are required for high availability.

### Resource Pools ###

- Compute:
    - KVM: Runs nova computes and associated services. Runs on nodes of role type COMPUTE-ROLE. The example lists 1 node.
    - ESX: (below listed resources will be provisioned for all activated clusters)
        - One instance of Nova compute Proxy per cluster
        - One instance of OVSvApp per node

*User shall add required information related to compute proxy and OVSvApp Nodes*

*Additional resource nodes can be added to the configuration.*

*Minimal Swift Resources are provided by the control plane*

### Deployer Node ###


This configuration runs the lifecycle-manager (formerly referred to as the deployer) on a control plane node.
You need to include this node address in your servers.yml definition. This function does not need a dedicated network.

*The minimum server count for this example is therefore 4 servers (Control Plane (x3) + 1 activated vCenter cluster having atleast 1 host)*

An example set of servers are defined in ***data/servers.yml***.   You will need to modify this file to reflect your specific environment.


### Networking ###

The example requires the following networks:

IPMI/iLO network, connected to the lifecycle-manager and the IPMI/iLO ports of all servers

A pair of bonded NICs which are used by the following networks:

- External API - This is the network that users will use to make requests to the cloud
- External VM - This is the network that will be used to provide access to VMs (via floating IP addresses)
- Guest - This is the network that will carry traffic between VMs on private networks within the cloud
- Cloud Management - This is the network that will be used for all internal traffic between the cloud services, This network is also
used to install and configure the nodes. This network needs to be on an untagged VLAN
- SES - This is the network that control plane and compute nodes clients will use to talk to the external SES

Note that the EXTERNAL\_API network must be reachable from the EXTERNAL\_VM network if you want VMs to be able to make API calls to the cloud
and user can choose bonded nic or dedicated nic and can feed VLANs to any network interface based on the nic availability.

TRUNK network is the network that will be used to apply security group rules on tenant traffic. It is managed internally by Ardana cloud and
is restricted to the vCenter environment.

ESX-CONF-NET network (of ESX-CONF network-group) represents a network that is used only to configure the ESX compute nodes in the cloud.  This deployer network should be different from the pxe-based deployer network used by cobbler to standup the cloud controller cluster.

The Data Center Management network (which hosts the vcenter server) must be reachable from the Cloud Management network so that the controllers,
compute proxy and OVSvApp nodes can communicate to the vcenter server.

An example set of networks are defined in ***data/networks.yml***.    You will need to modify this file to reflect your environment.

The example uses the devices hed3 & hed4 as a bonded network for all services.   If you need to modify these
for your environment they are defined in ***data/net_interfaces.yml*** The network devices eth3 & eth4 are renamed to devices hed4 & hed5 using the PCI bus mappings secified in  ***data/nic_mappings.yml***. You may need to modify the PCI bus addresses to match your system.

### Local Storage ###

All servers should present a single OS disk, protected by a RAID controller. This disk needs to be at least 512GB in capacity.   In addition the example configures one additional disk depending on the role of the server:

- Controllers:  /dev/sdb is configured to be used by Swift
- Compute Severs:  /dev/sdb is configured as an additional Volume Group to be used for VM storage

Additional discs can be configured for any of these roles by editing the corresponding ***data/disks_*.yml*** file
