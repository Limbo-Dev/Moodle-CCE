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

## Values Configurations
note that there are 2 ways to deploy moodle on CCE, K8s manifests and Helm Chart, the major difference between them is the dificulty during configuration, if you are not familiar with Helm you're advice to use K8s Manifist for extra configurations, and you do know how helm works you can use [helm playground](https://helm-playground.com/) to see the output manifest using Helm Charts.
### Kubernetes Manifest
#### Deployment File
In this file you will find out the image that is being used to deploy moodle and the deployment configuration, the section that you should get more information about are:
- image: the image that is being used is from bitnami, if you are wanting to deploy a newest version of moodle you should take a look of their [docker image](https://hub.docker.com/r/bitnami/moodle).
- environment variables: the environment variables that is being used in this demo is the default values from the image, you can change it depending on your personal configurations
- resources: change the resources limits and requests as you need.
- containers ports: these are the default container from the [docker image](https://hub.docker.com/r/bitnami/moodle), as you can see they are using port 8080 and 8443, avoiding port 80 and 443 due to cybersecurity reasons.
- volume mounts: the mountpath is important to leave it as default, because the path is where the instalation script install the moodle files.
- image pull secrets: the name is the secret object that is being created with the secret manifest it contains the Access Key and Secret Key values.
- node selector: this section can be deleted if you do not want to deploy the pods on a specific node.

#### StatefulSet

#### PVC and PV

#### Secret

#### Services

#### HPA (Horizontal Pod AutoScaling)

### Helm Chart
