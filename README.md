# jitsi-on-kubernetes

#### Deployment files:

#### Steps:
1. `https://github.com/ajaysheoran2323/jitsi-on-kubernetes.git`
2. Do the changes in files:

    https://github.com/ajaysheoran2323/jitsi-on-kubernetes/blob/master/deployments/jvb.yaml#L24
    
    https://github.com/ajaysheoran2323/jitsi-on-kubernetes/blob/master/deployments/web-ingress.yaml#L8
    
    https://github.com/ajaysheoran2323/jitsi-on-kubernetes/blob/master/deployments/web-ingress.yaml#L17-L18
    

3. go to deployment folder and run the yaml files one by one:
   ```
   kubectl apply -f jicofo.yaml
   kubectl apply -f jvb.yaml
   kubectl apply -f prosody.yaml
   kubectl apply -f web.yaml
   kubectl apply -f jvb-svc.yaml
   kubectl apply -f prosody-svc.yaml
   kubectl apply -f web-svc.yaml
   kubectl apply -f web-ingress.yaml
   ```

#### Different topology

Thus the default behaviour of advertised the internal IP of jvb, is not really suitable in many cases.
Kubernetes offers multiple possibilities to work around the problem. Not all options are available depending on the Kubernetes cluster setup.
The chart tries to make all options available without enforcing one.

##### Option 1: service of type `LoadBalancer`

This requires a cloud setup that enables a Loadbalancer attachement.
This could be enabled via values:

```yaml
jvb:
  service:
    type: LoadBalancer

  # Depending on the cloud, publicIP cannot be know in advance, so deploy first, without the next option.
  # Next: redeploy with the following option set to the public IP you retrieved from the API.
  publicIP: 1.2.3.4
```

In this case you're not allowed to change the `jvb.replicaCount` to more than `1`, UDP packets will be routed to random `jvb`, which would not allow for a working video setup.

##### Option 2: NodePort and node with Public IP or external loadbalancer

```yaml
jvb:
  service:
    type: NodePort
  UDPPort: 30000
  TCPPort: 30443

  publicIP: <LoadBalancer's public IP>
```

In this case you're not allowed to change the `jvb.replicaCount` to more than `1`, UDP packets will be routed to random `jvb`, which would not allow for a working video setup.


##### Option 3: hostPort and node with Public IP

Assuming that the node knows the PublicIP it holds, you can enable this setup:

```yaml
jvb:
  useHostPort: true
  # This option requires kubernetes >= 1.17
  useNodeIP: true
```

In this case you can have more the one `jvb`
