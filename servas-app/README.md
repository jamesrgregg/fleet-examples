```
servas-app/
  ├── Chart.yaml
  ├── values.yaml
  └── templates/
      ├── deployment.yaml
      ├── service.yaml
      └── pvc.yaml
```      

To use Rancher Fleet to deploy the servas-app using the Helm charts we just created, you need to follow these steps:

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
  namespace: fleet-default
spec:
  repo: "https://github.com/your-username/your-repo-name.git"  # Replace with your Git repository URL
  branch: main  # Replace with the branch name where Helm charts are stored
  paths:
    - "servas-app"  # Path to the Helm chart directory
  targets:
    - clusterSelector:
        matchLabels:
          environment: production  # Replace with your cluster label
      helm:
        releaseName: servas-app
        values:
          env:
            DB_USERNAME: user
            DB_PASSWORD: password
            DB_DATABASE: servas
```

2. **Apply the `GitRepo` Resource:**
   - Apply the `GitRepo` resource to your Rancher Fleet installation.

```sh
kubectl apply -f servas-app-gitrepo.yaml
```

### Summary

1. **Push the Helm chart files to a Git repository.**
2. **Create and apply a Fleet `GitRepo` resource that references your Git repository.**

