# heketi-glusterfs-playbook


gfs1
  ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
  systemctl daemon-reload
  systemctl start heketi
  systemctl status heketi
  systemctl enable heketi
  
  
  
  
  k8s:
    echo -n "PASSWORD" | base64
    
    
vim gluster-secret.yml

apiVersion: v1
kind: Secret
metadata:
  name: heketi-secret
  namespace: kube-system
type: "kubernetes.io/glusterfs"
data:
  # echo -n "PASSWORD" | base64
  key: $KEY
  
  kubectl apply -f gluster-secret.yml
kubectl get secret -n kube-system



vim storage-class.yml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gfs1-ssd
parameters:
  resturl: http://192.168.100.23:8080
  restuser: admin
  secretName: heketi-secret
  secretNamespace: kube-system
  volumetype: replicate:2
provisioner: kubernetes.io/glusterfs
reclaimPolicy: Delete
volumeBindingMode: Immediate
  
 kubectl apply -f storage-class.yml


vim deploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-gluster
  namespace: default
  labels:
    app: nginx-gluster
spec:
  revisionHistoryLimit: 0
  replicas: 1
  selector:
    matchLabels:
      app: nginx-gluster
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: nginx-gluster
        tier: frontend
    spec:
      containers:
      - name: nginx-gluster
        image: gcr.io/google_containers/nginx-slim:0.8
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: nginx-gluster
        volumeMounts:
        - name: gluster-vol1
          mountPath: /usr/share/nginx/html
      volumes:
      - name: gluster-vol1
        persistentVolumeClaim:
          claimName: test
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      
      
     vim pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: test
 annotations:
   volume.beta.kubernetes.io/storage-class: gfs1-ssd
spec:
 accessModes:
  - ReadWriteMany
 resources:
   requests:
     storage: 3Gi

 
