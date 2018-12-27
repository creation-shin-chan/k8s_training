# k8s_training

# Helm installation
###### Install helm client
`curl https://storage.googleapis.com/kubernetes-helm/helm-v2.12.0-rc.2-linux-amd64.tar.gz | tar zxv`

###### Add cluster-admin to tiller
`kubectl edit clusterrolebindings cluster-admin`  

and then add the following line to the end.
```
- kind: ServiceAccount
  name: default
  namespace: kube-system
```


# Deploy nginx-ingress
Let's install ingress service with nginx using helm.  
`helm install stable/nginx-ingress --name k8s-train --set rbac.create=true --namespace kube-system --set controller.kind=DaemonSet,controller.hostNetwork=true`


# Deploy StorageOS
###### git clone the repository
`git clone https://github.com/storageos/deploy.git`
###### Deploy operator
```
cd deploy/k8s/deploy-storageos/cluster-operator
./deploy-operator.sh
kubectl label nodes <worker node> node-role.kubernetes.io/worker=true
kubectl create -f - <<END
apiVersion: v1
kind: Secret
metadata:
  name: "storageos-api"
  namespace: "default"
  labels:
    app: "storageos"
type: "kubernetes.io/storageos"
data:
  # echo -n '<secret>' | base64
  apiUsername: c3RvcmFnZW9z
  apiPassword: c3RvcmFnZW9z
END
kubectl create -f examples/basic.yaml
```
###### Persistent Volume
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: fast
```
###### Mount the persistent volume
```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: myfrontend
    image: nginx
    volumeMounts:
    - mountPath: "/var/www/html"
      name: mypd
  volumes:
  - name: mypd
    persistentVolumeClaim:
      claimName: myclaim
  nodeSelector:
    node-role.kubernetes.io/worker: "true"
```

###### Deploy MySQl StatefulSet with persistent volume
```
cd ../../examples
kubectl create -f ./mysql
```

### Deploy jenkins with helm
`helm install stable/jenkins --set rbac.install=true,Persistence.StorageClass=fast,Master.ServiceType=ClusterIP,Master.HostName=<Public URL>`

Jenkins admin password to access the UI.  
`printf $(kubectl get secret --namespace default hardy-aardwolf-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`

Sample job definition
```
def label = "slave-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'golang', image: 'golang:1.10.5-alpine3.7', ttyEnabled: true, command: 'cat')
  ]) {

    node(label) {
        stage('Test Stage') {
            container('golang') {
                stage('Go version') {
                    sh 'go version'
                }
            }
        }
    }    
}
```

# Ingress + Letsencrypt with cert-manager

###### Deploy cert-manager
```
helm install \
  --name cert-manager \
  --namespace kube-system \
  --set ingressShim.defaultIssuerName=letsencrypt-staging \
  --set ingressShim.defaultIssuerKind=ClusterIssuer \
  stable/cert-manager
```

###### Create issuer
```
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: <me@example.com>
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-sec-staging
    # Enable HTTP01 validations
    http01: {}
```
###### Sample deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: letsencrypt-staging
    kubernetes.io/tls-acme: “true”
spec:
  rules:
  - host: <URL>
    http:
      paths:
      - path: /
        backend:
          serviceName: my-webapp
          servicePort: 80
  tls:
  - secretName: tls-staging-cert
    hosts:
    - <URL>
---
kind: Service
apiVersion: v1
metadata:
  name: my-webapp
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```



# Other tools
- **helm**
  - Application manager for Kubernetes. 
- **StorageOS**
  - Persistent volume provisionor for containers. 
- **Kaniko**
  - a tool to build container images from a Dockerfile, inside a container or Kubernetes cluster.
- **cert-manager**
  - Kubernetes add-on to automate the management and issuance of TLS certificates from various issuing sources.
- **Ingress**
  - nginx
  - traefik
- **weaveworks**
  - sock-shop
  - weave-net
  - weave scope
- **Istio**
- **Linkerd**
- **Prometheus**
- **InfluxDB**
 
