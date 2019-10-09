imagefs and Persistent Volume Claim
------------------------------------
EKS supports two types of file system: 

1. ``nodefs`` for pods-scheduling EC2 nodes' root volumes,
2. ``imagefs`` for the containers, which may be used to store the logs and transactions.


Now ``imagefs`` is ephemeral storage by default that lives as long as the container does. To persist its data when the container dies or terminates, a storage class model needs to be brought into EKS that is poised to create persistent storage. Within the AWS environment, you have access to different EBS types, and namely 
the default volume type *General Purpose SSD* (`gp2 <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html>`_). 

The ``ebs-gp2-storage.yaml`` here is a very lightweight manifest to create storage class. The key here is the annotation put into the metadata section that creates a default class of *true* for this particular storage class:

.. code-block:: bash

    kubectl apply -f ebs-gp2-storage.yaml


Persistent Volume
^^^^^^^^^^^^^^^^^

Now with ``gp2`` class in place, and *Retain* as its reclaim policy, a standard Kubernetes mapping can be used to attach storage that maintains persistence even after deleting the ``PersistentVolumeClaim (pvc)`` and the pod that claimed the storage. 

- ``pvc``:                  is a claim, the capability that connects the pod with the persistent volume. 
- ``pv``:                   the persistent volume is the underlying actual storage connection. That's the element that's going to remain persistently.

Mounting in Action
^^^^^^^^^^^^^^^^^^

:``nginx-volume.yaml``: In a ``Deployment API`` for a web server that runs the ``nginx:1.7.9`` Docker image, a 3Gi volume is mounted into its web directory. The ``Service`` domain on top routes the request to the Deployment. 

.. code-block:: bash

    kubectl apply -f nginx-volume.yaml

For the most part, this looks like any other Deployment up to the point of ``spec`` for ``volumes`` provisioned through the ``persistentVolumeClaim``. All it does is it associates the ``nginx-pvc`` with this particular volume name internally. Then, within the container file system, the ``nginx-pvc``, mapped to the volume name, is going to be mounted on ``/www``. 

.. code-block:: yaml

      containers:
      - image: nginx:1.7.9
        imagePullPolicy: Always
        name: nginx
        volumeMounts:
          - mountPath: "/www"
            name: nginx-pvc


Now let's see the ``/www`` directory once this pod is up and running to see how large this volume actually is. If you also look at the ``pvc``, this is what actually creates the underlying storage. 

.. code-block:: bash

    kubectl get pvc -o wide     
    kubectl exec -it $(kubectl get pod -l app=nginx-volume -o jsonpath={.items..metadata.name}) -- df -h /www

The ``exec`` gets a shell into the running container of the ``nginx-volume`` and gives the disk information in a human-readable fashion for the mounted ``pv``.


