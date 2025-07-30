# Install Flux

The below steps are found on:

https://fluxcd.io/flux/get-started/


#### Install the Flux CLI

To install the CLI with Homebrew run:
```sh
brew install fluxcd/tap/flux
```

#### Create a GitHub personal access token with repo permissions

The below steps are found on:

https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens


Go to profile —> setting —> developer settings —> personal access tokens —> tokens (classic) —> generate new token —> generate new classic token

Under select scopes set repo with all child’s 

Give it a name

Generate token

Copy token for use with the export command to bootstrap Flux


#### Export your credentials

Export your GitHub personal access token and username:
```sh
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>
```

#### Check your Kubernetes cluster

Check you have everything needed to run Flux by running the following command:
```sh
flux check --pre
```
The output is similar to:
```sh
► checking prerequisites
✔ kubernetes 1.28.0 >=1.25.0
✔ prerequisites checks passed
```

#### Install Flux onto your cluster

This command bootstraps Flux with a GitHub repository

It sets up Flux to sync from the Homelab repository (your GitHub account), on the main branch, and watches the path ./clusters/production for Kubernetes manifests.

flux bootstrap github \
--owner=$GITHUB_USER \
--repository={repo} \
--branch=main \
--path=./{directory}/{directory} \
--personal

Example:

flux bootstrap github \
--owner=$GITHUB_USER \
--repository=Homelab \
--branch=main \
--path=./clusters/production \
--personal

The output is similar to:
```sh
► connecting to github.com
✔ repository created
✔ repository cloned
✚ generating manifests
✔ components manifests pushed
► installing components in flux-system namespace
deployment "source-controller" successfully rolled out
deployment "kustomize-controller" successfully rolled out
deployment "helm-controller" successfully rolled out
deployment "notification-controller" successfully rolled out
✔ install completed
► configuring deploy key
✔ deploy key configured
► generating sync manifests
✔ sync manifests pushed
► applying sync manifests
◎ waiting for cluster sync
✔ bootstrap finished
```

# Flux commands:

```sh
flux get kustomizations --watch
```
The output is similar to:

```sh
NAME       	REVISION          	SUSPENDED	READY	MESSAGE                              
apps       	main@sha1:4892475b	False    	True 	Applied revision: main@sha1:4892475b	
flux-system	main@sha1:4892475b	False    	True 	Applied revision: main@sha1:4892475b	
monitoring 	main@sha1:4892475b	False    	True 	Applied revision: main@sha1:4892475b
```

Forces a manual sync of the Git source named {repo}, so that Flux immediately pulls the latest changes from the Git repository.
```sh
flux reconcile source git {repo}
```
The output is similar to:

```sh
► annotating GitRepository flux-system in flux-system namespace
✔ GitRepository annotated
◎ waiting for GitRepository reconciliation
✔ fetched revision main@sha1:4892475ba2694c5c111f0c3059ae5f76841bd727
```