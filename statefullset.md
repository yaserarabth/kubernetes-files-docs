# service_state.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      name: mysql
```
```
kubectl apply -f mysql-headless.yaml
```
# MySQL StatefulSet YAML
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-stateful
spec:
  serviceName: mysql-headless
  replicas: 2
  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0

        ports:
        - containerPort: 3306

        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root123
        - name: MYSQL_DATABASE
          value: mydb

        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql

  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      storageClassName: gp2
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
```
```
kubectl apply -f mysql-stateful.yaml
```
```
kubectl get pods
```
```
kubectl exec -it mysql-stateful-0 -- mysql -uroot -p
```
```
root123
```
```
kubectl get pvc

```
```
mysql-storage-mysql-stateful-0
mysql-storage-mysql-stateful-1
mysql-storage-mysql-stateful-2
```
### You can verify inside pod: in eks access machine 
```
kubectl exec -it mysql-stateful-0 -- bash
```
```
df -h
```
```
ls /var/lib/mysql
```

```
mysql-stateful-0
mysql-stateful-1
```
### DNS names:

```
mysql-stateful-0.mysql-headless.default.svc.cluster.local
mysql-stateful-1.mysql-headless.default.svc.cluster.local

```

## Applications connect using these names.

```
Example app connection:
```
```
Host=mysql-stateful-0.mysql-headless
Port=3306
User=root
Password=root123

```
### How communication happens:

```
Application Pod
      ↓
Service DNS
      ↓
StatefulSet Pod
      ↓
MySQL running inside container

```
```
Why StatefulSet needs Headless Service?

Because StatefulSet gives:

Stable pod names
Stable network identity
Stable storage

Without headless service, pods get random IPs only.

```
```
nslookup mysql-stateful-0.mysql-headless
```
### You’ll see pod IP.

### install mysql client and connect:

```
mysql -h mysql-stateful-0.mysql-headless -u root -p

```



