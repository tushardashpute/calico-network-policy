# calico-network-policy

Assumption: You already have eks cluster deployed.

# Step 1: Create 2 namespaces test1 and test2 to deploy two nginx application and check the connectivity between them.

k create ns test1
namespace/test1 created

k create ns test2
namespace/test2 created

k apply -f nginx-development.yaml -n test1
deployment.apps/nginx-deployment created
k apply -f nginx-development.yaml -n test2
deployment.apps/nginx-deployment created

k get pods -A -o wide|grep -i nginx 
test1         nginx-deployment-66b6c48dd5-2njk8   1/1     Running   0          69s     192.168.62.179   ip-192-168-34-213.us-east-2.compute.internal   <none>       
test2         nginx-deployment-66b6c48dd5-kvl7s   1/1     Running   0          65s     192.168.31.206   ip-192-168-5-20.us-east-2.compute.internal     <none>       

Now to test the connectivity we will issue the curl command from each namespace nginx to call other namespace nginx pod IP.
  
 k exec nginx-deployment-66b6c48dd5-2njk8 -n test1 -- curl 192.168.31.206

  You will get output like <title>Welcome to nginx!</title>

  
  k exec nginx-deployment-66b6c48dd5-kvl7s -n test2 -- curl 192.168.62.179
  You will get output like <title>Welcome to nginx!</title>
  
 
 # Step 2: Install calico on eks cluster and again check the connetivity between pods in different namespaces.
 
 Download the Calico networking manifest for the Kubernetes API datastore.

curl https://docs.projectcalico.org/v3.10/manifests/calico.yaml -O

  If you are using pod CIDR 192.168.0.0/16, skip to the next step. If you are using a different pod CIDR, use the following commands to set an environment variable called POD_CIDR containing your pod CIDR and replace 192.168.0.0/16 in the manifest with your pod CIDR.

POD_CIDR="<your-pod-cidr>" \
sed -i -e "s?192.168.0.0/16?$POD_CIDR?g" calico.yaml
Apply the manifest using the following command.

kubectl apply -f calico.yaml

  If you wish to enforce application layer policies and secure workload-to-workload communications with mutual TLS authentication, continue to Enabling application layer policy (optional). 
  
 kubectl set env daemonset/calico-node -n kube-system IP_AUTODETECTION_METHOD=can-reach=www.google.com
daemonset.apps/calico-node env updated
  
  k exec nginx-deployment-66b6c48dd5-2njk8 -n test1 -- curl 192.168.31.206

 You will get "curl: (7) Failed to connect to 192.168.31.206 port 80: No route to host"
  
  k exec nginx-deployment-66b6c48dd5-kvl7s -n test2 -- curl 192.168.62.179

 You will get "curl: (7) Failed to connect to 192.168.62.179 port 80: No route to host"

 # Step 3 : Create network policy on test1 ns to allow incoming trafic from ns test2.
  
  
