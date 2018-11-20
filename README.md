Instruction of using the container cluster (Kubernetes, k8s)

---

## Requesting access
Use [this form](https://support.epfl.ch/help?id=epfl_sc_cat_item&sys_id=8cd2b9284f1b1b00fe35adee0310c769&sysparm_category=7707db6d4fd94300fe35adee0310c708) to request access (use Accréditation=MLO).


## Kubernetes basics
Please refer to [this repository](https://github.com/EPFL-IC/caas) for your basic setup.


### Running a job
There are two approaches to running pods on the container cluster:
* Like in the `Kubernetes basics`, with command: `[sleep, infinity]`, and then connecting to the pod over ssh to run an experiment
    * This can be convenient for playing around. You can temporarily spin up as many nodes as you want
    * But you pay GPU time you don't use.
* Use something like `command: [run, my, experiment]`.
    * This makes debugging slightly harder, but as soon as your job finishes, the pod gets status `Completed`, and you (Martin) will stop paying for the pod.


### Storage across icclusters (`mounting /mlo-container-scratch`)
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

### Custom your own docker image
Go to `https://ic-registry.epfl.ch` and use your gaspar to login in.

There already has a group project named `mlo`. Please ask the owner of the group project to give you the corresponding permission so that you can push your docker image to that repository.

Once you get the image and have the permission, you can push to the remote host, e.g.,
```sh
docker push ic-registry.epfl.ch/mlo/ml:1.0
```

### Some deployment template
You can find some provided templates, e.g.,
* [`job` mode](https://github.com/epfml/kubernetes-setup/templates/pod_job).
* [`standalone` mode](https://github.com/epfml/kubernetes-setup/templates/pod_standalone).
* [`cluster` mode](https://github.com/epfml/kubernetes-setup/templates/pod_cluster).


## Some Tips
* By default, a Docker container will run as root. This means that the files you write in the shared storage are owned by root. You can solve this by changing the default user in Docker ([example from Tao](https://github.com/IamTao/beta-kubernetes/blob/29515feb07e953bf602339a7548461aeeaa59de2/images/base/Dockerfile#L56-L72))
* To avoid the error `sudo: no tty present and no askpass program specified`, please use `sudo -S xxx`.
