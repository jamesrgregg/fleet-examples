```
.
└── servas-app
    ├── Chart.yaml
    ├── README.md
    ├── templates
    │   ├── configmap.yaml
    │   ├── db-deployment.yaml
    │   ├── db-service.yaml
    │   ├── pvc.yaml
    │   ├── secret.yaml
    │   ├── servas-deployment.yaml
    │   └── service.yaml
    └── values.yaml
```      

To use Rancher Fleet to deploy the servas-app using the Helm charts, by following these steps:

1. **Create a Git Repository for the Helm Charts:**
   - Push the Helm chart files to a Git repository. Rancher Fleet will use this repository to deploy the application.

2. **Configure Rancher Fleet:**
   - Create a Fleet GitRepo resource that points to your Git repository containing the Helm charts.
   - Fleet will monitor this repository and deploy the Helm charts to the specified clusters.

### Step 1: Push Helm Charts to Git Repository

First, let's organize the Helm chart files and push them to a Git repository. You can create a new repository on GitHub or use an existing one.

### Step 2: Create Fleet GitRepo Resource

Next, create a `GitRepo` custom resource in Rancher Fleet to reference your Git repository.

Here are the steps to achieve this:

1. **Create a `GitRepo` Custom Resource in Rancher Fleet:**
   - Create a Fleet `GitRepo` resource YAML file that references your Git repository.

```yaml name=servas-app-gitrepo.yaml
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: servas-app
  namespace: fleet-local
spec:
  repo: https://github.com/jamesrgregg/fleet-examples
  branch: main
  paths:
    - servas-app
```

2. **Apply the `GitRepo` Resource:**
   - Apply the `GitRepo` resource to your Rancher Fleet installation.

```sh
kubectl apply -f servas-app-gitrepo.yaml
```
_Note: GitRepo definition is created in fleet-local namespace but the deployment of the application will be found in default namespace._

### Summary

1. **Push the Helm chart files to a Git repository.**
2. **Create and apply a Fleet `GitRepo` resource that references your Git repository.**

### Troubleshooting
1. Fleet is able to get the servas-app image and mariadb image, but the database container crashes.
The db container shows the following:

```
mysqld: Please consult the Knowledge Base to find out how to run mysqld as root!
2025-03-15 18:48:24 0 [ERROR] Aborting
```
Proposed fix: 
 - Run as a non-root user by setting `securityContext` in the deployment.yaml
 Reference: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/


 ```
apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mariadb
    spec:
      # ... other deployment configurations ...
      template:
        spec:
          containers:
            - name: mariadb
              image: mariadb:10.7 # or your desired version
              # ... other container configurations ...
              securityContext:
                runAsUser: 999 # example user id
                runAsGroup: 999 # example group id
              volumeMounts:
                - name: mariadb-data
                  mountPath: /var/lib/mysql
          volumes:
            - name: mariadb-data
              persistentVolumeClaim:
                claimName: mariadb-pvc # your persistent volume claim

 ```
 
2. GitRepo shows resources limited.
3. Do not run app and db inside the same pod as this bad practice.  

Proposed Fix:
 - Need to separate the deployments - frontend / backend just like the guestbook app.

