# kubernetes
Notes on tutorials, projects using k8s 

# Project_1: Creating a EKS cluster and deploy 2048 game

I'm watching this YT https://www.youtube.com/watch?v=wyad99QMKtc&t=24s to use to EKS 

## Create a cluster 
- Using the AWS Management Console 
- Configuration options allow EKS with Auto Mode - this is not in the video. However, there is an option for a custom configuration which I will try to be aligned with the video.
- Cluster name: eks-cluster-1 
- K8s version 1.34
- Created a new role to assign to the cluster in IAM console. However, the role was missing recommended managed policies and a trust policy, so I added those permissions and updated the role.
- Used the default VPC, but I deleted 2 of the 5 subnets, so there are only 3 subnets.
- At step 4 (select add-ons), I left the default add-ons which are more than what was in the video (Node monitoring agent, Amazon VPC CNI, CoreDNS, kube-proxy, Amazon EKS Pod Identity Agent, External DNS, Metrics Server) and left the default versions at step 5. 

## Creating a node group
- Creating a IAM polices for the node group (AmazonEC2ContainerRegistryReadOnly; AmazonEKS_CNI_Policy; AmazonEKSWorkerNodePolicy)
- node group: eks-node-group-2

## Authenticate to the cluster using CloudShell 
- Viewing account details: 
`aws sts get-caller-idenity`
- Creating kubeconfig file: `aws eks update-kubeconfig --region region-code --name my-cluster`
- Confirmed kube file was created: `cat .kube/config`

## Creating the pod and configuration file for the pod
- installing nano: `sudo yum install nano -y`
- inserted the pod with the code: 
`apiVersion: v1
kind: Pod
metadata:
   name: 2048-pod
   labels:
      app: 2048-ws
spec:
   containers:
   - name: 2048-container
     image: blackicebird/2048
     ports:
       - containerPort: 80
`
- creating the pod: `kubectl apply -f 2048-pod.yaml`
- confirming the pod was created: `kubectl get pods`
- Got an error message, and the pod did not run. Troubleshooted through AI,`kubectl describe pod 2048-pod.yaml` the suggestion was to edit my code because they was an architecture compatibility issue with my AWS environment. New code: 
`apiVersion: v1
kind: Pod
metadata:
   name: 2048-pod
   labels:
      app: 2048-ws
spec:
   containers:
   - name: 2048-container
     image: public.ecr.aws/kishorj/docker-2048:latest
     ports:
       - containerPort: 80
`
- deleting the old failed pod: `kubectl delete pod 2048-pod`
- applying the new version: `kubectl apply -f 2048-pod.yaml
- verifying the pods: `kubectl get pods -w`
- Pod created and running!

## Creating service to be the endpoint load balancer
`apiVersion: v1
kind: Service
metadata:
   name: mygame-svc
spec:
   selector:
      app: 2048-ws
   ports:
   - protocol: TCP
     port: 80
     targetPort: 80
   type: LoadBalancer
`
- applying the service: `kubectl apply -f mygame-svc.yaml`

