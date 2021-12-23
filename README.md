# calico-network-policy

Assumption: You already have eks cluster deployed.

# Step 1:

Create 2 namespaces test1 and test2 to deploy two nginx application and check the connectivity between them.

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
  
 
 # Step 2:
 
 Install calico on eks cluster.
  
  
 kubectl set env daemonset/calico-node -n kube-system IP_AUTODETECTION_METHOD=can-reach=www.google.com
daemonset.apps/calico-node env updated
  
  

  
