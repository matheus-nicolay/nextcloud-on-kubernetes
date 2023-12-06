# Nextcloud on Kubernetes
The project is about setting up the complete configuration for using the Nextcloud open-source drive in an on-premise Kubernetes cluster (or baremetal). In addition to deploying the database and application, the project includes the installation of Cert Manager (to generate SSL certificates), Nginx Ingress Controller, MetalLB (on-premise load balancer), and the NFS Provisioner.

## Prerequisites for on-premise cluster
### Ingress Controller
- Install NGINX Ingress Controller:
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml`

### Cert Manager
- Required to change the 'email' field for create Let's Encrypt account:

`cert-manager/prod_issuer.yaml`
`cert-manager/staging_issuer.yaml`

- Apply/install cert-manager:
`kubectl apply -f ./cert-manager/`

### MetalLB LoadBalancer
- Use your own IP addreses by changing the "addresses:" parameter on the file `metallb/ipaddresspool.yaml`
- Apply to create IPAddressPool resource:
`kubectl apply -f metallb/ipaddresspool.yaml`

- If BGP is used, change the "ASN" and "PeerAddress" specifications on file `metallb/bgpadvertisement.yaml` and apply:
`kubectl apply -f metallb/bgpadvertisement.yaml`

- If Layer2 is used, apply:
`kubectl apply -f metallb/layer2advertisement.yaml`

### NFS Provisioner
- In nfs-provisioner/nfs-deployment.yaml two different pods are deployed, one representing nfs for HDD and the other for SSD. The NFS server address can be changed in the following snippet:

```
env:
    - name: PROVISIONER_NAME
      value: ff1.dev/ssd 
    - name: NFS_SERVER
      value: 10.0.0.2 #Server IP
    - name: NFS_PATH
      value: /ssd01 #NFS server path
```

## Nextcloud components
## Ingress
- Change the domain of Nextcloud on ./nextcloud/ingress.yml and apply the ingress configuration:
`vi ./nextcloud/ingress.yml`
`kubectl apply -f ./nextcloud/ingress.yml`

## Persistent volumes
In this project, persistent volumes are separated into configuration and files, seeking to place files on the lowest performing disk and using the DB and config on the faster disk. The size of the PVC can be edited on the files, using "storage" parameter.

`kubectl apply -f ./nextcloud/app/nextcloud-config-pvc.yaml`
`kubectl apply -f ./nextcloud/app/nextcloud-data-pvc.yaml`
`kubectl apply -f ./nextcloud/db/db-pvc.yaml`

## Deployment
Deployment can be done in two different methods. To use Nextcloud in the FPM-Alpine version with nginx use:
`kubectl apply -f ./nextcloud/app/nc-fpm-alpine/`

To use with Apache integrated into a single container:
`kubectl apply -f ./nextcloud/app/nc-apache/`

## Database
Apply statefulset of the Redis database, used for cache:
`kubectl apply -f ./nextcloud/db/redis/redis-statefulset.yaml`

Apply statefulset of the Postgres database:
`kubectl apply -f ./nextcloud/db/postgres/postgres-statefulset.yaml`