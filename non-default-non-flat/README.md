### Non default partition to non-default partition peering, without a flat network
1. Deploy 2 clusters on GKE and 2 clusters on EKS using the consul-k8s
   terraform. This will ensure the clusters on GKE have the network requirements
   for partitions, and the clusters on EKS have the network requirements for
   partitions. We will deploy 2 partitions in GKE and 2 partitions in EKS.
1. Deploy a default partition in dc 1 using dc1-server-ap.yaml on GKE
2. Copy the ca cert and ca key secrets over to the second cluster on GKE. Edit
   the values file dc1-client-ap.yaml to point to your own ca cert and key.
3. Deploy a non-default partition in dc1 using dc1-client-ap.yaml on the second
   GKE cluster.
4. Do the same as steps 1-3 on EKS, using dc2-server-ap.yaml and
   dc2-client-ap.yaml.
5. Deploy the acceptor.yml on the non-default partition in GKE.
6. Deploy dialer.yml, backend.yml, and exported-svc.yml on the non-default
   partition in EKS.
7. Deploy frontend.yml on the non-default partition in GKE.
8. Exec onto the frontend app, and curl localhost:1234 to see a response from
   the backend app from EKS!
