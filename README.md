# acmeair-gitops

## Deploying to both containers and virtual machines with OpenShift Virtualization
The acmeair-ocpvirt directory will create an application that includes WebSphere Liberty microservices running in containers as well as a RHEL virtual machine running on the cluster via OpenShift Virtualization.

The RHEL VM definition contains cloud-init scripting to attach a Red Hat subscription with subscription-manager, mount a secondary /data disk, install MongoDB to store data on the /data disk, and start the MongoDB service. 

### Prerequisites

1. OpenShift Virtualization operator installed and a hyperconverged object created
2. A valid Red Hat subscription with access to rhel-9-for-s390x-baseos-rpms and rhel-9-for-s390x-appstream-rpms
   1. Red Hat offers a no-cost developer subscrition. Follow the instructions [here](https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux)

### Installation procedure

1. Create a project named `acmeair`

    ```
    oc new-project acmeair
    ```

2. Create a public/private key pair that will be used to ssh to the RHEL virtual machine

    ```
    ssh-keygen -t rsa -b 4096 -C "<your_email>"
    ```

    When prompted to `Enter file in which to save the key`, save it somewhere accessible with a name you can use in the following step, e.g. `/foo/bar/ocpv_id_rsa`.

3. Create a secret that contains the public key

    ```
    oc -n acmeair create secret generic ocpvirt-ssh --from-file=key=ocpv_id_rsa.pub
    ```

4. Create a secret that contains your Red Hat credentials

    ```
    oc -n acmeair create secret generic rhsm-creds --from-literal rhsm_username=<your_username> --from-literal rhsm_password=<your_password>
    ```

5. Modify the `.spec.host` value for each route under the routes-ocpvirt directory

    Change `ocpvirt.cpolab.ibm.com` to your domain, which can be identified with the following command

    ```
    oc get dns cluster -o jsonpath='{.spec.baseDomain}{"\n"}'
    ```
   
6. Deploy all of the YAML files under the acmeair-ocpvirt directory

    ```
    oc apply -f acmeair-ocpvirt/deployments -f acmeair-ocpvirt/routes-ocpvirt -f acmeair-ocpvirt/virtualization
    ```

    Alternatively, deploy the YAML files with a GitOps solution like [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) or [Red Hat Advanced Cluster Management for Kubernetes](https://www.redhat.com/en/technologies/management/advanced-cluster-management).