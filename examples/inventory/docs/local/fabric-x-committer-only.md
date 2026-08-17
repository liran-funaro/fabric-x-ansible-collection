# local/fabric-x-committer-only.yaml

[`fabric-x-committer-only.yaml`](../../local/fabric-x-committer-only.yaml) runs the committer against YugabyteDB with no ordering service at all. The load generator embeds a mock orderer, cuts the blocks itself, and serves them to the committer sidecar.

Use it to measure the committer in isolation, when a number should describe the committer rather than the orderer in front of it. It is also the quickest full pipeline to bring up, because it drops four orderer organizations, their Fabric CAs, and sixteen orderer processes.

## Table of Contents <!-- omit in toc -->

- [Pipeline](#pipeline)
- [Inventory Details](#inventory-details)
- [Where the Genesis Block Comes From](#where-the-genesis-block-comes-from)
- [Differences From the Other Local Inventories](#differences-from-the-other-local-inventories)
- [Shaping the Workload](#shaping-the-workload)

## Pipeline

```text
loadgen (mock orderer) -> sidecar -> coordinator -> verifier -> validator-committer -> YugabyteDB
```

The load generator holds both ends of the measurement: it generates and orders the transactions, and it receives their statuses back from the sidecar's block delivery stream. Everything between those two points is the committer.

## Inventory Details

All long-running services run as host binaries, so `tmux` has to be installed on the target machines. Only the Fabric CA database runs as a container.

This inventory deploys these logical services on the local machine:

- 1 Fabric CA server and 1 PostgreSQL database for Fabric CA state, for the single peer organization.
- 1 committer with validator, verifier, coordinator, and sidecar. No query service, since the workload does not query committed versions.
- 1 YugabyteDB master and 1 YugabyteDB tablet in cluster `1`.
- 1 load generator, which also serves the mock orderer on port `7050`.
- Monitoring with node exporter, cAdvisor, Prometheus, Grafana, Loki, and Alloy.

There are no orderer organizations, no orderer Fabric CAs, and no Block Explorer.

## Where the Genesis Block Comes From

With a real ordering service, Armageddon and configtxgen build the genesis config block from the orderer topology. There is no orderer topology here, so `loadgen make-artifacts` generates the crypto material and the config block instead, during the genesis block phase of `make setup`. The block is then fetched to the control node and handed to the sidecar.

Both halves have to come from that one generation step. The mock orderer signs the blocks it serves with the consenter identities generated next to the config block, and the sidecar verifies those signatures against the orderer organization carried inside it. A config block from any other source makes the sidecar reject every block.

For the same reason the sidecar reaches the mock orderer over plaintext. The sidecar resolves the CA certificates for its orderer connection from the config block, and the orderer TLS certificate generated with the artifacts does not carry the load generator's host among its subject alternative names, so server verification could not succeed. Every other connection in the deployment still uses TLS and mTLS.

`make init` does nothing here. It runs fxconfig, which submits the namespace-creation transactions through an ordering service, and the only one is inside the load generator. The load generator creates the namespaces itself instead, which is why `loadgen_generate_namespace` is set.

## The Load Generator Cannot Be Restarted On Its Own

The mock orderer keeps its chain in memory. Restart the load generator and the orderer starts again from an empty chain, while the sidecar's ledger and the state database are still where the previous run left them. The sidecar asks for the block after its current height, and the fresh orderer will never produce a block with that number, so block delivery stops: the ledger height stays put, the load generator keeps generating into its own orderer, and no transaction commits again.

The loadgen-side counters mislead in the same situation for a second reason. A restarted load generator subscribes to the sidecar's delivery stream from block 0, so it counts every transaction already in the ledger as committed. `loadgen_transaction_committed_total` then reports the whole history rather than the current run, which can be orders of magnitude above `loadgen_transaction_sent_total`.

So restarting the load generator means restarting the run:

```shell
make teardown && make setup && make start
```

To change the rate on a running deployment, use `make limit-rate LIMIT=<tps>` rather than restarting. Only a new binary or a changed config needs the full cycle, and that cycle has to clear the sidecar ledger and the state database with it.

## Differences From the Other Local Inventories

| | Other local inventories | This inventory |
| --- | --- | --- |
| Ordering service | 4 orderer groups, 16 processes | mock orderer inside the load generator |
| Genesis config block | Armageddon and configtxgen | `loadgen make-artifacts` |
| Sidecar to orderer | TLS or mTLS | plaintext |
| Namespace creation | `make init` (fxconfig) | the load generator's namespaces phase |
| Load generators | any number | exactly one |

Only one load generator is allowed. Each one would generate its own crypto material and cut its own chain, and the sidecar can only verify blocks against the single config block it bootstrapped with. Raise `loadgen_workers` if one load generator cannot saturate the committer.

## Shaping the Workload

Out of the box every transaction slot gets a fresh unique key. Nothing is ever read back, so no transaction conflicts with another and every one should commit. That measures the pipeline's ceiling, at the cost of a state database and a ledger that grow with every transaction, which is worth watching on a long run.

To measure something else:

- `loadgen_key_backref_rate`, with `loadgen_key_lookback_window`, makes transactions reference earlier keys and so introduces commit-time conflicts.
- `loadgen_queries_rate` makes reads carry committed versions fetched from a query service, which this inventory does not deploy; add a `query-service` committer host first.
