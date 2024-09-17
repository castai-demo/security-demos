# Showing the functionality of attack path on CAST AI

## Introduction
Before proceeding to this use-case, check the 'Attack path' under Security. 
You will not be seeing anything there because this cluster currently is not being exposed to the internet through any ingress controller or service.
During the demo (where you will actually have the attack paths present as the following configurations will already be set up), show the 
'Vulnerabilities' menu first and explain how the customer will have to deal with 1000s of CVEs if they are going that route.
Next show them how attack paths can get them to priorotize which ones to be fixed first because of high risk.

Step 1: Setup the cluster and online boutique application - See the README.md file for instructions. Once the pods are up, go to Step 2.

Step 2: Go to your CAST AI console and add this cluster (Make sure to select GKE if you are building this is GCP). 
        Enable CAST AI security (copy paste the generated script). This will install the kvisor agent.

Step 3: Click on Dashboard and verify it is populated. Click on 'Vulnerabilities' -> shows a list of all the images with CVEs. 
        Click on 'Attack path'. You most likely will not see anything here because we have not created an ingress to expose a service to internet.
        
Step 4: Let's add an ingress controller and expose an nginx service to the internet.

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

```
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --create-namespace --namespace ingress-nginx
```
Step 5: 
After few minutes, you should see something like this:
```
giri@cloudshell:~/microservices-demo (demos-321800)$ kubectl get service --namespace ingress-nginx nginx-ingress-ingress-nginx-controller
NAME                                     TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
nginx-ingress-ingress-nginx-controller   LoadBalancer   34.118.234.125   34.46.43.231   80:30640/TCP,443:31409/TCP   7m3s
```
```
kubectl get service --namespace ingress-nginx nginx-ingress-ingress-nginx-controller
```
Step 6:
Encrypting traffic:
Configure tls with private key, csr and certificate prior to this step and copy the appropriate keys to the ingress-config.yaml 

Generate a private key:
```
openssl genrsa -out tls.key 2048
```

Create a certificate signing request:
```
openssl req -new -key tls.key -out tls.csr
```

Generate self-signed certificate:
```
openssl x509 -req -in tls.csr -signkey tls.key -out tls.crt -days 365
```
Step 7: 
Next, apply this yaml to create an nginx ingress service, expose the deployment with type 'LoadBalancer' and an ingress resource:
```
https://github.com/castai-demo/security-demos/blob/master/ingress-config.yaml
```
Step 8: Give few minutes for the attack path to get generated (if nothing shows up, troubleshoot your ingress service to check if an external IP is assigned to the service)
        You should see something similar to this:
        giri@cloudshell:~ (demos-321800)$ kubectl get service --namespace ingress-nginx nginx-ingress-ingress-nginx-controller
NAME                                     TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
nginx-ingress-ingress-nginx-controller   LoadBalancer   34.118.234.193   35.225.194.245   80:32562/TCP,443:31251/TCP   3h

Step 9: Demoing the attack path:
        ![Screenshot 2024-08-29 at 5 46 31 PM](https://github.com/user-attachments/assets/642940dd-3fb3-4a96-a5dc-89349e97d6bd)

Click on the nginx service and ngnix deployment to show the misconfigurations in these resources.
Utlimately the image that is part of the container is all the way to the right. 
This image will probably have multiple vulnerabilities and these can be seen once you click on it (opens a new window to take you to vulnerabilities page).

Step 10: Finally, the use-case can be described as to how we prioritize only those workloads that are exposed to the internet 
         where attackers can exploit the vulnerabilities. Fixing these first will considerably reduce the risk of attacks.


