# nginx-class-demo
using NGNIX classes to route traffic

For this demo we will be using 3 different `deployments` which will be fronted by 3 different `ingress-controllers`

1. create the namespaces
```bash
kubectl create ns ingress-apple
kubectl create ns ingress-banana
kubectl create ns ingress-coffee
```

1. Install ingress-nginx on each namespace

```bash
helm install ingress-apple ingress-nginx/ingress-nginx --set controller.ingressClass=apple --namespace ingress-apple
helm install ingress-banana ingress-nginx/ingress-nginx  --set controller.ingressClass=banana --namespace ingress-banana
helm install ingress-coffee ingress-nginx/ingress-nginx --set controller.ingressClass=coffee --namespace ingress-coffee
```
1. create the ingress objects on each namespace

```bash
cat << EOF > ingress-apple.yaml
apiVersion: networking.k8s.io/v1 
kind: Ingress
metadata:
  name: ingress-apple
  namespace: ingress-apple
  annotations:
    ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: apple
spec:
  rules:
  - http:
      paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port: 
                number: 5678
---
apiVersion: networking.k8s.io/v1 
kind: Ingress
metadata:
  name: ingress-banana
  namespace: ingress-banana
  annotations:
    ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: banana
spec:
  rules:
  - http:
      paths:
        - path: /banana
          pathType: Prefix
          backend:
            service:
              name: banana-service
              port: 
                number: 5678
---
apiVersion: networking.k8s.io/v1 
kind: Ingress
metadata:
  name: ingress-coffee
  namespace: ingress-coffee
  annotations:
    ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: coffee
spec:
  rules:
  - http:
      paths:
        - path: /coffee
          pathType: Prefix
          backend:
            service:
              name: banana-service
              port: 
                number: 5678

EOF
```

1. deploy the applications

```bash
cat << EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apple-app
  namespace: ingress-apple
  labels:
    app: apple
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apple
  strategy: {}
  template:
    metadata:
      labels:
        app: apple
    spec:
      containers:
      - image: hashicorp/http-echo
        name: http-echo
        args:
          - "-text=apple"
---
kind: Service
apiVersion: v1
metadata:
  name: apple-service
  namespace: ingress-apple
spec:
  selector:
    app: apple
  ports:
    - port: 5678 # Default port for image
--
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banana-app
  namespace: ingress-banana
  labels:
    app: banana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: banana
  strategy: {}
  template:
    metadata:
      labels:
        app: banana
    spec:
      containers:
      - image: hashicorp/http-echo
        name: http-echo
        args:
          - "-text=banana"
---
kind: Service
apiVersion: v1
metadata:
  name: banana-service
  namespace: ingress-banana
spec:
  selector:
    app: banana
  ports:
    - port: 5678 # Default port for image
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coffee-app
  namespace: ingress-coffee
  labels:
    app: coffee
spec:
  replicas: 1
  selector:
    matchLabels:
      app: coffee
  strategy: {}
  template:
    metadata:
      labels:
        app: coffee
    spec:
      containers:
      - image: hashicorp/http-echo
        name: http-echo
        args:
          - "-text=coffee"
---
kind: Service
apiVersion: v1
metadata:
  name: coffee-service
  namespace: ingress-coffee
spec:
  selector:
    app: coffee
  ports:
    - port: 5678 # Default port for image
EOF
```

1. Testing

You should be able to test the different ingress controllers by (for example) calling `curl` againt the LoadBalancer ip address for a specific app. 
Here is an example of how to do it for the Banana App

```bash
BANANA_INGRESS=$(kubectl get svc -l app.kubernetes.io/instance=ingress-banana -o=jsonpath="{.items[].status.loadBalancer.ingress[0].ip}")
$ curl $BANANA_INGRESS/banana
banana
```
