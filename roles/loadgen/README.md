# hyperledger.fabricx.loadgen

> Deploys and manages the Fabric-X Load Generator across binary, container, and Kubernetes modes for workload, metrics, TLS, and log collection workflows.

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
  - [effective\_address](#effective_address)
  - [get\_metrics](#get_metrics)
  - [limit\_rate](#limit_rate)
  - [prometheus/get\_scrapers](#prometheusget_scrapers)
  - [config/transfer](#configtransfer)
  - [config/mtls/monitoring/transfer](#configmtlsmonitoringtransfer)
  - [config/rm](#configrm)
  - [crypto/setup](#cryptosetup)
  - [crypto/cryptogen/transfer](#cryptocryptogentransfer)
  - [crypto/fabric\_ca/enroll](#cryptofabric_caenroll)
  - [crypto/fetch](#cryptofetch)
  - [crypto/rm](#cryptorm)
  - [bin/build](#binbuild)
  - [bin/install](#bininstall)
  - [bin/rm](#binrm)
  - [make\_artifacts](#make_artifacts)
  - [bin/make\_artifacts](#binmake_artifacts)
  - [container/make\_artifacts](#containermake_artifacts)
  - [bin/start](#binstart)
  - [bin/stop](#binstop)
  - [bin/transfer](#bintransfer)
  - [bin/fetch\_logs](#binfetch_logs)
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
  - [openshift/start](#openshiftstart)
  - [openshift/ping](#openshiftping)
  - [openshift/rm](#openshiftrm)

## Role Defaults

See [`defaults/main.yaml`](defaults/main.yaml) for the generated role defaults and inline variable descriptions.

## ansible-doc

You can view the role documentation in your terminal running:

```shell
ansible-doc -t role hyperledger.fabricx.loadgen
```

## Tasks

### start

> Start the load generator

Start the Loadgen runtime selected by the deployment mode flags. Container mode is the default, binary mode starts the installed `loadgen` process, and Kubernetes mode applies Services and a Deployment. The runtime consumes the rendered Loadgen configuration and crypto material prepared by the config and crypto entry points.

```yaml
- name: Start the load generator
  vars:
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
    # Run the container runtime.
    loadgen_use_container: "{{ (not loadgen_use_bin) and (not loadgen_use_k8s) and (not loadgen_use_openshift) }}"
    # Run the binary runtime.
    loadgen_use_bin: false
    # Use Kubernetes resources.
    loadgen_use_k8s: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: start
```

### stop

> Stop the load generator

Stop the active Loadgen runtime selected by the deployment mode flags. Stops the local binary process or container without removing configuration, crypto material, logs, or Kubernetes resources.

```yaml
- name: Stop the load generator
  vars:
    # Run the container runtime.
    loadgen_use_container: "{{ (not loadgen_use_bin) and (not loadgen_use_k8s) and (not loadgen_use_openshift) }}"
    # Run the binary runtime.
    loadgen_use_bin: false
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: stop
```

### teardown

> Remove runtime artifacts

Remove runtime resources for the selected Loadgen deployment mode. Deletes the local container or Kubernetes workload resources while leaving generated config, crypto material, and fetched artifacts intact.

```yaml
- name: Remove runtime artifacts
  vars:
    # Run the container runtime.
    loadgen_use_container: "{{ (not loadgen_use_bin) and (not loadgen_use_k8s) and (not loadgen_use_openshift) }}"
    # Run the binary runtime.
    loadgen_use_bin: false
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: teardown
```

### wipe

> Remove all load generator data

Remove Loadgen runtime resources, binary artifacts, generated configuration, and crypto material from the host. Use this lifecycle entry point for a full role-local cleanup before rebuilding config or credentials.

```yaml
- name: Remove all load generator data
  vars:
    # Run the binary runtime.
    loadgen_use_bin: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: wipe
```

### fetch_logs

> Collect runtime logs

Collect Loadgen logs for the selected deployment mode. Binary, container, and Kubernetes modes delegate to their mode-specific log collection tasks and store the fetched runtime output as role artifacts.

```yaml
- name: Collect runtime logs
  vars:
    # Run the container runtime.
    loadgen_use_container: "{{ (not loadgen_use_bin) and (not loadgen_use_k8s) and (not loadgen_use_openshift) }}"
    # Run the binary runtime.
    loadgen_use_bin: false
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: fetch_logs
```

### ping

> Check the monitoring endpoint

Verify that the Loadgen monitoring endpoint is reachable. Uses direct host access for binary and container deployments and delegates to the Kubernetes ping task when `loadgen_use_k8s` is enabled.

```yaml
- name: Check the monitoring endpoint
  vars:
    # Prometheus metrics port exposed by Loadgen.
    loadgen_metrics_port: 9443
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: ping
```

### effective_address

> Resolve the effective connection addresses

Compute the host-and-port values used to reach the Loadgen instance from outside its own host. Sets `loadgen_effective_metrics_address` as an Ansible fact on the calling host. Resolution priority is OpenShift Route, then Kubernetes NodePort, then the plain host port. Accepts a `loadgen_host` variable so the task can be called from any host in the inventory, not just the Loadgen host itself. All Loadgen-specific variables are read from `hostvars[loadgen_host]` rather than from the calling host's scope.

```yaml
- name: Resolve the effective connection addresses
  vars:
    # Names the inventory host that provides the target Loadgen instance.
    loadgen_host: "loadgen1"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: effective_address
```

### get_metrics

> Fetch exported metrics

Query the Loadgen Prometheus metrics endpoint over HTTP or HTTPS. Delegates address resolution to the `effective_address` entry point.

```yaml
- name: Fetch exported metrics
  vars:
    # Assert the committed transaction metric and report aborted transactions when fetching metrics.
    loadgen_assert_metrics: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: get_metrics
```

### limit_rate

> Update the runtime rate limit

Send a control-plane HTTP request that changes the active generated transaction rate. Delegates address resolution to the `effective_address` entry point.

```yaml
- name: Update the runtime rate limit
  vars:
    # Maximum generated transaction rate.
    loadgen_limit_rate: 2500
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: limit_rate
```

### prometheus/get_scrapers

> Build Prometheus scrape targets

Build Prometheus scrape target definitions for all Loadgen hosts. Includes monitoring endpoint and TLS artifact paths consumed by the Prometheus role when scraping Loadgen metrics.

```yaml
- name: Build Prometheus scrape targets
  vars:
    # Inventory hosts running Loadgen instances.
    loadgen_hosts:
      - "loadgen1"
      - "loadgen2"
    # Local artifacts directory used for fetched TLS and MSP files.
    fetched_artifacts_dir: "/tmp/fabricx-artifacts"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: prometheus/get_scrapers
```

### config/transfer

> Dispatch configuration rendering

Render the Loadgen configuration file and transfer config-side support artifacts. The generated config contains orderer router and assembler targets, optional committer sidecar access, TLS and mTLS paths, workload profile settings, stream limits, and logging behavior. For Kubernetes deployments, also publishes the rendered config and trusted CA bundles as a ConfigMap.

```yaml
- name: Dispatch configuration rendering
  vars:
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Config mount path inside a container or pod.
    loadgen_container_config_dir: /config
    # Effective config directory used inside rendered Loadgen configuration.
    loadgen_config_dir: "{{ loadgen_remote_config_dir if loadgen_use_bin else loadgen_container_config_dir }}"
    # Rendered Loadgen config filename.
    loadgen_config_file: config-loadgen.yaml
    # Run the binary runtime.
    loadgen_use_bin: false
    # Coordinator host to apply load to directly, taking the sidecar and the ordering service out of the path so the coordinator's own ceiling can be measured. Setting it selects the coordinator adapter in place of the sidecar adapter. Use it when an end-to-end measurement cannot say which of two close stages is the limit, since an end-to-end run only ever reports the slowest one.
    loadgen_coordinator_host: "committer-coordinator"
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
    # Enable mTLS for the main endpoint.
    loadgen_use_mtls: false
    # Committer inventory hosts used by the dispatcher to derive client targets.
    committer_hosts:
      - "committer-sidecar1"
      - "committer-validator1"
    # Orderer inventory hosts used by the dispatcher to derive client targets.
    orderer_hosts:
      - "orderer-router1"
      - "orderer-assembler1"
    # Additional mTLS client identities trusted by the main endpoint.
    loadgen_mtls_clients:
      - "orderer-router1"
      - "committer-sidecar1"
    # Additional mTLS organizations trusted by the main endpoint.
    loadgen_mtls_orgs:
      - name: "Org1"
        domain: "org1.example.com"
      - name: "OrdererOrg1"
        domain: "ordererorg1.example.com"
    # Additional mTLS client identities trusted by the monitoring endpoint.
    loadgen_monitoring_mtls_clients:
      - "prometheus1"
      - "node-exporter1"
    # Additional mTLS organizations trusted by the monitoring endpoint.
    loadgen_monitoring_mtls_orgs:
      - name: "MonitoringOrg"
        domain: "monitoring.example.com"
    # Prometheus metrics port exposed by Loadgen.
    loadgen_metrics_port: 9443
    # gRPC control port exposed by Loadgen.
    loadgen_rpc_port: 7051
    # Render the config transaction block section.
    loadgen_generate_config_block: false
    # Render the namespace creation section. Set `true` when the workload should create namespace records before sending load.
    loadgen_generate_namespace: true
    # Render the transaction load section. Set `true` for a normal benchmark workload.
    loadgen_generate_load: true
    # Generated key size in bytes.
    loadgen_key_size: 32
    # Size in bytes of the per-transaction metadata. `0` means a nil value. Leave unset to omit the key and use the committer's own default.
    loadgen_tx_metadata_size: 150
    # Average number of reads per transaction whose committed version is fetched from the query service before the transaction is signed. `0`, the default, disables querying entirely and reads keep nil versions. Must not exceed the version-bearing reads (`loadgen_read_only_tx_keys` + `loadgen_read_write_tx_keys`), and is independent of `loadgen_key_backref_rate`. When greater than `0` the loadgen role emits a top-level `query-client` connection to the `query-service` committer hosts, so at least one must exist in the inventory. Each query is a synchronous round-trip that blocks its loadgen worker before the transaction is signed, so the achievable rate is bounded by `loadgen_workers` divided by the query latency, independent of the rate limit. Measured on a single-machine YugabyteDB deployment at roughly 99ms mean query latency. The default `loadgen_workers` of one per vCPU (32) capped throughput at about 315 tps however high the rate limit was set, while 512 workers reached about 4800 tps and querying disabled reached about 15000 tps. Raise `loadgen_workers` when enabling this, or lower the rate so only a fraction of transactions query.
    loadgen_queries_rate: 1
    # Average number of backward key references (reused keys) per transaction; the remaining transaction slots create fresh keys. This is what creates commit-time contention. `0`, the default, gives every slot a fresh unique key — no reuse and no contention. Below `1` it behaves as a probability, so `0.3` means roughly 30% of transactions carry a single reference. Must not exceed the total slot count.
    loadgen_key_backref_rate: 1
    # Controls how far back references point. A reference is drawn from the keys that existed this many transactions ago. `0`, the default, draws the newest keys, which may still be in flight, so conflicting transactions can land in the same block. Larger values draw older, committed keys. Irrelevant when `loadgen_key_backref_rate` is `0`.
    loadgen_tx_reference_gap: 100
    # How many keys a backward reference is spread over, counting back from the gap position. `0`, the default, steps straight back from the gap position, producing the most contention. Larger values spread references out and reduce contention. Irrelevant when `loadgen_key_backref_rate` is `0`.
    loadgen_key_lookback_window: 1024
    # Random seed used to build repeatable transaction streams.
    loadgen_tx_seed: 12345
    # Worker goroutine count used by the load profile.
    loadgen_workers: 16
    # Maximum generated block size.
    loadgen_block_max_size: 500
    # Minimum generated block size.
    loadgen_block_min_size: 1
    # Preferred block flush interval.
    loadgen_block_preferred_rate: "1s"
    # Enable read-only transactions in the load profile. Set `true` to include read-only query traffic.
    loadgen_generate_read_only_tx: true
    # Read-only key count per transaction.
    loadgen_read_only_tx_keys: 2
    # Enable write-only transactions in the load profile. Set `true` to include blind-write traffic.
    loadgen_generate_write_only_tx: true
    # Write-only key count per transaction.
    loadgen_write_only_tx_keys: 4
    # Write-only value size in bytes.
    loadgen_write_only_tx_val_size: 256
    # Enable read-write transactions in the load profile. Set `true` to include endorsement-style read-write traffic.
    loadgen_generate_read_write_tx: true
    # Read-write key count per transaction.
    loadgen_read_write_tx_keys: 2
    # Read-write value size in bytes.
    loadgen_read_write_tx_val_size: 128
    # Signature scheme used for generated identities.
    loadgen_key_scheme: "ECDSA"
    # Optional conflict injection block consumed by the load profile.
    loadgen_conflicts_settings:
      invalid_signatures: 1
      dependencies:
        - source: 1
          target: 2
    # Monitoring endpoint rate limit in requests per second.
    loadgen_monitoring_rate_limit_requests_per_second: 50
    # Monitoring endpoint rate limit burst size.
    loadgen_monitoring_rate_limit_burst: 100
    # Prefix used by the latency sampler.
    loadgen_latency_sampler_prefix: "loadgen_lg_1"
    # Portion of transactions sampled for latency tracking.
    loadgen_latency_sampler_portion: 0.01
    # Histogram distribution used for latency buckets. `uniform` spreads `loadgen_latency_buckets` equal width buckets over `loadgen_max_latency`; `fixed` uses the explicit bounds in `loadgen_latency_values`; `empty` turns latency tracking off. The load generator panics on startup for anything it does not recognize.
    loadgen_latency_distribution: "uniform"
    # Upper latency bound tracked by the histogram. Ignored when `loadgen_latency_distribution=fixed`.
    loadgen_max_latency: "5s"
    # Number of latency histogram buckets. Ignored when `loadgen_latency_distribution=fixed`.
    loadgen_latency_buckets: 1000
    # Latency histogram bucket upper bounds, in seconds. Required when `loadgen_latency_distribution=fixed` and ignored otherwise. Unlike `loadgen_latency_buckets`, the spacing need not be even, so a handful of bounds can cover both a healthy latency and an overloaded one. A latency above the last bound only reaches the overflow bucket, where it still counts towards the mean but no longer towards any quantile.
    loadgen_latency_values:
      - 0.005
      - 0.05
      - 0.5
      - 5
      - 30
    # Maximum generated transaction rate.
    loadgen_limit_rate: 2500
    # Batch size used by the stream pipeline.
    loadgen_stream_batches: 10
    # Channel buffer size used by the stream pipeline.
    loadgen_stream_buffers_size: 64
    # Log level specification.
    loadgen_log_level: info
    # Log message format template.
    loadgen_log_format: "%{color}%{time:2006-01-02 15:04:05.000 MST} [%{module}] %{shortfunc} -> %{level:.4s}%{color:reset} %{message}"
    # Local artifacts directory used for fetched TLS and MSP files.
    fetched_artifacts_dir: "/tmp/fabricx-artifacts"
    # Channel identifier rendered into generated transactions.
    channel_id: "fabricx-channel"
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Filename of the genesis config block placed in the Loadgen config directory.
    loadgen_config_block_file: config-block.pb.bin
    # Path on the control node of the genesis config block the load generator bootstraps from. Written by configtxgen when a real ordering service is deployed, and by `loadgen make-artifacts` when `use_mock_orderer` is set.
    genesis_config_block_path: "string"
    # Inventory-wide switch that selects the mock orderer topology for every role that takes part in it. Feeds `loadgen_use_mock_orderer`.
    use_mock_orderer: false
    # Run the workload against a mock orderer embedded in the load generator process instead of a real ordering service. The load generator cuts the blocks itself and serves them to the committer sidecar, which measures the committer without an ordering service in the path. It also generates the crypto material and the genesis config block, because the mock orderer signs the blocks it serves and the sidecar verifies them against the matching config block. Requires exactly one host in the `load_generators` group and no `fabric_x_orderers` group in the inventory.
    loadgen_use_mock_orderer: "{{ use_mock_orderer | default(false) }}"
    # Port the embedded mock orderer listens on when `use_mock_orderer` is set. Advertised in the generated config block, so the committer sidecar dials the load generator on this port.
    loadgen_mock_orderer_port: 7050
    # Number of cut blocks the embedded mock orderer buffers between the workload submitting them and the sidecar fetching them, which bounds the transactions in flight. The mock orderer's own default buffers tens of millions of transactions, so an overloaded committer is absorbed rather than felt: the submitted rate stays at the configured rate while only the committed rate shows the real drain rate, and end-to-end latency grows past anything the latency histogram can represent. Keep this a small multiple of `committer_sidecar_waiting_txs_limit` divided by the block size so saturation shows up as a drop in the submitted rate.
    loadgen_mock_orderer_out_block_capacity: 64
    # Base remote data directory that feeds `loadgen_remote_artifacts_dir`.
    remote_data_dir: "/var/hyperledger/fabricx/loadgen/lg-1/data"
    # Effective artifacts directory used inside rendered Loadgen configuration.
    loadgen_config_artifacts_dir: "{{ loadgen_remote_artifacts_dir if loadgen_use_bin else loadgen_container_artifacts_dir }}"
    # Directory on the load generator host holding the generated crypto material and genesis config block. Kept outside the config directory because that directory is mounted read-only in container mode.
    loadgen_remote_artifacts_dir: "{{ remote_data_dir }}/artifacts"
    # Artifacts mount path inside a container or pod.
    loadgen_container_artifacts_dir: /artifacts
    # Fault tolerance level of the ordering service rendered into the Loadgen config.
    loadgen_orderer_fault_tolerance_level: "BFT"
    # Maximum number of TX entries held in memory for latency tracking.
    loadgen_monitoring_latency_max_tracked_txs: 10000
    # Broadcast goroutine count used by the orderer client.
    loadgen_broadcast_parallelism: 8
    # Optional stopping limit for generated blocks.
    loadgen_limit_blocks: 100
    # Optional stopping limit for generated transactions.
    loadgen_limit_transactions: 100000
    # Enable mTLS for the monitoring endpoint.
    loadgen_monitoring_use_mtls: "{{ loadgen_use_mtls }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: config/transfer
```

### config/mtls/monitoring/transfer

> Transfer monitoring mTLS CA bundles

Transfer CA bundles trusted by the Loadgen monitoring endpoint. Copies client and organization CA files into the config tree so the metrics listener can verify Prometheus or other monitoring clients when mTLS is enabled.

```yaml
- name: Transfer monitoring mTLS CA bundles
  vars:
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Local artifacts directory used for fetched TLS and MSP files.
    fetched_artifacts_dir: "/tmp/fabricx-artifacts"
    # Additional mTLS client identities trusted by the monitoring endpoint.
    loadgen_monitoring_mtls_clients:
      - "prometheus1"
      - "node-exporter1"
    # Additional mTLS organizations trusted by the monitoring endpoint.
    loadgen_monitoring_mtls_orgs:
      - name: "MonitoringOrg"
        domain: "monitoring.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: config/mtls/monitoring/transfer
```

### config/rm

> Remove rendered configuration

Remove host-side rendered Loadgen configuration. Also removes the Kubernetes ConfigMap when Kubernetes deployment mode is enabled.

```yaml
- name: Remove rendered configuration
  vars:
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: config/rm
```

### crypto/setup

> Prepare crypto material

Prepare Loadgen MSP, user, and TLS material through the configured crypto source. Delegates to cryptogen transfer or Fabric CA enrollment, then publishes Kubernetes Secret material when Kubernetes mode is enabled.

```yaml
- name: Prepare crypto material
  vars:
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: crypto/setup
```

### crypto/cryptogen/transfer

> Transfer cryptogen artifacts

Transfer MSP, user, and TLS artifacts generated by cryptogen to the Loadgen host config directory. The copied paths are consumed by the rendered orderer client, sidecar client, server, and monitoring TLS sections.

```yaml
- name: Transfer cryptogen artifacts
  vars:
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Local cryptogen output directory.
    cryptogen_artifacts_dir: "/tmp/fabricx-crypto"
    # Crypto identity name used for MSP and TLS file names.
    loadgen_crypto_name: "{{ organization.peer.name | default(inventory_hostname) }}"
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: crypto/cryptogen/transfer
```

### crypto/fabric_ca/enroll

> Enroll identities with Fabric CA

Enroll Loadgen peer, user, and optional TLS identities against Fabric CA. Writes MSP and TLS artifacts under the remote config directory for use by the generated Loadgen config and later fetch or Kubernetes transfer tasks.

```yaml
- name: Enroll identities with Fabric CA
  vars:
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Local artifacts directory used for fetched TLS and MSP files.
    fetched_artifacts_dir: "/tmp/fabricx-artifacts"
    # Crypto identity name used for MSP and TLS file names.
    loadgen_crypto_name: "{{ organization.peer.name | default(inventory_hostname) }}"
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Real machine host.
    actual_host: "myvpc.cloud.ibm.com"
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
    # Specifies the OpenShift Route host.
    loadgen_openshift_metrics_route: "loadgen-metrics.apps.example.com"
    # Specifies the OpenShift Route host.
    loadgen_openshift_rpc_route: "loadgen-rpc.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: crypto/fabric_ca/enroll
```

### crypto/fetch

> Fetch generated certificates

Fetch generated Loadgen MSP signcerts and TLS certificates back to the control node. Stores artifacts under the fetched artifacts directory so other roles can trust Loadgen endpoints or reuse generated crypto outputs.

```yaml
- name: Fetch generated certificates
  vars:
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Local artifacts directory used for fetched TLS and MSP files.
    fetched_artifacts_dir: "/tmp/fabricx-artifacts"
    # Crypto identity name used for MSP and TLS file names.
    loadgen_crypto_name: "{{ organization.peer.name | default(inventory_hostname) }}"
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: crypto/fetch
```

### crypto/rm

> Remove crypto material

Remove Loadgen MSP, user, and TLS artifacts from the host config directory. Also removes the Kubernetes Secret when Kubernetes deployment mode is enabled.

```yaml
- name: Remove crypto material
  vars:
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: crypto/rm
```

### bin/build

> Build the load generator binary

Build the `loadgen` binary from the configured Fabric-X source repository. Uses the shared binary helper role and the configured Git host, repository, revision, and Go package path.

```yaml
- name: Build the load generator binary
  vars:
    # Binary name used by the shared bin role.
    loadgen_bin_name: loadgen
    # Environment variables exported for the load generator process in host-binary mode. Ignored in container, Kubernetes, and OpenShift modes. Set per host to tune the generator alone. The Go garbage collector is the usual reason: ECDSA signing allocates about 6 KB and 59 objects per signature against Ed25519's 184 B and 4, so a generator forced onto ECDSA by a namespace policy is limited by collection rather than by cryptography, and raising `GOGC` recovers most of the difference.
    loadgen_bin_env: {}
    # Git host used for binary builds.
    loadgen_git_hub_url: github.com
    # Git repository that provides the Loadgen source.
    loadgen_git_repo: hyperledger/fabric-x-committer
    # Git revision used for binary builds and installs.
    loadgen_git_commit: v1.0.4
    # Go package path for the Loadgen binary.
    loadgen_source_code_package: cmd/loadgen
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: bin/build
```

### bin/install

> Install the load generator binary

Install the `loadgen` binary through the shared binary helper role. Consumes the configured Go package and source revision so binary deployments can start the local process.

```yaml
- name: Install the load generator binary
  vars:
    # Binary name used by the shared bin role.
    loadgen_bin_name: loadgen
    # Environment variables exported for the load generator process in host-binary mode. Ignored in container, Kubernetes, and OpenShift modes. Set per host to tune the generator alone. The Go garbage collector is the usual reason: ECDSA signing allocates about 6 KB and 59 objects per signature against Ed25519's 184 B and 4, so a generator forced onto ECDSA by a namespace policy is limited by collection rather than by cryptography, and raising `GOGC` recovers most of the difference.
    loadgen_bin_env: {}
    # Go package used for binary installation.
    loadgen_bin_package: "{{ loadgen_git_hub_url }}/{{ loadgen_git_repo }}/{{ loadgen_source_code_package }}"
    # Git host used for binary builds.
    loadgen_git_hub_url: github.com
    # Git repository that provides the Loadgen source.
    loadgen_git_repo: hyperledger/fabric-x-committer
    # Git revision used for binary builds and installs.
    loadgen_git_commit: v1.0.4
    # Go package path for the Loadgen binary.
    loadgen_source_code_package: cmd/loadgen
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: bin/install
```

### bin/rm

> Remove the load generator binary

Remove the installed `loadgen` binary managed by the shared binary helper role. Does not remove generated configuration, crypto material, or runtime logs.

```yaml
- name: Remove the load generator binary
  vars:
    # Binary name used by the shared bin role.
    loadgen_bin_name: loadgen
    # Environment variables exported for the load generator process in host-binary mode. Ignored in container, Kubernetes, and OpenShift modes. Set per host to tune the generator alone. The Go garbage collector is the usual reason: ECDSA signing allocates about 6 KB and 59 objects per signature against Ed25519's 184 B and 4, so a generator forced onto ECDSA by a namespace policy is limited by collection rather than by cryptography, and raising `GOGC` recovers most of the difference.
    loadgen_bin_env: {}
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: bin/rm
```

### make_artifacts

> Generate the crypto material and genesis config block

Render the minimal artifacts config and run `loadgen make-artifacts` on the load generator host, then fetch the generated genesis config block to the control node for the committer sidecar. Only meaningful when `use_mock_orderer` is set. With a real ordering service, configtxgen and Armageddon build the config block instead. The artifacts directory is removed and regenerated whenever the rendered config changes, because `make-artifacts` otherwise reuses an existing config block and would keep advertising a stale orderer address.

```yaml
- name: Generate the crypto material and genesis config block
  vars:
    # Run the binary runtime.
    loadgen_use_bin: false
    # Run the container runtime.
    loadgen_use_container: "{{ (not loadgen_use_bin) and (not loadgen_use_k8s) and (not loadgen_use_openshift) }}"
    # Use Kubernetes resources.
    loadgen_use_k8s: false
    # Selects the OpenShift deployment branch.
    loadgen_use_openshift: false
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Base remote data directory that feeds `loadgen_remote_artifacts_dir`.
    remote_data_dir: "/var/hyperledger/fabricx/loadgen/lg-1/data"
    # Filename of the minimal rendered config that drives `loadgen make-artifacts`.
    loadgen_artifacts_config_file: config-loadgen-artifacts.yaml
    # Effective artifacts directory used inside rendered Loadgen configuration.
    loadgen_config_artifacts_dir: "{{ loadgen_remote_artifacts_dir if loadgen_use_bin else loadgen_container_artifacts_dir }}"
    # Directory on the load generator host holding the generated crypto material and genesis config block. Kept outside the config directory because that directory is mounted read-only in container mode.
    loadgen_remote_artifacts_dir: "{{ remote_data_dir }}/artifacts"
    # Artifacts mount path inside a container or pod.
    loadgen_container_artifacts_dir: /artifacts
    # Directory on the control node the generated genesis config block is fetched into.
    loadgen_artifacts_dir: "string"
    # Filename of the genesis config block placed in the Loadgen config directory.
    loadgen_config_block_file: config-block.pb.bin
    # Port the embedded mock orderer listens on when `use_mock_orderer` is set. Advertised in the generated config block, so the committer sidecar dials the load generator on this port.
    loadgen_mock_orderer_port: 7050
    # Number of peer organizations generated into the artifacts when `use_mock_orderer` is set. Their MSP identities endorse the namespace-creation transactions, and the generated config block requires all of them to sign, so raising this adds a signature per namespace transaction.
    loadgen_peer_organization_count: 1
    # Channel identifier rendered into generated transactions.
    channel_id: "fabricx-channel"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: make_artifacts
```

### bin/make_artifacts

> Generate the artifacts with the binary runtime

Invoke `loadgen make-artifacts --config=...` as a local binary process.

```yaml
- name: Generate the artifacts with the binary runtime
  vars:
    # Binary name used by the shared bin role.
    loadgen_bin_name: loadgen
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Filename of the minimal rendered config that drives `loadgen make-artifacts`.
    loadgen_artifacts_config_file: config-loadgen-artifacts.yaml
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: bin/make_artifacts
```

### container/make_artifacts

> Generate the artifacts with the container runtime

Invoke `loadgen make-artifacts --config=...` in a short-lived container. The config directory is mounted read-only, as it is for the running load generator, so the artifacts get a separate writable mount.

```yaml
- name: Generate the artifacts with the container runtime
  vars:
    # Container name used by the runtime.
    loadgen_container_name: "{{ inventory_hostname }}"
    # Loadgen container image.
    loadgen_image: "{{ loadgen_registry_endpoint }}/{{ loadgen_image_name }}:{{ loadgen_image_tag }}"
    # Image name used by the Loadgen container.
    loadgen_image_name: fabric-x-loadgen
    # Image tag used by the Loadgen container.
    loadgen_image_tag: 1.0.4
    # Image registry endpoint.
    loadgen_registry_endpoint: "{{ lookup('env', 'LOADGEN_REGISTRY_ENDPOINT') or 'docker.io/hyperledger' }}"
    # Config mount path inside a container or pod.
    loadgen_container_config_dir: /config
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Base remote data directory that feeds `loadgen_remote_artifacts_dir`.
    remote_data_dir: "/var/hyperledger/fabricx/loadgen/lg-1/data"
    # Filename of the minimal rendered config that drives `loadgen make-artifacts`.
    loadgen_artifacts_config_file: config-loadgen-artifacts.yaml
    # Artifacts mount path inside a container or pod.
    loadgen_container_artifacts_dir: /artifacts
    # Directory on the load generator host holding the generated crypto material and genesis config block. Kept outside the config directory because that directory is mounted read-only in container mode.
    loadgen_remote_artifacts_dir: "{{ remote_data_dir }}/artifacts"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: container/make_artifacts
```

### bin/start

> Start the binary runtime

Start Loadgen as a local binary process using the rendered config file. Waits on the monitoring port after invoking `loadgen start --config=...`.

```yaml
- name: Start the binary runtime
  vars:
    # Binary name used by the shared bin role.
    loadgen_bin_name: loadgen
    # Environment variables exported for the load generator process in host-binary mode. Ignored in container, Kubernetes, and OpenShift modes. Set per host to tune the generator alone. The Go garbage collector is the usual reason: ECDSA signing allocates about 6 KB and 59 objects per signature against Ed25519's 184 B and 4, so a generator forced onto ECDSA by a namespace policy is limited by collection rather than by cryptography, and raising `GOGC` recovers most of the difference.
    loadgen_bin_env: {}
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Rendered Loadgen config filename.
    loadgen_config_file: config-loadgen.yaml
    # Prometheus metrics port exposed by Loadgen.
    loadgen_metrics_port: 9443
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: bin/start
```

### bin/stop

> Stop the binary runtime

Stop the local Loadgen binary process managed by the shared binary helper role. Leaves the remote config directory and logs available for inspection or collection.

```yaml
- name: Stop the binary runtime
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: bin/stop
```

### bin/transfer

> Transfer the load generator binary

Transfer a prebuilt `loadgen` binary through the shared binary helper role. Used by binary deployments when the executable is built elsewhere and then staged onto the target host.

```yaml
- name: Transfer the load generator binary
  vars:
    # Binary name used by the shared bin role.
    loadgen_bin_name: loadgen
    # Environment variables exported for the load generator process in host-binary mode. Ignored in container, Kubernetes, and OpenShift modes. Set per host to tune the generator alone. The Go garbage collector is the usual reason: ECDSA signing allocates about 6 KB and 59 objects per signature against Ed25519's 184 B and 4, so a generator forced onto ECDSA by a namespace policy is limited by collection rather than by cryptography, and raising `GOGC` recovers most of the difference.
    loadgen_bin_env: {}
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: bin/transfer
```

### bin/fetch_logs

> Fetch binary logs

Collect logs emitted by a binary-based Loadgen runtime. Fetches process logs without changing the running state or removing generated artifacts.

```yaml
- name: Fetch binary logs
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: bin/fetch_logs
```

### container/start

> Start the container runtime

Start Loadgen as a local container with the rendered config directory mounted read-only. Exposes the Prometheus metrics and gRPC ports and waits for the monitoring port to become reachable.

```yaml
- name: Start the container runtime
  vars:
    # Container name used by the runtime.
    loadgen_container_name: "{{ inventory_hostname }}"
    # Loadgen container image.
    loadgen_image: "{{ loadgen_registry_endpoint }}/{{ loadgen_image_name }}:{{ loadgen_image_tag }}"
    # Image registry endpoint.
    loadgen_registry_endpoint: "{{ lookup('env', 'LOADGEN_REGISTRY_ENDPOINT') or 'docker.io/hyperledger' }}"
    # Image name used by the Loadgen container.
    loadgen_image_name: fabric-x-loadgen
    # Image tag used by the Loadgen container.
    loadgen_image_tag: 1.0.4
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Config mount path inside a container or pod.
    loadgen_container_config_dir: /config
    # Rendered Loadgen config filename.
    loadgen_config_file: config-loadgen.yaml
    # Prometheus metrics port exposed by Loadgen.
    loadgen_metrics_port: 9443
    # gRPC control port exposed by Loadgen.
    loadgen_rpc_port: 7051
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: container/start
```

### container/stop

> Stop the container runtime

Stop the local Loadgen container. Preserves the container definition, image reference, mounted configuration, crypto material, and logs for later cleanup or collection.

```yaml
- name: Stop the container runtime
  vars:
    # Container name used by the runtime.
    loadgen_container_name: "{{ inventory_hostname }}"
    # Loadgen container image.
    loadgen_image: "{{ loadgen_registry_endpoint }}/{{ loadgen_image_name }}:{{ loadgen_image_tag }}"
    # Image name used by the Loadgen container.
    loadgen_image_name: fabric-x-loadgen
    # Image tag used by the Loadgen container.
    loadgen_image_tag: 1.0.4
    # Image registry endpoint.
    loadgen_registry_endpoint: "{{ lookup('env', 'LOADGEN_REGISTRY_ENDPOINT') or 'docker.io/hyperledger' }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: container/stop
```

### container/rm

> Remove the container runtime

Remove the local Loadgen container runtime resources. Leaves host-side generated configuration and crypto material under the remote config directory.

```yaml
- name: Remove the container runtime
  vars:
    # Container name used by the runtime.
    loadgen_container_name: "{{ inventory_hostname }}"
    # Loadgen container image.
    loadgen_image: "{{ loadgen_registry_endpoint }}/{{ loadgen_image_name }}:{{ loadgen_image_tag }}"
    # Image name used by the Loadgen container.
    loadgen_image_name: fabric-x-loadgen
    # Image tag used by the Loadgen container.
    loadgen_image_tag: 1.0.4
    # Image registry endpoint.
    loadgen_registry_endpoint: "{{ lookup('env', 'LOADGEN_REGISTRY_ENDPOINT') or 'docker.io/hyperledger' }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: container/rm
```

### container/fetch_logs

> Fetch container logs

Collect logs from a containerized Loadgen runtime. Reads container logs for the configured container name without modifying runtime or config state.

```yaml
- name: Fetch container logs
  vars:
    # Container name used by the runtime.
    loadgen_container_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: container/fetch_logs
```

### k8s/start

> Start the Kubernetes deployment

Create or update Kubernetes resources for Loadgen. Ensures the namespace exists, applies the Service, optional NodePort and LoadBalancer Services, and Deployment, and mounts generated ConfigMap and Secret artifacts into the pod.

```yaml
- name: Start the Kubernetes deployment
  vars:
    # Committer inventory hosts used by the dispatcher to derive client targets.
    committer_hosts:
      - "committer-sidecar1"
      - "committer-validator1"
    # Orderer inventory hosts used by the dispatcher to derive client targets.
    orderer_hosts:
      - "orderer-router1"
      - "orderer-assembler1"
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Crypto identity name used for MSP and TLS file names.
    loadgen_crypto_name: "{{ organization.peer.name | default(inventory_hostname) }}"
    # Loadgen container image.
    loadgen_image: "{{ loadgen_registry_endpoint }}/{{ loadgen_image_name }}:{{ loadgen_image_tag }}"
    # Image registry endpoint.
    loadgen_registry_endpoint: "{{ lookup('env', 'LOADGEN_REGISTRY_ENDPOINT') or 'docker.io/hyperledger' }}"
    # Image name used by the Loadgen container.
    loadgen_image_name: fabric-x-loadgen
    # Image tag used by the Loadgen container.
    loadgen_image_tag: 1.0.4
    # Config mount path inside a container or pod.
    loadgen_container_config_dir: /config
    # Rendered Loadgen config filename.
    loadgen_config_file: config-loadgen.yaml
    # Deployment rollout wait timeout in seconds.
    loadgen_k8s_wait_timeout: 120
    # Pod FSGroup used for mounted config and secrets.
    loadgen_k8s_fs_group: 10001
    # Kubernetes namespace used for loadgen resources.
    k8s_namespace: "fabricx-loadgen"
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to Loadgen resources.
    loadgen_k8s_part_of: "fabric-x-loadgen-{{ organization.name }}"
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
    # Enable mTLS for the main endpoint.
    loadgen_use_mtls: false
    # Enable mTLS for the monitoring endpoint.
    loadgen_monitoring_use_mtls: "{{ loadgen_use_mtls }}"
    # Additional mTLS client identities trusted by the main endpoint.
    loadgen_mtls_clients:
      - "orderer-router1"
      - "committer-sidecar1"
    # Additional mTLS organizations trusted by the main endpoint.
    loadgen_mtls_orgs:
      - name: "Org1"
        domain: "org1.example.com"
      - name: "OrdererOrg1"
        domain: "ordererorg1.example.com"
    # Additional mTLS client identities trusted by the monitoring endpoint.
    loadgen_monitoring_mtls_clients:
      - "prometheus1"
      - "node-exporter1"
    # Additional mTLS organizations trusted by the monitoring endpoint.
    loadgen_monitoring_mtls_orgs:
      - name: "MonitoringOrg"
        domain: "monitoring.example.com"
    # Optional image pull secret used by Kubernetes workloads.
    k8s_image_pull_secret: "fabricx-registry-pull"
    # Prometheus metrics port exposed by Loadgen.
    loadgen_metrics_port: 9443
    # gRPC control port exposed by Loadgen.
    loadgen_rpc_port: 7051
    # Filename of the genesis config block placed in the Loadgen config directory.
    loadgen_config_block_file: config-block.pb.bin
    # Kubernetes NodePort value used by the external metrics Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    loadgen_k8s_metrics_node_port: 30090
    # Kubernetes NodePort value used by the external gRPC control Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    loadgen_k8s_rpc_node_port: 30051
    # Set to `true` to create a LoadBalancer Service entry that exposes the metrics port externally. When undefined or `false`, the metrics port is not included in the LoadBalancer Service.
    loadgen_k8s_loadbalancer_expose_metrics_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the gRPC control port externally. When undefined or `false`, the gRPC control port is not included in the LoadBalancer Service.
    loadgen_k8s_loadbalancer_expose_rpc_port: false
    # Optional Kubernetes container resource requests and limits.
    k8s_resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: k8s/start
```

### k8s/ping

> Check that the Loadgen Kubernetes service is reachable

Probes configured Kubernetes NodePort values and LoadBalancer-exposed service ports for external reachability.

```yaml
- name: Check that the Loadgen Kubernetes service is reachable
  vars:
    # Prometheus metrics port exposed by Loadgen.
    loadgen_metrics_port: 9443
    # gRPC control port exposed by Loadgen.
    loadgen_rpc_port: 7051
    # Kubernetes NodePort value used by the external metrics Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    loadgen_k8s_metrics_node_port: 30090
    # Kubernetes NodePort value used by the external gRPC control Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    loadgen_k8s_rpc_node_port: 30051
    # Set to `true` to create a LoadBalancer Service entry that exposes the metrics port externally. When undefined or `false`, the metrics port is not included in the LoadBalancer Service.
    loadgen_k8s_loadbalancer_expose_metrics_port: false
    # Set to `true` to create a LoadBalancer Service entry that exposes the gRPC control port externally. When undefined or `false`, the gRPC control port is not included in the LoadBalancer Service.
    loadgen_k8s_loadbalancer_expose_rpc_port: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: k8s/ping
```

### k8s/rm

> Remove Kubernetes resources

Remove the Kubernetes Deployment and Services created for Loadgen. Does not remove the ConfigMap or Secret; use the Kubernetes config and crypto remove entry points for those generated artifacts.

```yaml
- name: Remove Kubernetes resources
  vars:
    # Kubernetes namespace used for loadgen resources.
    k8s_namespace: "fabricx-loadgen"
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
    # Kubernetes NodePort value used by the external metrics Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    loadgen_k8s_metrics_node_port: 30090
    # Set to `true` to create a LoadBalancer Service entry that exposes the metrics port externally. When undefined or `false`, the metrics port is not included in the LoadBalancer Service.
    loadgen_k8s_loadbalancer_expose_metrics_port: false
    # Kubernetes NodePort value used by the external gRPC control Service port. Defining this variable enables the NodePort Service; the value is set as the static `nodePort` in the Service spec.
    loadgen_k8s_rpc_node_port: 30051
    # Set to `true` to create a LoadBalancer Service entry that exposes the gRPC control port externally. When undefined or `false`, the gRPC control port is not included in the LoadBalancer Service.
    loadgen_k8s_loadbalancer_expose_rpc_port: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: k8s/rm
```

### k8s/fetch_logs

> Fetch pod logs

Collect logs from the Kubernetes pod running Loadgen. Uses the configured Kubernetes resource name to fetch pod output without changing workload state.

```yaml
- name: Fetch pod logs
  vars:
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: k8s/fetch_logs
```

### k8s/config/transfer

> Publish the Kubernetes ConfigMap

Publish the rendered Loadgen configuration and trusted CA bundles as a Kubernetes ConfigMap. The ConfigMap is consumed by the Loadgen Deployment and includes orderer, sidecar, TLS, mTLS, workload, stream, and logging config content.

```yaml
- name: Publish the Kubernetes ConfigMap
  vars:
    # Kubernetes namespace used for loadgen resources.
    k8s_namespace: "fabricx-loadgen"
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to Loadgen resources.
    loadgen_k8s_part_of: "fabric-x-loadgen-{{ organization.name }}"
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Rendered Loadgen config filename.
    loadgen_config_file: config-loadgen.yaml
    # Filename of the genesis config block placed in the Loadgen config directory.
    loadgen_config_block_file: config-block.pb.bin
    # Sidecar host targeted by the orderer and sidecar clients.
    committer_sidecar_host: "committer-sidecar1"
    # Enable mTLS for the main endpoint.
    loadgen_use_mtls: false
    # Enable mTLS for the monitoring endpoint.
    loadgen_monitoring_use_mtls: "{{ loadgen_use_mtls }}"
    # Additional mTLS client identities trusted by the main endpoint.
    loadgen_mtls_clients:
      - "orderer-router1"
      - "committer-sidecar1"
    # Additional mTLS organizations trusted by the main endpoint.
    loadgen_mtls_orgs:
      - name: "Org1"
        domain: "org1.example.com"
      - name: "OrdererOrg1"
        domain: "ordererorg1.example.com"
    # Additional mTLS client identities trusted by the monitoring endpoint.
    loadgen_monitoring_mtls_clients:
      - "prometheus1"
      - "node-exporter1"
    # Additional mTLS organizations trusted by the monitoring endpoint.
    loadgen_monitoring_mtls_orgs:
      - name: "MonitoringOrg"
        domain: "monitoring.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: k8s/config/transfer
```

### k8s/config/rm

> Remove the Kubernetes ConfigMap

Remove the Kubernetes ConfigMap created for Loadgen configuration. Leaves host-side rendered config files intact for inspection, regeneration, or non-Kubernetes deployments.

```yaml
- name: Remove the Kubernetes ConfigMap
  vars:
    # Kubernetes namespace used for loadgen resources.
    k8s_namespace: "fabricx-loadgen"
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: k8s/config/rm
```

### k8s/crypto/transfer

> Publish the Kubernetes Secret

Publish Loadgen MSP and TLS material as a Kubernetes Secret. The Secret is consumed by the Loadgen Deployment and is built from fetched or remote crypto artifacts for the selected identity.

```yaml
- name: Publish the Kubernetes Secret
  vars:
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Base remote config directory that feeds `loadgen_remote_config_dir`.
    remote_config_dir: "/var/hyperledger/fabricx/loadgen/lg-1/config"
    # Remote config directory used by Loadgen.
    loadgen_remote_config_dir: "{{ remote_config_dir }}"
    # Crypto identity name used for MSP and TLS file names.
    loadgen_crypto_name: "{{ organization.peer.name | default(inventory_hostname) }}"
    # Kubernetes namespace used for loadgen resources.
    k8s_namespace: "fabricx-loadgen"
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to Loadgen resources.
    loadgen_k8s_part_of: "fabric-x-loadgen-{{ organization.name }}"
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: k8s/crypto/transfer
```

### k8s/crypto/rm

> Remove the Kubernetes Secret

Remove the Kubernetes Secret created for Loadgen MSP and TLS material. Leaves host-side crypto artifacts and fetched local artifacts untouched.

```yaml
- name: Remove the Kubernetes Secret
  vars:
    # Kubernetes namespace used for loadgen resources.
    k8s_namespace: "fabricx-loadgen"
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: k8s/crypto/rm
```

### openshift/start

> Start the OpenShift deployment

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Start the OpenShift deployment
  vars:
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
    # Value for the Kubernetes `app.kubernetes.io/part-of` label applied to Loadgen resources.
    loadgen_k8s_part_of: "fabric-x-loadgen-{{ organization.name }}"
    # Organization definition consumed by crypto, config, and Kubernetes templates.
    organization:
      name: "Org1"
      domain: "org1.example.com"
      role: "peer"
      fabric_ca_host: "fca-org1"
      peer:
        name: "orderer-loadgen"
        secret: "orderer-loadgenPWD"
      users:
        - name: "orderer-loadgen"
          secret: "orderer-loadgenPWD"
      namespaces:
        - id: 0
          policy: "threshold"
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
    # Specifies the OpenShift Route host.
    loadgen_openshift_metrics_route: "loadgen-metrics.apps.example.com"
    # Specifies the OpenShift Route host.
    loadgen_openshift_rpc_route: "loadgen-rpc.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: openshift/start
```

### openshift/ping

> Check the OpenShift deployment

Checks configured OpenShift Routes and reuses the Kubernetes service ping flow.

```yaml
- name: Check the OpenShift deployment
  vars:
    # Enable TLS for the main endpoint.
    loadgen_use_tls: false
    # Enable TLS for the monitoring endpoint.
    loadgen_monitoring_use_tls: "{{ loadgen_use_tls }}"
    # Specifies the OpenShift Route host.
    loadgen_openshift_metrics_route: "loadgen-metrics.apps.example.com"
    # Specifies the OpenShift Route host.
    loadgen_openshift_rpc_route: "loadgen-rpc.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: openshift/ping
```

### openshift/rm

> Remove the OpenShift deployment

Reuses the Kubernetes workload flow and manages OpenShift Routes for configured HTTP-capable ports.

```yaml
- name: Remove the OpenShift deployment
  vars:
    # Kubernetes resource name used for the Deployment, Service, Secret, and optional NodePort Service.
    loadgen_k8s_resource_name: "{{ inventory_hostname }}"
    # Specifies the OpenShift Route host.
    loadgen_openshift_metrics_route: "loadgen-metrics.apps.example.com"
    # Specifies the OpenShift Route host.
    loadgen_openshift_rpc_route: "loadgen-rpc.apps.example.com"
  ansible.builtin.include_role:
    name: hyperledger.fabricx.loadgen
    tasks_from: openshift/rm
```
