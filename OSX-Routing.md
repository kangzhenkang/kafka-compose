#  Set up Routing (For Mac users only)
## The Issue
The minimal os, boot2docker, is used by osx to run docker containers. It runs as a VM on your box and then runs the docker engine/host.  
The exposed container ports can still be reached by going to the VM's ip (run `boot2docker ip`), but the issue is that Kafka nodes use a hostname/ip address to advertised themselves, this advertised address is what clients use to send and consume messages.  
Containers within boot2docker end up with ip addresses that aren't reachable from the host/local machine, this stops clients outside of the boot2docker vm from sending/consuming to and from the actual Kafka Nodes.
## The Solution (not perfect but good enough) 
For mac users to use the kafka node hostname/ip addesses, we'll need to route the subnet ip range that the docker containers are using to the boot2docker vm ip (remember we are exposing the container ports on this ip already).  
You'll need to carry out the following steps  
1) Find out the boot2docker vm's ip by running ``boot2docker ip``   
2) Find out the subnet range being used by the containers, by running ``docker ps | cut -d' ' -f1 | tail -n +2 | xargs -I {} docker inspect --format '{{.Config.Image }} {{ .NetworkSettings.IPAddress }}' {}``  
You should see a bunch of image names and ips like
> samsara/kafka:latest 172.17.0.8  
> samsara/kafka:latest 172.17.0.7  
> samsara/kafka:latest 172.17.0.6  
> samsara/zookeeper:latest 172.17.0.5  

3) Check for any existing routing rules by running ``netstat -rn | grep 172.17``   
4) If a rule exists, just make sure you can reach the container ips and any ports.   
5) If a rule doesn't exist, create one by running ``sudo route -n add 172.17.0.0/24 $(boot2docker ip)``. Then check you can reach the container ips and ports  

To remove a routing rule run ``sudo route -n delete 172.17.0.0/24 $(boot2docker ip)``   
The above routing rule is temporary, so it will need to be re-created if the system is restarted.  

