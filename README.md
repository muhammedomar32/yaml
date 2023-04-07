     i try to deploy MinIO distributed Cluster on Kubernetes , i wrote yaml file in the below link , i got errors logs from the pods , could you please review my yaml and support me for run minio as Cluster MinIO on Kubernetes cluster 4 replicas, running on minikube please check my yaml file here

[![enter image description here][1]][1]


  [1]: https://i.stack.imgur.com/sVc5W.jpg
##
--------------------
    #Create Namespace
        apiVersion: v1    
        kind: Namespace   
        metadata:
          name: minio
        ---
    #Create Service
        apiVersion: v1    
        kind: Service
        metadata:
          name: minio
          labels:
            app: minio
        spec:
          clusterIP: None
          ports:
            - port: 9000
              name: minio
          selector:
            app: minio
        ---
    #Create PODs
        apiVersion: apps/v1
        kind: StatefulSet
        metadata:
          name: minio
        spec:
          serviceName: minio
          replicas: 4
          selector:
            matchLabels:
              app: minio 
          template:
            metadata:
              annotations:
                pod.alpha.kubernetes.io/initialized: "true"
              labels:
                app: minio
            spec:
              containers:
              - name: minio
                env:
                - name: MINIO_ACCESS_KEY
                  value: "minio"
                - name: MINIO_SECRET_KEY
                  value: "minio123"
                image: minio/minio:RELEASE.2020-10-03T02-19-42Z
                args:
                - server
                - http://minio-0.minio.minio.svc.cluster.local/data
                - http://minio-1.minio.minio.svc.cluster.local/data
                - http://minio-2.minio.minio.svc.cluster.local/data
                - http://minio-3.minio.minio.svc.cluster.local/data
                ports:
                - containerPort: 9000
                volumeMounts:
                - name: data
                  mountPath: /data
          volumeClaimTemplates:
          - metadata:
              name: data
            spec:
              storageClassName: standard
              accessModes:
                - ReadWriteOnce
              resources:
                requests:
                  storage: 10Gi
        ---
    #Create LoadBalancer
        apiVersion: v1
        kind: Service
        metadata:
          name: minio-service
        spec:
          type: LoadBalancer
          ports:
            - port: 9000
              targetPort: 9000    
              protocol: TCP    
          selector:
            app: minio    
