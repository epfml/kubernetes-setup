Instruction for using the container cluster (Kubernetes, k8s)

You can refer to [this repository](https://github.com/EPFL-IC/caas) for more information, below are steps for most common setups.

---

## Table of Contents
- [Requesting access](#requesting-access)
- [Installing Kubernetes](#installing-kubernetes)
- [Setting up Kubernetes](#setting-up-kubernetes)
- [Using Kubernetes](#using-kubernetes)
- [Note on Storage across icclusters](#note-on-storage-across-icclusters)
- [Other resources and deployment templates](#other-resources-and-deployment-templates)

## Requesting access
Use [this form](https://support.epfl.ch/help?id=epfl_sc_cat_item&sys_id=8cd2b9284f1b1b00fe35adee0310c769&sysparm_category=7707db6d4fd94300fe35adee0310c708) to request access (use Accréditation=MLO).

Once you have been approved, you will get an email from the IC with a zip file named <your-name>.zip.
Unzip it and put these files into the `.kube` folder of your home directory. Then rename `<your-name>.config` to `config`.

```bash
cd ~
mkdir .kube
# Save your-name.zip in the .kube folder
mv .kube/*.config .kube/config
```

## Installing the Kubernetes client on your personal machine

### Ubuntu/Debian
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

### MacOS
```bash
brew install kubernetes-cli
```

### Others
Follow instructions on [the kubernetes docs](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl).


## Setting up Kubernetes

To use a kubernetes pod, you need to:
 - [Create a Dockerfile to describe the experimental environment](#creating-a-dockerfile)
 - [Build a Docker image from it](#building-a-docker-image)
 - [Push the docker image to ic-registry.epfl.ch/mlo/](#pushing-the-docker-image)
 - [Create a kubernetes config file](#creating-a-kubernetes-config-file)

### Creating a Dockerfile

You can build your own Dockerfile based on this [basic example](https://github.com/epfml/kubernetes-setup/blob/master/templates/pod-simple/Dockerfile).\
You can specify the software and Python packages you need in there.

To make sure you can access data on the network storage `mlodata1` and `mloraw`, you should make sure that the main user in your docker image has the same user ID as you have on EPFL's system.\
To achieve this, put your Gaspar ID after `NB_USER=` and your UID after `NB_UID=`.\
You can get you uid by using the `id` command on a an iccluster node.

The `FROM` line allows you to choose an image to start from. You can choose from images on the [Dockerhub](https://hub.docker.com/) (or elsewhere).

### Building a Docker image

Once you are happy with the Dockerfile, go to the directory of the Dockerfile and run:
```bash
docker build . -t <your-tag>
```
Replace `<your-tag>` by the name you want to give to this Docker image.\
It is good practice to put your username first, for example `jaggi-base`.

### Pushing the Docker image
When you start a pod on the Kubernetes cluster, you tell it to run your docker image.\
The cluster will search for your image on EPFLs internal docker repository [Harbor](https://ic-registry.epfl.ch/).\
You should upload your create image to this repository.

Go have a look at https://ic-registry.epfl.ch and use your gaspar credentials to login in. \
There already is a group project named `mlo`. Please ask someone in the lab already using kubernetes to add you to the mlo group so that you can push your Docker image to that repository.

Now take the following steps:


#### Login to Harbor on your personal machine

Login to the server by running the following command

```bash
docker login ic-registry.epfl.ch
```

and enter the credentials:

Username: `robot$mlo-image-publisher`
Password:
```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1ODg5MjIwNjEsImlhdCI6MTU4NjMzMDA2MSwiaXNzIjoiaGFyYm9yLXRva2VuLWlzc3VlciIsImlkIjozLCJwaWQiOjYsImFjY2VzcyI6W3siUmVzb3VyY2UiOiIvcHJvamVjdC82L3JlcG9zaXRvcnkiLCJBY3Rpb24iOiJwdXNoIiwiRWZmZWN0IjoiIn1dfQ.RI9AhLg94Y1piWKZ4cR-UWOFw39fX3NnuoPM93Ux6T2BR6azMYNUDGSSD8-p17dX82UkaJ4jAYfa19b6e1VudT-YM21QWjbWjWnnnpLfe0wEpL8Y9ddP8kbgfWzhZKxt_RNe8Wl7QjTxQYjp-bdKw1CY1v8NGoltf3nILLYbr8g4RiODQsB-XBCVfCxFUZcQkhy39hq9ckPbEs3jh2HuN7s3IRQGfRAbkQ5DKo1wp967Zkf1LYopF6-W8hZyWq69XzsqixX6UaF8izaZVCGkqPqw1DlgKp6Lropwnb8GT9TjV_kUfX6A6Ju3yEgBtcFEOWCKDYeRlFgPuu1DW3Sy7dnHeDOUNvSeB8ANgY-QesSasQ8LSrFjIcsG9fZ8I_NBCx0CEQCenoRMQHpUqDFZoyFbMaq8zrEO9yshEhHggLoTTd6GHDByxqmWN15dfCZrHHGKmSwW34t5q_a6fsELuAtCmy8j-FvdbB3zQiJF8dj58DKDbIya4R8GdoFq0hOUopZsetUpHAhMwnJ3TRJrJVo7IXzzjT6i5q85qoOHEwPpr0UJHK05zGSXsjoKzTMG26togEnd6GlApuzWpEF21f0eYHib-pkJY1oCcQpobiFKrSwvcYVyjUMaxFMLm1le16Lpk83CEXgstSgSPx_lB1qwMK7zauqpFhrHI-Fc7mQ
```

This is a 'one-in-a-lifetime' steps.

#### Actually pushing the Docker image
To push an image to a private registry (and not the central Docker registry) you must tag it with the registry hostname.\
Then you can push it:
```bash
docker tag <your-tag> ic-registry.epfl.ch/mlo/<your-tag>
docker push ic-registry.epfl.ch/mlo/<your-tag>
```

### Creating a Kubernetes pod config file
Have a look at (and download) this simple [kubernetes config file](https://github.com/epfml/kubernetes-setup/blob/master/templates/pod-simple/pod-gpu-mlodata.yaml).
Fill all elements that are in \<brackets\> .\
`<your-pod-name>` needs not be the same as `<your-docker-image-tag>` but again it is good practice to put your name first for the pod name, for example `jaggi-pod`.

In this config file,
 - you can change: `nvidia.com/gpu: 1` to request more or fewer gpus
 - you can see at the end that mlodata1 is mounted. You can remove it or change it for mloscratch
 - you specify which command is run when launching the pod. Here it will sleep for 60 seconds and then stop

 #### Commands

- To have a container run forever, you can use:
   ```yaml
   command: [sleep, infinity]
  ```
  and then you can [connect to the pod through ssh](#ssh-to-a-pod) and run your jobs from there.

  **If you do this, make sure to [delete the pod](#deleting-a-pod) once you are done to free the resource !**

- To run more complex or multiple commands, you can do:
  ```yaml
   command: ["/bin/bash", "-c"]
   args: ["command1; command2 && command3"]
  ```

  For example:
  ```yaml
   command: ["/bin/bash", "-c"]
   args: ["cd /mlodata1/jaggi/ml && python automl.py"]
  ```

  _The resource will be automatically freed once the command has run. The pod gets status `Completed` but is not deleted._


## Using Kubernetes
### Creating a pod
Go to the directory where your kubernetes config file is and run:
```bash
kubectl create -f <your-configfile-name>.yaml
```

### Checking pods status
```bash
kubectl get pods  # get all pods
kubectl get pods -l user=jaggi  # filter by label (defined in the config file)
kubectl get pod jaggi-pod  # get by pod name
```

### SSH to a pod
```bash
kubectl exec -it jaggi-pod /bin/bash
```

### Deleting a pod
```
kubectl delete pod jaggi-pod
```

### Getting information on a pod
Useful for debugging
```bash
kubectl describe pod jaggi-pod
kubectl get pod jaggi-pod -o yaml
kubectl logs jaggi-pod
```

## Note on Storage across icclusters
### (`mounting /mlo-container-scratch`)
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
### (`mounting /mlodata1`)
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


## Other Resources and deployment templates
Here you can find some kubernetes templates:
* [`job` mode](https://github.com/epfml/kubernetes-setup/tree/master/templates/pod-job).
* [`standalone` mode](https://github.com/epfml/kubernetes-setup/tree/master/templates/pod-standalone).
* [`cluster` mode](https://github.com/epfml/kubernetes-setup/tree/master/templates/pod-cluster).

And some personalized Dockerfile:
- [A Dockerfile from Thijs](https://github.com/epfml/job-monitor/blob/master/docker/worker/Dockerfile)
- [A Dockerfile from Tao](https://github.com/IamTao/beta-kubernetes/blob/master/images/base/Dockerfile)

and some more [documentation from Tao's github](https://github.com/IamTao/beta-kubernetes).

## Some Tips (deprecated)
* By default, a Docker container will run as root. This means that the files you write in the shared storage are owned by root. This is solved by changing the default user in Docker (which is already done in the simple [Dockerfile](https://github.com/epfml/kubernetes-setup/blob/master/templates/pod-simple/Dockerfile#L32-L45))
(Here another [example from Tao](https://github.com/IamTao/beta-kubernetes/blob/29515feb07e953bf602339a7548461aeeaa59de2/images/base/Dockerfile#L56-L72))
* To avoid the error `sudo: no tty present and no askpass program specified`, please use `sudo -S xxx`.
