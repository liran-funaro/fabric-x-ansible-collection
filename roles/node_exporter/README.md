# hyperledger.fabricx.node_exporter

> Runs Prometheus Node Exporter in container or Kubernetes mode to collect machine state metrics such as RAM, disk, CPU, and network usage.

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
  - [get\_host\_set](#get_host_set)
  - [config/transfer](#configtransfer)
  - [config/rm](#configrm)
  - [config/transfer\_grafana\_dashboard](#configtransfer_grafana_dashboard)
  - [crypto/setup](#cryptosetup)
  - [crypto/fetch](#cryptofetch)
  - [crypto/rm](#cryptorm)
  - [crypto/openssl/generate\_cert](#cryptoopensslgenerate_cert)
  - [bin/install](#bininstall)
  - [bin/start](#binstart)
  - [bin/stop](#binstop)
  - [container/start](#containerstart)
  - [container/stop](#containerstop)
  - [container/rm](#containerrm)
  - [container/fetch\_logs](#containerfetch_logs)
  - [k8s/start](#k8sstart)
  - [k8s/ping](#k8sping)
  - [k8s/rm](#k8srm)
  - [k8s/fetch\_logs](#k8sfetch_logs)
  - [k8s/config/transfer](#k8sconfigtransfer)
  - [k8s/config/rm](#k8sconfigrm)
  - [k8s/crypto/transfer](#k8scryptotransfer)
  - [k8s/crypto/rm](#k8scryptorm)
  - [prometheus/get\_scrapers](#prometheusget_scrapers)
  - [openshift/start](#openshiftstart)
  - [openshift/ping](#openshiftping)
  - [openshift/rm](#openshiftrm)

## Role Defaults

See [`defaults/main.yaml`](defaults/main.yaml) for the generated role defaults and inline variable descriptions.

## ansible-doc

You can view the role documentation in your terminal running:

```shell
ansible-doc -t role hyperledger.fabricx.node_exporter
```

## Tasks

### start

> Start Node Exporter

Starts Node Exporter using the backend selected for the host. In container mode, launches the local `node_exporter_container_name` container and publishes `node_exporter_port`. In Kubernetes mode, creates the Service and DaemonSet in `k8s_namespace` and waits for rollout completion.

```yaml
- name: Start Node Exporter
  vars:
    # Installs Node Exporter as a host binary under systemd instead of as a container. Node Exporter measures the machine rather than any one experiment, so binary mode enables the service and leaves it running across `make stop` and `make teardown`. A gap in host metrics between runs is what makes two runs hard to compare. Also the only mode available on machines that cannot pull an image. The release archive is downloaded once on the control node and copied from there.
    node_exporter_use_bin: false
    # Enables the container backend.
    node_exporter_use_container: "{{ (not node_exporter_use_bin) and (not node_exporter_use_k8s) and (not node_exporter_use_openshift) }}"
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: start
```

### stop

> Stop Node Exporter

Stops the container-backed Node Exporter workload for the host. Kubernetes deployments use `teardown` instead of this entry point.

```yaml
- name: Stop Node Exporter
  vars:
    # Installs Node Exporter as a host binary under systemd instead of as a container. Node Exporter measures the machine rather than any one experiment, so binary mode enables the service and leaves it running across `make stop` and `make teardown`. A gap in host metrics between runs is what makes two runs hard to compare. Also the only mode available on machines that cannot pull an image. The release archive is downloaded once on the control node and copied from there.
    node_exporter_use_bin: false
    # Enables the container backend.
    node_exporter_use_container: "{{ (not node_exporter_use_bin) and (not node_exporter_use_k8s) and (not node_exporter_use_openshift) }}"
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: stop
```

### teardown

> Remove Node Exporter runtime resources

Removes Node Exporter runtime resources for the enabled backend. Container mode removes the local runtime container. Kubernetes mode removes the Service, optional NodePort Service, and DaemonSet.

```yaml
- name: Remove Node Exporter runtime resources
  vars:
    # Enables the container backend.
    node_exporter_use_container: "{{ (not node_exporter_use_bin) and (not node_exporter_use_k8s) and (not node_exporter_use_openshift) }}"
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: teardown
```

### wipe

> Remove Node Exporter data and runtime resources

Removes runtime resources, generated TLS material, and transferred configuration for Node Exporter. Use this to fully reset either deployment mode on the current host.

```yaml
- name: Remove Node Exporter data and runtime resources
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: wipe
```

### fetch_logs

> Collect Node Exporter logs

Collects logs from the active Node Exporter backend for this host. Container mode reads the container log stream. Kubernetes mode reads the matching pod logs from the DaemonSet.

```yaml
- name: Collect Node Exporter logs
  vars:
    # Enables the container backend.
    node_exporter_use_container: "{{ (not node_exporter_use_bin) and (not node_exporter_use_k8s) and (not node_exporter_use_openshift) }}"
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: fetch_logs
```

### ping

> Check Node Exporter reachability

Verifies that the Node Exporter metrics port is reachable on the current host. Kubernetes deployments verify the optional NodePort exposure when it is enabled.

```yaml
- name: Check Node Exporter reachability
  vars:
    # Sets the TCP port exposed by Node Exporter and seeds the default Kubernetes NodePort value.
    node_exporter_port: 9100
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: ping
```

### get_host_set

> Build the Node Exporter host group

Adds one inventory host per unique `ansible_host` with a defined `node_exporter_port` to the `node_exporter_hosts` group. Example hosts include `worker-1`, `worker-2`, and `worker-3`.

```yaml
- name: Build the Node Exporter host group
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: get_host_set
```

### config/transfer

> Transfer Node Exporter configuration

Creates the Node Exporter configuration directory and renders the web configuration file when TLS is enabled. Container mode writes the config under `node_exporter_remote_config_dir` for bind-mounting into the runtime container. Kubernetes mode also applies the ConfigMap that supplies the same config to the DaemonSet.

```yaml
- name: Transfer Node Exporter configuration
  vars:
    # Installs Node Exporter as a host binary under systemd instead of as a container. Node Exporter measures the machine rather than any one experiment, so binary mode enables the service and leaves it running across `make stop` and `make teardown`. A gap in host metrics between runs is what makes two runs hard to compare. Also the only mode available on machines that cannot pull an image. The release archive is downloaded once on the control node and copied from there.
    node_exporter_use_bin: false
    # Config directory as the running exporter sees it, used inside rendered files. A container reads its config through a bind mount and a host binary reads it directly, so a rendered path that assumed the mount sent the binary looking for its certificate under the container's directory and the service died on startup.
    node_exporter_config_dir: "{{ node_exporter_remote_config_dir if node_exporter_use_bin else node_exporter_container_config_dir }}"
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Sets the configuration mount point inside the Node Exporter container.
    node_exporter_container_config_dir: /var/config
    # Sets the rendered Node Exporter web configuration filename.
    node_exporter_web_config_file: web-config.yaml
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Sets the Node Exporter TLS private key filename.
    node_exporter_tls_private_key_file: server.key
    # Sets the Node Exporter TLS certificate filename.
    node_exporter_tls_cert_file: server.crt
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: config/transfer
```

### config/rm

> Remove Node Exporter configuration

Removes transferred Node Exporter configuration files from the remote host. Kubernetes mode also removes the ConfigMap used by the DaemonSet.

```yaml
- name: Remove Node Exporter configuration
  vars:
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: config/rm
```

### config/transfer_grafana_dashboard

> Transfer the Grafana dashboard for Node Exporter

Copies the bundled Node Exporter dashboard into Grafana by delegating to the Grafana role.

```yaml
- name: Transfer the Grafana dashboard for Node Exporter
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: config/transfer_grafana_dashboard
```

### crypto/setup

> Generate Node Exporter TLS material

Generates TLS material for Node Exporter when TLS is enabled. Container mode writes certs and keys under `node_exporter_remote_config_dir`/tls. Kubernetes mode also applies the Secret that mounts the same artifacts into the DaemonSet.

```yaml
- name: Generate Node Exporter TLS material
  vars:
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: crypto/setup
```

### crypto/fetch

> Fetch Node Exporter TLS certificates

Fetches the generated Node Exporter CA certificate and server certificate from the remote host when TLS is enabled. Writes the artifacts into `fetched_artifacts_dir` for later reuse.

```yaml
- name: Fetch Node Exporter TLS certificates
  vars:
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the local directory used to store fetched TLS artifacts.
    fetched_artifacts_dir: "/tmp/fabricx-artifacts/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: crypto/fetch
```

### crypto/rm

> Remove Node Exporter TLS material

Removes generated TLS material for Node Exporter. Kubernetes mode also removes the TLS Secret used by the DaemonSet.

```yaml
- name: Remove Node Exporter TLS material
  vars:
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Enables the Kubernetes backend or cleanup path when true.
    node_exporter_use_k8s: false
    # Selects the OpenShift deployment branch.
    node_exporter_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: crypto/rm
```

### crypto/openssl/generate_cert

> Generate a self-signed TLS certificate for Node Exporter

Delegates to the OpenSSL role to generate a self-signed certificate and private key for Node Exporter. Uses the organization data and TLS filenames to place the artifacts under the Node Exporter config path.

```yaml
- name: Generate a self-signed TLS certificate for Node Exporter
  vars:
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Sets the Node Exporter TLS private key filename.
    node_exporter_tls_private_key_file: server.key
    # Sets the Node Exporter TLS certificate filename.
    node_exporter_tls_cert_file: server.crt
    # Provides organization data used to build the OpenSSL subject. When organization data does not define a domain value, the inventory hostname is used instead.
    organization:
      domain: "node-exporter.example.org"
      common_name: "node-exporter.example.org"
      organization_name: "Example Org"
    # Specifies the OpenShift Route host.
    node_exporter_openshift_route: "node-exporter-metrics.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: crypto/openssl/generate_cert
```

### bin/install

> Install the Node Exporter release on a machine

Downloads the Node Exporter release archive once onto the control node and unpacks the binary from there onto the machine, so machines with no route to the internet are served too.

```yaml
- name: Install the Node Exporter release on a machine
  vars:
    # Sets the Node Exporter image tag used in the default image reference.
    node_exporter_image_tag: v1.12.1
    # Selects the Node Exporter release used in binary mode. Derived from `node_exporter_image_tag` without its leading `v`, so binary and container mode track the same version by default.
    node_exporter_release_version: "{{ node_exporter_image_tag | regex_replace('^v', '') }}"
    # Names the directory inside the release archive, which is also the archive stem.
    node_exporter_release_name: "node_exporter-{{ node_exporter_release_version }}.linux-{{ 'arm64' if ansible_facts['architecture'] == 'aarch64' else 'amd64' }}"
    # Names the Node Exporter release archive.
    node_exporter_release_archive: "{{ node_exporter_release_name }}.tar.gz"
    # Sets the URL the release archive is downloaded from, on the control node.
    node_exporter_release_url: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_release_version }}/{{ node_exporter_release_archive }}"
    # Sets the control node working directory that holds downloaded release archives.
    control_node_dir: "/data1/fabric-x/control-node"
    # Sets the control node directory holding the downloaded release archive.
    node_exporter_control_release_dir: "{{ control_node_dir }}/node-exporter"
    # Sets the control node path of the downloaded release archive.
    node_exporter_control_release_archive: "{{ node_exporter_control_release_dir }}/{{ node_exporter_release_archive }}"
    # Sets the directory the Node Exporter binary is installed into.
    node_exporter_install_dir: /usr/local/bin
    # Sets the installed Node Exporter binary path used by the systemd unit.
    node_exporter_bin_path: "{{ node_exporter_install_dir }}/node_exporter"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: bin/install
```

### bin/start

> Run Node Exporter as a systemd service

Installs the unit, enables it and starts it, then waits for the metrics port. Enabled deliberately, so host metrics survive a reboot and span experiments.

```yaml
- name: Run Node Exporter as a systemd service
  vars:
    # Names the systemd service used in binary mode.
    node_exporter_service_name: node-exporter
    # Runs the Node Exporter service as this user. The default reads every mount point and process without special casing. A dedicated unprivileged user works too, at the cost of some collectors.
    node_exporter_system_user: root
    # Sets the installed Node Exporter binary path used by the systemd unit.
    node_exporter_bin_path: "{{ node_exporter_install_dir }}/node_exporter"
    # Sets the directory the Node Exporter binary is installed into.
    node_exporter_install_dir: /usr/local/bin
    # Address the exporter listens on. Empty means every interface.
    node_exporter_bind_address: ""
    # Sets the TCP port exposed by Node Exporter and seeds the default Kubernetes NodePort value.
    node_exporter_port: 9100
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Sets the rendered Node Exporter web configuration filename.
    node_exporter_web_config_file: web-config.yaml
    # Directory the textfile collector reads, for metrics written by other tooling.
    node_exporter_textfile_dir: /var/lib/node_exporter/textfile_collector
    # Appends extra command line flags to the Node Exporter service.
    node_exporter_extra_flags:

  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: bin/start
```

### bin/stop

> Stop the Node Exporter service

Does nothing unless `node_exporter_stop_with_deployment` is set, so that stopping a deployment does not create a hole in the machine's metrics.

```yaml
- name: Stop the Node Exporter service
  vars:
    # Names the systemd service used in binary mode.
    node_exporter_service_name: node-exporter
    # Stops and disables the Node Exporter service when the deployment is stopped. Off by default, so host metrics continue across experiments. Turn it on when releasing a machine entirely.
    node_exporter_stop_with_deployment: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: bin/stop
```

### container/start

> Start Node Exporter in a container

Starts the Node Exporter container with the configured image, port, mounts, and optional TLS web configuration. The container name comes from `node_exporter_container_name` and the image from `node_exporter_image`. The runtime binds the host root filesystem plus `node_exporter_remote_config_dir` for config and TLS material.

```yaml
- name: Start Node Exporter in a container
  vars:
    # Sets the container name used for the Node Exporter runtime.
    node_exporter_container_name: node-exporter
    # Sets the registry endpoint used to build the default Node Exporter image reference.
    node_exporter_registry_endpoint: "{{ lookup('env', 'NODE_EXPORTER_REGISTRY_ENDPOINT') or 'docker.io/prom' }}"
    # Sets the Node Exporter image name used in the default image reference.
    node_exporter_image_name: node-exporter
    # Sets the Node Exporter image tag used in the default image reference.
    node_exporter_image_tag: v1.12.1
    # Sets the full Node Exporter image reference.
    node_exporter_image: "{{ node_exporter_registry_endpoint }}/{{ node_exporter_image_name }}:{{ node_exporter_image_tag }}"
    # Sets the TCP port exposed by Node Exporter and seeds the default Kubernetes NodePort value.
    node_exporter_port: 9100
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Sets the configuration mount point inside the Node Exporter container.
    node_exporter_container_config_dir: /var/config
    # Sets the rendered Node Exporter web configuration filename.
    node_exporter_web_config_file: web-config.yaml
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Sets the bind-mount propagation flags for the host root filesystem volume (`/:/host:<flags>`). On Linux the default is `rslave,ro`, required for Docker bind-mount propagation. On macOS, Docker Desktop with VirtioFS does not support `rslave`, so `ro` is used instead.
    node_exporter_root_fs_flags: "{{ 'ro' if ansible_facts.system == 'Darwin' else 'rslave,ro' }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: container/start
```

### container/stop

> Stop the Node Exporter container

Stops the Node Exporter container by delegating to the shared container role. Uses the configured container name and image reference to identify the runtime.

```yaml
- name: Stop the Node Exporter container
  vars:
    # Sets the container name used for the Node Exporter runtime.
    node_exporter_container_name: node-exporter
    # Sets the registry endpoint used to build the default Node Exporter image reference.
    node_exporter_registry_endpoint: "{{ lookup('env', 'NODE_EXPORTER_REGISTRY_ENDPOINT') or 'docker.io/prom' }}"
    # Sets the Node Exporter image name used in the default image reference.
    node_exporter_image_name: node-exporter
    # Sets the Node Exporter image tag used in the default image reference.
    node_exporter_image_tag: v1.12.1
    # Sets the full Node Exporter image reference.
    node_exporter_image: "{{ node_exporter_registry_endpoint }}/{{ node_exporter_image_name }}:{{ node_exporter_image_tag }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: container/stop
```

### container/rm

> Remove the Node Exporter container

Removes the Node Exporter container by delegating to the shared container role. Leaves the config and TLS paths untouched so a later start can reuse them if desired.

```yaml
- name: Remove the Node Exporter container
  vars:
    # Sets the container name used for the Node Exporter runtime.
    node_exporter_container_name: node-exporter
    # Sets the registry endpoint used to build the default Node Exporter image reference.
    node_exporter_registry_endpoint: "{{ lookup('env', 'NODE_EXPORTER_REGISTRY_ENDPOINT') or 'docker.io/prom' }}"
    # Sets the Node Exporter image name used in the default image reference.
    node_exporter_image_name: node-exporter
    # Sets the Node Exporter image tag used in the default image reference.
    node_exporter_image_tag: v1.12.1
    # Sets the full Node Exporter image reference.
    node_exporter_image: "{{ node_exporter_registry_endpoint }}/{{ node_exporter_image_name }}:{{ node_exporter_image_tag }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: container/rm
```

### container/fetch_logs

> Fetch logs from the Node Exporter container

Collects logs from the Node Exporter container by delegating to the shared container role. This is the container-mode log path used by the top-level fetch_logs entry point.

```yaml
- name: Fetch logs from the Node Exporter container
  vars:
    # Sets the container name used for the Node Exporter runtime.
    node_exporter_container_name: node-exporter
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: container/fetch_logs
```

### k8s/start

> Start Node Exporter on Kubernetes

Creates the Kubernetes Service, optional NodePort and LoadBalancer Services, and DaemonSet for Node Exporter. Uses `k8s_namespace`, `node_exporter_k8s_resource_name`, and `node_exporter_port` to shape the workload identity and service exposure. Waits for the DaemonSet rollout before returning.

```yaml
- name: Start Node Exporter on Kubernetes
  vars:
    # Sets how long to wait for the Node Exporter DaemonSet rollout.
    node_exporter_k8s_wait_timeout: 120
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to Node Exporter resources.
    node_exporter_k8s_part_of: monitoring
    # Sets the TCP port exposed by Node Exporter and seeds the default Kubernetes NodePort value.
    node_exporter_port: 9100
    # Kubernetes NodePort value used by the external metrics Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    node_exporter_k8s_node_port: 31000
    # Sets the registry endpoint used to build the default Node Exporter image reference.
    node_exporter_registry_endpoint: "{{ lookup('env', 'NODE_EXPORTER_REGISTRY_ENDPOINT') or 'docker.io/prom' }}"
    # Sets the Node Exporter image name used in the default image reference.
    node_exporter_image_name: node-exporter
    # Sets the Node Exporter image tag used in the default image reference.
    node_exporter_image_tag: v1.12.1
    # Sets the full Node Exporter image reference.
    node_exporter_image: "{{ node_exporter_registry_endpoint }}/{{ node_exporter_image_name }}:{{ node_exporter_image_tag }}"
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Sets the configuration mount point inside the Node Exporter container.
    node_exporter_container_config_dir: /var/config
    # Sets the rendered Node Exporter web configuration filename.
    node_exporter_web_config_file: web-config.yaml
    # Sets the Node Exporter TLS certificate filename.
    node_exporter_tls_cert_file: server.crt
    # Sets the Node Exporter TLS private key filename.
    node_exporter_tls_private_key_file: server.key
    # Sets the Kubernetes namespace used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet. Required when `node_exporter_use_k8s` is true.
    k8s_namespace: "monitoring"
    # Sets the optional image pull secret used by the Node Exporter DaemonSet.
    k8s_image_pull_secret: "regcred"
    # Overrides the DaemonSet readiness probe initial delay. The template defaults to 10 seconds.
    k8s_readiness_probe_initial_delay_seconds: 10
    # Overrides the DaemonSet readiness probe period. The template defaults to 10 seconds.
    k8s_readiness_probe_period_seconds: 10
    # Overrides the DaemonSet readiness probe timeout. The template defaults to 5 seconds.
    k8s_readiness_probe_timeout_seconds: 5
    # Overrides the DaemonSet readiness probe failure threshold. The template defaults to 3 failures.
    k8s_readiness_probe_failure_threshold: 3
    # Overrides the DaemonSet liveness probe initial delay. The template defaults to 30 seconds.
    k8s_liveness_probe_initial_delay_seconds: 30
    # Overrides the DaemonSet liveness probe period. The template defaults to 15 seconds.
    k8s_liveness_probe_period_seconds: 15
    # Overrides the DaemonSet liveness probe timeout. The template defaults to 5 seconds.
    k8s_liveness_probe_timeout_seconds: 5
    # Overrides the DaemonSet liveness probe failure threshold. The template defaults to 5 failures.
    k8s_liveness_probe_failure_threshold: 5
    # Set to `true` to create a LoadBalancer Service entry that exposes the metrics port externally. When undefined or `false`, the metrics port is not included in the LoadBalancer Service.
    node_exporter_k8s_loadbalancer_expose_metrics_port: false
    # UID the Node Exporter container runs as.
    node_exporter_k8s_run_as_user: 0
    # Optional Kubernetes container resource requests and limits.
    k8s_resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: k8s/start
```

### k8s/ping

> Check that Node Exporter Kubernetes service is reachable

Probes configured Kubernetes NodePort values and LoadBalancer-exposed service ports for external reachability.

```yaml
- name: Check that Node Exporter Kubernetes service is reachable
  vars:
    # Sets the TCP port exposed by Node Exporter and seeds the default Kubernetes NodePort value.
    node_exporter_port: 9100
    # Kubernetes NodePort value used by the external metrics Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    node_exporter_k8s_node_port: 31000
    # Set to `true` to create a LoadBalancer Service entry that exposes the metrics port externally. When undefined or `false`, the metrics port is not included in the LoadBalancer Service.
    node_exporter_k8s_loadbalancer_expose_metrics_port: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: k8s/ping
```

### k8s/rm

> Remove Node Exporter Kubernetes resources

Removes the Kubernetes DaemonSet and Services created for Node Exporter. Targets the workload by `k8s_namespace` and `node_exporter_k8s_resource_name`.

```yaml
- name: Remove Node Exporter Kubernetes resources
  vars:
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
    # Sets the Kubernetes namespace used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet. Required when `node_exporter_use_k8s` is true.
    k8s_namespace: "monitoring"
    # Kubernetes NodePort value used by the external metrics Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    node_exporter_k8s_node_port: 31000
    # Set to `true` to create a LoadBalancer Service entry that exposes the metrics port externally. When undefined or `false`, the metrics port is not included in the LoadBalancer Service.
    node_exporter_k8s_loadbalancer_expose_metrics_port: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: k8s/rm
```

### k8s/fetch_logs

> Fetch logs from Node Exporter pods

Collects logs from Node Exporter pods by delegating to the shared Kubernetes role. This is the Kubernetes-mode log path used by the top-level fetch_logs entry point.

```yaml
- name: Fetch logs from Node Exporter pods
  vars:
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: k8s/fetch_logs
```

### k8s/config/transfer

> Apply the Node Exporter Kubernetes ConfigMap

Ensures the target namespace exists and applies the ConfigMap used by the Node Exporter DaemonSet. The ConfigMap points at `node_exporter_remote_config_dir` content and the rendered web config when TLS is enabled.

```yaml
- name: Apply the Node Exporter Kubernetes ConfigMap
  vars:
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to Node Exporter resources.
    node_exporter_k8s_part_of: monitoring
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Sets the rendered Node Exporter web configuration filename.
    node_exporter_web_config_file: web-config.yaml
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Sets the Kubernetes namespace used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet. Required when `node_exporter_use_k8s` is true.
    k8s_namespace: "monitoring"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: k8s/config/transfer
```

### k8s/config/rm

> Remove the Node Exporter Kubernetes ConfigMap

Deletes the ConfigMap used by the Node Exporter Kubernetes deployment. Keeps the namespace and runtime pods untouched so teardown can be handled separately.

```yaml
- name: Remove the Node Exporter Kubernetes ConfigMap
  vars:
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
    # Sets the Kubernetes namespace used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet. Required when `node_exporter_use_k8s` is true.
    k8s_namespace: "monitoring"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: k8s/config/rm
```

### k8s/crypto/transfer

> Apply the Node Exporter Kubernetes TLS Secret

Ensures the target namespace exists and applies the Secret that stores Node Exporter TLS material. The Secret name matches `node_exporter_k8s_resource_name`.

```yaml
- name: Apply the Node Exporter Kubernetes TLS Secret
  vars:
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to Node Exporter resources.
    node_exporter_k8s_part_of: monitoring
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Sets the base remote deployment directory used by `node_exporter_remote_config_dir`.
    remote_deploy_dir: "/opt/fabricx/node-exporter"
    # Sets the remote Node Exporter configuration directory.
    node_exporter_remote_config_dir: "{{ remote_deploy_dir }}/node-exporter/config"
    # Sets the Node Exporter TLS private key filename.
    node_exporter_tls_private_key_file: server.key
    # Sets the Node Exporter TLS certificate filename.
    node_exporter_tls_cert_file: server.crt
    # Sets the Kubernetes namespace used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet. Required when `node_exporter_use_k8s` is true.
    k8s_namespace: "monitoring"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: k8s/crypto/transfer
```

### k8s/crypto/rm

> Remove the Node Exporter Kubernetes TLS Secret

Deletes the Secret that stores Node Exporter TLS material for Kubernetes deployments. Targets the Secret named after `node_exporter_k8s_resource_name`.

```yaml
- name: Remove the Node Exporter Kubernetes TLS Secret
  vars:
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
    # Sets the Kubernetes namespace used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet. Required when `node_exporter_use_k8s` is true.
    k8s_namespace: "monitoring"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: k8s/crypto/rm
```

### prometheus/get_scrapers

> Build Prometheus scrape targets for Node Exporter

Builds the scrape service definitions Prometheus uses to collect metrics from the selected Node Exporter hosts. When any host has `node_exporter_use_k8s` set to `true`, produces a single scrape job with `kubernetes_sd_configs` that discovers Node Exporter pods via the Kubernetes API using the pod role. Container-mode hosts produce per-host `static_configs` scrape jobs as before.

```yaml
- name: Build Prometheus scrape targets for Node Exporter
  vars:
    # Lists the inventory hosts exposed as Prometheus scrape targets.
    node_exporter_hosts:
      - "worker-1"
      - "worker-2"
      - "worker-3"
    # Sets the local directory used to store fetched TLS artifacts.
    fetched_artifacts_dir: "/tmp/fabricx-artifacts/node-exporter"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: prometheus/get_scrapers
```

### openshift/start

> Start the OpenShift deployment

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Start the OpenShift deployment
  vars:
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to Node Exporter resources.
    node_exporter_k8s_part_of: monitoring
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Specifies the OpenShift Route host.
    node_exporter_openshift_route: "node-exporter-metrics.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: openshift/start
```

### openshift/ping

> Check the OpenShift deployment

Checks configured OpenShift Routes and reuses the Kubernetes service ping flow.

```yaml
- name: Check the OpenShift deployment
  vars:
    # Enables the TLS web configuration and certificate paths when true.
    node_exporter_use_tls: false
    # Specifies the OpenShift Route host.
    node_exporter_openshift_route: "node-exporter-metrics.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: openshift/ping
```

### openshift/rm

> Remove the OpenShift deployment

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Remove the OpenShift deployment
  vars:
    # Sets the Kubernetes object name used for Node Exporter resources, including the Service, optional NodePort Service, and DaemonSet.
    node_exporter_k8s_resource_name: "{{ inventory_hostname }}"
    # Specifies the OpenShift Route host.
    node_exporter_openshift_route: "node-exporter-metrics.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.node_exporter
    tasks_from: openshift/rm
```
