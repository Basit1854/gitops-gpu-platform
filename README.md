# gitops-gpu-platform

A GPU cloud platform on Kubernetes, built end to end and fully reproducible from code. Terraform provisions the infrastructure, Argo CD delivers everything through Git, and the platform serves models with vLLM and runs multi-node training through Slurm. Every failure mode encountered along the way is documented in the runbook.

## Stack

| Layer | Tools |
|---|---|
| Infrastructure | Terraform, managed Kubernetes |
| Delivery | Argo CD, Helm, Kustomize |
| GPU | NVIDIA GPU Operator, DCGM, MIG |
| Networking | CNI, Gateway API, NCCL, RDMA |
| Workloads | vLLM inference, Slurm with Soperator for training |
| Observability | Prometheus, Grafana, OpenTelemetry |

## Roadmap

- [x] Project setup and tooling
- [ ] Container runtime internals: namespaces, cgroups, containerd
- [ ] Kubernetes core: scheduling, workloads, resource management
- [ ] Networking and storage: CNI, Ingress, NetworkPolicy, CSI, StatefulSets
- [ ] Infrastructure as code and observability: Terraform, Prometheus, Grafana
- [ ] GitOps delivery: Helm, Kustomize, Argo CD app of apps
- [ ] GPU enablement: driver chain, GPU Operator, DCGM monitoring
- [ ] Multi-node GPU: NVLink, InfiniBand, RDMA, NCCL benchmarks
- [ ] Inference serving: vLLM with autoscaling on queue depth
- [ ] Distributed training: Slurm on Kubernetes with Soperator
- [ ] Performance engineering: topology, NCCL tuning, storage bottlenecks
- [ ] Cluster operations: etcd backup and restore, upgrades, recovery
- [ ] Multi-cluster GitOps and SRE: secrets, alerts, SLOs, incident response
- [ ] Automation: diagnostics tooling, CI pipeline, architecture walkthrough

## Runbook

See [runbook/](runbook/). Each entry covers the symptom, first commands, cause, fix and prevention.
