# 2netInterfaceAWS
2 Net Interface same network on AWS - Centos/RHEL


This playbook config 2 or more ec2 instance on AWS for manage netowrk traffic with 2 interface on same Net, this is useful for create virtual ip for cluster.

Example:
Postgres cluster with virtual ip for pgpool 

You can define a vip network interface that isn't attach to any ec2 instance, when pgpool select master attach the network interface to the ec2 instance. 

Note: to do this you must install aws cli on the ec2 that can belong at cluster pgpool, and create 2 script one for escalation up and one for de-escalation

Here an example for escalation script and de-escalation

```bash
#!/bin/bash

###Find the ec2 instance ID
INSTANCEID=`/bin/cat /sys/devices/virtual/dmi/id/board_asset_tag`


###Check Instance ID  
#echo $INSTANCEID
###
###This is useful for wait command on AWS that complete from other ec2 instance that have the vip and detach card some time aws is slow to process cli command
echo "Wait 5 second before claim vip interface"
sleep 5
echo "Attach VIP Interface"
###You must define the network interface on AWS console or cli and keep the netowrk interface id eni-xxxxxx
/usr/local/bin/aws ec2 attach-network-interface --instance-id $INSTANCEID --network-interface-id eni-11111111111111111 --device-index 1


echo "Wait 3 second before Config the interface"
###This is useful to wait kernel 
sleep 3
/sbin/ifup eth0
```