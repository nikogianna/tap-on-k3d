# Setting up a k3d - TAP cluster

These are instructions on how to set up and use a k3d cluster containing Tanzu Application Platform with own wildcard certificates and wildcard DNS.   

# k3d Cluster

We need to have a cluster where the 443 port of the loadbalancer container will be forwarded, since we are going to be using https with TLS termination on the httpproxy of the contour that comes with TAP. For that reason the **k3d-cluster-config.yml** file can be used. The command to do so is:

``k3d cluster create --config k3d-cluster-config.yml``

If the context doesn't switch automatically, do so using :

``kubectl config get-contexts``
``kubectl config use-context <appropriate context>``

# TLS secret

 Firstly, we need to create the kubectl secret to hold our certificate. We can do that with:

``kubectl create secret tls default-cert --key <key file> --cert <crt file>``   

Where key file is the private key file and crt file is the certificate file. Of course the name of the secret can be anything we choose, as long as that is reflected on the tap-values file that is mentioned on the next section. 

# Install TAP

After the cluster is running use the script to install TAP on it. Use of the **tap-values.yaml** is recommended for basic installation. The only arguments needed are the Tanzu network username and password.

``. tap-install.sh <username> <password>``

Notice the dot at the beginning. It is important in order for the script to run within the existing shell and save the variables to the current environment. 

This will take some time (TAP is a big installation).

# Prepare development environment

After the installation is complete we can set the cluster so that we can start creating workloads. Firstly we need to create a kubectl secret to hold the credentials for the registry where the built images are going to be pushed. An example command fo that is:

```
kubectl create secret -n default docker-registry registry-credentials --docker-server=<server url> --docker-username=<username> --docker-password=<password>
```
Next, we will create the necessary service account, role and rolebinding for the development namespace. This is the namespace where all the workloads are going to be created. All we have to do is apply the secret-placeholders.yaml file. 

``kubectl apply -n default -f secret-placeholders.yaml``

In this example we chose the default namespace. If another namespace is chosen, the above commands have to be adjusted accordingly.

# Create workload

We can now create a workload and watch it be deployed on the cluster. One command is enough.

```
tanzu apps workload create -n workload cat-app --git-repo  https://github.com/nikogianna/cat-app  --git-branch master --type web --label app.kubernetes.io/part-of=cat-app2 --yes
```

We can watch the pods with
 
 ``kubectl get po -w`` 

or the logs with

 ``tanzu apps workload tail <workload name> --since 20m --timestamp``
  
  or use
 
   ``tanzu apps workload get <workload name> -n <namespace>``
 
   to monitor the workload progress.

After a few minutes the app should be deployed on the domain that was set in the **tap-values.yaml** file and it should be secured with TLS.

