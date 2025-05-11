 oc login https://api.alsh-cluster.24o1.p1.openshiftapps.com:6443 --username cluster-admin --password tS7h3-okahV-s5f7s-gUwrV

## Create Users

```bash

oc create user amee
oc create user berry
oc create user kelly
oc create user jack
oc create user nash

```

## Create Project and add description

Creates a new project with description in a specified namespace

```bash

oc new-project achernar --description="Satellite Constellation"
oc new-project arcturus --description="Satellite Constellation"
oc new-project antares --description="Satellite Constellation"
oc new-project aldebaran --description="Satellite Constellation"
oc new-project altair --description="Satellite Constellation"

```

## Grant User Privileges in a specified namespace

```bash

oc adm policy add-role-to-user admin amee -n achernar
oc adm policy add-role-to-user view berry -n arcturus
oc adm policy add-role-to-user admin kelly -n antares
oc adm policy add-role-to-user admin kelly -n altair

```


## 1. Create an htpasswd file:

   - **htpasswd -c htpass-file amee** - This command creates a new htpass-file and writes the username "amee" along with the hashed password into it. The -c flag is used to create a new file.

   - **htpasswd htpass-file berry** - Without the -c flag, this command appends the username "berry" and its hashed password to the existing htpass-file.

   - **htpasswd htpass-file kelly** - Similar to the previous command, this appends the username "kelly" and its hashed password to the existing htpass-file.

   - **htpasswd htpass-file jack** - Appends the username "jack" and its hashed password to the existing htpass-file.

   - **htpasswd htpass-file nash** - Appends the username "nash" and its hashed password to the existing htpass-file.

## 2. Verify the users and their passwords:

   - **cat htpass-file** - This command is used to display the contents of the htpass-file, showing the users and their hashed passwords to verify the file's content.

## 3. Create a secret:

   - **oc create secret generic users-secret --from-file=htpasswd=./htpass-file** - This command creates a generic secret named "users-secret" using the htpass-file. The --from-file flag specifies the source file to create the secret. This is a standard Kubernetes/OpenShift approach to store sensitive information.

## 4. Create a secret in a specific namespace:

   - **oc create secret generic users-secret --from-file=htpasswd=./htpass-file -n openshift-config** - Similar to the above command, this creates the same secret, but within the "openshift-config" namespace.

## 5. Edit the OAuth configuration:

   - **oc edit oauth cluster** - This command opens the OAuth cluster configuration for editing.

## 6. Add the secret to the configuration:

   - Within the configuration, you would need to find the spec.identityProviders.htpasswd.fileData.name section and add the secret name "users-secret". This step associates the htpass-file with the OAuth identity provider.

7. View secrets and users:
   - oc get secrets -n openshift-config - View the list of secrets within the "openshift-config" namespace.

   - oc get users - Lists all the users within the cluster.

   - oc get secrets -n openshift-config - Another command to view the secrets within the "openshift-config" namespace.

8. Creating users:
   - oc create user [username] - A set of commands to create users "amee", "berry", "kelly", "jack", and "nash" within the OpenShift cluster. These commands are required after associating the htpass-file with the OAuth configuration to ensure these users can authenticate against OpenShift using htpasswd authentication method.


## Resource Quotas and LimitRange

https://stackoverflow.com/questions/54929714/in-kubernetes-what-is-the-difference-between-resourcequota-vs-limitrange-object

A. **Resource Quotas**

When several users or teams share a cluster with a fixed number of nodes, there is a concern that one team could use more than its fair share of resources. Resource quotas are used to address this concern. 

A resource quota provides constraints that limit resource consumption per namespace. 
It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that namespace.

```bash

apiVersion: v1
kind: ResourceQuota
metadata:
  name: example
  namespace: default
spec:
  hard:
    pods: '4' <--------------- (1)
    requests.cpu: '1' <--------------- (2)
    requests.memory: 1Gi <--------------- (3)
    limits.cpu: '2' <--------------- (4)
    limits.memory: 2Gi <--------------- (5)

```

1. The total number of pods that can exist in the project.
2. Across all pods, the sum of CPU requests cannot exceed 1 core.
3. Across all pods, the sum of memory requests cannot exceed 1Gi.
4. Across all pods, the sum of CPU limits cannot exceed 2 cores.
5. Across all pods, the sum of memory limits cannot exceed 2Gi.

B. **LimitRange**

Within a namespace, a Pod can consume as much CPU and memory as is allowed by the ResourceQuotas that apply to that namespace. As a cluster operator, or as a namespace-level administrator, you might also be concerned about making sure that a single object cannot monopolize all available resources within a namespace.

A LimitRange is a policy to constrain the resource allocations (limits and requests) that you can specify for each applicable object kind (such as Pod or PersistentVolumeClaim) in a namespace.

LimitRange defines default, maximum, minimum of CPU and memory consumption by workloads in a namespace. 

**Note**: If you have a quota applied in a namespace, every Pod must request for resources like CPU and memory in their manifests. Otherwise the Pod creation will fail. But if you have a LimitRange enforced with default memory and CPU requests that could be avoided.

```bash

apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "resource-limits"
spec:
  limits:
    - type: "Container"
      max: <--------------- (1)
        cpu: "2"
        memory: "1Gi"
      min: <--------------- (2)
        cpu: "100m"
        memory: "4Mi"
      default: <--------------- (3)
        cpu: "300m"
        memory: "200Mi"
      defaultRequest: <--------------- (4)
        cpu: "200m"
        memory: "100Mi"
```

1. The maximum amount of CPU and memory that a single container in a pod can request
2. The minimum amount of CPU and memory that a single container in a pod can request
3. The default amount of CPU and memort that a container can use if not specified in the Pod spec.
4. The default amount of CPU and that a container can request if not specified in the Pod spec.


## Remove the ability to provision new projects from all users

```bash
 oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
 
 ```
 Self-provisioner provide users with the ability to create projects,so we need to remove it in order to prevent users from been able to create projects.

**https://computingforgeeks.com/prevent-users-from-creating-projects-in-openshift-kubernetes-cluster/?expand_article=1**

**https://medium.com/@tcij1013/openshift-project-and-user-management-guide-612dd89af60b**

## ResourceQuotas and LimitRange

**https://freedomben.medium.com/configuring-default-resource-requests-limits-and-quotas-on-your-new-openshift-4-cluster-daef7d1a84f5**

**https://docs.openshift.com/container-platform/3.11/dev_guide/compute_resources.html**

## Allocate persistent volume to the internal docker registry - create PV, PVC and add volume

A. **Persisten Volume (PV)**
 
A persistent volume (PV) is a piece of storage in the Kubernetes cluster, while a persistent volume claim (PVC) is a request for storage.

Persistent volumes helps persist data even if your container or pods restarts or get terminated

```bash

apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
spec:
  capacity: <---------------- (1)
    storage: 5Gi
  accessModes:
    - ReadWriteOnce <---------------- (2)
  hostPath: <---------------- (3)
    path: /tmp
    server: 172.17.0.2

```
1. Specifies the amount of storage that this PersistentVolume can provide
2. ReadWriteOnce, meaning the volume can be mounted as read-write by a single Node.
3. Specifies that the PersistentVolume will be backed by a directory on the host. It sets the path to "/tmp" and the server to "172.17.0.2," which reflects the IP address of the node where the volume is located.

B. **Persisten Volume Claim (PVC)**

In a real ecosystem, a system admin will create the PersistentVolume then a developer will create a PersistentVolumeClaim which will be referenced in a pod. 

When a developer needs persistent storage for an application in the cluster, they request that storage by creating a persistent volume claim (PVC) and then mounting the volume to a path in the pod.

A PersistentVolumeClaim is created by specifying the minimum size and the access mode they require from the persistentVolume.

```bash

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources: <--------------- (1)
    requests:
      storage: 5Gi <--------------- (2)
```
1. Specifies the resource requirements for the PersistentVolumeClaim.
2. Describes the amount of storage requested for the PersistentVolumeClaim. 

C. **Pods to claim the volume**

```bash

apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes: <------------- (1)
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: test-pvc
  containers: <------------- (2)
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts: <------------- (3)
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage

```
1. The volumes that should be available to the containers in the Pod. Here, it specifies a PersistentVolumeClaim named **test-pvc** as seen above, that the pod will use.
2. Describes the container that should be run in the pods.
3. Describes how the volumes in the Pod should be mounted into the container. In this case, it mounts the **task-pv-storage** volume to the path "/usr/share/nginx/html" in the container.


## Hello-Openshift app  and secure edge route

A Hello-Openshift app is a simple application that displays the message "Hello from OpenShift!" It is a good example of how to create and deploy a simple application in OpenShift.

A secure edge route is a way to expose an application in OpenShift to the public internet in a secure way. It uses a TLS certificate to encrypt traffic between the client and the server.

To create a Hello-Openshift app and secure edge route, you can use the following steps:

Hello-Openshift app Create an HTTPs based route

```bash  

    oc new-project atmos —description=“Atmos”

 ```
Create a new OpenShift project called atmos with the description "Atmos".

 ```bash
 oc new-app centos/ruby-25-centos7~https://github.com/sclorg/ruby-ex.git --name="atmos"
./gen-certs.sh atmos.atmos.10.0.2.204.xip.io

 ```
1. Create a new OpenShift application called atmos using the centos/ruby-25-centos7~https://github.com/sclorg/ruby-ex.git image.

2. Generate a TLS certificate and key for the atmosapplication

```bash
oc create route edge --service=atmos --cert=atmos.atmos.10.0.2.204.xip.io.crt --key=atmos.atmos.10.0.2.204.xip.io.key --hostname=atmos.atmos.10.0.2.204.xip.io

```
3. Create an edge route for the atmos application that uses the generated TLS certificate and key and exposes the atmos service.

Access application on **http://atmos.atmos.10.0.2.204.xip.io**


### Create a new app, make a change and rebuild:

# Hello-Openshift app  and secure edge route

1. **Create a new OpenShift application called myapp using the openshift/hello-openshift image**

```bash
oc new-app openshift/hello-openshift --name=myapp
oc expose service/myapp
oc status

```

2. **Generate Self-Signed TLS certificates for the application

```bash 

openssl genrsa -out key.key
openssl req -new -key -out csr.csr -subj "/hostname"
openssl x509 -req -in csr.csr -signkey key.key -out crt.crt

```

3. **Create an edge route for the atmos application that uses the generated TLS certificate and key and exposes the atmos service.**

```bash

oc create route edge --service=myapp --hostname=hostname --key=key.key --cert=crt.crt 

oc get route

```

## S2I and rebuild

**https://ebrudalkir6.medium.com/openshift-s2i-source-to-image-cbacf877159e**

In the context of OpenShift, S2I (Source-to-Image) and rebuild are important concepts related to application deployment and continuous integration/continuous delivery (CI/CD) processes.

### Source-to-Image (S2I):

S2I is a feature in OpenShift that streamlines the process of building and deploying applications from source code. It provides a framework for automating the creation of container images directly from application source code. 

With S2I, developers can focus on writing and maintaining their application code without needing to manage the complexities of creating and maintaining container images manually.

`oc new-app https://github.com/IBM-Cloud/openshift-node-app --name=<app-name> --strategy=docker --as-deployment-config`

 - This reates a new application named new-app using the source code specified in the GitHub repository "https://github.com/IBM-Cloud/openshift-node-app." 
 
- The `--strategy=docker` flag sets the build strategy as Docker, and `--as-deployment-config` creates a deployment configuration for the application.


S2I simplifies the build process by abstracting away the details of building and packaging applications, allowing developers to provide their application source code repository (such as Git) as input. It then automatically orchestrates the build process to create a ready-to-run container image. S2I can be customized through builder images, which include the required build tools, runtime environment, and configuration files specific to the application stack or programming language.

### Rebuild:

Rebuild refers to the process of regenerating a container image for an application. In OpenShift, when triggered, a rebuild will initiate the S2I process to re-create the container image using the updated source code or configuration. This allows for application updates, bug fixes, or new features to be incorporated into the container image and deployed to the OpenShift cluster.

### Importance and Benefits:

S2I and rebuild are crucial elements for smooth and efficient application deployment in OpenShift. Here are some of their key benefits:

1. Simplified Build Process: S2I abstracts away the complexities of creating container images, making it easier for developers to build and deploy applications without deep expertise in containerization.

2. Consistency and Automation: S2I ensures consistent build and deployment processes across different applications and teams. It provides automation for building, testing, and packaging applications, enabling standardized and reproducible pipelines.

3. Faster Iteration and Continuous Integration: With S2I and rebuild, developers can rapidly iterate on their application code, triggering rebuilds whenever changes occur. This supports continuous integration and allows for quick feedback on code changes.

4. Improved Deployment and Rollback: By rebuilding container images, application updates and fixes can be easily incorporated and deployed to OpenShift clusters. Additionally, if issues arise, the process allows for simple rollback to previous known working versions of the application.

Overall, S2I and rebuild are important components in OpenShift that contribute to the efficient and streamlined development, build, and deployment of containerized applications. They help simplify the CI/CD pipeline, support faster development iterations, and enable organizations to deliver reliable and up-to-date applications to their users.


## Scale Replicas

Scaling replicas refers to adjusting the number of instances or copies of a specific workload or application running in an openshift cluster. OpenShift provides various methods to scale replicas based on the desired configuration and requirements of your application.

To scale replicas in OpenShift, you can use the `oc scale` command or manipulate the desired replica count in the YAML file.

1. **Using the `oc scale` Command**: 

   Syntax: `oc scale --replicas=<replica_count> <object_type>/<object_name>`

   Replace `<replica_count>` with the desired number of replicas, `<object_type>` with the type of object (e.g., `deployment`, `replicaset`), and `<object_name>` with the name of the object.

   Example: `oc scale --replicas=3 deployment/my-app`

   This command scales the specified deployment named `my-app` to 3 replicas.

2. **Modifying the YAML file**:

   Locate the YAML file that defines the desired object (e.g., Deployment, StatefulSet) and update the `spec.replicas` field to the desired replica count.

   Example:
   ```yaml
   ...
   spec:
     replicas: 3
   ...  
   ```
   This YAML snippet sets the number of replicas to 3.

   Apply the changes using `oc apply -f <filename.yaml>`.

Scaling replicas in OpenShift allows you to dynamically adjust the number of running instances to meet demand, improve availability, and achieve desired performance characteristics for your applications.

Please note that scaling replicas may require cluster administrator privileges or appropriate RBAC permissions. Additionally, scaling depends on the workload type and the application's ability to be scaled horizontally.

9. Wordpress app
10. install metrics
11. Application from 3rd party templates
