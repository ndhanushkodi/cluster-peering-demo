# Test Cluster Peering Service Mesh
1. Create 2 K8s clusters. Take note of which instructions apply to each
   individual peer or BOTH peers.
2. Install 1st peer (replace path with where you've cloned consul-k8s repo to.
   It needs to point to the charts/consul directory) :
```
helm install nitya1 ../../../consul-k8s/charts/consul -f peer1-consul-values.yml
```
2. Install 2nd peer:
```
helm install nitya2 ../../../consul-k8s/charts/consul -f peer2-consul-values.yml
```
3. Apply mesh.yml to the **BOTH** clusters:
```
k apply -f mesh.yml
```
3. Apply proxydefaults.yml to **BOTH** clusters:
```
k apply -f proxydefaults.yml
```
3. Verify that the mesh and proxydefaults resources exist on your cluster with
   no errors. 
3. Apply acceptor.yml to the FIRST cluster:
```
k apply -f acceptor.yml
```
4. This will cause Consul to generate a peering token called `api-token`. Get
   that token.
```
k get secret api-token -oyaml > api-token.yml
```
5. Apply the token to the SECOND cluster:
```
k apply -f api-token.yml
```
6. Apply to the SECOND cluster, the dialer resource, backend application, and exported service CR (you only need the intention.yml if you enable acls):
```
k apply -f dialer.yml; k apply -f backend.yml; k apply -f exportedsvc.yml; k apply -f intention.yml;
```
8. On the FIRST cluster, deploy frontend, which has an annotation describing it's explicit upstream
   backend:
```
k apply -f frontend.yml
```
9. Exec onto frontend on the FIRST cluster, and try to reach backend. The local listener for the
    upstream is `localhost:1234`, it should return a response from backend successfully:
```
curl localhost:1234
```
10. [Optional: Intentions] If you edit the intention to remove the last 2 lines that allow the
    "frontend" service, you should see the same request in step 9 fail.

