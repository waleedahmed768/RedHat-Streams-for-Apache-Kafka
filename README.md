# RedHat-Streams-for-Apache-Kafka
**1. Getting Started Overview**

This guide shows how to install and start using Streams for Apache Kafka on OpenShift: create a Kafka cluster, connect applications, send/receive messages. 

The recommended path uses the Operator from the OperatorHub in OpenShift, which simplifies installation and supports automatic updates. 

**Prerequisites:**

A Red Hat account. 


JDK 11 or later. 


An OpenShift cluster (version 4.14 to 4.19) and the oc CLI installed/configured. 


Additional resources: links to the full product overview and deploying/managing guide. 


**2. Installing the Streams for Apache Kafka Operator from OperatorHub**

You create a new project (namespace) in OpenShift (example uses streams-kafka). 


In the OperatorHub (in the OpenShift console) you search for “Streams for Apache Kafka”, select it, and install it. 


During installation you pick: update channel (stable, or version‐specific), installation mode (specific namespace or all namespaces), and update approval (automatic vs manual). 
Red Hat Docs

After installation succeeds, you verify under Installed Operators in the console. 
Red Hat Docs

**3. Deploying Kafka Components using the Operator**

Once the Operator is installed, you can deploy Kafka components (via the custom resources it provides). 
Red Hat Docs
+1

Example components you can create: Kafka cluster, Kafka NodePool, Kafka Connect, MirrorMaker2, Topic, User, Bridge, Connector, Rebalance. 
Red Hat Docs

The guide shows how to create a Kafka cluster named my-cluster (via YAML) with specific settings. For example: replication factors, listeners (plain and TLS), version (e.g., 4.0.0). 
Red Hat Docs

Then create node pools: one for brokers, one for controllers. Example spec shows kind: KafkaNodePool with roles broker and controller. 
Red Hat Docs

Note: the example uses ephemeral storage for evaluation only; in production, persistent volumes should be used. 
Red Hat Docs

**4. Creating an OpenShift Route to Access a Kafka Cluster**

To allow external clients (outside the OpenShift cluster) to connect, you can expose the Kafka bootstrap via an OpenShift Route. 
Red Hat Docs

Procedure: edit the Kafka custom resource, under listeners add a listener of type route, e.g.:

- name: listener1
  port: 9094
  type: route
  tls: true


Red Hat Docs

After saving, you get the route’s hostname (via console or oc get routes …) and extract the cluster CA certificate (via oc extract secret/…). 
Red Hat Docs

You then create a local truststore with keytool referencing the CA certificate so that a client can authenticate with TLS. 
Red Hat Docs

Important warning: The Route hostname can easily exceed Kubernetes/OpenShift naming limits (max ~63 characters) so the cluster name + listener name + namespace must fit. 
Red Hat Docs
**
**5. Sending and Receiving Messages from a Topic****

This section walks through both in-cluster and external (local client) producers & consumers. 
Red Hat Docs
+1

In‐cluster deployment: Use oc run pods with the Kafka image (example: registry.redhat.io/amq-streams/kafka-40-rhel9:3.0.1) to run kafka-console-producer.sh and kafka-console-consumer.sh. For example:

oc run kafka-producer -ti \
  --image=registry.redhat.io/amq-streams/kafka-40-rhel9:3.0.1 \
  --rm=true --restart=Never \
  -- bin/kafka-console-producer.sh \
  --bootstrap-server my-cluster-kafka-bootstrap:9092 \
  --topic my-topic


Red Hat Docs

And similarly for the consumer using --from-beginning. 
Red Hat Docs

From a local machine: Use the downloaded Kafka binaries, specify the bootstrap server via the route (hostname:443), plus TLS truststore properties:

kafka-console-producer.sh \
  --bootstrap-server <route-hostname>:443 \
  --producer-property security.protocol=SSL \
  --producer-property ssl.truststore.password=password \
  --producer-property ssl.truststore.location=client.truststore.jks \
  --topic my-topic


Red Hat Docs

**Then start the consumer similarly. **
Red Hat Docs

After sending some messages via the producer, you switch to the consumer logs and verify you see the messages. 
Red Hat Docs

**6. Deploying the Streams for Apache Kafka Console**

After your Kafka cluster is up and running, you can install the Kafka Console UI (provided by Streams for Apache Kafka) to manage and monitor the cluster. 
Red Hat Docs

The guide references a separate “console guide” for more details on how to connect and use the UI. 
Red Hat Docs

Appendix A. Using your Subscription

The product is distributed via a subscription. The guide outlines how to manage your subscription in the Red Hat, Inc. Customer Portal. 
Red Hat Docs

Steps include logging into access.redhat.com, activating the subscription, downloading software, installing packages via dnf. 
Red Hat Docs

Key Takeaways & Relevance

The guide is very hands-on: step-by-step from operator install → cluster creation → routing → send/receive → console install.

It’s targeted at users who already have an OpenShift platform (which matches you, since you’re working with OpenShift in your cluster).

It emphasises operator-based deployment (which aligns with Kubernetes/Openshift best practices) and supports external access for clients (useful for real application integration).

It also highlights production vs eval differences (e.g., ephemeral storage vs persistent, etc).

One thing to keep in mind: If you already have Kafka experience (e.g., managing brokers, topics, etc) you’ll still need to align with the OpenShift operator model (custom resources, node pools) and route/secret config for external access.



4.10.4.2. Connecting to the broker from external clients 

When you expose an acceptor to external clients (that is, by setting the value of the expose parameter to true), the Operator automatically creates a dedicated service and route for each broker pod in the deployment.

An external client can connect to the broker by specifying the full host name of the route created for the broker pod. You can use a basic curl command to test external access to this full host name. For example:

$ curl https://my-broker-deployment-0-svc-rte-my-openshift-project.my-openshift-domain
Copy to Clipboard
The full host name of the route for the broker pod must resolve to the node that is hosting the OpenShift router. The OpenShift router uses the host name to determine where to send the traffic inside the OpenShift internal network. By default, the OpenShift router listens to port 80 for non-secured (that is, non-SSL) traffic and port 443 for secured (that is, SSL-encrypted) traffic. For an HTTP connection, the router automatically directs traffic to port 443 if you specify a secure connection URL (that is, https), or to port 80 if you specify a non-secure connection URL (that is, http).

If you want external clients to load balance connections across the brokers in the cluster:

Enable load balancing by configuring the haproxy.router.openshift.io/balance roundrobin option on the OpenShift route for each broker pod.
If an external client uses the Core protocol, set the useTopologyForLoadBalancing=false key in the client’s connection URL.

Setting the useTopologyForLoadBalancing=false key prevents a client from using the AMQ Broker Pod DNS names that are in the cluster topology information provided by the broker. The Pod DNS names resolve to internal IP addresses, which an external client cannot access.

If your brokers have durable subscription queues or request/reply queues, be aware of the caveats associated with using these queues when load balancing client connections. For more information, see Section 4.10.4.4, “Caveats to load balancing client connections when you have durable subscription queues or reply/request queues”.

If you don’t want external clients to load balance connections across the brokers in the cluster:

In each client’s connection URL, specify the full host name of the route for each broker pod. The client attempts to connect to the first host name in the connection URL. However, if the first host name is unavailable, the client automatically connects to the next host name in the connection URL, and so on.
If an external client uses the Core protocol, set the useTopologyForLoadBalancing=false key in the client’s connection URL to prevent the client from using the cluster topology information provided by the broker.
For non-HTTP connections:

Clients must explicitly specify the port number (for example, port 443) as part of the connection URL.
For one-way TLS, the client must specify the path to its trust store and the corresponding password, as part of the connection URL.
For two-way TLS, the client must also specify the path to its key store and the corresponding password, as part of the connection URL.
Some example client connection URLs, for supported messaging protocols, are shown below.

External Core client, using one-way TLS


tcp://my-broker-deployment-0-svc-rte-my-openshift-project.my-openshift-domain:443?useTopologyForLoadBalancing=false&sslEnabled=true \
&trustStorePath=~/client.ts&trustStorePassword=<password>
Copy to Clipboard

Note
The useTopologyForLoadBalancing key is explicitly set to false in the connection URL because an external Core client cannot use topology information returned by the broker. If this key is set to true or you do not specify a value, it results in a DEBUG log message.

External Core client, using two-way TLS


tcp://my-broker-deployment-0-svc-rte-my-openshift-project.my-openshift-domain:443?useTopologyForLoadBalancing=false&sslEnabled=true \
&keyStorePath=~/client.ks&keyStorePassword=<password> \
&trustStorePath=~/client.ts&trustStorePassword=<password>
Show more


Toggle word wrap
External OpenWire client, using one-way TLS


ssl://my-broker-deployment-0-svc-rte-my-openshift-project.my-openshift-domain:443"

# Also, specify the following JVM flags
-Djavax.net.ssl.trustStore=~/client.ts -Djavax.net.ssl.trustStorePassword=<password>
Show more

External OpenWire client, using two-way TLS


ssl://my-broker-deployment-0-svc-rte-my-openshift-project.my-openshift-domain:443"

# Also, specify the following JVM flags
-Djavax.net.ssl.keyStore=~/client.ks -Djavax.net.ssl.keyStorePassword=<password> \
-Djavax.net.ssl.trustStore=~/client.ts -Djavax.net.ssl.trustStorePassword=<password>
Show more
Copy to Clipboard
External AMQP client, using one-way TLS


amqps://my-broker-deployment-0-svc-rte-my-openshift-project.my-openshift-domain:443?transport.verifyHost=true \
&transport.trustStoreLocation=~/client.ts&transport.trustStorePassword=<password>


External AMQP client, using two-way TLS


amqps://my-broker-deployment-0-svc-rte-my-openshift-project.my-openshift-domain:443?transport.verifyHost=true \
&transport.keyStoreLocation=~/client.ks&transport.keyStorePassword=<password> \
&transport.trustStoreLocation=~/client.ts&transport.trustStorePassword=<password>
Show more


4.10.4.3. Connecting to the Broker using a NodePort 

As an alternative to using a route, an OpenShift administrator can configure a NodePort to connect to a broker pod from a client outside OpenShift. The NodePort should map to one of the protocol-specific ports specified by the acceptors configured for the broker.

By default, NodePorts are in the range 30000 to 32767, which means that a NodePort typically does not match the intended port on the broker Pod.

To connect from a client outside OpenShift to the broker via a NodePort, you specify a URL in the format <protocol>://<ocp_node_ip>:<node_port_number>.
