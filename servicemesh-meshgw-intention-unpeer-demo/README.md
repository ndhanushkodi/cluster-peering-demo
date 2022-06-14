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
   merged into main and released soon). The images being used are currently built off of
   `main` for hashicorp/consul and `peering` for hashicorp/consul-k8s.
3. Apply acceptor.yml to the first cluster:
```
k apply -f acceptor.yml
```
4. This will cause Consul to generate a peering token called `api-token`. Get
   that token.
```
k get secret api-token -oyaml > api-token.yml
```
5. Apply the token to the second cluster:
```
k apply -f api-token.yml
```
6. Apply the dialer resource, backend application, and exported service CR:
```
k apply -f dialer.yml; k apply -f backend.yml; k apply -f exported-svc.yml; k apply -f intention.yml;
```
7. Go back to the first cluster and confirm that the peering connection is up by
   checking if the endpoints of backend deployed in peer api show up on the
   first cluster. (Need to exec onto a consul server):
```
curl "localhost:8500/v1/health/connect/backend?peer=api&pretty"
```
8. Deploy frontend, which has an annotation describing it's explicit upstream
   backend:
```
k apply -f frontend.yml
```
9. Exec onto frontend, and try to reach backend. The local listener for the
    upstream is `localhost:1234`, it should return successfully:
```
curl localhost:1234
```
10. [Optional: Intentions] If you edit the intention to remove the last 2 lines that allow the
    "frontend" service, you should see the same request in step 9 fail.
11. [Optional: Deleting endpoints and Unpeering] If you delete the exported service, you should see
    that the endpoints for backend no longer show up if you run the command in
    step 7.


    **This next part does not yet work but will after the last deletion PR is merged"
    Similarly, if you delete the peering resource (PeeringAcceptor and
    PeeringDialer) in both clusters, you
    should also see the command in step 7 return an empty set of instances.
