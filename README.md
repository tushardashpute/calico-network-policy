# calico-network-policy

Assumption: You already have eks cluster deployed.

# Steps 1:

[root@ip-172-31-36-53 opt]# k create ns test1
namespace/test1 created
[root@ip-172-31-36-53 opt]# k create ns test2
namespace/test2 created
[root@ip-172-31-36-53 opt]# 
[root@ip-172-31-36-53 opt]# 
[root@ip-172-31-36-53 opt]# k apply -f nginx-development.yaml -n test1
deployment.apps/nginx-deployment created
[root@ip-172-31-36-53 opt]# k apply -f nginx-development.yaml -n test2
deployment.apps/nginx-deployment created

[root@ip-172-31-36-53 opt]# k get pods -A -o wide|grep -i nginx 
test1         nginx-deployment-66b6c48dd5-2njk8   1/1     Running   0          69s     192.168.62.179   ip-192-168-34-213.us-east-2.compute.internal   <none>           <none>
test2         nginx-deployment-66b6c48dd5-kvl7s   1/1     Running   0          65s     192.168.31.206   ip-192-168-5-20.us-east-2.compute.internal     <none>           <none>

# k exec nginx-deployment-66b6c48dd5-2njk8 -n test1 -- curl 192.168.31.206
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   234k      0 --:--:-- --:--:-- --:--:--  298k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

  
  # k exec nginx-deployment-66b6c48dd5-kvl7s -n test2 -- curl 192.168.62.179
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

  
 # Step 2:
 
 Install calico on eks cluster.
  
  
 # kubectl set env daemonset/calico-node -n kube-system IP_AUTODETECTION_METHOD=can-reach=www.google.com
daemonset.apps/calico-node env updated
  
  

  
