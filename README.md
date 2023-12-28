# calico-network-policy

Assumption: You already have eks cluster deployed.

**Step 1: Create sample microservice application**

      Source Code for Microservice Application : https://github.com/tushardashpute/microservice-kubernetes.git
 
      sh kubernetes-deploy.sh

            $ sh kubernetes-deploy.sh
            deployment.apps/apache created
            service/apache exposed
            deployment.apps/catalog created
            service/catalog exposed
            deployment.apps/customer created
            service/customer exposed
            deployment.apps/order created
            service/order exposed
  
Now to test the connectivity, port-forward the svc appache and hit the customer service.

 kubectl port-forward svc/apache 80:80

 Open the http://localhost in the browser

 ![image](https://github.com/tushardashpute/calico-network-policy/assets/74225291/9b824090-8a3a-48d4-9206-6e48a17858af)

 You can now navigate to any of the above available options, for example I am navigating to customer.

 ![image](https://github.com/tushardashpute/calico-network-policy/assets/74225291/8e9e2209-0658-4445-96c0-681560a67a86)

 
 **Step 2: Install calico on eks cluster and again check the connetivity between pods in different namespaces.**
 
Download the [Calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises) networking manifest for the Kubernetes API datastore.

- Install the operator on your cluster.

            kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml
            
- Download the custom resources necessary to configure Calico.

            curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml -O
            
If you are using pod CIDR 192.168.0.0/16, skip to the next step. If you are using a different pod CIDR, use the following commands to set an environment variable called POD_CIDR containing your pod CIDR and replace 192.168.0.0/16 in the manifest with your pod CIDR.

POD_CIDR="<your-pod-cidr>" \
sed -i -e "s?192.168.0.0/16?$POD_CIDR?g" custom-resources.yaml
Apply the manifest using the following command.

- Create the manifest in order to install Calico.

kubectl apply -f custom-resources.yaml

- Verify Calico installation in your cluster.

        watch kubectl get pods -n calico-system


              Every 2s: kubectl get pods -n calico-system                                                                                               2023-12-28 17:23:31
            
            NAME                                       READY   STATUS    RESTARTS   AGE
            calico-kube-controllers-5f5f8d5fb9-hj57z   1/1     Running   0          6h12m
            calico-node-qsf5m                          1/1     Running   0          6h12m
            calico-typha-759c4b8c47-cjdb8              1/1     Running   0          6h12m
            csi-node-driver-f7tqz                      2/2     Running   0          6h12m


 **Step 3 : Create network policy default deny on default ns**
  
            apiVersion: networking.k8s.io/v1
            kind: NetworkPolicy
            metadata:
              name: default-deny
            spec:
              podSelector: {}
              policyTypes:
              - Ingress

In this policy we are denying all ingress traffic coming to pods in default namespace.

kubectl apply -f defautl_deny.yaml

            $ kubectl apply -f default_deny.yaml
            networkpolicy.networking.k8s.io/default-deny created

now try to access the customer option again from UI, you will get below error.

![image](https://github.com/tushardashpute/calico-network-policy/assets/74225291/5a834107-c7f4-49f8-8b65-c2844a9191f6)

Step 4 : Create network policy to allow traffic from apache app to customer app

            kind: NetworkPolicy
            apiVersion: networking.k8s.io/v1
            metadata:
              name: customer-allow-prod
            spec:
              podSelector:
                matchLabels:
                  app: customer
              ingress:
              - from:
                - namespaceSelector:
                    matchLabels:
                      kubernetes.io/metadata.name: default

Here we are allowing trafic from all the pods in the default namespace to the pods with the lables "app: customer"

            $ kubectl apply -f traffic_to_customer.yaml
            networkpolicy.networking.k8s.io/customer-allow-prod created

Now again try to access the customer page:

![image](https://github.com/tushardashpute/calico-network-policy/assets/74225291/65329441-9521-42d4-af8e-b09a897eab0a)
