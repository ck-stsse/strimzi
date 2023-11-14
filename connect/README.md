# <a name="_1wcd6ofrjeuj"></a>Strimzi Connect Cluster to Confluent Cloud (Docker)

This document walks you through the prerequisites and steps to setup a Kafka Connector using Strimzi, to connect to a Confluent Cloud cluster.
## <a name="_5rja607z0n5"></a>Prerequisites
1. Docker image with all the connector plugins required for the setup.([Create a Docker image from the base Kafka Connect image](https://strimzi.io/docs/operators/in-development/deploying#creating-new-image-from-base-str) )
1. TLS certificates from Confluent Cloud.
1. Confluent Cloud API keys with necessary access to the topics.
1. [Sample configuration](https://github.com/ck-stsse/strimzi/tree/main/connect)
## <a name="_qwf7k8s4n901"></a>Strimzi Connect Cluster Setup
1. ### <a name="_wtyoza2j2me"></a>Deploying Kafka Connect
   1. Refer [sample configuration](https://github.com/ck-stsse/strimzi/blob/main/connect/connect.yaml) for Strimzi Connect cluster.
   1. Create the docker image with all the required kafka connect plugins. This will be used to create the Connect Cluster.
   1. Create a namespace for your Strimzi cluster.
      kubectl create namespace strimzi
   1. Install Strimzi operator in the created namespace.
      kubectl create -f 'https://strimzi.io/install/latest?namespace=strimzi' -n strimzi
   1. Now, in order to connect to Confluent Cloud we would need two secrets, one for [authentication](https://docs.confluent.io/cloud/current/access-management/authenticate/api-keys/api-keys.html#use-api-keys-to-control-access-in-ccloud) and another for [data encryption](https://docs.confluent.io/cloud/current/cp-component/clients-cloud-config.html#connect-a-net-application-to-ccloud).
   1. Refer to [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/) to understand more.
   1. Create my-cluster-cluster-ca-cert for TLS encryption using ca.crt, obtained using [Confluent Cloud Certificates](https://letsencrypt.org/certs/).
      kubectl create secret generic my-cluster-cluster-ca-cert --from-file=ca.crt=./ca.crt -n strimzi

1. Create my-connect-secret-name for authentication using the Confluent Cloud API Secret.
   kubectl create secret generic my-connect-secret-name --from-file=my-password-field-name=./MY-PASSWORD.txt -n strimzi
1. Update the bootstrap server with the Confluent Cloud bootstrap server in the connect configuration.
1. Make sure that the replication factor for the configs in connnect.yaml is 3.
1. Deploy connect cluster in the created namespace
   kubectl -n strimzi apply -f connect.yaml
1. ### <a name="_akliss3i1igd"></a>Deploying Kafka Connector
   1. Refer the [sample configuration](https://github.com/ck-stsse/strimzi/blob/main/connect/connector.yaml) for MYSQL Source Connector which references the Connect Cluster created in the last step.
   1. Make sure the connector class is one of the plugins mentioned in the docker image used in connect.yaml config.
   1. Update other database configs for the connector accordingly.
   1. Deploy connector.yaml to the current namespace.
      kubectl -n strimzi apply -f connect.yaml
## <a name="_sv1s5rcr2omw"></a>References
1. Confluent cloud certificates - <https://letsencrypt.org/certificates/>

<https://docs.confluent.io/cloud/current/cp-component/clients-cloud-config.html#connect-a-net-application-to-ccloud>

2. MySQL Connector Blog- <https://strimzi.io/blog/2020/01/27/deploying-debezium-with-kafkaconnector-resource/>
3. Authentication - <https://strimzi.io/docs/operators/in-development/configuring#type-KafkaClientAuthenticationPlain-reference>

