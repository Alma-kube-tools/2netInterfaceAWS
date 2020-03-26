# 2netInterfaceAWS
2 Net Interface same network on AWS - Centos/RHEL


This playbook config 2 or more ec2 instance on AWS for manage netowrk traffic with 2 interface on same Net, this is useful for create virtual ip for cluster.

Before you can use this playbook config the group_vars all with this mandatory variable:

This an example you must change and with your value for vipip you can find the ip of the interface that you have defined on aws console under interfaces and you want to use for VIP.
```
Net is the network in CIDR format 
Gateway is default gateway for the network
table1name is the routing table name for the 1 network interface of ec2 instance 
table2name is the routing table name for VIP interface
defaultdevice is the first network interface that will be created on first deploy
vipdevice is the name for the vip interface that you attach and detach at runtime
vipip is the ip for the interface that you use for vip
defaultnetmask is the netmask of the network is useful for build the config file 
```
```yaml
net: xxx.xxx.xxx.xxx/24
gateway: gateway_ip
table1name: t1
table2name: t2
defaultdevice: ens5 
vipdevice: eth0
vipip: xxx.xxx.xxx.xxx
defaultnetmask: 255.255.255.0
```
Playbook launch command
ansible-playbook 2netnode.yml -i ./hostinv  --private-key=/path/to/your/private/key 

Example:
Postgres cluster with virtual ip for pgpool 

You can define a vip network interface that isn't attached to any ec2 instance, when pgpool select master attach the network interface to the ec2 instance. 

###### Note: to do this you must install aws cli on the ec2 that can belong at cluster pgpool, and create 2 script one for escalation up and one for de-escalation

Here an example for escalation script and de-escalation

Escalation up 
```bash
#!/bin/bash

###Find the ec2 instance ID
INSTANCEID=`/bin/cat /sys/devices/virtual/dmi/id/board_asset_tag`


###Check Instance ID  
#echo $INSTANCEID
###
###This is useful for wait command on AWS that complete from other ec2 instance that have the vip
### and detach card some time aws is slow to process cli command
echo "Wait 5 second before claim vip interface"
sleep 5
echo "Attach VIP Interface"
###You must define the network interface on AWS console or cli and keep the netowrk 
### interface id eni-xxxxxx
/usr/local/bin/aws ec2 attach-network-interface --instance-id $INSTANCEID --network-interface-id eni-11111111111111111 --device-index 1


echo "Wait 5 second before Config the interface"
###This is useful to wait kernel 
sleep 5
/sbin/ifup eth0
```
De escalation
```bash
#!/bin/bash
###Retrive Interface attach ID from ec2 instance with aws cli
INTERFACEATTACHID=`/usr/local/bin/aws ec2 describe-network-interfaces --network-interface-ids eni-08770b5291f7eee93 |grep AttachmentId |awk '{print $2}'|/bin/rev|/bin/cut -c 2-|/bin/rev|cut -d '"' -f 2`

###check for view Interface ID
#echo $INTERFACEATTACHID


###bring down the network Interface
echo "Shutdown VIP ethernet Interface"
/sbin/ifdown eth0 


echo "Detach Network Interface from AWS"
`/usr/local/bin/aws ec2 detach-network-interface --attachment-id $INTERFACEATTACHID`
```
