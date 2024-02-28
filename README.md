# Moodle-CCE
## Moodle project mounted on HuaweiCloud
First is needed to create Huawei Cloud resources, such as CCE Cluster and a ECS to manage kubernetes cluster and run Helm commands, following the [hardware recommendations for moodle](https://docs.moodle.org/403/en/Installing_Moodle) it should be:
- Disk space: 200Mb as minimum, but 5Gb is more realistic
- CPU: 1GHz as minimum, and 2Ghz is recommended
- Memory: 512Mb as minimum, 1Gb is recommended and 8Gb for large production server

Knowing this we can start buying the required resources:
- For a dev environment or a small scale production server we can use a node with this specs:
  - VCPU: 4Ghz
  - Memory: 8Gb
- For a large scale production server:
  - VCPU: 4GHz
  - Memory: 16Gb

Now that we know the spec that we need, lets start by buying the resources:
<p align="center">
 <img src="https://github.com/Limbo-Dev/Moodle-CCE/blob/main/images/moodle-cce.png" />
</p>p
for [more information](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0028.html) about the configurations of CCE Cluster configuration visit the page.

follow this configuration:
- basic settings:
  - type: CCE Standard Cluster
  - Cluster Version: 1.25 or the newest one that allow to use docker as [container engine](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0405.html#cce_10_0405__clusterUpgradeTo1_28)
  - th rest leave as default or configure it as desired
- addons:
  - leave it as default or select desired addons
  - configuration can be as default or a desired configuration

after the completion of the cluster creation we need to buy a worker node, for [more information](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0363.html) about the creation of the node visit the page.

following this configuration:
- Basic settings:
  - Specifications: 4VCPUs, 8GiB or higher spec depending on your needs
  - Container Engine: Docker
  - OS: ubuntu or as you like
  - Disk size: leave it as default
# Kubernetes Architecture
For this demo the architecture will be looking like this:

<p align="center">
  <img src="https://github.com/Limbo-Dev/Moodle-CCE/blob/main/images/kubernetes-moodle-hld.png"/>
</p>

now that we know what are we trying to deploy is time to configure values.

## Values Configurations
note that there are 2 ways to deploy moodle on CCE, K8s manifests and Helm Chart, the major difference between them is the dificulty during configuration, if you are not familiar with Helm you're advice to use K8s Manifist for extra configurations, and you do know how helm works you can use [helm playground](https://helm-playground.com/) to see the output manifest using Helm Charts.
### Kubernetes Manifest
#### Deployment File
In this file you will find out the image that is being used to deploy moodle and the deployment configuration, the section that you should get more involved are:
- image: the image that is being used is from bitnami, if you are wanting to deploy a newest version of the image you should take a look of their [Moodle Docker image](https://hub.docker.com/r/bitnami/moodle).
- environment variables: these variables and their values are the default configuration values from the image, you can change it depending on your personal configurations, find more information about the environment variables in their [Moodle Docker image](https://hub.docker.com/r/bitnami/moodle).
- resources: change the resources limits and requests as you need.
- containers ports: these are the default container from the [Moodle Docker image](https://hub.docker.com/r/bitnami/moodle), as you can see they are using port 8080 and 8443, avoiding port 80 and 443 due to cybersecurity reasons, thats why in this manifest we are using port 8080 and 8443.
- volume mounts: the mountpath is important to leave it as default, because the path is where the instalation script install the moodle files.
- image pull secrets: the name is the secret object that is being created with the secret manifest it contains the Access Key and Secret Key values.
- node selector: this section can be deleted if you do not want to deploy the pods on a specific node.

#### StatefulSet
In this file you will find out the image that is being used to deploy the database, in this case will be MariaDB, the section that you should get more involved with are:
- image: the image that is being used is from bitnami, if you are wanting to use a newest version of the image you should look are their [Mariadb Docker image](https://hub.docker.com/r/bitnami/mariadb) to get more information about it.
- Environment Variables: these variables and values that is being used is the default configuration values from the image, you can change it depending on your personal configurations find more information about the environment variables in their [Mariadb Docker image](https://hub.docker.com/r/bitnami/mariadb).
- container ports: the default port for the database is 3306 as you can see in [Mariadb Docker image](https://hub.docker.com/r/bitnami/mariadb) environment variables, you change this port just by adding a new environment variable with the same name as the image information says, be aware that if you change it, you would need to open the same port you change it for.
- volume mounts: the mountpath is important to leave it as default, because the path is where the instalation script install the moodle files.
- image pull secrets: the name is the secret object that is being created with the secret manifest it contains the Access Key and Secret Key values.
- node selector: this section can be deleted if you do not want to deploy the pods on a specific node.

#### PVC and PV
By default this K8s architecture is dynamically provisioning the storages, for this demo we are using Elastic Volume Services storage, but there are more storage services [Elastic Volume Service](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0613.html), [Scalable File Service](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0617.html) and [Object Storage Service](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0628.html), in these documentations you can find out more information about Huawei Cloud Storage services for dynamic and static provision for CCE.

so the section that you should be aware in this demo are:
- disk type: depending on what type of disk you want to use, such as HDD, SSD or Ultra high SSD, it will more expensive or cheaper, you can find out the types of disk [here](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0615.html)
- region and zone: in this section you need to be carefull, before deploying be aware that the region and zone is the same where the worker node is deployed, this is so important because the dynamic storage provided by EVS(Elastic Volume Service) is going to be attached to the worker node so it can be attached to the pod, follow the next recommendations:
  - Verify that the region and zone are the same where the worker node was deployed, visit [here](https://developer.huaweicloud.com/intl/en-us/endpoint) to get the correct values for each region and zone.
  - Use nodeSelector in the deployment and stateful manifest to deploy in a specific worker node in case you more than 1 worker node
#### Secret
This file is just used to store your enconded Access key and Secret key, the only used that I could find out was during OBS(Object Storage Services) dynamic and static provision, since it need to access a storage that is not being attached to the worker node.

Use this command to encode your AK/SK
```
echo -n <your access key>|base64
echo -n <your secret key>|base64
```
#### Services
Note: Is important that you only modify the LB(Elastic Load Balancer Services is being used) service since the clusterip is being used to communicate application and database pod through the port 3306, unless you changed the database access port, be aware that the Load Balancer service is dyanamically provided.

The Load Balancer is using Elastic Load Balancer service from Huawei Cloud visit [here](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0681.html), the section that you should get more information about it are:
- annotations:
  - elb.class: is important to leave as union
  - elb.autocreate: in this section you will find out the configuration of the ELB and EIP(Elastic IP Services)
- ports: do not change the target ports unless you are modifying the directly the [Moodle Docker image](https://hub.docker.com/r/bitnami/moodle) to change the ports that you want to access to the services.
#### HPA (Horizontal Pod AutoScaling)
In this manifest you can find the autoscaling policies for Moodle pod, depending on the memory usage it will scale out.

Only deployment kind is supported by Huawei Cloud CCE Cluster, Statefulset is not supported yet.
### Helm Chart
