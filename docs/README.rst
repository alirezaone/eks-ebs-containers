eks-ebs-containers
------------------
This is a guide around mounting persistent AWS/EBS for containers into EKS clusters. In order to have EKS environment available to you to run Kubernetes, there are a couple of precursor stages to do.

First, EKS needs to be able to create resources in a constrained environment (VPC) to launch its environment. Then, since EKS is managing the control plane for you, you can add worker nodes. Depending on the EKS-optimized AMI for your region, there's a CloudFormation template (`us-west-2 <https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-08-30/amazon-eks-nodegroup.yaml>`_). Grab the worker ARN (NodeInstanceRole) from the CloudFormation output to authenticate those worker nodes into your EKS cluster. And you're going to do that by applying a config map, ``aws-auth-cm.yaml`` document for passing in information into the control plane:

.. code-block:: yaml

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: aws-auth
      namespace: kube-system
    data:
      mapRoles: 
        - rolearn: arn:aws:iam::<Your_NodeInstanceRole_ARN>
          username: system:node:{{EC2PrivateDNSName}}
          groups:
            - system:bootstrappers
            - system:nodes

Install & Run aws-iam-authenticator Binary
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This binary is required to authenticate your EKS users against the backend IAM system.

*Linux:*
.. code-block:: bash

    curl -sLO https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/aws-iam-authenticator

*MacOS:*
.. code-block:: bash
    curl -sLO https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/darwin/amd64/aws-iam-authenticator

*Windows:*
.. code-block:: bash
    curl -sLO https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/windows/amd64/aws-iam-authenticator.exe

Create Your kubeconfig File
^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can create the `kubeconfig <https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html>`_ file by AWS CLI or manaually. On CLI, simply run ``update-kubeconfig`` command:
.. code-block:: bash
    aws eks --region <region_name> update-kubeconfig --name <cluster_name>

Verify Kubernetes access
^^^^^^^^^^^^^^^^^^^^^^^^

To confirm your access to online nodes, run: 
.. code-block:: bash
    kubectl get nodes

Here, a manifest like a simple hostname service is launched where it creates a little micro webserver with a *www directory* in it. Look up its ``yaml`` for the full disclosure: 
.. code-block:: bash
    kubectl apply -f hostname.yaml
