<p align="center">
  <a href="https://getbootstrap.com/">
    <img src="https://docs.strangebee.com/cortex/images/cortex-logo.png" alt="Bootstrap logo">
  </a>
</p>

<h3 align="center">Kubcortex</h3>
<p align="center">
  Analysis platform for security researchers - on a Kubernetes cluster in a local development environment.

>Cortex solves two common problems frequently encountered by SOCs, CSIRTs and security researchers in the course of threat intelligence, digital forensics and incident response:
How to analyze observables they have collected, at scale, by querying a single tool instead of several?
How to actively respond to threats and interact with the constituency and other teams?

Comes along with TheHive project. [TheHive](https://thehive-project.org/) is a scalable 3-in-1 open-source and free Security Incident Response Platform designed to make life easier for SOCs, CSIRTs, CERTs and any information security practitioner dealing with security incidents that need to be investigated and acted upon swiftly.

## Terminology
| Term | Meaning | 
|----------|----------|
| MISP | Malware Information Sharing Platform  |
| SOCs | Security operations center |
| CSIRTs  | Computer security incident response team |
| CERT  | Cybersecurity Emergency Response Teams |

## Setup
We are going to install Cortex in a minikube environment in Linux.

I will use Kali Linux 2024.1, but you can use any x86-64 Linux Distro.

Requirements:
- x86-64 Linux VM with:
    - 2 CPUs or more
    - 2GB of free memory
    - 20GB of free disk space
    - Internet connection



---
</br>

### Docker setup and service configuration:
```sh
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker --now
sudo usermod -aG docker $USER
```
You can check the state of the docker daemon like this:
```sh
$ sudo systemctl status docker      
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Mon 2024-04-29 13:05:00 EDT; 8min ago
TriggeredBy: ● docker.socket
[...]
Apr 29 13:05:00 kali systemd[1]: Started docker.service - Docker Application Container Engine.
Apr 29 13:05:00 kali dockerd[1005]: time="2024-04-29T13:05:00.630768653-04:00" level=info msg="API listen on /run/docker.sock"
```

Add yourself to the docker group:
```sh
sudo usermod -aG docker $USER && newgrp docker
```
If you don't follow this process, a message like this one wil appear with minikube start:
```sh
$ minikube start           
😄  minikube v1.33.0 on Debian kali-rolling
👎  Unable to pick a default driver. Here is what was considered, in preference order:
    ▪ docker: Not healthy: "docker version --format {{.Server.Os}}-{{.Server.Version}}:{{.Server.Platform.Name}}" exit status 1: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied
    ▪ docker: Suggestion: Add your user to the 'docker' group: 'sudo usermod -aG docker $USER && newgrp docker' <https://docs.docker.com/engine/install/linux-postinstall/>
💡  Alternatively you could install one of these drivers:
    ▪ kvm2: Not installed: exec: "virsh": executable file not found in $PATH
    ▪ podman: Not installed: exec: "podman": executable file not found in $PATH
    ▪ qemu2: Not installed: exec: "qemu-system-x86_64": executable file not found in $PATH
    ▪ virtualbox: Not installed: unable to find VBoxManage in $PATH

❌  Exiting due to DRV_NOT_HEALTHY: Found driver(s) but none were healthy. See above for suggestions how to fix installed drivers.
```
---
</br>

### Minikube setup:
```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
minikube start
$ minikube start
😄  minikube v1.33.0 on Debian kali-rolling
✨  Automatically selected the docker driver. Other choices: none, ssh
📌  Using Docker driver with root privileges
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.43 ...
💾  Downloading Kubernetes v1.30.0 preload ...
    > preloaded-images-k8s-v18-v1...:  342.90 MiB / 342.90 MiB  100.00% 16.37 M
    > gcr.io/k8s-minikube/kicbase...:  480.29 MiB / 480.29 MiB  100.00% 19.21 M
🔥  Creating docker container (CPUs=2, Memory=3700MB) ...
🐳  Preparing Kubernetes v1.30.0 on Docker 26.0.1 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

$ minikube kubectl -- get pods -A
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubectl:  49.07 MiB / 49.07 MiB [------------] 100.00% 38.00 MiB p/s 1.5s
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-7db6d8ff4d-n6ljv           1/1     Running   0          7s
kube-system   etcd-minikube                      1/1     Running   0          21s
kube-system   kube-apiserver-minikube            1/1     Running   0          21s
kube-system   kube-controller-manager-minikube   1/1     Running   0          21s
kube-system   kube-proxy-2b4hg                   1/1     Running   0          8s
kube-system   kube-scheduler-minikube            1/1     Running   0          21s
kube-system   storage-provisioner                1/1     Running   0          20s
```

Links:
- [Docker install](https://www.kali.org/docs/containers/installing-docker-on-kali/)
- [Minukube Config](https://minikube.sigs.k8s.io/docs/start/)

</br>
</br>

## Apply manifests
Now we are going to download the manifest and apply it to the minikube cluster.
```sh
git clone https://github.com/argb10/kubcortex.git
minikube kubectl -- apply -f kubcortex/manifests
```
The app should be up and running. You can access a service running in minikube with the service command.
```sh
$ minikube service cortex
|-----------|--------|-------------|--------------|
| NAMESPACE |  NAME  | TARGET PORT |     URL      |
|-----------|--------|-------------|--------------|
| default   | cortex |             | No node port |
|-----------|--------|-------------|--------------|
😿  service default/cortex has no node port
❗  Services [default/cortex] have type "ClusterIP" not meant to be exposed, however for local development minikube allows you to access this !
🏃  Starting tunnel for service cortex.
|-----------|--------|-------------|------------------------|
| NAMESPACE |  NAME  | TARGET PORT |          URL           |
|-----------|--------|-------------|------------------------|
| default   | cortex |             | http://127.0.0.1:39769 |
|-----------|--------|-------------|------------------------|
🎉  Opening service default/cortex in default browser...
```
Browser will open and you will see something like this:
![alt text](/images/image.png)


After admin config, we will see something like this:
![alt text](/images/image-1.png)


Follow the [guide](https://docs.strangebee.com/cortex/user-guides/first-start/) for more information.

---

</br></br></br>


# Manifest generation
Since the vendor already provides a docker-compose.yml (which we cannot use for minikube), I translated the vendor information in some manifest that we can consume.
I'm using [kompose](https://kompose.io/getting-started/):
```sh
curl -L https://github.com/kubernetes/kompose/releases/download/v1.33.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
curl https://raw.githubusercontent.com/TheHive-Project/Cortex/master/docker/cortex/docker-compose.yml -o docker-compose.yml
$ kompose convert
WARN The "job_directory" variable is not set. Defaulting to a blank string. 
WARN The "job_directory" variable is not set. Defaulting to a blank string. 
WARN The "job_directory" variable is not set. Defaulting to a blank string. 
WARN /home/kali/Documents/cortex/docker-compose.yml: `version` is obsolete 
WARN Service "elasticsearch" won't be created because 'ports' is not specified 
WARN Volume mount on the host "/path/to/data" isn't supported - ignoring path on the host 
INFO Kubernetes file "cortex-service.yaml" created 
INFO Kubernetes file "cortex-deployment.yaml" created 
INFO Kubernetes file "elasticsearch-deployment.yaml" created 
INFO Kubernetes file "elasticsearch-claim0-persistentvolumeclaim.yaml" created
```
<a name="Manifestgen"></a>
As we can see in the warnings, there are some drawbacks to using kompose.
- To solve the connectivity problem I removed the "elasticsearch-deployment.yaml" and added the nodes to the "cortex-deployment.yaml".
- Since elasticsearch is going to be ONLY the DB of this app, it does not need to be exposed with a service (also recommended, if we follow the least privilege principle).
- Since we are not loading data into "/path/to/data", a value mount is needed.
- Probably we do not need to load config into the cortex, if so we will have to create a volume mount for "application.conf".



## Useful resources
To check all the services and config via GUI, you can use the dashboard creation.
Go to the terminal an use this:
```sh
minikube addons enable metrics-server
minikube dashboard
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:42067/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

## Assumptions
- This is just a temporary deployment, a temporary minikube is enough.
- As commented in [Manifest generation](#Manifestgen), some warning were skipped for this exercise. If for any means we need to load configuration, expose elasticsearch or add more service to the deployment the manifest will need to be changed.
