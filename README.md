# Setup and Listen messages from Kafka Broker 

#EXECUTE THE KAFKA APP 

This will initialize the kafka listener for the kafka-topic. 

1. Build and export the code to a runnable kaflis.jar
2. Copy the kaflis.properties to the same folder as kafkapp.jar
3. Configure the properties as suited by the environment 
4. Execute the listener jar file:  
	   java -cp . -jar kafkapp.jar -resources ./  
	   nohup java -cp . -jar kafkapp.jar -resources ./ &>/dev/null &  
5. Keep track of the kafkapp.log and kafkapp_Error.log dump at ./logs/



#SETUP KAFKA BROKER root@192.168.56.101

Reference: http://kafka.apache.org/090/documentation.html
Download: https://www.apache.org/dyn/closer.cgi?path=/kafka/0.9.0.0/kafka_2.11-0.9.0.0.tgz

1. Extract this in /opt folder
2. Configure $KAFKA_HOME/config/server.properties  
	    advertised.host.name=192.168.56.101  


#KAFKA RESTART SCRIPT root@192.168.56.101
```
# Set Kafka Environment  
export KAFKA_HOME="/opt/kafka_2.11-0.9.0.0/"

echo "Kafka Home: ",$KAFKA_HOME
echo "1. Stop the various processes(VK Scheduler, Kafka, Zookeeper)"
/opt/vertica/packages/kafka/bin/vkconfig shutdown
$KAFKA_HOME/bin/kafka-server-stop.sh
$KAFKA_HOME/bin/zookeeper-server-stop.sh

#
# nohup command for background process
# nohup sh script.sh &>/dev/null &
#
echo "\n2. Start the various processes(Zookeeper,Kafka, Vertica-Kafka Scheduler)"
echo "2.a Starting Zookeeper"
nohup sh $KAFKA_HOME/bin/zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties &>/dev/null &
sleep 5
echo "2.b Starting Kafka"
nohup sh $KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties &>/dev/null &
sleep 5
echo "2.c Starting Vertica Kafka Scheduler"
nohup sh /opt/vertica/packages/kafka/bin/vkconfig launch --username dbadmin --password test &>/dev/null &

echo "\nStartup completed"
echo "Kafka Logs: tail -f /opt/kafka_2.11-0.9.0.0/logs/server.log"
echo "Kafka-vertica log: tail -f /opt/vertica/log/vkafka-sched.log"
```


#Vertica Kafka integration verification

A topic named "kaflis-topic" exists with a single partition and only one replica  

   export KAFKA_HOME="/opt/kafka_2.11-0.9.0.0/"  

1. Create & See list of topics created in Kafka  
   $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper 192.168.56.101:2181 --replication-factor 1 --partitions 1 --topic kaflis-topic  
   $KAFKA_HOME/bin/kafka-topics.sh --list --zookeeper 192.168.56.101:2181  

2. Send some messages  
   $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list 192.168.56.101:9092 --topic kaflis-topic  

3. Start a consumer to list messages subscribed to topic kaflis-topic  
   $KAFKA_HOME/bin/kafka-console-consumer.sh --zookeeper 192.168.56.101:2181 --topic kaflis-topic --from-beginning  

4. Vertica Kafka configuration  
   Created and Configured default Vertica Scheduler. The messages and topic in Kafka are pushed to Vertica public table kafka_tgt.   
   Associate listener to the Topic kaflis-topic  
   /opt/vertica/packages/kafka/bin/vkconfig topic --add --target public.kafka_tgt --rejection-table public.kafka_rej --topic kaflis-topic --username dbadmin --password test  

5. Check the Vertica public table kafka_tgt for any messages produced by using Step 2  
















