# Hello Jenkins X

Ref: https://jenkins-x.io/

Get the `jx` client binary (https://jenkins-x.io/getting-started/install/):

```sh
mkdir ~/bin
curl -L https://github.com/jenkins-x/jx/releases/download/v1.2.142/jx-linux-amd64.tar.gz | tar xzv -C ~/bin
export PATH=$PATH:$HOME/bin
```

Let Jenkins X create a new GKE cluster and install itself on it (https://jenkins-x.io/getting-started/create-cluster/):

```sh
jx create cluster gke --skip-login  --default-admin-password=myStrongPassWord123 -n my-jx-cluster
# go through installation instructions
# obtain GitHub token:
# https://github.com/settings/tokens/new?scopes=repo,read:user,user:email,write:repo_hook
# add DNS entry for JX service
# https://console.cloud.google.com/net-services/dns/zones/
```

Go ahead and create a quickstart project (https://jenkins-x.io/developing/create-quickstart/):

```sh
jx create quickstart
# run through instructions (using python-http)
...
Watch pipeline activity via:    jx get activity -f python-http -w
Browse the pipeline log via:    jx get build logs itamaro/python-http/master
Open the Jenkins console via    jx console
You can list the pipelines via: jx get pipelines
When the pipeline is complete:  jx get applications
...

jx get activity -f python-http -w
```

The app itself will fail because, so fix it and push to trigger an update. Go through a pull request to demonstrate the PR workflow with Jenkins X (see https://github.com/itamaro/python-http-jx-quickstart/commit/9a5265329cef588cbcf0d4247a083660b86fe860).

Cleanup the GKE cluster:

```sh
gcloud container clusters delete my-jx-cluster
```
