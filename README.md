# Set Up
- To generate `tsconfig.json`, the lecture tells you to run `tsc --init`, but this requires a globally-installed typescript. Since you don't have it, use `npx` instead:
```
npx --package typescript tsc --init
```

- To bypass Chrome warning that `ingress-nginx` uses an unsafe self-signed certificate, you just need to click anywhere on the browser screen
then type out `thisisunsafe`. What a hack!

# Remote Development Environment
- Our entire K8s cluster will be running on Google Cloud VM
- Why we use Google Cloud instead of AWS?
  - Skaffold is developed by Google.
- We need to add configuration to tell Skaffold to use some node on the cloud.
- If we change a synced file
  - Skaffold will just take that changed file and place it on the remote Pod
- If we change a not-synced file
  - Skaffold will rebuild image
  - It will reach out to Google Cloud Build (a GCP service), which takes the source code and Dockerfile and pass it into Docker Builder, which builds the updated image. The reason this is off-loaded to Google Cloud Build is to alleviate the load on our local machine. 
  - Skaffold reaches out to the deployment, telling it a new image is available.
  - The deployment restarts the Pods using the new image.

## Kubectl Contexts
- Behind the scenes, kubectl uses contexts: different connection settings to tell kubectl to connect to different clusters.
- The context we are currently using is the default context created when we first install Docker for Mac/Windows.
- We need to create a new context to connec to the GCP cluster.
  - We can install GCloud SDK which teaches kubectl how to connect to the cloud cluster.

### Installing Gcloud
- Follow the steps [here](https://cloud.google.com/sdk/docs/install-sdk) until (but not including) the `gloud init`
- Then at the command line, authenticate with `gcloud auth login`. You will then be redirected to a browser link, you need to accept permissions grant
- After authenticating, you can now run `gcloud init`, and follow prompts
- Then run `gcloud container clusters get-credentials <cluster_name>` i.e `ticketing-dev`.
  - If you open Docker Desktop icon on the top, scroll to Kubernetes, you will see a new kubectl context with some name ending with `ticketing-dev`. It works!!
  - By switching between different contexts you change what cluster `kubectl` connects to and what it sees i.e the running pods/services/deployments.

Remaining steps to setup remote development environment:
- Enable Google Cloud Build
- Update `skaffold.yaml` to use GCB
  - Add `googleCloudBuild` under `build` section
  - Rename the image to `us.gcr.io/${projectId}/${context}` i.e `us.gcr.io/tidy-alchemy-358010/auth`, because this is the image name Google Cloud Build expects. This has to be done in `auth-depl.yaml` as well.
- Setup `ingress-nginx` on our remote Google Cloud cluster (remember we have only done this for the local cluster)
  - Running the steps from [here](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start) again, followed by
  the specific steps for [GCE-GKE](https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke)
  - After this, a load balancer will be created for our cluster.
- Update our hosts file again so accessing `ticketing.dev` will redirect us to the IP address of the cloud cluster
  - What we really want is the IP address of the load balancer, as this is where traffic needs to go to before entering
  the cluster.
  - Find the load balancer by go to `Networking/Network Services/Load Balancing` on the hamburger menu.
- Finally, restart Skaffold.
  - Note here the Cloud Build interaction might fail i.e you will not see your builds in the History section. Try `gcloud auth application-default login`
  - If you change some files and there are some cryptic errors like [this](https://github.com/GoogleContainerTools/skaffold/issues/7587), just run `rm -rf ~/.skaffold/cache` to clear the cache.


