Instruction of using the container cluster (Kubernetes, k8s)

---

## Requesting access
Use [this form](https://support.epfl.ch/help?id=epfl_sc_cat_item&sys_id=8cd2b9284f1b1b00fe35adee0310c769&sysparm_category=7707db6d4fd94300fe35adee0310c708) to request access (use Accréditation=MLO).

Once you have been approved, you will get an email from the IC with a zip file named <your-name>.zip.
Unzip it and put these files into the `.kube` folder of your home directory. Then rename `<your-name>.config` to `config`.
```
cd ~
mkdir .kube
# Save your-name.zip in the .kube folder
mv .kube/*.config .kube/config
```

You can refer to [this repository](https://github.com/EPFL-IC/caas) for more information, below are steps for most common setups.

## Installing Kubernetes (Ubuntu/Debian)

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

Check that it is working by running:
```bash
kubectl get pods
```

## Kubernetes basics

To use a kubernetes pod, you need to:
 - Create a dockerfile with your needed config
 - Build this docker image
 - Push the docker image to ic-registry.epfl.ch/mlo/
 - Create a kubernetes config file
 - Create a pod
 
### Creating a dockerfile

If you are new to docker, have a look at this very simple [Dockerfile](todo). You should guess what is happening and add your own config.

Put your gaspar id after `NB_USER=` and your uid after `NB_UID=`.\
You can get you uid by using the `id` commmand on a cluster.

The `FROM` line allows you to choose an image to start from. You can choose from images on the [Dockerhub](https://hub.docker.com/) (or elsewhere).

### Building the Docker image

Once you are happy with the Dockerfile, go to the directory of the Dockerfile and run:
```bash
docker build . -t <your-tag>
```
Replace `<your-tag>` by the name you want to give to this docker image. It is good practice to put your name first, for example `jaggi-base`.

### Pushing the Docker image
When you will create a pod, the server will need a Docker image to build a container for you.\
The server will go look for the docker image on https://ic-registry.epfl.ch/, so you should put your Docker image there.

#### Login to ic-registry.epfl.ch/mlo/
Login to the server by running the following command and entering your epfl credentials:
```bash
docker login ic-registry.epfl.ch
```
This is a 'one-in-a-lifetime' steps.

#### Actually pushing the Docker image
To push an image to a private registry (and not the central Docker registry) you must tag it with the registry hostname.\
Then you can push it:
```bash
docker tag <your-tag> ic-registry.epfl.ch/mlo/<your-tag>
docker push ic-registry.epfl.ch/mlo/<your-tag>
```

### Creating a kubernetes config file
Have a look at (and download) this simple [kubernetes config file](todo).
Fill all elements that are in brackets.\
`<your-pod-name>` needs not be the same as `<your-docker-image-tag>` but again it is good practice to put your name first for the pod name.

In this config file,
 - you can change: `nvidia.com/gpu: 1` to request more gpus.\
 - you can see at the end that mlodata1 is mounted. You can remove it or change it for mloscratch.
 - you specify which command is run when launching the pod. Here it will sleep for 60 seconds and then die.
 
To run more complex or multiple commands, you can do:
```yaml
 command: ["/bin/bash", "-c"]
 args: ["command1; command2 && command3"]
```

#### Approaches to running a job
There are two approaches to running pods on the container cluster:
* Like in the `Kubernetes basics`, with command: `[sleep, infinity]`, and then connecting to the pod over ssh to run an experiment
    * This can be convenient for playing around. You can temporarily spin up as many nodes as you want
    * But you pay GPU time you don't use.
* Use something like `command: [run, my, experiment]`.
    * This makes debugging slightly harder, but as soon as your job finishes, the pod gets status `Completed`, and you (Martin) will stop paying for the pod.

### Creating a pod
Go to the directory where your kubernetes config file is and run:
```bash
kubectl create -f <your-configfile-name>.yaml
```

#### Useful kubernetes commands
Check which pods are running
```bash
kubectl get pods  # get all pods
kubectl get pods -l user=jaggi  # filter by label (defined in the config file)
kubectl get pod jaggi-pod  # get by pod name
```

Delete a pod:
```
kubectl delete pod jaggi-pod
```

Debug a pod:
```bash
kubectl describe pod jaggi-pod
kubectl get pod jaggi-pod -o yaml
```



### Note on Storage across icclusters
#### (`mounting /mlo-container-scratch`)
Follow the instructions in `Kubernetes basics`, and use
```yaml
volumeMounts:
- mountPath: /scratch
   name: mlo-scratch
   subPath: YOUR_USERNAME
```

and

```yaml
volumes:
- name: mlo-scratch
   persistentVolumeClaim:
   claimName: mlo-scratch
```
#### (`mounting /mlodata1`)
```yaml
spec:
  volumes:
  - name: mlodata1
    persistentVolumeClaim:
      claimName: pv-mlodata1
  containers:
  - name:  ubuntu
    volumeMounts:
    - mountPath: /mlodata1
      name: mlodata1
```

### Custom your own docker image
Go to `https://ic-registry.epfl.ch` and use your gaspar to login in.

There already has a group project named `mlo`. Please ask the owner of the group project to give you the corresponding permission so that you can push your docker image to that repository.

Once you get the image and have the permission, you can push to the remote host, e.g.,
```sh
docker push ic-registry.epfl.ch/mlo/ml:1.0
```

### Some deployment template
You can find some provided templates, e.g.,
* [`job` mode](https://github.com/epfml/kubernetes-setup/tree/master/templates/pod-job).
* [`standalone` mode](https://github.com/epfml/kubernetes-setup/tree/master/templates/pod-standalone).
* [`cluster` mode](https://github.com/epfml/kubernetes-setup/tree/master/templates/pod-cluster).

## Some Tips
* By default, a Docker container will run as root. This means that the files you write in the shared storage are owned by root. You can solve this by changing the default user in Docker ([example from Tao](https://github.com/IamTao/beta-kubernetes/blob/29515feb07e953bf602339a7548461aeeaa59de2/images/base/Dockerfile#L56-L72))
* To avoid the error `sudo: no tty present and no askpass program specified`, please use `sudo -S xxx`.
