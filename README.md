# kubernetes-aws


# kubernetes - aws 

https://medium.com/containermind/how-to-create-a-kubernetes-cluster-on-aws-in-few-minutes-89dda10354f4

* brew install awscli

* brew install kops

aws configure
AWS Access Key ID [None]: AccessKeyValue
AWS Secret Access Key [None]: SecretAccessKeyValue
Default region name [None]: us-east-1
Default output format [None]:


ICC11575:Workspaces msah$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************WS7A shared-credentials-file    
secret_key     ****************h1jL shared-credentials-file    
    region                us-east-1      config-file    ~/.aws/config
ICC11575:Workspaces msah$ 


S3 Bucket: 
https://s3.console.aws.amazon.com/s3/#

bucket_name=msah-kops-store
aws s3api create-bucket \
--bucket ${bucket_name} \
--region us-east-1

Enable versioning for the above S3 bucket:
aws s3api put-bucket-versioning --bucket msah-kops-store --versioning-configuration Status=Enabled


Provide a name for the Kubernetes cluster and set the S3 bucket URL in the following environment variables:

export KOPS_CLUSTER_NAME=msah.k8s.local
export KOPS_STATE_STORE=s3://msah-kops-store

Add above code block can be added to the ~/.bash_profile or ~/.profile file depending on the operating system to make them available on all terminal environments.

Create a Kubernetes cluster definition using kops by providing the required node count, node size, and AWS zones. The node size or rather the EC2 instance type would need to be decided according to the workload that you are planning to run on the Kubernetes cluster:


ICC11575:kubernetes-aws msah$ kops create cluster \
> --node-count=2 \
> --node-size=t2.medium \
> --zones=us-east-1a \
> --name=${KOPS_CLUSTER_NAME}
I0914 13:05:27.236536   86197 create_cluster.go:472] Inferred --cloud=aws from zone "us-east-1a"
I0914 13:05:27.679930   86197 subnets.go:184] Assigned CIDR 172.20.32.0/19 to subnet us-east-1a
Previewing changes that will be made:


*********************************************************************************

A new kubernetes version is available: 1.9.9
Upgrading is recommended (try kops upgrade cluster)

More information: https://github.com/kubernetes/kops/blob/master/permalinks/upgrade_k8s.md#1.9.9

*********************************************************************************


SSH public key must be specified when running with AWS (create with `kops create secret --name msah.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub`)
ICC11575:kubernetes-aws msah$ 





* EDIT CLUSTER - Verification
kops edit cluster --name ${KOPS_CLUSTER_NAME}


* create the Kubernetes cluster on AWS by executing kops update command:
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes

* Execute the validate command to check its status and wait until the cluster becomes ready


* NOTES: 

** While doing kops update cluster --name ${KOPS_CLUSTER_NAME} --yes - got the error SSH public key must be specified when running with AWS (create with `kops create secret --name msah.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub`)

** kops create secret --name msah.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub 
   ******* GAVE ERROR ~/.ssh/id_rsa.pub doesnt exist
   ******* Create this file in ~/.ssh folder 
   ******* It gave a different error "Error fingerprinting for user admin: No ssh key found "
   ******* Tried lot of things, nothing worked 
   ******* Tried to kops validate cluster 
        ********** This gave error context doesnt exist for msah.k8s.local
        ********** kubectl config get-contexts -- Shows nothing 
        ********** kubectl config set-cluster with my cluster 
        ********** kubectl config set-context with my cluster 
    ******* ssh-keygen to create the ssh key 
Then it works -- NOT SURE WHat made it work. Creating ssh key using ssh-keygen or adding context in kubectl config 

However, more errors:

ICC11575:kubernetes-aws msah$ kops update cluster --name ${KOPS_CLUSTER_NAME} --yes

*********************************************************************************

A new kubernetes version is available: 1.9.9
Upgrading is recommended (try kops upgrade cluster)

More information: https://github.com/kubernetes/kops/blob/master/permalinks/upgrade_k8s.md#1.9.9

*********************************************************************************

I0914 16:36:12.856754   93036 apply_cluster.go:456] Gossip DNS: skipping DNS validation
I0914 16:36:13.220129   93036 executor.go:91] Tasks: 0 done / 77 total; 30 can run
I0914 16:36:13.852102   93036 vfs_castore.go:731] Issuing new certificate: "apiserver-aggregator-ca"
I0914 16:36:14.061849   93036 vfs_castore.go:731] Issuing new certificate: "ca"
I0914 16:36:14.425151   93036 executor.go:91] Tasks: 30 done / 77 total; 24 can run
I0914 16:36:14.972658   93036 vfs_castore.go:731] Issuing new certificate: "kubecfg"
I0914 16:36:14.989828   93036 vfs_castore.go:731] Issuing new certificate: "apiserver-proxy-client"
I0914 16:36:15.141766   93036 vfs_castore.go:731] Issuing new certificate: "kube-controller-manager"
I0914 16:36:15.234457   93036 vfs_castore.go:731] Issuing new certificate: "kubelet"
I0914 16:36:15.251745   93036 vfs_castore.go:731] Issuing new certificate: "kubelet-api"
I0914 16:36:15.288663   93036 vfs_castore.go:731] Issuing new certificate: "apiserver-aggregator"
I0914 16:36:15.316804   93036 vfs_castore.go:731] Issuing new certificate: "kops"
I0914 16:36:15.324249   93036 vfs_castore.go:731] Issuing new certificate: "kube-scheduler"
I0914 16:36:15.354690   93036 vfs_castore.go:731] Issuing new certificate: "kube-proxy"
I0914 16:36:16.844206   93036 executor.go:91] Tasks: 54 done / 77 total; 19 can run
I0914 16:36:17.426893   93036 launchconfiguration.go:341] waiting for IAM instance profile "masters.msah.k8s.local" to be ready
I0914 16:36:17.465450   93036 launchconfiguration.go:341] waiting for IAM instance profile "nodes.msah.k8s.local" to be ready
I0914 16:36:27.955414   93036 executor.go:91] Tasks: 73 done / 77 total; 3 can run
I0914 16:36:28.967606   93036 vfs_castore.go:731] Issuing new certificate: "master"
I0914 16:36:29.295762   93036 executor.go:91] Tasks: 76 done / 77 total; 1 can run
W0914 16:36:29.678827   93036 executor.go:118] error running task "LoadBalancerAttachment/api-master-us-east-1a" (9m59s remaining to succeed): error attaching autoscaling group to ELB: ValidationError: Provided Load Balancers may not be valid. Please ensure they exist and try again.
	status code: 400, request id: dbbce914-b85d-11e8-a895-292bd6e6c694
I0914 16:36:29.678865   93036 executor.go:133] No progress made, sleeping before retrying 1 failed task(s)
I0914 16:36:39.683766   93036 executor.go:91] Tasks: 76 done / 77 total; 1 can run
I0914 16:36:40.332740   93036 executor.go:91] Tasks: 77 done / 77 total; 0 can run
I0914 16:36:40.746195   93036 update_cluster.go:291] Exporting kubecfg for cluster
kops has set your kubectl context to msah.k8s.local

Cluster changes have been applied to the cloud.



Changes may require instances to restart: kops rolling-update cluster

******* AFTER THIS : 
AWS has instances running. 


NEED TO LOOK AT THIS ERROR from above: BUT BEcause it is about LoadBalancers, moving on for now. 
W0914 16:36:29.678827   93036 executor.go:118] error running task "LoadBalancerAttachment/api-master-us-east-1a" (9m59s remaining to succeed): error attaching autoscaling group to ELB: ValidationError: Provided Load Balancers may not be valid. Please ensure they exist and try again.


Validating the instances:
ICC11575:kubernetes-aws msah$ kops rolling-update cluster
NAME			STATUS	NEEDUPDATE	READY	MIN	MAX	NODES
master-us-east-1a	Ready	0		1	1	1	1
nodes			Ready	0		2	2	2	2

No rolling-update required.
ICC11575:kubernetes-aws msah$ 


Validation using kops: 
`
kops validate cluster
`

ICC11575:kubernetes-aws msah$ kops validate cluster
Validating cluster msah.k8s.local

INSTANCE GROUPS
NAME			ROLE	MACHINETYPE	MIN	MAX	SUBNETS
master-us-east-1a	Master	c4.large	1	1	us-east-1a
nodes			Node	t2.medium	2	2	us-east-1a

NODE STATUS
NAME				ROLE	READY
ip-172-20-35-202.ec2.internal	node	True
ip-172-20-42-124.ec2.internal	node	True
ip-172-20-47-99.ec2.internal	master	True

Your cluster msah.k8s.local is ready
ICC11575:kubernetes-aws msah$ 


* need to deploy the Kubernetes dashboard to access the cluster via its web based user interface:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

ICC11575:kubernetes-aws msah$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
secret/kubernetes-dashboard-certs created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
ICC11575:kubernetes-aws msah$



* Execute the below command to find the admin user’s password:
ICC11575:kubernetes-aws msah$ kops get secrets kube --type secret -oplaintext
pSd3HHbrE5ZpsqnAQEca9NH3k0ru70Lq
ICC11575:kubernetes-aws msah$ 


* Execute the below command to find the Kubernetes master hostname:

ICC11575:kubernetes-aws msah$ kubectl cluster-info
Kubernetes master is running at https://api-msah-k8s-local-724nhq-199393921.us-east-1.elb.amazonaws.com
KubeDNS is running at https://api-msah-k8s-local-724nhq-199393921.us-east-1.elb.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
ICC11575:kubernetes-aws msah$ 


* Access the Kubernetes dashboard using the following URL:
https://api-msah-k8s-local-724nhq-199393921.us-east-1.elb.amazonaws.com/ui

Screenshot: images/login-screen

username: admin
password: pSd3HHbrE5ZpsqnAQEca9NH3k0ru70Lq

* GET ADMIN TOKEN 
kops get secrets admin --type secret -oplaintext

screenshot: images/admin-token

Enter the token got from here

* Console screenshot - images/kube-console




