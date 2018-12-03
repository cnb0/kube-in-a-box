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

Some useful vagrant commands:
* `vagrant up` - Starts and provisions the VM on your first run.
* `vagrant provision` - Runs the provisioners, i.e. Ansible scripts that configure this system
* `vagrant halt` - Suspends the VM
* `vagrant snapshot` - Snapshots the VM in its current state

## Kubectl 
`kubectl` is a command line utility that is extremely useful for managing Kubernetes.  You can easily install it on your mac with brew, or you can use my [mac-dev-setup](https://github.com/nickmaccarthy/mac-dev-setup) playbook which installs it for you as well.  With `kubectl` installed and configured, you can now manage the kubernetes cluster runningon KIAB from your own mac's terminal.  When KIAB runs intially a `kubeconfig` is created which is synced to your local machine.  You can simply place this file `./kube-config` under `~/.kube/config` and your default config will now point to KIAB.

## Useful kubectl aliases
Here are some useful `kubectrl` aliases I have used to help show pods, services, etc that are running in KIAB.  Simply add these to your local `.bashrc` file or equivelent on your local machine:

```
alias pods="kubectl get pods --output=wide --all-namespaces"
alias nodes="kubectl get nodes --all-namespaces"
alias services="kubectl get services --all-namespaces"
alias cjobs="kubectl get cronjobs --all-namespaces"
alias deployments="kubectl get deployments --all-namespaces"
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

## FAQ:
Q: Why no just run minikube or kubernetes in Docker?
A: I can, and I do sometimes, however I was finding somethings about minikube to be a real pain to work with on my mac, espicially with later versions of OSx, so I thought it was just easier to run it all in a VM ( which is essentially what Minikube is doing anyways )

## Useful Resources
A primer on using kubectl from a docker user
- https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/
