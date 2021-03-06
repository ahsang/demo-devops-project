# demo-devops-project

## Objectives

-  Automating the build of the container of a test golang app
-  Setting up the TLS Offloading load balancer
-  Defining the whole infrastructure as code

### Tools Used
- Ansible: Used to setup a baremetal ubuntu machine to host a kubernetes cluster containing the running app.
- Terraform: Used to provision the kubernetes components step by step to ensure every step is automated and can be created/destroyed smoothly

### Requirements for setting up ubuntu (20.04 lts or 18.04 lts) based host 
- Ansible >=2.11.6
### Steps for setting up ubuntu based host 
- Run the playbook.yaml ansible-playbook to setup the vm and install microk8s cluster along with dns(to allow kubernetes pods to access the internet) and metallb addons (To provide a loadbalancer to kubernetes for the nginx ingress)
    ```
    ansible-playbook -i '<host-ip>,' -u <username> playbook.yaml 
    ```
    
    To apply the kubernetes components as well please run
    ```
    ansible-playbook -i '<host-ip>,' -u <username> playbook.yaml -e apply_changes=true
    ```
- Point the subdomain DNS A record to the public ip of the host

> This assumes the host is already accessible via ssh

## Hoping to understand, what does the playbook do? Here goes:
- Sets up [microk8s](https://microk8s.io/) in the host
- Enables microk8s [metallb](https://metallb.universe.tf/) addon with the address range limited to the host ip and the microk8s dns addon
- Sets up kubectl
- Clones the infrastracture as code terraform project [repo](https://github.com/ahsang/certmanager-demo) to /tmp/certmanager-demo inside the host
- Runs terraform init and terraform plan (Intentionally defaults at plan to save time) or runs terraform apply to provision the components.


## Next Steps
- ssh into the host
- navigate to /tmp/cert-manager
- run either
    ```
    terraform apply tfplan
    ```
    or 
    ```
    terraform apply -var 'subdomain=test.domain.com'

### Expected result
- A deployment for our [test app](https://github.com/ahsang/golang-cicd) using the [helm chart](https://github.com/ahsang/certmanager-demo/tree/master/outyet) on the kubernetes cluster running inside our host
- The [app docker image](https://hub.docker.com/repository/docker/ahsangondal/golang-cicd) is hosted in docker hub public registry which is set to auto build in case of any updates to the code
- the app will be running behind an nginx ingress load balancer
- [cert-manager](https://cert-manager.io/) will be deployed to generate lets encrypt ssl certs automatically so the app is accessible over tls on the given subdomain
- you will be able to navigate to https://test.zeezouworld.com (or custom subdomain) and see the test app with a valid certificate



## License

MIT

