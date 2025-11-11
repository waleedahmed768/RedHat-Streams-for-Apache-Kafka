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
