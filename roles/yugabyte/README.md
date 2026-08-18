# hyperledger.fabricx.yugabyte

> Deploys and manages YugabyteDB masters and tablet servers for Fabric-X in container or Kubernetes mode, including TLS, initialization config, logs, data cleanup, and Prometheus scraper metadata.

## Table of Contents <!-- omit in toc -->

- [Role Defaults](#role-defaults)
- [ansible-doc](#ansible-doc)
- [Tasks](#tasks)
  - [start](#start)
  - [stop](#stop)
  - [teardown](#teardown)
  - [wipe](#wipe)
  - [fetch\_logs](#fetch_logs)
  - [ping](#ping)
  - [k8s/ping](#k8sping)
  - [crypto/setup](#cryptosetup)
  - [crypto/fetch](#cryptofetch)
  - [crypto/rm](#cryptorm)
  - [crypto/openssl/generate\_csr](#cryptoopensslgenerate_csr)
  - [crypto/openssl/fetch\_csr](#cryptoopensslfetch_csr)
  - [crypto/openssl/transfer\_cert](#cryptoopenssltransfer_cert)
  - [crypto/cryptogen/transfer](#cryptocryptogentransfer)
  - [crypto/fabric\_ca/enroll](#cryptofabric_caenroll)
  - [config/transfer](#configtransfer)
  - [config/rm](#configrm)
  - [config/transfer\_grafana\_dashboard](#configtransfer_grafana_dashboard)
  - [container/start](#containerstart)
  - [container/stop](#containerstop)
  - [container/rm](#containerrm)
  - [container/fetch\_logs](#containerfetch_logs)
  - [container/master/start](#containermasterstart)
  - [container/tablet/start](#containertabletstart)
  - [k8s/start](#k8sstart)
  - [k8s/rm](#k8srm)
  - [k8s/fetch\_logs](#k8sfetch_logs)
  - [k8s/config/transfer](#k8sconfigtransfer)
  - [k8s/master/start](#k8smasterstart)
  - [k8s/master/rm](#k8smasterrm)
  - [k8s/tablet/start](#k8stabletstart)
  - [k8s/tablet/rm](#k8stabletrm)
  - [k8s/crypto/transfer](#k8scryptotransfer)
  - [bin/start](#binstart)
  - [bin/install](#bininstall)
  - [bin/master/start](#binmasterstart)
  - [bin/tablet/start](#bintabletstart)
  - [bin/stop](#binstop)
  - [bin/fetch\_logs](#binfetch_logs)
  - [data/rm](#datarm)
  - [prometheus/get\_scrapers](#prometheusget_scrapers)
  - [openshift/start](#openshiftstart)
  - [openshift/rm](#openshiftrm)
  - [openshift/ping](#openshiftping)
  - [openshift/master/start](#openshiftmasterstart)
  - [openshift/master/rm](#openshiftmasterrm)
  - [openshift/tablet/start](#openshifttabletstart)
  - [openshift/tablet/rm](#openshifttabletrm)

## Role Defaults

See [`defaults/main.yaml`](defaults/main.yaml) for the generated role defaults and inline variable descriptions.

## ansible-doc

You can view the role documentation in your terminal running:

```shell
ansible-doc -t role hyperledger.fabricx.yugabyte
```

## Tasks

### start

> Start a YugabyteDB cluster

Builds master and tablet topology facts from `yugabyte_cluster`, derives the master RPC endpoint list, and dispatches each host to the container or Kubernetes startup path. Master hosts run `yb-master` with a replication factor based on the master host list; tablet hosts run `yb-tserver` and the first tablet initializes the configured database.

```yaml
- name: Start a YugabyteDB cluster
  vars:
    # Lists the inventory hosts that belong to the YugabyteDB cluster.
    yugabyte_cluster:
      - "yb-master-1"
      - "yb-master-2"
      - "yb-master-3"
      - "yb-tserver-1"
      - "yb-tserver-2"
      - "yb-tserver-3"
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Enables container mode for the YugabyteDB role.
    yugabyte_use_container: "{{ (not yugabyte_use_bin) and (not yugabyte_use_k8s) and (not yugabyte_use_openshift) }}"
    # Enables binary mode for the YugabyteDB role, unpacking the release archive on the host and running yb-master or yb-tserver directly under tmux. Unlike the Fabric-X components, YugabyteDB is not built from source. The archive is downloaded once onto the control node and unpacked from there onto each database host, so the database hosts themselves need no route to the internet.
    yugabyte_use_bin: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: start
```

### stop

> Stop YugabyteDB runtime

Stops the YugabyteDB runtime for container deployments. Kubernetes mode is managed through the removal entry points because StatefulSets and Services are reconciled resources.

```yaml
- name: Stop YugabyteDB runtime
  vars:
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Enables container mode for the YugabyteDB role.
    yugabyte_use_container: "{{ (not yugabyte_use_bin) and (not yugabyte_use_k8s) and (not yugabyte_use_openshift) }}"
    # Enables binary mode for the YugabyteDB role, unpacking the release archive on the host and running yb-master or yb-tserver directly under tmux. Unlike the Fabric-X components, YugabyteDB is not built from source. The archive is downloaded once onto the control node and unpacked from there onto each database host, so the database hosts themselves need no route to the internet.
    yugabyte_use_bin: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: stop
```

### teardown

> Remove YugabyteDB runtime artifacts

Removes the active YugabyteDB runtime in the selected deployment mode and then removes persisted role-managed data. Container mode deletes the running container and host data directory; Kubernetes mode deletes StatefulSets, Services, optional NodePort Services, and PVCs. Binary mode stops the process and deletes its data directories, but keeps the unpacked release so that a redeploy does not download and unpack it again, in the same way that container mode keeps the image.

```yaml
- name: Remove YugabyteDB runtime artifacts
  vars:
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Enables container mode for the YugabyteDB role.
    yugabyte_use_container: "{{ (not yugabyte_use_bin) and (not yugabyte_use_k8s) and (not yugabyte_use_openshift) }}"
    # Enables binary mode for the YugabyteDB role, unpacking the release archive on the host and running yb-master or yb-tserver directly under tmux. Unlike the Fabric-X components, YugabyteDB is not built from source. The archive is downloaded once onto the control node and unpacked from there onto each database host, so the database hosts themselves need no route to the internet.
    yugabyte_use_bin: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: teardown
```

### wipe

> Wipe YugabyteDB state

Runs teardown, removes persisted data, and deletes role-managed TLS and initialization configuration artifacts. Use this entry point when the YugabyteDB node should be returned to a clean state before regeneration.

```yaml
- name: Wipe YugabyteDB state
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: wipe
```

### fetch_logs

> Collect YugabyteDB logs

Collects YugabyteDB logs through the selected deployment mode. Container mode fetches logs from the named container; Kubernetes mode fetches pod logs selected by the resource labels; binary mode fetches the tmux session's log file.

```yaml
- name: Collect YugabyteDB logs
  vars:
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Enables container mode for the YugabyteDB role.
    yugabyte_use_container: "{{ (not yugabyte_use_bin) and (not yugabyte_use_k8s) and (not yugabyte_use_openshift) }}"
    # Enables binary mode for the YugabyteDB role, unpacking the release archive on the host and running yb-master or yb-tserver directly under tmux. Unlike the Fabric-X components, YugabyteDB is not built from source. The archive is downloaded once onto the control node and unpacked from there onto each database host, so the database hosts themselves need no route to the internet.
    yugabyte_use_bin: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: fetch_logs
```

### ping

> Check YugabyteDB service ports

Selects the expected master or tablet service ports for the current host and delegates reachability checks. Masters expose RPC and webserver ports; tablets expose YSQL, RPC, webserver, YSQL web UI, YCQL, and YCQL web UI ports.

```yaml
- name: Check YugabyteDB service ports
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Sets the master webserver port.
    yugabyte_master_webserver_port: 7000
    # Sets the master RPC bind port.
    yugabyte_master_rpc_bind_port: 7100
    # Sets the tablet YSQL bind port.
    yugabyte_tablet_pgsql_bind_port: 5433
    # Sets the tablet RPC bind port.
    yugabyte_tablet_rpc_bind_port: 9100
    # Sets the tablet webserver port.
    yugabyte_tablet_webserver_port: 9000
    # Sets the tablet YSQL web UI port.
    yugabyte_tablet_pgsql_web_port: 13000
    # Sets the tablet YCQL bind port.
    yugabyte_tablet_cql_bind_port: 9042
    # Sets the tablet YCQL web UI port.
    yugabyte_tablet_cql_web_port: 12000
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: ping
```

### k8s/ping

> Check that the YugabyteDB Kubernetes service is reachable

Probes configured Kubernetes NodePort values and LoadBalancer-exposed service ports for external reachability.

```yaml
- name: Check that the YugabyteDB Kubernetes service is reachable
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
    # Kubernetes NodePort value used by the external master RPC Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_master_rpc_node_port: 32100
    # Kubernetes NodePort value used by the external master webserver Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_master_webserver_node_port: 32000
    # Kubernetes NodePort value used by the external tablet YSQL Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_pgsql_node_port: 31433
    # Kubernetes NodePort value used by the external tablet RPC Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_rpc_node_port: 32101
    # Kubernetes NodePort value used by the external tablet webserver Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_webserver_node_port: 32001
    # Kubernetes NodePort value used by the external tablet YSQL web UI Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_pgsql_web_node_port: 32300
    # Kubernetes NodePort value used by the external tablet YCQL bind Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_cql_bind_node_port: 32042
    # Kubernetes NodePort value used by the external tablet YCQL web UI Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_cql_web_node_port: 32200
    # Set to `true` to create a LoadBalancer Service entry that exposes the master RPC port externally. When undefined or `false`, the master RPC port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_master_rpc_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the master webserver port externally. When undefined or `false`, the master webserver port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_master_webserver_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YSQL port externally. When undefined or `false`, the tablet YSQL port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_pgsql_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet RPC port externally. When undefined or `false`, the tablet RPC port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_rpc_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet webserver port externally. When undefined or `false`, the tablet webserver port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_webserver_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YSQL web UI port externally. When undefined or `false`, the tablet YSQL web UI port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_pgsql_web_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YCQL bind port externally. When undefined or `false`, the tablet YCQL bind port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_cql_bind_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YCQL web UI port externally. When undefined or `false`, the tablet YCQL web UI port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_cql_web_port: false
    # Sets the master RPC bind port.
    yugabyte_master_rpc_bind_port: 7100
    # Sets the master webserver port.
    yugabyte_master_webserver_port: 7000
    # Sets the tablet YSQL bind port.
    yugabyte_tablet_pgsql_bind_port: 5433
    # Sets the tablet RPC bind port.
    yugabyte_tablet_rpc_bind_port: 9100
    # Sets the tablet webserver port.
    yugabyte_tablet_webserver_port: 9000
    # Sets the tablet YSQL web UI port.
    yugabyte_tablet_pgsql_web_port: 13000
    # Sets the tablet YCQL bind port.
    yugabyte_tablet_cql_bind_port: 9042
    # Sets the tablet YCQL web UI port.
    yugabyte_tablet_cql_web_port: 12000
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/ping
```

### crypto/setup

> Prepare YugabyteDB TLS assets

Prepares TLS assets for YugabyteDB when TLS is enabled, using the configured crypto source. The flow can generate CSRs, fetch certificates, transfer cryptogen material, enroll through Fabric CA, and create the Kubernetes Secret used by pods.

```yaml
- name: Prepare YugabyteDB TLS assets
  vars:
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Provides the organization metadata consumed by the crypto entry points. The mapping is expected to expose `domain`, `role`, `peer.name`, `peer.secret`, and `fabric_ca_host` when relevant.
    organization:
      domain: "org1.example.com"
      role: "peer"
      peer:
        name: "yb-tserver-1"
        secret: "yb-tserver-1PWD"
      fabric_ca_host: "fca-org1"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: crypto/setup
```

### crypto/fetch

> Fetch YugabyteDB TLS certificates

Copies generated YugabyteDB node and CA certificates from the remote host to the control-node artifact directory. Fetched artifacts are reused by certificate transfer, Kubernetes Secret generation, and TLS-enabled Prometheus scraper configuration.

```yaml
- name: Fetch YugabyteDB TLS certificates
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Defines the control-node directory that stores fetched YugabyteDB artifacts. Required when TLS-enabled tasks need access to fetched CA or certificate artifacts, such as when `yugabyte_use_tls` or webserver TLS is enabled.
    fetched_artifacts_dir: "/tmp/fabric-x/artifacts/yugabyte"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: crypto/fetch
```

### crypto/rm

> Remove YugabyteDB TLS artifacts

Deletes the remote YugabyteDB TLS directory and, in Kubernetes mode, removes the generated TLS Secret. This removes role-managed key, certificate, and CA material without changing database data.

```yaml
- name: Remove YugabyteDB TLS artifacts
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: crypto/rm
```

### crypto/openssl/generate_csr

> Generate a YugabyteDB TLS CSR

Builds the YugabyteDB TLS SAN list from host addresses and organization metadata, then delegates CSR generation to the OpenSSL role. The generated key, CSR, and extension file are written under the remote YugabyteDB TLS configuration path.

```yaml
- name: Generate a YugabyteDB TLS CSR
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Provides the organization metadata consumed by the crypto entry points. The mapping is expected to expose `domain`, `role`, `peer.name`, `peer.secret`, and `fabric_ca_host` when relevant.
    organization:
      domain: "org1.example.com"
      role: "peer"
      peer:
        name: "yb-tserver-1"
        secret: "yb-tserver-1PWD"
      fabric_ca_host: "fca-org1"
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_master_webserver_route: "yugabyte-master-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_webserver_route: "yugabyte-tablet-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_pgsql_web_route: "yugabyte-tablet-pgsql-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_cql_web_route: "yugabyte-tablet-cql-web.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: crypto/openssl/generate_csr
```

### crypto/openssl/fetch_csr

> Fetch a YugabyteDB TLS CSR

Copies the YugabyteDB CSR and OpenSSL extension file from the remote TLS directory to the control-node artifact directory. Use this before signing the node certificate outside the target host.

```yaml
- name: Fetch a YugabyteDB TLS CSR
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Defines the control-node directory that stores fetched YugabyteDB artifacts. Required when TLS-enabled tasks need access to fetched CA or certificate artifacts, such as when `yugabyte_use_tls` or webserver TLS is enabled.
    fetched_artifacts_dir: "/tmp/fabric-x/artifacts/yugabyte"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: crypto/openssl/fetch_csr
```

### crypto/openssl/transfer_cert

> Transfer a signed YugabyteDB TLS certificate

Copies the signed YugabyteDB node certificate and trusted organization TLS CA certificate to the remote TLS directory. The transferred files are consumed by container volume mounts or by Kubernetes Secret generation.

```yaml
- name: Transfer a signed YugabyteDB TLS certificate
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Defines the control-node directory that stores fetched YugabyteDB artifacts. Required when TLS-enabled tasks need access to fetched CA or certificate artifacts, such as when `yugabyte_use_tls` or webserver TLS is enabled.
    fetched_artifacts_dir: "/tmp/fabric-x/artifacts/yugabyte"
    # Provides the organization metadata consumed by the crypto entry points. The mapping is expected to expose `domain`, `role`, `peer.name`, `peer.secret`, and `fabric_ca_host` when relevant.
    organization:
      domain: "org1.example.com"
      role: "peer"
      peer:
        name: "yb-tserver-1"
        secret: "yb-tserver-1PWD"
      fabric_ca_host: "fca-org1"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: crypto/openssl/transfer_cert
```

### crypto/cryptogen/transfer

> Copy cryptogen TLS material for YugabyteDB

Transfers the YugabyteDB TLS key, certificate, and CA certificate generated by cryptogen to the target host. The source path is resolved from the organization domain and peer identity, with the inventory host used as the peer name when no peer name is provided.

```yaml
- name: Copy cryptogen TLS material for YugabyteDB
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Defines the control-node directory that stores cryptogen-generated artifacts.
    cryptogen_artifacts_dir: "/tmp/fabric-x/crypto-config"
    # Provides the organization metadata consumed by the crypto entry points. The mapping is expected to expose `domain`, `role`, `peer.name`, `peer.secret`, and `fabric_ca_host` when relevant.
    organization:
      domain: "org1.example.com"
      role: "peer"
      peer:
        name: "yb-tserver-1"
        secret: "yb-tserver-1PWD"
      fabric_ca_host: "fca-org1"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: crypto/cryptogen/transfer
```

### crypto/fabric_ca/enroll

> Enroll YugabyteDB TLS material with Fabric CA

Copies the Fabric CA TLS root when needed and delegates YugabyteDB TLS enrollment to the Fabric CA role. Enrollment uses organization metadata, peer credentials, and the actual external host so generated certificates include the expected SANs.

```yaml
- name: Enroll YugabyteDB TLS material with Fabric CA
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Defines the control-node directory that stores fetched YugabyteDB artifacts. Required when TLS-enabled tasks need access to fetched CA or certificate artifacts, such as when `yugabyte_use_tls` or webserver TLS is enabled.
    fetched_artifacts_dir: "/tmp/fabric-x/artifacts/yugabyte"
    # Real machine host.
    actual_host: "myvpc.cloud.ibm.com"
    # Provides the organization metadata consumed by the crypto entry points. The mapping is expected to expose `domain`, `role`, `peer.name`, `peer.secret`, and `fabric_ca_host` when relevant.
    organization:
      domain: "org1.example.com"
      role: "peer"
      peer:
        name: "yb-tserver-1"
        secret: "yb-tserver-1PWD"
      fabric_ca_host: "fca-org1"
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_master_webserver_route: "yugabyte-master-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_webserver_route: "yugabyte-tablet-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_pgsql_web_route: "yugabyte-tablet-pgsql-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_cql_web_route: "yugabyte-tablet-cql-web.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: crypto/fabric_ca/enroll
```

### config/transfer

> Transfer YugabyteDB initialization config

Renders the YugabyteDB initialization SQL script that creates the configured database user and database. Container mode places the script in the remote config directory; Kubernetes mode also creates the ConfigMap mounted by tablet pods.

```yaml
- name: Transfer YugabyteDB initialization config
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Names the SQL initialization script used by tablet pods.
    yugabyte_init_script_file: 01-yb-init.sql
    # Sets the YugabyteDB database name created by the initialization SQL script.
    yugabyte_db: "fabricx"
    # Sets the YugabyteDB database user created by the initialization SQL script.
    yugabyte_user: "fabricx_user"
    # Sets the password for the YugabyteDB database user. Store this value in Ansible Vault.
    yugabyte_password: "my_yugabyte_password"
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: config/transfer
```

### config/rm

> Remove YugabyteDB configuration

Deletes the remote YugabyteDB configuration directory and, in Kubernetes mode, removes the generated initialization ConfigMap. This cleans role-managed config without removing running resources unless paired with teardown or wipe.

```yaml
- name: Remove YugabyteDB configuration
  vars:
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: config/rm
```

### config/transfer_grafana_dashboard

> Transfer the YugabyteDB Grafana dashboard

Selects the built-in YugabyteDB dashboard JSON file and delegates the copy step to the Grafana role. The dashboard complements the Prometheus scraper configuration generated by this role.

```yaml
- name: Transfer the YugabyteDB Grafana dashboard
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: config/transfer_grafana_dashboard
```

### container/start

> Dispatch YugabyteDB container startup

Selects the master or tablet container startup path for the current host. The selected path starts either `yb-master` or `yb-tserver` with the role-managed data, config, TLS, and port settings.

```yaml
- name: Dispatch YugabyteDB container startup
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: container/start
```

### container/stop

> Stop a YugabyteDB container

Stops the container associated with the current YugabyteDB host. This leaves host data and configuration directories in place for a later restart.

```yaml
- name: Stop a YugabyteDB container
  vars:
    # Names the YugabyteDB container associated with the current host.
    yugabyte_container_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: container/stop
```

### container/rm

> Remove a YugabyteDB container

Deletes the container associated with the current YugabyteDB host. This removes the runtime container only; data cleanup is handled by `data/rm` or `teardown`.

```yaml
- name: Remove a YugabyteDB container
  vars:
    # Names the YugabyteDB container associated with the current host.
    yugabyte_container_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: container/rm
```

### container/fetch_logs

> Fetch logs from a YugabyteDB container

Delegates log collection for the current YugabyteDB container. Collected logs come from the named master or tablet container for troubleshooting.

```yaml
- name: Fetch logs from a YugabyteDB container
  vars:
    # Names the YugabyteDB container associated with the current host.
    yugabyte_container_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: container/fetch_logs
```

### container/master/start

> Start a YugabyteDB master container

Creates the host data directory, assembles the `yb-master` command line, and starts the master container. The container publishes master RPC and webserver ports, mounts the data directory, and mounts TLS material when TLS is enabled.

```yaml
- name: Start a YugabyteDB master container
  vars:
    # Sets the shared remote data directory consumed by YugabyteDB.
    remote_data_dir: "/var/hyperledger/fabric-x/yugabyte/data"
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Sets the remote data directory used by YugabyteDB tasks.
    yugabyte_remote_data_dir: "{{ remote_data_dir }}"
    # Sets the in-container data directory used by YugabyteDB.
    yugabyte_container_data_dir: /var/data
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Names the YugabyteDB container associated with the current host.
    yugabyte_container_name: "{{ inventory_hostname }}"
    # Sets the registry endpoint used to resolve the YugabyteDB image.
    yugabyte_registry_endpoint: "{{ lookup('env', 'YUGABYTE_REGISTRY_ENDPOINT') or 'docker.io/yugabytedb' }}"
    # Sets the YugabyteDB image name.
    yugabyte_image_name: yugabyte
    # Sets the YugabyteDB image tag.
    yugabyte_image_tag: 2025.2.1.0-b141
    # Sets the YugabyteDB container image.
    yugabyte_image: "{{ yugabyte_registry_endpoint }}/{{ yugabyte_image_name }}:{{ yugabyte_image_tag }}"
    # Lists the master RPC endpoints used to bootstrap YugabyteDB tablets and health checks.
    yugabyte_master_endpoints: "yb-master-1.example.com:7100,yb-master-2.example.com:7100,yb-master-3.example.com:7100"
    # Sets the master RPC bind port.
    yugabyte_master_rpc_bind_port: 7100
    # Sets the master webserver port.
    yugabyte_master_webserver_port: 7000
    # Provides the ordered list of master hosts used to compute replication factors.
    yugabyte_master_hosts:
      - "yb-master-1"
      - "yb-master-2"
      - "yb-master-3"
    # Sets the YugabyteDB log verbosity threshold.
    yugabyte_logs_level: 3
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables node-to-node TLS for YugabyteDB.
    yugabyte_node_to_node_use_tls: "{{ yugabyte_use_tls }}"
    # Enables client-to-server TLS for YugabyteDB RPC and SQL access.
    yugabyte_client_to_server_use_tls: "{{ yugabyte_use_tls }}"
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: container/master/start
```

### container/tablet/start

> Start a YugabyteDB tablet container

Creates the host data directory, assembles the `yb-tserver` command line, starts the tablet container, and initializes the database on the first tablet host. The container publishes YSQL, RPC, webserver, and YCQL ports, mounts initialization config, and mounts TLS material when TLS is enabled.

```yaml
- name: Start a YugabyteDB tablet container
  vars:
    # Sets the shared remote data directory consumed by YugabyteDB.
    remote_data_dir: "/var/hyperledger/fabric-x/yugabyte/data"
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Sets the remote data directory used by YugabyteDB tasks.
    yugabyte_remote_data_dir: "{{ remote_data_dir }}"
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Sets the in-container data directory used by YugabyteDB.
    yugabyte_container_data_dir: /var/data
    # Names the SQL initialization script used by tablet pods.
    yugabyte_init_script_file: 01-yb-init.sql
    # Names the YugabyteDB container associated with the current host.
    yugabyte_container_name: "{{ inventory_hostname }}"
    # Sets the registry endpoint used to resolve the YugabyteDB image.
    yugabyte_registry_endpoint: "{{ lookup('env', 'YUGABYTE_REGISTRY_ENDPOINT') or 'docker.io/yugabytedb' }}"
    # Sets the YugabyteDB image name.
    yugabyte_image_name: yugabyte
    # Sets the YugabyteDB image tag.
    yugabyte_image_tag: 2025.2.1.0-b141
    # Sets the YugabyteDB container image.
    yugabyte_image: "{{ yugabyte_registry_endpoint }}/{{ yugabyte_image_name }}:{{ yugabyte_image_tag }}"
    # Lists the master RPC endpoints used to bootstrap YugabyteDB tablets and health checks.
    yugabyte_master_endpoints: "yb-master-1.example.com:7100,yb-master-2.example.com:7100,yb-master-3.example.com:7100"
    # Sets the tablet YSQL bind port.
    yugabyte_tablet_pgsql_bind_port: 5433
    # Sets the tablet RPC bind port.
    yugabyte_tablet_rpc_bind_port: 9100
    # Sets the tablet webserver port.
    yugabyte_tablet_webserver_port: 9000
    # Sets the tablet YSQL web UI port.
    yugabyte_tablet_pgsql_web_port: 13000
    # Sets the tablet YCQL bind port.
    yugabyte_tablet_cql_bind_port: 9042
    # Sets the tablet YCQL web UI port.
    yugabyte_tablet_cql_web_port: 12000
    # Provides the ordered list of tablet hosts used to initialize the first tablet.
    yugabyte_tablet_hosts:
      - "yb-tserver-1"
      - "yb-tserver-2"
      - "yb-tserver-3"
    # Sets the YugabyteDB log verbosity threshold.
    yugabyte_logs_level: 3
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables node-to-node TLS for YugabyteDB.
    yugabyte_node_to_node_use_tls: "{{ yugabyte_use_tls }}"
    # Enables client-to-server TLS for YugabyteDB RPC and SQL access.
    yugabyte_client_to_server_use_tls: "{{ yugabyte_use_tls }}"
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: container/tablet/start
```

### k8s/start

> Dispatch YugabyteDB Kubernetes startup

Selects the master or tablet Kubernetes startup path for the current host. The selected path applies the Service, optional NodePort Service, StatefulSet, PVC template, image pull secret reference, and probe settings for the node type.

```yaml
- name: Dispatch YugabyteDB Kubernetes startup
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/start
```

### k8s/rm

> Dispatch YugabyteDB Kubernetes removal

Selects the master or tablet Kubernetes removal path for the current host. The selected path removes the StatefulSet, ClusterIP Service, and optional NodePort Service for the node type.

```yaml
- name: Dispatch YugabyteDB Kubernetes removal
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/rm
```

### k8s/fetch_logs

> Fetch logs from YugabyteDB pods

Delegates pod log collection for the Kubernetes resource associated with the current host. Pods are selected by the role-managed Kubernetes resource label for the master or tablet StatefulSet.

```yaml
- name: Fetch logs from YugabyteDB pods
  vars:
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/fetch_logs
```

### k8s/config/transfer

> Create a YugabyteDB ConfigMap

Creates the ConfigMap that exposes the initialization SQL script to tablet pods. The ConfigMap is only populated for tablet components because tablets run the YSQL initialization command.

```yaml
- name: Create a YugabyteDB ConfigMap
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Names the SQL initialization script used by tablet pods.
    yugabyte_init_script_file: 01-yb-init.sql
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to YugabyteDB resources.
    yugabyte_k8s_part_of: yugabyte
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/config/transfer
```

### k8s/master/start

> Start a YugabyteDB master StatefulSet

Applies the master ClusterIP Service, optional NodePort and LoadBalancer Services, and StatefulSet for the current YugabyteDB master node. The StatefulSet runs `yb-master`, configures replication from the master host list, attaches persistent storage, mounts TLS Secrets when enabled, and waits for readiness when requested.

```yaml
- name: Start a YugabyteDB master StatefulSet
  vars:
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to YugabyteDB resources.
    yugabyte_k8s_part_of: yugabyte
    # Sets the YugabyteDB container image.
    yugabyte_image: "{{ yugabyte_registry_endpoint }}/{{ yugabyte_image_name }}:{{ yugabyte_image_tag }}"
    # Sets the registry endpoint used to resolve the YugabyteDB image.
    yugabyte_registry_endpoint: "{{ lookup('env', 'YUGABYTE_REGISTRY_ENDPOINT') or 'docker.io/yugabytedb' }}"
    # Sets the YugabyteDB image name.
    yugabyte_image_name: yugabyte
    # Sets the YugabyteDB image tag.
    yugabyte_image_tag: 2025.2.1.0-b141
    # Lists the master RPC endpoints used to bootstrap YugabyteDB tablets and health checks.
    yugabyte_master_endpoints: "yb-master-1.example.com:7100,yb-master-2.example.com:7100,yb-master-3.example.com:7100"
    # Sets the master RPC bind port.
    yugabyte_master_rpc_bind_port: 7100
    # Sets the master webserver port.
    yugabyte_master_webserver_port: 7000
    # Provides the ordered list of master hosts used to compute replication factors.
    yugabyte_master_hosts:
      - "yb-master-1"
      - "yb-master-2"
      - "yb-master-3"
    # Sets the YugabyteDB log verbosity threshold.
    yugabyte_logs_level: 3
    # Sets the in-container data directory used by YugabyteDB.
    yugabyte_container_data_dir: /var/data
    # Waits for YugabyteDB Kubernetes resources to become ready.
    yugabyte_k8s_wait: true
    # Sets the Kubernetes readiness wait timeout in seconds.
    yugabyte_k8s_wait_timeout: 300
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables node-to-node TLS for YugabyteDB.
    yugabyte_node_to_node_use_tls: "{{ yugabyte_use_tls }}"
    # Enables client-to-server TLS for YugabyteDB RPC and SQL access.
    yugabyte_client_to_server_use_tls: "{{ yugabyte_use_tls }}"
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
    # Kubernetes NodePort value used by the external master RPC Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_master_rpc_node_port: 32100
    # Kubernetes NodePort value used by the external master webserver Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_master_webserver_node_port: 32000
    # Sets the image pull secret used by Kubernetes deployments when defined.
    k8s_image_pull_secret: "registry-pull-secret"
    # Sets the storage class used by Kubernetes PersistentVolumeClaims when defined.
    k8s_storage_class: "fast-ssd"
    # Sets the requested persistent storage size for Kubernetes deployments.
    k8s_storage_size: "20Gi"
    # Overrides the readiness probe initial delay used by Kubernetes templates when defined.
    k8s_readiness_probe_initial_delay_seconds: 30
    # Overrides the readiness probe period used by Kubernetes templates when defined.
    k8s_readiness_probe_period_seconds: 10
    # Overrides the readiness probe timeout used by Kubernetes templates when defined.
    k8s_readiness_probe_timeout_seconds: 5
    # Overrides the readiness probe failure threshold used by Kubernetes templates when defined.
    k8s_readiness_probe_failure_threshold: 12
    # Overrides the liveness probe initial delay used by Kubernetes templates when defined.
    k8s_liveness_probe_initial_delay_seconds: 60
    # Overrides the liveness probe period used by Kubernetes templates when defined.
    k8s_liveness_probe_period_seconds: 20
    # Overrides the liveness probe timeout used by Kubernetes templates when defined.
    k8s_liveness_probe_timeout_seconds: 5
    # Overrides the liveness probe failure threshold used by Kubernetes templates when defined.
    k8s_liveness_probe_failure_threshold: 6
    # Set to `true` to create a LoadBalancer Service entry that exposes the master RPC port externally. When undefined or `false`, the master RPC port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_master_rpc_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the master webserver port externally. When undefined or `false`, the master webserver port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_master_webserver_port: false
    # Optional Kubernetes container resource requests and limits.
    k8s_resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/master/start
```

### k8s/master/rm

> Remove a YugabyteDB master StatefulSet

Deletes the master StatefulSet and its Services for the current YugabyteDB master node. PersistentVolumeClaims are left for `data/rm` so runtime removal and data removal stay separate.

```yaml
- name: Remove a YugabyteDB master StatefulSet
  vars:
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Kubernetes NodePort value used by the external master RPC Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_master_rpc_node_port: 32100
    # Set to `true` to create a LoadBalancer Service entry that exposes the master RPC port externally. When undefined or `false`, the master RPC port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_master_rpc_port: false
    # Kubernetes NodePort value used by the external master webserver Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_master_webserver_node_port: 32000
    # Set to `true` to create a LoadBalancer Service entry that exposes the master webserver port externally. When undefined or `false`, the master webserver port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_master_webserver_port: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/master/rm
```

### k8s/tablet/start

> Start a YugabyteDB tablet StatefulSet

Applies the tablet ClusterIP Service, optional NodePort and LoadBalancer Services, and StatefulSet for the current YugabyteDB tablet node, then initializes the database on the first tablet. The StatefulSet runs `yb-tserver`, connects to the configured masters, attaches persistent storage, mounts the initialization ConfigMap and TLS Secret when enabled, and waits for readiness when requested.

```yaml
- name: Start a YugabyteDB tablet StatefulSet
  vars:
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to YugabyteDB resources.
    yugabyte_k8s_part_of: yugabyte
    # Sets the YugabyteDB container image.
    yugabyte_image: "{{ yugabyte_registry_endpoint }}/{{ yugabyte_image_name }}:{{ yugabyte_image_tag }}"
    # Sets the registry endpoint used to resolve the YugabyteDB image.
    yugabyte_registry_endpoint: "{{ lookup('env', 'YUGABYTE_REGISTRY_ENDPOINT') or 'docker.io/yugabytedb' }}"
    # Sets the YugabyteDB image name.
    yugabyte_image_name: yugabyte
    # Sets the YugabyteDB image tag.
    yugabyte_image_tag: 2025.2.1.0-b141
    # Lists the master RPC endpoints used to bootstrap YugabyteDB tablets and health checks.
    yugabyte_master_endpoints: "yb-master-1.example.com:7100,yb-master-2.example.com:7100,yb-master-3.example.com:7100"
    # Sets the tablet YSQL bind port.
    yugabyte_tablet_pgsql_bind_port: 5433
    # Sets the tablet RPC bind port.
    yugabyte_tablet_rpc_bind_port: 9100
    # Sets the tablet webserver port.
    yugabyte_tablet_webserver_port: 9000
    # Sets the tablet YSQL web UI port.
    yugabyte_tablet_pgsql_web_port: 13000
    # Sets the tablet YCQL bind port.
    yugabyte_tablet_cql_bind_port: 9042
    # Sets the tablet YCQL web UI port.
    yugabyte_tablet_cql_web_port: 12000
    # Sets the YugabyteDB log verbosity threshold.
    yugabyte_logs_level: 3
    # Sets the in-container data directory used by YugabyteDB.
    yugabyte_container_data_dir: /var/data
    # Waits for YugabyteDB Kubernetes resources to become ready.
    yugabyte_k8s_wait: true
    # Sets the Kubernetes readiness wait timeout in seconds.
    yugabyte_k8s_wait_timeout: 300
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables node-to-node TLS for YugabyteDB.
    yugabyte_node_to_node_use_tls: "{{ yugabyte_use_tls }}"
    # Enables client-to-server TLS for YugabyteDB RPC and SQL access.
    yugabyte_client_to_server_use_tls: "{{ yugabyte_use_tls }}"
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
    # Kubernetes NodePort value used by the external tablet YSQL Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_pgsql_node_port: 31433
    # Kubernetes NodePort value used by the external tablet RPC Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_rpc_node_port: 32101
    # Kubernetes NodePort value used by the external tablet webserver Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_webserver_node_port: 32001
    # Kubernetes NodePort value used by the external tablet YSQL web UI Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_pgsql_web_node_port: 32300
    # Kubernetes NodePort value used by the external tablet YCQL bind Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_cql_bind_node_port: 32042
    # Kubernetes NodePort value used by the external tablet YCQL web UI Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_cql_web_node_port: 32200
    # Names the SQL initialization script used by tablet pods.
    yugabyte_init_script_file: 01-yb-init.sql
    # Provides the ordered list of tablet hosts used to initialize the first tablet.
    yugabyte_tablet_hosts:
      - "yb-tserver-1"
      - "yb-tserver-2"
      - "yb-tserver-3"
    # Sets the image pull secret used by Kubernetes deployments when defined.
    k8s_image_pull_secret: "registry-pull-secret"
    # Sets the storage class used by Kubernetes PersistentVolumeClaims when defined.
    k8s_storage_class: "fast-ssd"
    # Sets the requested persistent storage size for Kubernetes deployments.
    k8s_storage_size: "20Gi"
    # Overrides the readiness probe initial delay used by Kubernetes templates when defined.
    k8s_readiness_probe_initial_delay_seconds: 30
    # Overrides the readiness probe period used by Kubernetes templates when defined.
    k8s_readiness_probe_period_seconds: 10
    # Overrides the readiness probe timeout used by Kubernetes templates when defined.
    k8s_readiness_probe_timeout_seconds: 5
    # Overrides the readiness probe failure threshold used by Kubernetes templates when defined.
    k8s_readiness_probe_failure_threshold: 12
    # Overrides the liveness probe initial delay used by Kubernetes templates when defined.
    k8s_liveness_probe_initial_delay_seconds: 60
    # Overrides the liveness probe period used by Kubernetes templates when defined.
    k8s_liveness_probe_period_seconds: 20
    # Overrides the liveness probe timeout used by Kubernetes templates when defined.
    k8s_liveness_probe_timeout_seconds: 5
    # Overrides the liveness probe failure threshold used by Kubernetes templates when defined.
    k8s_liveness_probe_failure_threshold: 6
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YSQL port externally. When undefined or `false`, the tablet YSQL port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_pgsql_port: false
    # Optional Kubernetes container resource requests and limits.
    k8s_resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet RPC port externally. When undefined or `false`, the tablet RPC port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_rpc_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet webserver port externally. When undefined or `false`, the tablet webserver port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_webserver_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YSQL web UI port externally. When undefined or `false`, the tablet YSQL web UI port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_pgsql_web_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YCQL bind port externally. When undefined or `false`, the tablet YCQL bind port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_cql_bind_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YCQL web UI port externally. When undefined or `false`, the tablet YCQL web UI port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_cql_web_port: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/tablet/start
```

### k8s/tablet/rm

> Remove a YugabyteDB tablet StatefulSet

Deletes the tablet StatefulSet and its Services for the current YugabyteDB tablet node. PersistentVolumeClaims are left for `data/rm` so runtime removal and data removal stay separate.

```yaml
- name: Remove a YugabyteDB tablet StatefulSet
  vars:
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Kubernetes NodePort value used by the external tablet YSQL Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_pgsql_node_port: 31433
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YSQL port externally. When undefined or `false`, the tablet YSQL port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_pgsql_port: false
    # Kubernetes NodePort value used by the external tablet RPC Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_rpc_node_port: 32101
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet RPC port externally. When undefined or `false`, the tablet RPC port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_rpc_port: false
    # Kubernetes NodePort value used by the external tablet webserver Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_webserver_node_port: 32001
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet webserver port externally. When undefined or `false`, the tablet webserver port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_webserver_port: false
    # Kubernetes NodePort value used by the external tablet YSQL web UI Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_pgsql_web_node_port: 32300
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YSQL web UI port externally. When undefined or `false`, the tablet YSQL web UI port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_pgsql_web_port: false
    # Kubernetes NodePort value used by the external tablet YCQL bind Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_cql_bind_node_port: 32042
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YCQL bind port externally. When undefined or `false`, the tablet YCQL bind port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_cql_bind_port: false
    # Kubernetes NodePort value used by the external tablet YCQL web UI Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    yugabyte_k8s_tablet_cql_web_node_port: 32200
    # Set to `true` to create a LoadBalancer Service entry that exposes the tablet YCQL web UI port externally. When undefined or `false`, the tablet YCQL web UI port is not included in the LoadBalancer Service.
    yugabyte_k8s_loadbalancer_expose_tablet_cql_web_port: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/tablet/rm
```

### k8s/crypto/transfer

> Create a YugabyteDB TLS Secret

Creates the Kubernetes Secret that exposes the YugabyteDB TLS key pair and CA certificate to pods. The Secret is mounted by master and tablet StatefulSets when YugabyteDB TLS is enabled.

```yaml
- name: Create a YugabyteDB TLS Secret
  vars:
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to YugabyteDB resources.
    yugabyte_k8s_part_of: yugabyte
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: k8s/crypto/transfer
```

### bin/start

> Dispatch YugabyteDB binary startup

Installs the YugabyteDB release and selects the master or tablet startup path for the current host.

```yaml
- name: Dispatch YugabyteDB binary startup
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: bin/start
```

### bin/install

> Install the YugabyteDB release archive

Downloads the release archive once onto the control node and unpacks it onto the current host. Runs the release's `post_install.sh`, which rewrites the library paths the binaries were linked against; `yb-master` and `yb-tserver` do not start without it. Fetching on the control node rather than on each host is what allows database hosts with no route to the internet.

```yaml
- name: Install the YugabyteDB release archive
  vars:
    # Selects the YugabyteDB release used in binary mode. Defaults to `yugabyte_image_tag` so that binary and container mode run the same version unless one is set deliberately.
    yugabyte_release_version: "{{ yugabyte_image_tag }}"
    # Strips the build suffix from `yugabyte_release_version`. YugabyteDB uses both forms. The archive filename carries the full build, while the download path and the directory inside the archive use the version alone, so `2025.2.1.0-b141` is published at `releases/2025.2.1.0/` and unpacks into `yugabyte-2025.2.1.0`.
    yugabyte_release_base_version: "{{ yugabyte_release_version | regex_replace('-b[0-9]+$', '') }}"
    # Names the YugabyteDB release archive to download.
    yugabyte_release_archive: "yugabyte-{{ yugabyte_release_version }}-linux-x86_64.tar.gz"
    # Lists the packages that provide the `en_US.UTF-8` locale in binary mode. YugabyteDB asks initdb for that locale by name, so a host without it cannot start a tablet server at all. Container mode is unaffected because the image ships it. The default covers the RedHat family, where the locale arrives ready to use. On Debian and Ubuntu, installing `locales` is not by itself enough; the locale also has to be generated, so add that step or preseed the image.
    yugabyte_locale_packages:
      - "{{ 'glibc-langpack-en' if ansible_facts['os_family'] == 'RedHat' else 'locales' }}"
    # Sets the URL the release archive is downloaded from.
    yugabyte_release_url: "https://software.yugabyte.com/releases/{{ yugabyte_release_base_version }}/{{ yugabyte_release_archive }}"
    # Sets the control node directory holding the downloaded release archive.
    yugabyte_control_release_dir: "{{ control_node_dir }}/yugabyte"
    # Sets the control node path of the downloaded release archive.
    yugabyte_control_release_archive: "{{ yugabyte_control_release_dir }}/{{ yugabyte_release_archive }}"
    # Sets the host directory the release archive is unpacked into. Point this at a data disk. The unpacked release is a few GB, which is more than a small root filesystem can usually spare. Keyed on `remote_deploy_dir` rather than the per-host `remote_node_dir`, because it has to resolve to the same absolute path on every node. A master bootstrapping the cluster replicates the path of its initial system catalog snapshot through Raft, so a follower whose release sits elsewhere cannot find that snapshot and aborts. Sharing the path also means one copy of the release per machine rather than one per node.
    yugabyte_install_dir: "{{ remote_deploy_dir }}/yugabyte"
    # Sets the base remote deployment directory that feeds `yugabyte_install_dir`. Shared by every node on a machine, and identical across machines.
    remote_deploy_dir: "/data1/fabric-x"
    # Sets the unpacked release directory holding `bin/yb-master` and `bin/yb-tserver`. Named after `yugabyte_release_base_version`, because that is what the archive unpacks into: the build suffix appears in the filename but not in the directory.
    yugabyte_home_dir: "{{ yugabyte_install_dir }}/yugabyte-{{ yugabyte_release_base_version }}"
    # Sets the YugabyteDB image tag.
    yugabyte_image_tag: 2025.2.1.0-b141
    # Sets the control node working directory that feeds `yugabyte_control_release_dir`.
    control_node_dir: "./out/control-node"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: bin/install
```

### bin/master/start

> Start a YugabyteDB master binary

Creates the master data directories, assembles the `yb-master` command line, and starts it under tmux. Uses the same flags as the container path, reading TLS material from the host config directory instead of a bind mount.

```yaml
- name: Start a YugabyteDB master binary
  vars:
    # Names the tmux session and log file used by the YugabyteDB binary on this host.
    yugabyte_bin_name: "{{ inventory_hostname }}"
    # Sets the unpacked release directory holding `bin/yb-master` and `bin/yb-tserver`. Named after `yugabyte_release_base_version`, because that is what the archive unpacks into: the build suffix appears in the filename but not in the directory.
    yugabyte_home_dir: "{{ yugabyte_install_dir }}/yugabyte-{{ yugabyte_release_base_version }}"
    # Sets the host directory the release archive is unpacked into. Point this at a data disk. The unpacked release is a few GB, which is more than a small root filesystem can usually spare. Keyed on `remote_deploy_dir` rather than the per-host `remote_node_dir`, because it has to resolve to the same absolute path on every node. A master bootstrapping the cluster replicates the path of its initial system catalog snapshot through Raft, so a follower whose release sits elsewhere cannot find that snapshot and aborts. Sharing the path also means one copy of the release per machine rather than one per node.
    yugabyte_install_dir: "{{ remote_deploy_dir }}/yugabyte"
    # Sets the base remote deployment directory that feeds `yugabyte_install_dir`. Shared by every node on a machine, and identical across machines.
    remote_deploy_dir: "/data1/fabric-x"
    # Selects the YugabyteDB release used in binary mode. Defaults to `yugabyte_image_tag` so that binary and container mode run the same version unless one is set deliberately.
    yugabyte_release_version: "{{ yugabyte_image_tag }}"
    # Strips the build suffix from `yugabyte_release_version`. YugabyteDB uses both forms. The archive filename carries the full build, while the download path and the directory inside the archive use the version alone, so `2025.2.1.0-b141` is published at `releases/2025.2.1.0/` and unpacks into `yugabyte-2025.2.1.0`.
    yugabyte_release_base_version: "{{ yugabyte_release_version | regex_replace('-b[0-9]+$', '') }}"
    # Sets the YugabyteDB image tag.
    yugabyte_image_tag: 2025.2.1.0-b141
    # Sets the remote data directory used by YugabyteDB tasks.
    yugabyte_remote_data_dir: "{{ remote_data_dir }}"
    # Sets the shared remote data directory consumed by YugabyteDB.
    remote_data_dir: "/var/hyperledger/fabric-x/yugabyte/data"
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Lists the data directories a master uses in binary mode. A master holds only cluster metadata, so one directory is normally enough.
    yugabyte_master_data_dirs:
      - "{{ yugabyte_remote_data_dir }}"
    # Appends extra command line flags to yb-master in binary mode.
    yugabyte_master_extra_flags:

    # Seconds to wait for a YugabyteDB binary to start serving on its port. A tablet server has to reach the masters and be assigned tablets before it accepts SQL, which on a cold cluster takes appreciably longer than a process start.
    yugabyte_bin_wait_timeout: 300
    # Lists the master RPC endpoints used to bootstrap YugabyteDB tablets and health checks.
    yugabyte_master_endpoints: "yb-master-1.example.com:7100,yb-master-2.example.com:7100,yb-master-3.example.com:7100"
    # Provides the ordered list of master hosts used to compute replication factors.
    yugabyte_master_hosts:
      - "yb-master-1"
      - "yb-master-2"
      - "yb-master-3"
    # Sets the master RPC bind port.
    yugabyte_master_rpc_bind_port: 7100
    # Sets the master webserver port.
    yugabyte_master_webserver_port: 7000
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Sets the YugabyteDB log verbosity threshold.
    yugabyte_logs_level: 3
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables node-to-node TLS for YugabyteDB.
    yugabyte_node_to_node_use_tls: "{{ yugabyte_use_tls }}"
    # Enables client-to-server TLS for YugabyteDB RPC and SQL access.
    yugabyte_client_to_server_use_tls: "{{ yugabyte_use_tls }}"
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: bin/master/start
```

### bin/tablet/start

> Start a YugabyteDB tablet server binary

Creates the tablet data directories, assembles the `yb-tserver` command line, starts it under tmux, and initializes the database from the first tablet host. Give `yugabyte_tablet_data_dirs` one directory per physical disk: a tablet server spreads its tablets over the directories it is given and works them in parallel.

```yaml
- name: Start a YugabyteDB tablet server binary
  vars:
    # Names the tmux session and log file used by the YugabyteDB binary on this host.
    yugabyte_bin_name: "{{ inventory_hostname }}"
    # Sets the unpacked release directory holding `bin/yb-master` and `bin/yb-tserver`. Named after `yugabyte_release_base_version`, because that is what the archive unpacks into: the build suffix appears in the filename but not in the directory.
    yugabyte_home_dir: "{{ yugabyte_install_dir }}/yugabyte-{{ yugabyte_release_base_version }}"
    # Sets the host directory the release archive is unpacked into. Point this at a data disk. The unpacked release is a few GB, which is more than a small root filesystem can usually spare. Keyed on `remote_deploy_dir` rather than the per-host `remote_node_dir`, because it has to resolve to the same absolute path on every node. A master bootstrapping the cluster replicates the path of its initial system catalog snapshot through Raft, so a follower whose release sits elsewhere cannot find that snapshot and aborts. Sharing the path also means one copy of the release per machine rather than one per node.
    yugabyte_install_dir: "{{ remote_deploy_dir }}/yugabyte"
    # Sets the base remote deployment directory that feeds `yugabyte_install_dir`. Shared by every node on a machine, and identical across machines.
    remote_deploy_dir: "/data1/fabric-x"
    # Selects the YugabyteDB release used in binary mode. Defaults to `yugabyte_image_tag` so that binary and container mode run the same version unless one is set deliberately.
    yugabyte_release_version: "{{ yugabyte_image_tag }}"
    # Strips the build suffix from `yugabyte_release_version`. YugabyteDB uses both forms. The archive filename carries the full build, while the download path and the directory inside the archive use the version alone, so `2025.2.1.0-b141` is published at `releases/2025.2.1.0/` and unpacks into `yugabyte-2025.2.1.0`.
    yugabyte_release_base_version: "{{ yugabyte_release_version | regex_replace('-b[0-9]+$', '') }}"
    # Sets the YugabyteDB image tag.
    yugabyte_image_tag: 2025.2.1.0-b141
    # Sets the remote data directory used by YugabyteDB tasks.
    yugabyte_remote_data_dir: "{{ remote_data_dir }}"
    # Sets the shared remote data directory consumed by YugabyteDB.
    remote_data_dir: "/var/hyperledger/fabric-x/yugabyte/data"
    # Sets the shared remote configuration directory consumed by YugabyteDB.
    remote_config_dir: "/opt/hyperledger/fabric-x/yugabyte/config"
    # Lists the data directories a tablet server uses in binary mode. Give it one directory per physical disk. A tablet server spreads its tablets over the directories it is given and reads and writes them in parallel, which a single directory cannot do however fast the disk behind it is.
    yugabyte_tablet_data_dirs:
      - "{{ yugabyte_remote_data_dir }}"
    # Appends extra command line flags to yb-tserver in binary mode.
    yugabyte_tablet_extra_flags:

    # Seconds to wait for a YugabyteDB binary to start serving on its port. A tablet server has to reach the masters and be assigned tablets before it accepts SQL, which on a cold cluster takes appreciably longer than a process start.
    yugabyte_bin_wait_timeout: 300
    # Seconds any single yb-admin readiness call may take before it is abandoned. yb-admin blocks inside its client setup when the masters are not answering, so it has to be bounded for the surrounding retry loop to make progress rather than spending its whole budget on one hang.
    yugabyte_bin_admin_timeout: 15
    # Lists the master RPC endpoints used to bootstrap YugabyteDB tablets and health checks.
    yugabyte_master_endpoints: "yb-master-1.example.com:7100,yb-master-2.example.com:7100,yb-master-3.example.com:7100"
    # Provides the ordered list of tablet hosts used to initialize the first tablet.
    yugabyte_tablet_hosts:
      - "yb-tserver-1"
      - "yb-tserver-2"
      - "yb-tserver-3"
    # Sets the tablet YSQL bind port.
    yugabyte_tablet_pgsql_bind_port: 5433
    # Sets the tablet RPC bind port.
    yugabyte_tablet_rpc_bind_port: 9100
    # Sets the tablet webserver port.
    yugabyte_tablet_webserver_port: 9000
    # Sets the tablet YSQL web UI port.
    yugabyte_tablet_pgsql_web_port: 13000
    # Sets the tablet YCQL bind port.
    yugabyte_tablet_cql_bind_port: 9042
    # Sets the tablet YCQL web UI port.
    yugabyte_tablet_cql_web_port: 12000
    # Sets the remote configuration directory used by YugabyteDB tasks.
    yugabyte_remote_config_dir: "{{ remote_config_dir }}"
    # Names the SQL initialization script used by tablet pods.
    yugabyte_init_script_file: 01-yb-init.sql
    # Sets the YugabyteDB log verbosity threshold.
    yugabyte_logs_level: 3
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables node-to-node TLS for YugabyteDB.
    yugabyte_node_to_node_use_tls: "{{ yugabyte_use_tls }}"
    # Enables client-to-server TLS for YugabyteDB RPC and SQL access.
    yugabyte_client_to_server_use_tls: "{{ yugabyte_use_tls }}"
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: bin/tablet/start
```

### bin/stop

> Stop a YugabyteDB binary

Stops the tmux session running `yb-master` or `yb-tserver` on the current host. This leaves the unpacked release, the data directories and the configuration in place for a later restart.

```yaml
- name: Stop a YugabyteDB binary
  vars:
    # Names the tmux session and log file used by the YugabyteDB binary on this host.
    yugabyte_bin_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: bin/stop
```

### bin/fetch_logs

> Fetch logs from a YugabyteDB binary

Collects the log file written by the tmux session running the YugabyteDB binary.

```yaml
- name: Fetch logs from a YugabyteDB binary
  vars:
    # Names the tmux session and log file used by the YugabyteDB binary on this host.
    yugabyte_bin_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: bin/fetch_logs
```

### data/rm

> Remove YugabyteDB persisted data

Deletes persisted YugabyteDB data for the selected deployment mode. Container mode removes the host data directory; Kubernetes mode removes the PVC associated with the StatefulSet volume claim; binary mode removes each configured data directory, which may be one per disk.

```yaml
- name: Remove YugabyteDB persisted data
  vars:
    # Sets the shared remote data directory consumed by YugabyteDB.
    remote_data_dir: "/var/hyperledger/fabric-x/yugabyte/data"
    # Enables container mode for the YugabyteDB role.
    yugabyte_use_container: "{{ (not yugabyte_use_bin) and (not yugabyte_use_k8s) and (not yugabyte_use_openshift) }}"
    # Enables Kubernetes mode for the YugabyteDB role.
    yugabyte_use_k8s: false
    # Selects the OpenShift deployment branch.
    yugabyte_use_openshift: false
    # Enables binary mode for the YugabyteDB role, unpacking the release archive on the host and running yb-master or yb-tserver directly under tmux. Unlike the Fabric-X components, YugabyteDB is not built from source. The archive is downloaded once onto the control node and unpacked from there onto each database host, so the database hosts themselves need no route to the internet.
    yugabyte_use_bin: false
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
    # Lists the data directories a master uses in binary mode. A master holds only cluster metadata, so one directory is normally enough.
    yugabyte_master_data_dirs:
      - "{{ yugabyte_remote_data_dir }}"
    # Lists the data directories a tablet server uses in binary mode. Give it one directory per physical disk. A tablet server spreads its tablets over the directories it is given and reads and writes them in parallel, which a single directory cannot do however fast the disk behind it is.
    yugabyte_tablet_data_dirs:
      - "{{ yugabyte_remote_data_dir }}"
    # Sets the remote data directory used by YugabyteDB tasks.
    yugabyte_remote_data_dir: "{{ remote_data_dir }}"
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Sets the Kubernetes namespace used by YugabyteDB resources.
    k8s_namespace: "fabricx-yugabyte"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: data/rm
```

### prometheus/get_scrapers

> Build Prometheus scrapers for YugabyteDB

Groups YugabyteDB hosts by cluster and assembles Prometheus scrape configuration for exposed master and tablet metrics endpoints. When webserver TLS is enabled, the generated scraper references the fetched organization TLS CA artifact for HTTPS scraping.

```yaml
- name: Build Prometheus scrapers for YugabyteDB
  vars:
    # Lists the inventory hosts that belong to the YugabyteDB clusters monitored by Prometheus.
    yugabyte_hosts:
      - "yb-master-1"
      - "yb-master-2"
      - "yb-master-3"
      - "yb-tserver-1"
      - "yb-tserver-2"
      - "yb-tserver-3"
    # Defines the control-node directory that stores fetched YugabyteDB artifacts. Required when TLS-enabled tasks need access to fetched CA or certificate artifacts, such as when `yugabyte_use_tls` or webserver TLS is enabled.
    fetched_artifacts_dir: "/tmp/fabric-x/artifacts/yugabyte"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: prometheus/get_scrapers
```

### openshift/start

> Dispatch YugabyteDB OpenShift startup

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Dispatch YugabyteDB OpenShift startup
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: openshift/start
```

### openshift/rm

> Dispatch YugabyteDB OpenShift removal

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Dispatch YugabyteDB OpenShift removal
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: openshift/rm
```

### openshift/ping

> Check the YugabyteDB OpenShift deployment

Checks configured OpenShift Routes and reuses the Kubernetes service ping flow.

```yaml
- name: Check the YugabyteDB OpenShift deployment
  vars:
    # Selects whether the current host is handled as a YugabyteDB master or tablet node.
    yugabyte_component_type: "tablet"
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_master_webserver_route: "yugabyte-master-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_webserver_route: "yugabyte-tablet-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_pgsql_web_route: "yugabyte-tablet-pgsql-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_cql_web_route: "yugabyte-tablet-cql-web.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: openshift/ping
```

### openshift/master/start

> Start the YugabyteDB master OpenShift deployment

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Start the YugabyteDB master OpenShift deployment
  vars:
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to YugabyteDB resources.
    yugabyte_k8s_part_of: yugabyte
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_master_webserver_route: "yugabyte-master-web.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: openshift/master/start
```

### openshift/master/rm

> Remove the YugabyteDB master OpenShift deployment

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Remove the YugabyteDB master OpenShift deployment
  vars:
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_master_webserver_route: "yugabyte-master-web.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: openshift/master/rm
```

### openshift/tablet/start

> Start the YugabyteDB tablet OpenShift deployment

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Start the YugabyteDB tablet OpenShift deployment
  vars:
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to YugabyteDB resources.
    yugabyte_k8s_part_of: yugabyte
    # Enables TLS asset handling for YugabyteDB.
    yugabyte_use_tls: false
    # Enables HTTPS for the YugabyteDB webserver.
    yugabyte_webserver_use_tls: "{{ yugabyte_use_tls }}"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_webserver_route: "yugabyte-tablet-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_pgsql_web_route: "yugabyte-tablet-pgsql-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_cql_web_route: "yugabyte-tablet-cql-web.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: openshift/tablet/start
```

### openshift/tablet/rm

> Remove the YugabyteDB tablet OpenShift deployment

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Remove the YugabyteDB tablet OpenShift deployment
  vars:
    # Names the Kubernetes resources associated with the current host, including the derived NodePort Service when enabled.
    yugabyte_k8s_resource_name: "{{ inventory_hostname }}"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_webserver_route: "yugabyte-tablet-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_pgsql_web_route: "yugabyte-tablet-pgsql-web.apps.example.com"
    # Specifies the OpenShift Route host.
    yugabyte_openshift_tablet_cql_web_route: "yugabyte-tablet-cql-web.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.yugabyte
    tasks_from: openshift/tablet/rm
```
