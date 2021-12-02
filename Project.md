# 打造IoT雲端資料處理平台

## 架構圖  
![all](https://i.imgur.com/igJ0sp9.png)


---

在k8s中amster主機 .kube 中的config檔案先指定namespace:  
```
contexts:
- context:
    cluster: kubernetes
    namespace: nifi(自訂名)
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
```
$ kubectl create ns nifi

### MQTT mosquitto 安裝  
$ mkdir mosquitto  
$ sudo nano mosquitto/configmap.yaml  
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mosquitto-config
data:
  mosquitto.conf: |-
    # Ip/hostname to listen to.
    # If not given, will listen on all interfaces
    #bind_address

    # Port to use for the default listener.
    port 1883

    # Allow anonymous users to connect?
    # If not, the password file should be created
    allow_anonymous true

    # The password file.
    # Use the `mosquitto_passwd` utility.
    # If TLS is not compiled, plaintext "username:password" lines bay be used
    # password_file /mosquitto/config/passwd
```

$ sudo nano mosquitto/deployment.yaml  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
spec:
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
      containers:
      - name: mosquitto
        image: eclipse-mosquitto:2.0
        resources:
          requests:
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 1883
        volumeMounts:
            - name: mosquitto-config
              mountPath: /mosquitto/config/mosquitto.conf
              subPath: mosquitto.conf
      volumes:
        - name: mosquitto-config
          configMap:
            name: mosquitto-config
```    

$ sudo nano mosquitto/service.yaml  
```
apiVersion: v1
kind: Service
metadata:
  name: mosquitto
spec:
  externalIPs:
  - 192.168.183.131(透過此固定IP連線)
  selector:
    app: mosquitto
  ports:
  - port: 1883
    targetPort: 1883
```

$ cd mosquitto;kubectl apply -f .  
$ kubectl get all  
```
NAME                                       READY   STATUS    RESTARTS   AGE
pod/mosquitto-76bcf4956f-wgsgp             1/1     Running   0          8d

NAME                               TYPE        
service/mosquitto                  ClusterIP   10.100.7.235     192.168.183.131   1883/TCP                        8d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mosquitto             1/1     1            1           8d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/mosquitto-76bcf4956f             1         1         1       8d
```
進入mosquitto發送/接收訊息
$ kubectl exec -it (pod/mosquitto-76bcf4956f-wgsgp) sh  
/ # mosquitto_sub -h localhost -t A  
/ # mosquitto_pub -h localhost -t A -m Message  
$ kubectl logs -l app=gateway-bridge -f --all-containers  

---

### Helm ingress 安裝  
```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh

更新ingress-nginx下的values.yaml
.....
externalIPs:
- 192.168.183.131(透過此固定IP連線)
.....
```

$ kubectl get all
```
NAME                                                 READY   STATUS    RESTARTS   AGE
pod/nifi-ingress-nginx-controller-6bb9dc86d5-c54mg   1/1     Running   0          8d

NAME                                              TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                      AGE
service/kubernetes                                ClusterIP      10.96.0.1        <none>            443/TCP                      25d
service/nifi-ingress-nginx-controller             LoadBalancer   10.103.21.101    192.168.183.131   80:32542/TCP,443:32477/TCP   8d
service/nifi-ingress-nginx-controller-admission   ClusterIP      10.108.160.184   <none>            443/TCP                      8d

NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nifi-ingress-nginx-controller   1/1     1            1           8d

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/nifi-ingress-nginx-controller-6bb9dc86d5   1         1         1       8d
```

---

### Nifi 安裝  
網頁化界面可以直接拉資料的流向  

部屬項目:  
NiFi Namespace（所有項目都將部署在這裡）  
Apache NiFi（每個都有自己的服務端點）  
Apache Zookeeper（僅可在集群內訪問）  
Secrets（基本身份驗證的 用戶名/密碼： admin:admin）  
Ingress（訪問端點）  

$ mkdir apachenifi  
$ sudo nano apachenifi/namespace.yaml  
```
apiVersion: v1
kind: Namespace
metadata:
  name: nifi
  labels:
    name: nifi
  annotations:
    app.kubernetes.io/name: nifi
    app.kubernetes.io/part-of: nifi
```

$ sudo nano apachenifi/statefulset.yaml  
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nifi
  namespace: nifi
  labels:
    name: nifi
    app: nifi
  annotations:
    app.kubernetes.io/name: nifi
    app.kubernetes.io/part-of: nifi
spec:
  serviceName: nifi
  replicas: 1
  selector:
    matchLabels:
      app: nifi
  template:
    metadata:
      labels:
        app: nifi
    spec:
      nodeSelector:
        beta.kubernetes.io/os: linux
        kubernetes.io/arch: amd64
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - topologyKey: "kubernetes.io/hostname"
              labelSelector:
                matchLabels:
                  app: nifi
      restartPolicy: Always
      containers:
      - name: nifi
        image: apache/nifi:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: nifi
        - containerPort: 8082
          name: cluster
        env:
          - name: NIFI_WEB_HTTP_PORT
            value: "8080"
          - name: NIFI_CLUSTER_IS_NODE
            value: "true"
          - name: NIFI_CLUSTER_NODE_PROTOCOL_PORT
            value: "8082"
          - name: NIFI_CLUSTER_NODE_ADDRESS
            value: "nifi"
          - name: NIFI_ZK_CONNECT_STRING
            value: "zookeeper:2181"
          - name: NIFI_ELECTION_MAX_WAIT
            value: "1 min"
        livenessProbe:
          exec:
            command:
              - pgrep
              - java
        readinessProbe:
          tcpSocket:
              port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
          limits:
            cpu: 400m
            memory: 1Gi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zookeeper
  namespace: nifi
  labels:
    name: zookeeper
    app: zookeeper
  annotations:
    app.kubernetes.io/name: zookeeper
    app.kubernetes.io/part-of: nifi
spec:
  serviceName: zookeeper
  replicas: 1
  selector:
    matchLabels:
      app: zookeeper
  template:
    metadata:
      labels:
        app: zookeeper
    spec:
      nodeSelector:
        kubernetes.io/os: linux
        kubernetes.io/arch: amd64
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - topologyKey: "kubernetes.io/hostname"
              labelSelector:
                matchLabels:
                  app: zookeeper
      restartPolicy: Always
      containers:
      - name: zookeeper
        image: zookeeper:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 2181
          name: zk
        - containerPort: 5111
          name: cmd
        env:
          - name: ALLOW_ANONYMOUS_LOGIN
            value: "yes"
          - name: ZOO_AUTOPURGE_PURGEINTERVAL
            value: "1"
          - name: ZOO_AUTOPURGE_SNAPRETAINCOUNT
            value: "1"
          - name: ZOO_STANDALONE_ENABLED
            value: "true"
        livenessProbe:
          exec:
            command:
              - which
              - java
        readinessProbe:
          tcpSocket:
              port: 2181
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 512Mi
```

$ sudo nano apachenifi/service.yaml  
```
apiVersion: v1
kind: Service
metadata:
  name: nifi
  namespace: nifi
  labels:
    app: nifi
  annotations:
    app.kubernetes.io/name: nifi
    app.kubernetes.io/part-of: nifi
spec:
  type: NodePort
  selector:
    app: nifi
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
    name: nifi
  - protocol: TCP
    port: 8082
    targetPort: 8082
    name: cluster
---
apiVersion: v1
kind: Service
metadata:
  name: zookeeper
  namespace: nifi
  labels:
    app: zookeeper
  annotations:
    app.kubernetes.io/name: zookeeper
    app.kubernetes.io/part-of: nifi
spec:
  type: ClusterIP
  selector:
    app: zookeeper
    "statefulset.kubernetes.io/pod-name": zookeeper-0
  ports:
  - protocol: TCP
    port: 2181
    targetPort: 2181
    name: zk
  - protocol: TCP
    port: 5111
    targetPort: 5111
    name: cmd
```

$ sudo nano apachenifi/ingress.yaml  
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nifi
  namespace: nifi
  labels:
    app: nifi
  annotations:
    app.kubernetes.io/name: nifi
    app.kubernetes.io/part-of: nifi
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: nifi-basic-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      server_tokens off;
spec:
  rules:
  - host: 
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nifi
            port:
              number: 8080
```

$ sudo nano apachenifi/secret.yaml  
```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: nifi-basic-auth
  namespace: nifi
  labels:
    app: nifi
  annotations:
    app.kubernetes.io/name: nifi
    app.kubernetes.io/part-of: nifi
data:
  auth: YWRtaW46JGFwcjEkSDY1dnBkTU8kMXAxOGMxN3BuZVFUT2ZjVC9TZkZzMQo=
```

$ sudo nano apachenifi/Kustomization.yaml  
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: nifi

resources:
- namespace.yaml
- statefulset.yaml
- secret.yaml
- service.yaml
- ingress.yaml
```

$ cd apachenifi;kubectl apply -f .  
$ kubectl get all  
```
NAME                                       READY   STATUS    RESTARTS   AGE
pod/mosquitto-76bcf4956f-wgsgp             1/1     Running   0          8d
pod/nifi-0                                 1/1     Running   0          8d
pod/zookeeper-0                            1/1     Running   0          8d

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP       PORT(S)                         AGE
service/mosquitto                  ClusterIP   10.100.7.235     192.168.183.131   1883/TCP                        8d
service/nifi                       NodePort    10.103.165.143   <none>            8080:32220/TCP,8082:30788/TCP   8d
service/zookeeper                  ClusterIP   10.106.3.89      <none>            2181/TCP,5111/TCP               8d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mosquitto             1/1     1            1           8d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/mosquitto-76bcf4956f             1         1         1       8d

NAME                               READY   AGE
statefulset.apps/nifi              1/1     8d
statefulset.apps/zookeeper         1/1     8d
```

上網打externalIP/ (ingress會幫我們倒到nifi網站)   
![nifi](https://i.imgur.com/FBk5EXh.png) 

選取 processor 去拉選套件: ConsumeMQTT
**Broker URI**: The URI to use to connect to the MQTT broker  
**Topic filter**: # = all topic  
**Max queue size**: This property specifies the maximum number of messages this processor will hold in memory at one time  
![connifi](https://i.imgur.com/fTbYRCb.png)  
![con](https://i.imgur.com/UyQ2Zuh.png)  

**Automatically Terminate Relationships**: 要把所有資訊都打勾, 資料就不會一直重複回流給自己  
**InfluxDB connection URL**: enter influxdb service name  
**user/password**: reference by influxdb-secret.yaml  
![db](https://i.imgur.com/SRST2Ez.png)
![db2](https://i.imgur.com/TKYoR3c.png)

---

### Kafka 安裝  
用 Helm 安裝 kafka, 至網站下載壓縮檔
https://artifacthub.io/packages/helm/bitnami/kafka  

$ sudo nano kafka-pv.yaml 
$ mkdir kafka-storage  
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-kafka
spec:
  capacity:
    storage: 8Gi 
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  #storageClassName: manual
  local:
    path: /home/bigred/kafka-storage
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - m1(hostname)
```

$ sudo nano zookeeper-pv.yaml 
$ mkdir zookeeper-storage  
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-kafka-zookeeper
spec:
  capacity:
    storage: 8Gi 
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  #storageClassName: manual
  local:
    path: /home/bigred/zookeeper-storage
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - m1(hostname)
```

$ kubectl apply -f kafka-pv.yaml 
$ kubectl apply -f zookeeper-pv.yaml 
$ kunectl get pv
```
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
data-kafka             8Gi        RWO            Retain           Bound    nifi/data-kafka-0                                     5d
data-kafka-zookeeper   8Gi        RWO            Retain           Bound    nifi/data-kafka-zookeeper-0                           5d
```

$ kunectl get pvc
```
NAME                     STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-kafka-0             Bound    data-kafka             8Gi        RWO                           5d
data-kafka-zookeeper-0   Bound    data-kafka-zookeeper   8Gi        RWO                           5d
```

$ cd kafka  
$ helm install kafka .  
$ helm list  
```
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
kafka   nifi            1               2021-07-10 07:38:10.602004087 +0000 UTC deployed        kafka-13.0.3    2.8.0
```

$ sudo nano kafkaclient.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: testclient
  namespace: nifi
spec:
  containers:
  - name: kafka
    image: confluentinc/cp-kafka:5.0.1
    command:
      - sh
      - -c
      - "exec tail -f /dev/null"
```

$ kubectl apply -f kafkaclient.yaml  
$ kubectl get all  
```
NAME                                       READY   STATUS    RESTARTS   AGE
pod/kafka-0                                1/1     Running   0          4d23h
pod/kafka-zookeeper-0                      1/1     Running   0          4d23h
pod/mosquitto-76bcf4956f-wgsgp             1/1     Running   0          8d
pod/nifi-0                                 1/1     Running   0          8d
pod/testclient                             1/1     Running   0          4d22h
pod/zookeeper-0                            1/1     Running   0          8d

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP       PORT(S)                         AGE
service/kafka                      ClusterIP   10.111.41.89     <none>            9092/TCP                        4d23h
service/kafka-headless             ClusterIP   None             <none>            9092/TCP,9093/TCP               4d23h
service/kafka-zookeeper            ClusterIP   10.99.55.54      <none>            2181/TCP,2888/TCP,3888/TCP      4d23h
service/kafka-zookeeper-headless   ClusterIP   None             <none>            2181/TCP,2888/TCP,3888/TCP      4d23h
service/mosquitto                  ClusterIP   10.100.7.235     192.168.183.131   1883/TCP                        8d
service/nifi                       NodePort    10.103.165.143   <none>            8080:32220/TCP,8082:30788/TCP   8d
service/zookeeper                  ClusterIP   10.106.3.89      <none>            2181/TCP,5111/TCP               8d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mosquitto             1/1     1            1           8d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-6f46b9c7b6               1         1         1       8d
replicaset.apps/mosquitto-76bcf4956f             1         1         1       8d

NAME                               READY   AGE
statefulset.apps/kafka             1/1     4d23h
statefulset.apps/kafka-zookeeper   1/1     4d23h
statefulset.apps/nifi              1/1     8d
statefulset.apps/zookeeper         1/1     8d

$ kubectl exec -it testclient -- /usr/bin/kafka-topics --zookeeper kafka-zookeeper:2181 --topic test1 --create --partitions 1 --replication-factor 1 (做資料保存1次)

#開啟一個終端機執行 producer
$ kubectl  exec -ti testclient -- /usr/bin/kafka-console-producer --broker-list kafka:9092 --topic test1
>hi

#另外一個終端機執行 consumer
$ kubectl exec -ti testclient -- /usr/bin/kafka-console-consumer --bootstrap-server kafka:9092 --topic test1
hi(看producer打什麼, consumer的訊息會自動跳出來)
```

---

### influxDB 安裝  
$ mkdir influxdb
$ sudo nano influxdb/influxdb.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: influxdb-deployment
spec:
  selector:
    matchLabels:
      app: influxdb
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: influxdb
    spec:
      containers:
        - image: influxdb:1.8.6
        #influxdb版本2.0以上有一些問題所以先不要用latest
          imagePullPolicy: IfNotPresent
          name: influxdb
          ports:
            - containerPort: 8086
          volumeMounts:
            - mountPath: /var/lib/influxdb
              name: influxdb-data
#            - mountPath: /etc/influxdb/influxdb.conf
#              name: influxdb-config
#              subPath: influxdb.conf
#              readOnly: true
          envFrom:
            - secretRef:
                name: influxdb-secrets
      volumes:
        - name: influxdb-data
          persistentVolumeClaim:
            claimName: influxdb-data
#        - name: influxdb-config
#          configMap:
#            name: influxdb-config
```

$ sudo nano influxdb/influxdb-service.yaml
```
apiVersion: v1  
kind: Service  
metadata:  
  name: influxdb-service  
spec:  
  selector:  
    app: influxdb  
  ports:  
    - protocol: TCP  
      port: 8086  
      targetPort: 8086
```

建立pv pvc 將跟目錄下的influxdb-storge mount 進container裡面
$ sudo nano influxdb/influxdb-pv-pvc.yaml
$ mkdir influxdb-storge
```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-local
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  local:
    path: "/home/bigred/influxdb-storge"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - m1 
---
apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
  name: influxdb-data  
spec:  
  accessModes:  
  - ReadWriteOnce  
  resources:  
    requests:  
      storage: 2Gi
```

$ sudo nano influxdb/influxdb-secret.yaml
```
apiVersion: v1  
kind: Secret  
metadata:  
  name: influxdb-secrets  
type: Opaque  
stringData:  
  INFLUXDB_CONFIG_PATH: /etc/influxdb/influxdb.conf  
  INFLUXDB_ADMIN_USER: admin  
  INFLUXDB_ADMIN_PASSWORD: kraken  
  INFLUXDB_DB: gatling  
  INFLUXDB_USER: user  
  INFLUXDB_USER_PASSWORD: kraken
```

$ sudo nano influxdb/influxdb-config.yaml
```
apiVersion: v1

kind: ConfigMap
metadata:
  name: influxdb-config
data:
  influxdb.conf: |+
    reporting-disabled = false
    bind-address = "127.0.0.1:8088"

    [meta]
      dir = "/var/lib/influxdb/meta"
      retention-autocreate = true
      logging-enabled = true
```

$ cd influxdb;kubectl apply -f .
$ kubectl get pv
```
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
data-kafka             8Gi        RWO            Retain           Bound    nifi/data-kafka-0                                     5d1h
data-kafka-zookeeper   8Gi        RWO            Retain           Bound    nifi/data-kafka-zookeeper-0                           5d1h
pv-local               2Gi        RWO            Retain           Bound    nifi/influxdb-data                                    7d18h
```

$ kubectl get pvc
```
NAME                     STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-kafka-0             Bound    data-kafka             8Gi        RWO                           5d1h
data-kafka-zookeeper-0   Bound    data-kafka-zookeeper   8Gi        RWO                           5d1h
influxdb-data            Bound    pv-local               2Gi        RWO                           7d18h
```

$ kubectl get all
```
NAME                                       READY   STATUS    RESTARTS   AGE
pod/influxdb-deployment-6b89f4f5f6-x76j8   1/1     Running   0          7d18h
pod/kafka-0                                1/1     Running   0          5d1h
pod/kafka-zookeeper-0                      1/1     Running   0          5d1h
pod/mosquitto-76bcf4956f-wgsgp             1/1     Running   0          8d
pod/nifi-0                                 1/1     Running   0          8d
pod/testclient                             1/1     Running   0          5d1h
pod/zookeeper-0                            1/1     Running   0          8d

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP       PORT(S)                         AGE
service/influxdb-service           ClusterIP   10.101.118.82    <none>            8086/TCP                        7d18h
service/kafka                      ClusterIP   10.111.41.89     <none>            9092/TCP                        5d1h
service/kafka-headless             ClusterIP   None             <none>            9092/TCP,9093/TCP               5d1h
service/kafka-zookeeper            ClusterIP   10.99.55.54      <none>            2181/TCP,2888/TCP,3888/TCP      5d1h
service/kafka-zookeeper-headless   ClusterIP   None             <none>            2181/TCP,2888/TCP,3888/TCP      5d1h
service/mosquitto                  ClusterIP   10.100.7.235     192.168.183.131   1883/TCP                        8d
service/nifi                       NodePort    10.103.165.143   <none>            8080:32220/TCP,8082:30788/TCP   8d
service/zookeeper                  ClusterIP   10.106.3.89      <none>            2181/TCP,5111/TCP               8d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/influxdb-deployment   1/1     1            1           7d18h
deployment.apps/mosquitto             1/1     1            1           8d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/influxdb-deployment-6b89f4f5f6   1         1         1       7d18h
replicaset.apps/mosquitto-76bcf4956f             1         1         1       8d

NAME                               READY   AGE
statefulset.apps/kafka             1/1     5d1h
statefulset.apps/kafka-zookeeper   1/1     5d1h
statefulset.apps/nifi              1/1     8d
statefulset.apps/zookeeper         1/1     8d
```

$ kubectl exec -it (pod/influxdb-deployment-6b89f4f5f6-x76j8) -- bash
root@influxdb-deployment-6b89f4f5f6-x76j8:/# influx --username admin --password kraken
Connected to http://localhost:8086 version 1.8.6
InfluxDB shell version: 1.8.6
\> show databases;
name: databases
name
\-----
gatling
_internal

---

### Grafana 安裝  
$ sudo nano grafana.yaml   
```
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: nifi
  labels:
    app: grafana
  name: grafana
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      securityContext:
        fsGroup: 472
        supplementalGroups:
        - 0    
      containers:
        - name: grafana
          image: grafana/grafana:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
              name: http-grafana
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /robots.txt
              port: 3000
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 30
            successThreshold: 1
            timeoutSeconds: 2
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 3000
            timeoutSeconds: 1            
          resources:
            requests:
              cpu: 250m
              memory: 750Mi
          volumeMounts:
            - mountPath: "/etc/grafana"
              name: config
      volumes:
        - name: config
          configMap:
            name: grafana-config
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: nifi
spec:
  type: ClusterIP
  selector:
    app: grafana
  ports:
    - protocol: TCP
      port: 3000
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: nifi
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2 
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /grafana(/|$)(.*)
            backend:
              serviceName: grafana
              servicePort: 3000
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-config
  namespace: nifi
data:
  grafana.ini: |
    [analytics]
    check_for_updates = true
    [server]
    domain = example.com
    root_url = %(protocol)s://%(domain)s:%(http_port)s/grafana/
    serve_from_sub_path = true
    [log]
    mode = console
    [paths]
    data = /var/lib/grafana/
    logs = /var/log/grafana
    plugins = /var/lib/grafana/plugins
    provisioning = /etc/grafana/provisioning
```

$ kubectl apply -f grafana.yaml  
$ kubectl get all  
```
NAME                                       READY   STATUS    RESTARTS   AGE
pod/grafana-6f46b9c7b6-hxwjp               1/1     Running   0          8d
pod/influxdb-deployment-6b89f4f5f6-x76j8   1/1     Running   0          7d18h
pod/kafka-0                                1/1     Running   0          5d1h
pod/kafka-zookeeper-0                      1/1     Running   0          5d1h
pod/mosquitto-76bcf4956f-wgsgp             1/1     Running   0          8d
pod/nifi-0                                 1/1     Running   0          8d
pod/testclient                             1/1     Running   0          5d1h
pod/zookeeper-0                            1/1     Running   0          8d

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP       PORT(S)                         AGE
service/grafana                    ClusterIP   10.105.194.81    <none>            3000/TCP                        8d
service/influxdb-service           ClusterIP   10.101.118.82    <none>            8086/TCP                        7d18h
service/kafka                      ClusterIP   10.111.41.89     <none>            9092/TCP                        5d1h
service/kafka-headless             ClusterIP   None             <none>            9092/TCP,9093/TCP               5d1h
service/kafka-zookeeper            ClusterIP   10.99.55.54      <none>            2181/TCP,2888/TCP,3888/TCP      5d1h
service/kafka-zookeeper-headless   ClusterIP   None             <none>            2181/TCP,2888/TCP,3888/TCP      5d1h
service/mosquitto                  ClusterIP   10.100.7.235     192.168.183.131   1883/TCP                        8d
service/nifi                       NodePort    10.103.165.143   <none>            8080:32220/TCP,8082:30788/TCP   8d
service/zookeeper                  ClusterIP   10.106.3.89      <none>            2181/TCP,5111/TCP               8d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana               1/1     1            1           8d
deployment.apps/influxdb-deployment   1/1     1            1           7d18h
deployment.apps/mosquitto             1/1     1            1           8d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-6f46b9c7b6               1         1         1       8d
replicaset.apps/influxdb-deployment-6b89f4f5f6   1         1         1       7d18h
replicaset.apps/mosquitto-76bcf4956f             1         1         1       8d

NAME                               READY   AGE
statefulset.apps/kafka             1/1     5d1h
statefulset.apps/kafka-zookeeper   1/1     5d1h
statefulset.apps/nifi              1/1     8d
statefulset.apps/zookeeper         1/1     8d
```

---

上網打externalIP/grafana (ingress會幫我們倒到grafana網站)  
![grafana](https://i.imgur.com/TvEjoFA.png)  

進去grafana建立influxdb資料庫, 帳號/密碼看influxdb-secret.yaml  
![influxdb](https://i.imgur.com/DeJaCAQ.png)  
![set](https://i.imgur.com/w2P8NL4.png)  

設定警訊通知  
![alter](https://i.imgur.com/KvqzMom.png)  
