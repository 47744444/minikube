# minikube
## 安裝minikube
### Visit https://minikube.sigs.k8s.io/docs/start/
## 啟動minikube
```
minikube start
```
## deploy.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: upload
spec:
  replicas: 3
  selector:
    matchLabels:
      app: upload
  strategy:
    type: RollingUpdate  
    rollingUpdate:
      maxUnavailable: 1  
      maxSurge: 1  
  template:
    metadata:
      labels:
        app: upload
    spec:
      containers:
      - name: upload
        image: zihrueiliou/uploadimg:v1.0.0
        ports:
        - containerPort: 5000  
        resources: {}
        volumeMounts:
        - mountPath: /code/app/static/images
          name: pv-storage
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: pv-storage
        persistentVolumeClaim:
          claimName: upload-pvc
```
## service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: upload-service
spec:
  selector:
    app: upload
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 5000
  type: LoadBalancer
```
## pv.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: upload-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: /data/upload-pv
```
## pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: upload-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
```
## 創建服務
```
kubectl apply -f upload-pvc.yaml
kubectl apply -f test-pv.yaml
kubectl create -f test.yaml
kubectl create -f testservice.yaml
kubectl get service
kubectl get pod
```
## 啟動ingree
```
minikube addons enable ingress
```
## ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: upload-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello.info
      http:
        paths:
          - path: /upload
            pathType: Prefix
            backend:
              service:
                name: upload-service
                port:
                  number: 5000 
```

