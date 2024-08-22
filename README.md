# ArgoCD Cluster Boostrapping

This repo serves to act as the source of truth for a GitOps-based ArgoCD-managed Kubernetes cluster.

In order to bootstrap a cluster with this repository, there are two steps: Installing the ArgoCD  Helm Chart and Installing the Bootstrap Helm Chart.

## Boostrapping

### Installing the ArgoCD Helm Chart

To install ArgoCD into your cluster, do the following:

1. Prepare the ArgoCD helm chart:

   ```
   helm dependency update argocd
   ```
2. Install the ArgoCD helm chart on the cluster:

   ```
   helm install argocd argocd -n argocd --create-namespace
   ```
3. Access the Argo-CD webpage by doing the following:

   ```
   kubectl port-forward svc/argocd-server 8080:443 -n argocd
   ```

   To login to the dashboard at `localhost:8080`, use the username "admin", and the password retrieved by this command:

   ```
   kubectl get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" -n argocd | base64 -d
   ```

Now ArgoCD is installed in your cluster and you can login through the WebUI.

#### Private Source Repo Setup

In the case that you want this source-of-truth repository to be private, you will need to add a secret to your cluster that contains the key to your repo.

See the [ArgoCD Documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#repositories) for more info.

There are multiple ways to achieve this, but methods that store an encrypted manifest of the secret in Git are preferred. One such way is to use Sealed Secrets.

#### Adding Private Repo Credentials using Sealed Secrets

TBD

### Installing the Bootstrap Helm Chart

`helm template bootstrap | kubectl apply -f -`

There you go! Now you have a Argo-CD git-ops cluster bootstrapped and ready to go!

## How It Works

1. Apply the bootstrap kustomize overlay corresponding to the environment (dev, staging, prod, etc.) you want.
2.
