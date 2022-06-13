# Test Cluster Peering Service Mesh
This uses mesh gateways for the dataplane between two peered clusters. TODO: Add
intentions and un-peering.

1. Create 2 Kubernetes clusters with a flat network. For example the terraform
   [here](https://github.com/hashicorp/consul-k8s/tree/main/charts/consul/test/terraform/eks)
   has VPC peered clusters.
2. Use `peer1-consul-values.yml` to install the Helm chart on the first cluster,
   and `peer2-consul-values.yml` on the second cluster. Use the Helm chart from the
   [peering branch](https://github.com/hashicorp/consul-k8s/tree/peering) of
   consul-k8s (This will be
   merged into main and released soon).
3. Apply acceptor.yml to the first cluster:
```
k apply -f acceptor.yml
```
4. This will cause Consul to generate a peering token called `api-token`. Get
   that token.
```
k get secret api-token -oyaml > api-token.yml
```
5. Edit `api-token.yml` to remove the ownerreference. (All the lines UNDER the key
   ownderReferences). This is a temporary step.
6. Apply the token to the second cluster:
```
k apply -f api-token.yml
```
7. Apply the dialer resource, backend application, and exported service CR:
```
k apply -f dialer.yml; k apply -f backend.yml; k apply -f exported-svc.yml
```
8. Go back to the first cluster and confirm that the peering connection is up by
   checking if the endpoints of backend deployed in peer api show up on the
   first cluster. (Need to exec onto a consul server):
```
curl "localhost:8500/v1/health/connect/backend?peer=api&pretty"
```
9. Deploy frontend, which has an annotation describing it's explicit upstream
   backend:
```
k apply -f frontend.yml
```
10. Exec onto frontend, and try to reach backend. The local listener for the
    upstream is `localhost:1234`:
```
curl localhost:1234
```
