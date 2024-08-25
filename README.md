# ArgoCD Cluster Boostrapping

This repo serves to act as the source of truth for a GitOps-based ArgoCD-managed Kubernetes cluster.

In order to bootstrap a cluster with this repository, there are two steps: Installing the ArgoCD  Helm Chart and Installing the Bootstrap Helm Chart.

## Cloning this Repo

To begin, clone or fork this repo and host it on Git.

Once that is done, retrieve the git url of the repo. Replace the repoURL fields in the following files with your git url:

* bootstrap/base/root-app.yaml
* root/base/bootstrap-app.yaml

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

### Configure Values (RepoURLS)

Assuming you created your own repository from this one, you need to make sure that the manifests point to your repo.

To acheive this, do the following:

1. In bootstrap/values.yaml, change the repoURLs to the url of your repo.
2. Do the same as in step #1 but in root/values.yaml.

As you may have noticed, there are other values that can be changed, as well as overridden depending on environment (dev, staging, prod), but we will get to those later.


### OPTIONAL: Private Source Repo Setup

In the case that you want this source-of-truth repository to be private, you will need to add a secret to your cluster that contains the key to your repo.

See the [ArgoCD Documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#repositories) for more info.

There are multiple ways to achieve this, but methods that store an encrypted manifest of the secret in Git are preferred. One such way is to use Sealed Secrets.

#### Adding Private Repo Credentials using Sealed Secrets

TBD

### Installing the Bootstrap Helm Chart

There you go! Now you have a Argo-CD git-ops cluster bootstrapped and ready to go!

## How It Works

1. Apply the bootstrap kustomize overlay corresponding to the environment (dev, staging, prod, etc.) you want.
2.

## Known Limitations

* If wanting to run multiple environments on the same cluster, use a solution like vClusters.
  * Reason: Multiple ArgoCD instances per cluster are not supported without use of a tool like the one above.
* Using Kustomize and Helm Charts together via the Kustomize helm chart inflator generator does not work for charts located in private helm repos. See the[Kustomize Docs here](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/helmcharts/#the-current-builtin).
  * Despite this, Kustomize + Helm is still viable for public charts, and the most common use-case for this will probably be third-party charts. Ideally, the only charts located in private repos are ones you would have less need to apply a Kustomization to in the first place.
  * NOTE: For the helm chart inflation generator to work with ArgoCD in the first place, the following needs to be enabled in ArgoCD's config map:`kustomize.buildOptions: --enable-helm `
