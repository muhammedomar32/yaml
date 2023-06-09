    i try to deploy MinIO distributed Cluster on Kubernetes , i wrote yaml file in the below link , i got errors logs from the pods , 
    could you please review my yaml and support me for run minio as Cluster MinIO on Kubernetes cluster 4 replicas, running on minikube please check my yaml file here

##
--------------------
    apiVersion: v1
    kind: Namespace
    metadata:
      name: minio # 
      labels:
        name: minio
    ---
    
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
    
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: minio
      namespace: minio
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
              # This ensures containers are allocated on separate hosts. Remove hostPort to allow multiple Minio containers on one host
            # These volume mounts are persistent. Each pod in the PetSet
            # gets a volume mounted based on this field.
            volumeMounts:
            - name: data
              mountPath: /data
      # These are converted to volume claims by the controller
      # and mounted at the paths mentioned above.
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
          # Uncomment and add storageClass specific to your requirements below. Read more https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1
          #storageClassName:
          
    ---
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
