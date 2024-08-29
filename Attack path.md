# Showing the functionality of attack path on CAST AI

## Introduction
Before proceeding to this use-case, check the 'Attack path' under Security. 
You will not be seeing anything there because this cluster currently is not being exposed to the internet through any ingress controller or service.
During the demo (where you will actually have the attack paths present as the following configurations will already be set up), show the 
'Vulnerabilities' menu first and explain how the customer will have to deal with 1000s of CVEs if they are going that route.
Next show them how attack paths can get them to priorotize which ones to be fixed first because of high risk.

Step 1: Let's add an ingress controller and expose the emailservice to the internet.

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

```
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --create-namespace --namespace ingress-nginx
```
Step 2: 
After few minutes, you should see something like this:
giri@cloudshell:~/microservices-demo (demos-321800)$ kubectl get service --namespace ingress-nginx nginx-ingress-ingress-nginx-controller
NAME                                     TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
nginx-ingress-ingress-nginx-controller   LoadBalancer   34.118.234.125   34.46.43.231   80:30640/TCP,443:31409/TCP   7m3s

```
kubectl get service --namespace ingress-nginx nginx-ingress-ingress-nginx-controller
```
Step 3:
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
Step 4: 
Next, apply this yaml to create an nginx ingress service, expose the deployment with type 'LoadBalancer' and an ingress resource:
```
https://github.com/castai-demo/security-demos/blob/master/ingress-config.yaml
```
