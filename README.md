https://aws.amazon.com/blogs/big-data/stream-twitter-data-into-amazon-redshift-using-amazon-msk-and-aws-glue-streaming-etl/
JSON Code Ref:
https://cloudformation-template-repo.s3.amazonaws.com/Kafka_Streaming_ETL_Redshift.json

```
aws cloudformation create-stack \
  --stack-name "$stack_name" \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=KeyName,ParameterValue=default-key ParameterKey=SSHLocation,ParameterValue=24.84.133.30/32


Commands:

# download jq to /usr/bin
sudo curl -qL -o /usr/bin/jq https://stedolan.github.io/jq/download/linux64/jq && chmod +x /usr/bin/jq

# this generate ~/.aws/config
aws configure set region us-west-1 --profile default
# or export AWS_DEFAULT_REGION=us-east-1

cluster_arn=$(aws kafka list-clusters | jq -r '.ClusterInfoList[0].ClusterArn')

# aws kafka get-bootstrap-brokers --cluster-arn $cluster_arn

zookeeper_string=$(aws kafka list-clusters | jq -r '.ClusterInfoList[0].ZookeeperConnectString')
broker_string=$(aws kafka get-bootstrap-brokers --cluster-arn $cluster_arn |\
 jq -r '.BootstrapBrokerString')
broker_tls_string=$(aws kafka get-bootstrap-brokers --cluster-arn $cluster_arn |\
 jq -r '.BootstrapBrokerStringTls')


# Create a topic, Note: the repliation-factor has to match the number of workers in MSK

kafka_topic=CovidTweets
/opt/kafka/bin/kafka-topics.sh --create --zookeeper $zookeeper_string \
  --replication-factor 2 --partitions 1 --topic $kafka_topic

# send a json using console-producer
/opt/kafka/bin/kafka-console-producer.sh --broker-list $broker_string --topic $kafka_topic < test-input.json


# list topics
/opt/kafka/bin/kafka-topics.sh --list --zookeeper $zookeeper_string

# check messages
/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server $broker_string --topic $kafka_topic --from-beginning
```