# Elasticsearch Clustering Guide

## Introduction
This guide demonstrates how to configure an Elasticsearch cluster with three nodes, where each node serves as both a Master Eligible Node and a Data Node (default configuration). It also includes instructions to integrate Kibana into the cluster.

---

## Install and Run Elasticsearch on All Nodes

1. **Download and Install Elasticsearch**:
   - Installing Java is not required as Elasticsearch comes with an integrated Java.
   - Use the following commands to download and install Elasticsearch (as root):
     ```bash
     wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.15.3-amd64.deb
     wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.15.3-amd64.deb.sha512
     shasum -a 512 -c elasticsearch-8.15.3-amd64.deb.sha512
     sudo dpkg -i elasticsearch-8.15.3-amd64.deb
     ```

2. **Post Installation**:
   After installation, Elasticsearch will generate security-related information, including:
   - A superuser password.
   - Commands to create enrollment tokens for additional nodes or Kibana.

   Example output:
   ```
   Authentication and authorization are enabled.
   TLS for the transport and HTTP layers is enabled and configured.
   The generated password for the elastic built-in superuser is: ErHQaG33QeEFNSer2odU
   ```

---

## Configure Cluster on Node 1

1. **Edit Configuration File**:
   - Open the configuration file and make the following changes:
     ```bash
     vi /etc/elasticsearch/elasticsearch.yml
     ```
   - Update the following lines:
     ```yaml
     cluster.name: elasticsearch
     network.host: 0.0.0.0
     ```

2. **Start Elasticsearch**:
   ```bash
   systemctl enable --now elasticsearch
   ```

3. **Generate Enrollment Token**:
   - Generate a token to join other nodes to the cluster and save it to a file:
     ```bash
     /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node > token
     ```

---

## Join Other Nodes to the Cluster

1. **Reconfigure Additional Nodes**:
   - Run the following command on additional nodes, replacing `<token>` with the content of the `token` file:
     ```bash
     /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token>
     ```
   - Confirm the reconfiguration process by typing `y` when prompted.

2. **Edit Configuration File**:
   - Update `/etc/elasticsearch/elasticsearch.yml`:
     ```yaml
     cluster.name: elasticsearch
     network.host: 0.0.0.0
     ```

3. **Start Elasticsearch**:
   ```bash
   systemctl enable --now elasticsearch
   ```

---

## Verify Clustering

1. **Check Node Status**:
   ```bash
   curl -u elastic --cacert /etc/elasticsearch/certs/http_ca.crt https://127.0.0.1:9200/_cat/nodes?v
   ```
   Example output:
   ```
   ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
   10.0.0.51  21           87          0   0.00    0.05    0.07    cdfhilmrstw *      node01.srv.world
   10.0.0.52  29           82          0   0.22    0.30    0.14    cdfhilmrstw -      node02.srv.world
   10.0.0.53  26           97          0   0.22    0.13    0.05    cdfhilmrstw -      node03.srv.world
   ```

2. **Check Cluster Health**:
   ```bash
   curl -u elastic --cacert /etc/elasticsearch/certs/http_ca.crt https://node01.srv.world:9200/_cluster/health?pretty
   ```
   Example output:
   ```json
   {
     "cluster_name": "elastic-cluster",
     "status": "green",
     "number_of_nodes": 3,
     "number_of_data_nodes": 3,
     "active_shards_percent_as_number": 100.0
   }
   ```

---

## Install and Join Kibana to Elasticsearch

1. **Download Kibana**:
   ```bash
   wget https://artifacts.elastic.co/downloads/kibana/kibana-8.15.3-amd64.deb
   ```

2. **Generate Kibana Enrollment Token**:
   - On Node 1, generate a Kibana enrollment token:
     ```bash
     /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana > kibana-token
     ```

3. **Configure Kibana**:
   - On the Kibana server, run the following command, replacing `<token>` with the token from `kibana-token`:
     ```bash
     /usr/share/kibana/bin/kibana-setup --enrollment-token <token>
     ```

4. **Edit Kibana Configuration**:
   - Update `/etc/kibana/kibana.yml`:
     ```yaml
     server.host: "0.0.0.0"
     server.publicBaseUrl: "https://fqdn-of-node-one:5601/"
     server.name: "node01-mvmco.ir"
     ```

5. **Start Kibana**:
   ```bash
   systemctl enable --now kibana
   ```

6. **Access Kibana**:
   - Open a browser and navigate to: `https://<hostname-or-IP>:5601/`
   - Log in using the `elastic` user and the generated password.

---

## References
- [Elasticsearch Setup Guide](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=elasticstack8&f=2)
- [Kibana Setup Guide](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=elasticstack8&f=3)
