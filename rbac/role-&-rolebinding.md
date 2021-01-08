The Role and RoleBinding define in role-&-rolebinding.yml file creates a Normal User

Normal users are assumed to be managed by an outside, independent service. It is assumed that 
a cluster-independent service manages normal users in the following ways:

    - an administrator distributing private keys
    - a user store like Keystone or Google Accounts
    - a file with a list of usernames and passwords

So we will use a production standard way to creating and authenticating a Normal User to access a Kubernetes Cluster. So we will be implement the first approach: an administrator (the k8s super user) creates a normal user and distributes private keys to the created user to enable the user authenticate securely to the K8s Cluster.

STEPS - Do these in GCP Cloud Shell
Create a user account for the new joiner: russell
$ sudo useradd russell

Grant password to the user. In prod, give the user key files (SSH keys) and not password
$ sudo passwd russell # enter your desired password

Create Namespace for our new user
$ kubectl create ns russ-playground

New user sign-on - Lets switch User
$ su - russell # enter password

To view the configuration you are using whenever you issue kubectl command, run:
$ kubectl config view

A User can only access the cluster after configuring his environment with the Cluster Name and Server URL provided by the Cluster Admin (Super User) by running:
$ kubectl config view # empty output
$ kubectl config set-cluster cloudkite.k8s.local --server=https://api-cloudkite-k8s-local.....com
$ kubectl config view # you will now see that the cluster name & server have been set

Next, set the context
$ kubectl config set-context russcontext --user russell --cluster cloudkite.k8s.local
$ kubectl config view
$ kubectl config use-context russcontext

In k8s, there is no built-in system of authenticating users. One way to enable a created User to be able to authenticate to the Cluster, is for the super user (cluster-admin) to issue/distribute to the created user: Normal User (russell) private keys (certificate) for him to be able to authenticate to the cluster.

As a Cluster Admin, how to issue a K8s signed X.509 certificate to a created user?

The k8s cluster only accept kubectl commands/requests that have been signed by a certificate, and that certificate must have been issued by the k8s cluster itself. -> So there is a Kubernetes Certificate Authority (CA) in every created Kubernetes cluster.
Be it a GKE, AKS, EKS, or KOPs managed k8s cluster, a certificate is created/issued automatically to the Cluster-Admin when the k8s cluster is created.
On the other hand, if a Cluster-Admin creates a user, this user must be issued a certificate to enable the use authenticate to the cluster.

Generate a private key for user: Russell
$ openssl genrsa -out private-key-russell.key 2048

Create a certificate signing request (.csr file) using the private key created above
$ openssl req -new -key private-key-russell.key -out russell-req.csr -subj "/CN=russell/O=russell"

NB: CN equals Username to be define in your RoleBinding/ClusterRoleBinding. O represents the organization our User is in and I think in terms of a Linux system, this just means their Linux Group name. And by default, a user will called Russell will be in Russell group.

NB: If you are setting this certificate for a website, at this point you have to send the created .csr file to the company you are buying the certificate from. And if they believe you are who you say you are, they will run that CSR file together with your private key through a cryptography routine, and out of the end will come a certificate.

CREATING THE FINAL CERTIFICATE
For this use case, we will need the K8s Cluster Certificate Authority. It will be responsible for approving the request and for generating the necessary certificate Russell will use to access the Cluster.

Now, we need to get access to two files in our K8s Cluster
    - A Private Key e.g cluster-key.key
    - A Certificate - to be use as the certificate authority e.g cluseter-ca.crt

Now we can generate the final certificate by approving the certificate sign request: russell-req.csr

$ openssl x509 -req \
    -in russell-req.csr \
    -CA cluseter-ca.crt \
    -CAkey cluster-key.key \
    -CAcreateserial \
    -out russell-ca.crt \
    -days 365

The above will create: russell-ca.crt