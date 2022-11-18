### Non default partition to non-default partition peering, without a flat network
1. Deploy 4 K8s clusters. Currently, I've not made sure this config supports PSPs.
1. Deploy a default partition in dc 1 using dc1-server-ap.yaml cluster 1.
2. Copy the ca cert and ca key secrets and the partition token secret over to the cluster 2. Edit
   the values file dc1-client-ap.yaml to point to your own ca cert and key. (Follow admin partitions docs to set up your client partition)
3. Deploy a non-default partition in dc1 using dc1-client-ap.yaml on cluster 2.
4. Do the same as steps 1-3 on clusters 3 and 4, using dc2-server-ap.yaml and
   dc2-client-ap.yaml.
4. Deploy the mesh.yml and proxydefaults.yml in the default partition in dc1 and default partition in dc2.
5. Deploy the acceptor.yml on the non-default partition in dc1.
6. Deploy dialer.yml, backend.yml, intention.yml and exported-svc.yml on the non-default
   partition in dc2. (edit intention.yml and exported-svc.yml with your peer and partition names)
7. Deploy frontend.yml on the non-default partition in dc 1.
8. Exec onto the frontend app, and curl localhost:1234 to see a response from
   the backend app! You can also use tproxy with this by removing the explicit upstream and dialing backend using http://backend.virtual.default.ns.ekspart2.ap.api.peer.consul.
