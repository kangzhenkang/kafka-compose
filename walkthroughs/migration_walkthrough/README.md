# Kafka Migration Walkthrough
This walkthrough deals with the scenario where you have an existing kafka cluster (e.g 1 zookeeper node 3 kafka nodes)  
And you are looking to add 3 new kafka nodes and decommission the 3 old kafka nodes.   

# Main Goal
This walkthrough simulates how you would replace brokers with new ones using docker-compose, it is not *100% realistic* (as you are running containers locally) but it should give you a good grounding on what's involved. The aim is to get you comfortable with the process.   


##  Pre-requisite
- The reader should have followed the instructions outlayed in the SETUP section of [main readme](../../README.md)   
  Please make sure you've carried out all the steps detailed in the SETUP section.   
- You should have 2 terminals open. One in this project root for running the docker-compose commands, the other for running kafka commands  


## The Walkthrough

### 1) Bring up a 3 kafka node cluster with some topics
- Go to the project's main directory and run  ``docker-compose -f configs/kafka-migration.yml  up -d``  
- Confirm your cluster is up by running  ``docker-compose -f configs/kafka-migration.yml ps``   
- Create a topic, by going to your kafka/confluent-kafka bin directory and running   
  ``./kafka-topics --zookeeper $(boot2docker ip):2181 --create --topic airbenders --partitions 3 --replication-factor 3``  
  ``./kafka-topics --zookeeper $(boot2docker ip):2181 --create --topic waterbenders --partitions 3 --replication-factor 3``  
- Confirm the topics have been created by running  ``./kafka-topics --zookeeper $(boot2docker ip):2181 --describe``  


### 2) Bring up 3 new additional Kafka nodes 
 - Edit the configs/kafka-migration.yml  and uncomment brokers 4 to 6  
 - Bring up the 3 new nodes by running  ``docker-compose -f configs/kafka-migration.yml  up -d --no-recreate  kafka4 kafka5 kafka6 ``    
 - Confirm your 6 node kafka cluster is up by running  ``docker-compose -f configs/kafka-migration.yml  ps``

### NOTE existing partitions aren't moved to new brokers
- The new brokers are now part of the cluster BUT no partitions will reside on them. So essentially they are doing nothing.   
  You can check this by running  ``./kafka-topics --zookeeper $(boot2docker ip):2181 --describe``  
- Create a new topic by running  ``./kafka-topics --zookeeper $(boot2docker ip):2181 --create --topic earthbenders --partitions 3 --replication-factor 3``   
- Some of the partitions in the new topic "earthbenders" will now end up on the new brokers (4, 5 or 6).  
  But the partitions for airbender and waterbender topics will stil be on the old brokers (1, 2 and 3)  
  Check by running  ``./kafka-topics --zookeeper $(boot2docker ip):2181 --describe``   
- At this stage you'll have quite an unbalanced cluster with brokers 1, 2 and 3 doing most of the work  

### 3) Move all topics onto the new brokers
- To make sure all topics are now on the new brokers (4 5 6), you need to use the re-assign tool
- Create a json file (topics-to-move.json) listing all the topics to be reassigned, something similar to the following  
```
{
    "topics":
        [{"topic": "airbenders"},
         {"topic": "waterbenders"},
         {"topic": "earthbenders"}],
        "version": 1
}
```
- Run re-assign tool to generate all the partitions that need to be moved, do this by running   
  ``./kafka-reassign-partitions --zookeeper $(boot2docker ip):2181 --topics-to-move-json-file topics-to-move.json --broker-list "4,5,6" --generate``  <- Note that we supply only the new brokers to --broker-list   
- The above command will generate the 2 blocks of json, the first is the current partition assignments for the topics, whilst the second block is the partition assignment you want to achieve for the topics.   
- Save the first block of json into a file called revert-partitions.json  and  save the second block of json into a file called reassign-partitions.json   
- Now run the re-assign tool to move all the partitions to brokers 4 5 6, by running   
``./kafka-reassign-partitions --zookeeper $(boot2docker ip):2181 --reassignment-json-file  reassign-partitions.json --execute``   
- This above command will again output 2 blocks of json. The first being the current partition assigments and the second being the assignment you are trying to achieved. If you already haven't saved these blocks into different files, please do so now  
- Verify the progress of the partition movement, as you can imagine depending on the amount of topics/partition/data this can take a while  
  To verify the progress run the following command   
  ``./kafka-reassign-partitions --zookeeper $(boot2docker ip):2181 --reassignment-json-file  ~/Documents/reassign-partitions.json --verify``   
- **NOTE** Again you'll have to wait for all this to be completely done. And it might take a while.  
  Producers and Consumers sending data will probably lose connections and have to reconnect as the various partition leaders that were being communicated to are moved from the old brokers to the new ones. Producers/Consumers should expect this anyways when talking to a kafka cluster and should be able to handle connection loss in a graceful way    


### 4) Decommision/shutdown the old brokers
- At this stage all the partitions should now be on brokers 4, 5 and 6. Confirm this by running  
``./kafka-topics --zookeeper $(boot2docker ip):2181 --describe``   
- You should NOT see any partitions on brokers 1 2 or 3   
- Shutdown brokers 1 2 and 3 by running this command    
  ``docker-compose -f configs/kafka-migration.yml  stop kafka1 kafka2 kafka3``   
- Congratulations!!! You have now decommissioned brokers 1 2 3, replaced them with brokers 4 5 6 and moved all the data across!!
- Once you are happy, to destroy everything run
``docker-compose -f configs/kafka-migration.yml  stop && docker-compose -f configs/kafka-migration.yml  rm``  
and  
``boot2docker restart``  

## THINGS TO NOTE
1) Whilst this migration is being done, it's essential that new topics aren't being created.  
2) You really have to wait for all the data/partitions to have been moved from brokers 1 2 3 before shutting them down, this can take a while.  

