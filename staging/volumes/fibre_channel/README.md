## Step 1. Setting up Fibre Channel Target

On your FC SAN Zone manager, allocate and mask LUNs so Kubernetes hosts can access them.

## Step 2. Creating the Pod with Fibre Channel persistent storage

Once you have installed Fibre Channel initiator and new Kubernetes, you can create a pod based on my example [fc.yaml](fc.yaml). In the pod JSON, you need to provide *targetWWNs* (array of Fibre Channel target's World Wide Names), *lun*, and the type of the filesystem that has been created on the lun, and *readOnly* boolean.

Once your pod is created, run it on the Kubernetes master:

```console
kubectl create -f ./your_new_pod.json
```

Here is my command and output:

```console
# kubectl create -f examples/volumes/fibre_channel/fc.yaml
# kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
fcpd      2/2       Running   0          10m
```

On the Kubernetes host, I got these in mount output

```console
#mount |grep /var/lib/kubelet/plugins/kubernetes.io
/dev/mapper/360a98000324669436c2b45666c567946 on /var/lib/kubelet/plugins/kubernetes.io/fc/500a0982991b8dc5-lun-2 type ext4 (ro,relatime,seclabel,stripe=16,data=ordered)
/dev/mapper/360a98000324669436c2b45666c567944 on /var/lib/kubelet/plugins/kubernetes.io/fc/500a0982991b8dc5-lun-1 type ext4 (rw,relatime,seclabel,stripe=16,data=ordered)
```

If you ssh to that machine, you can run `docker ps` to see the actual pod.

```console
# docker ps
CONTAINER ID        IMAGE                                  COMMAND             CREATED             STATUS              PORTS               NAMES
090ac457ddc2        kubernetes/pause                       "/pause"            12 minutes ago      Up 12 minutes                           k8s_fcpd-rw.aae720ec_fcpd_default_4024318f-4121-11e5-a294-e839352ddd54_99eb5415   
5e2629cf3e7b        kubernetes/pause                       "/pause"            12 minutes ago      Up 12 minutes                           k8s_fcpd-ro.857720dc_fcpd_default_4024318f-4121-11e5-a294-e839352ddd54_c0175742   
2948683253f7        k8s.gcr.io/pause:0.8.0   "/pause"            12 minutes ago      Up 12 minutes                           k8s_POD.7be6d81d_fcpd_default_4024318f-4121-11e5-a294-e839352ddd54_8d9dd7bf       
```

## Multipath

To leverage multiple paths for block storage, it is important to perform the
multipath configuration on the host.
If your distribution does not provide `/etc/multipath.conf`, then you can
either use the following minimalistic one:

    defaults {
        find_multipaths yes
        user_friendly_names yes
    }

or create a new one by running:

    $ mpathconf --enable

Finally you'll need to ensure to start or reload and enable multipath:

    $ systemctl enable multipathd.service
    $ systemctl restart multipathd.service

**Note:** Any change to `multipath.conf` or enabling multipath can lead to
inaccessible block devices, because they'll be claimed by multipath and
exposed as a device in /dev/mapper/*.

Some additional informations about multipath can be found in the
[iSCSI documentation](../iscsi/README.md)


<!-- BEGIN MUNGE: GENERATED_ANALYTICS -->
[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/examples/volumes/fibre_channel/README.md?pixel)]()
<!-- END MUNGE: GENERATED_ANALYTICS -->
