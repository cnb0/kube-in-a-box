# Kube In A Box (KIAB)

A local environment to run and test Kubernetes things.  

This will install Kubernetes, Docker and an OSS Nexus server for use as a docker registry within a Virtual machine.

## Requirements
* Vagrant 2+
* Virtualbox 
* kubectl ( `brew install kubernetes-cli` if on a Mac)

If you are on a Mac, simply run my [mac-dev-setup](https://github.com/nickmaccarthy/mac-dev-setup) and the pre-reqs should be met.

## Whats included?
Operating System: Cent OS 7
Software:
* Kubernetes 
* Nexus 3.7x
* Docker 

## Usage:
To get started, simply clone this repo, then within it from CLI, run the command `vagrant up`.  This should download the VM image, start it and run the "provisioners" which actually do all the setup of the verious services for KIAB.

Some useful vagrant commands (note, these should be run from within this directory):
* `vagrant up` - Starts and provisions the VM on your first run.
* `vagrant ssh` - SSH's you into the VM where it can be administered
* `vagrant provision` - Runs the provisioners, i.e. Ansible scripts that configure this system
* `vagrant halt` - Suspends the VM
* `vagrant snapshot` - Snapshots the VM in its current state

## Kubectl 
`kubectl` is a command line utility that is extremely useful for managing Kubernetes.  You can easily install it on your mac with brew, or you can use my [mac-dev-setup](https://github.com/nickmaccarthy/mac-dev-setup) playbook which installs it for you as well.  With `kubectl` installed and configured, you can now manage the kubernetes cluster runningon KIAB from your own mac's terminal.  When KIAB runs intially a `kubeconfig` is created which is synced to your local machine.  You can simply place this file `./kube-config` under `~/.kube/config` and your default config will now point to KIAB.

## Useful kubectl aliases
Here are some useful `kubectrl` aliases I have used to help show pods, services, etc that are running in KIAB.  Simply add these to your local `.bashrc` file or equivelent on your local machine (assuming you have a mac with brew installed):

```
alias pods="kubectl get pods --output=wide --all-namespaces"
alias nodes="kubectl get nodes --all-namespaces"
alias services="kubectl get services --all-namespaces"
alias cjobs="kubectl get cronjobs --all-namespaces"
alias deployments="kubectl get deployments --all-namespaces"

if [ -f $(brew --prefix)/etc/bash_completion ]; then
. $(brew --prefix)/etc/bash_completion
fi

kubectl completion bash > $(brew --prefix)/etc/bash_completion.d/kubectl

source <(kubectl completion bash)
```

## Docker
KIAB installs its own Docker daemon for Kubernetes to use.  This Docker daemon is available externally (i.e. the `docker` command on your mac) can utilize it while KIAB is up.   This is very useful because you dont have to manage two dockers, and the registry are local now.  

To use this, simply just create an Envrionment variable called `DOCKER_HOST` with the example below.  

Example:
```
export DOCKER_HOST="kiab.local:2375"
```

Perhaps add this to your `bash.rc` file in your `HOME_DIR` so it works in the future. 

## Nexus and docker registry
KIAB is currently running Nexus 3 as its docker registry.  This can be accesses on your local machine via [http://kiab.local:8081](http://kiab.local:8081).  To use this Nexus as a container registry you must add KIAB to your `insecure-registries` on your local machine if you are using its local docker.  Alternately you can use the docker deamon hosted by KIAB and this insecure-registy is already added ( see the Docker section above for more info on how to configure this).  Once that is completed, simply run the `docker login http://kiab.local:5000` -  username: `admin`, password: `admin123`.

The Username and Password the Nexus web GUI is `admin`/`admin123`

## FAQ:
Q: Why no just run minikube or kubernetes in Docker?
A: I do sometimes, however I was finding some bugs and issues around minikube to be a real pain to work with on my mac, espicially with later versions of OSx, so I thought it was just easier to run it all in a VM ( which is essentially what Minikube is doing anyways )

## Useful Resources
A primer on using kubectl from a docker user
- https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/


## Traefik as an Ingress Controller
If you set your `kube_ingress_proxy: traefik` in your `kiab.config.yaml`, KIAB will install [Traefik](https://www.traefik.io) as an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) for your KIAB instance.  Out of the box KIAB is going to map and expose the following endpoints for you:

Traefik UI - [http://kiab.local/](http://kiab.local)
Official Kubernetes Dashboard - [http://kiab.local/kube-dashboard](http://kiab.local/kube-dashboard) (note: there is no authentication for this)

If you wanted to add your own Ingress, simply add something like this:
```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: elk-kibana
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.frontend.rule.type: PathPrefixStrip
spec:
  rules:
  - host: kiab.local
    http:
      paths:
      - path: /elk-kibana
        backend:
          serviceName: elk-kibana
          servicePort: 5601
---