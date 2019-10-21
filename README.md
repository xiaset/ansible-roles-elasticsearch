Role Name
=========
Installation and configuration of the Elasticsearch v7.x in cluster mode and single mode + xpack security & xpack ssl transport.

Requirements
------------
- JVM must be pre-installed
- Ansible v2.5
- Supported OS:
  - RedHat 7 based
  - Debian 9 / Ubuntu 18.04
- Testing:
  - Molecule 2.22
  - Docker or Vagrant Provider


Role Variables
--------------
Currently only Elasticsearch version 7 can be used in this role:
```yaml
es_version: '7.x'
es_package_state: present
```

#### Service options:
```yaml
es_service_state: started
es_service_enabled: true
```

#### Path options:
```yaml
es_path_data: /var/lib/elasticsearch
es_path_logs: /var/log/elasticsearch
es_path_config: /etc/elasticsearch
```

#### Network options:
The node will bind to this hostname or IP address and publish (advertise) this host to other nodes in the cluster. Accepts an IP address, hostname, a special value, or an array of any combination of these. Note that any values containing a : (e.g., an IPv6 address or containing one of the special values) must be quoted because : is a special character in YAML. 0.0.0.0 is an acceptable IP address and will bind to all network interfaces. The value 0 has the same effect as the value 0.0.0.0.

`es_network_host: "{{ ansible_default_ipv4.address }}"`

The publish host is the single interface that the node advertises to other nodes in the cluster, so that those nodes can connect to it. Change the configuration when cluster nodes communicates through non default network interfaces (differ from NIC defined in the 'es_network_host' variable):

`es_network_publish_host: "{{ ansible_default_ipv4.address }}"`

Port to bind to for incoming HTTP requests:

`es_http_port: 9200`


#### Cluster options
If true, a node will elect itself master and will not join a cluster with any other node. Single node cluster mode:

`es_single_node: false`

A node can only join a cluster when it shares its cluster name with all the other nodes in the cluster:

`es_cluster_name: my-cluster`

In order to join a cluster, a node needs to know the hostname or IP address of at least some of the other nodes in the cluster. This setting provides the initial list of addresses this node will try to contact. Default: list of default IPv4 address of members of 'elasticsearch' group:

`es_discovery_seed_hosts: "{{ groups['elasticsearch'] | map('extract', hostvars, ['ansible_default_ipv4','address']) | list }}"`

Bootstrap the cluster using at least three master-eligible nodes. Default: list of default IPv4 address of members of 'elasticsearch' group:

`es_cluster_initial_master_nodes: "{{ groups['elasticsearch'] | map('extract', hostvars, ['ansible_default_ipv4','address']) | list }}"`


#### Security configuration
`es_security_enable_xpack: "false"`

`es_security_enable_ssl_transport: "false"`

`es_security_path_node_certificate: files/elastic-certificates.p12`

Dependencies
------------

JVM must be pre-installed


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables
passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: elasticsearch, x: 42 }

#### SSL Transport configuration:
1. generate CA certificate:
     bin/elasticsearch-certutil ca --days 3650 --out /tmp/elastic-stack-ca.p12
2. generate node certificates:
     bin/elasticsearch-certutil cert --ca /tmp/elastic-stack-ca.p12 --days 3650 --out /tmp/elastic-certificates.p12
3. copy node certificate to ansible 'files' directory (with default filename 'elastic-certificates.p12'), or configure path to certs explicitly by defining 'es_security_path_node_certificate' variable
4. set 'es_security_enable_xpack' role variable to 'true' (es_security_enable_xpack: true)
5. set 'es_security_enable_ssl_transport' role variable to 'true' (es_security_enable_ssl_transport: true)
6. run role to apply new security settings
7. login to any es node and setup passwords for elasticsearch built-in users:
     bin/elasticsearch-setup-passwords auto (The passwords will be randomly generated and printed to the console)
     or
     bin/elasticsearch-setup-passwords interactive (You will be prompted to enter passwords as the process progresses)
8. Check xpack security status:
     curl -u elastic http://elasticsearch:9200/_xpack?pretty

License
-------

BSD

Author Information
------------------
Maintainer: Aset Madraimov (xiaset@gmail.com) One Technologies (amadraimov@one.kz)
