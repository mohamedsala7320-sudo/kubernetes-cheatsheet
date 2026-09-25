##Kubernetes_Most_Important_Commands##
======================================

                                            #**Pods**#
                                          ==============
```bash
1.  kubectl run <pod_name> --image <image_name>  //=====> to create a pod.
2.  kubectl get pods                             //=====> to list the pods in your cluster and get info about it.
3.  kubectl get pods <pod_name>                  //=====> to get info about specific pod on your cluster.
4.  kubectl get pods -o wide                     //=====> to list pods with extra details like (Node IP and Node Name).
5.  kubectl get pods -n <namespace_name>         //=====> to list pods inside a specific namespace.
6.  kubectl describe pod <pod-name>              //=====> to view detailed info, events, and errors for a pod.
7.  kubectl delete pod <pod_name>                //=====> to delete a specific pod.
8.  kubectl exec -it <pod_name> -- bash          //=====> to open an interactive terminal inside a running pod
9.  kubectl exec -it <pod_name> -- sh            //=====> to open an interactive shell inside a pod (using it if bash not available)
10. kubectl get pods -n <namespace_name>         //=====> to list pods inside a specific namespace.
11. kubectl logs <pod_name>                      //=====> to view logs of a specific pod.
12. kubectl logs <pod_name> -f                   //=====> to stream/follow logs of a pod in real-time.
```
                              =====================================================

                                            #**Deployments**#
                                          =====================
```bash
1. kubectl create deployment <deployment_name> --image=<image_name>   //=======> to create a deployment.
2. kubectl get deployments                                            //=======> to list all deployments in your cluster.
3. kubectl scale deployment <deployment_name> --replicas=<number>     //=======> to scale up/down the number of pods.
4. kubectl delete deployment <deployment_name>                        //=======> to delete a deployment.
5. kubectl describe deployment <deployment_name>                      //=======> to get detailed info and events about a deployment.
```
                              ======================================================

                                              #**Services**#
                                           ===================
```bash
1. kubectl expose deployment <deployment_name> --type=NodePort --port=<port>   //=======> to expose a deployment as a service.
2. kubectl get services                                                        //=======> to list all services and their ports.
3. kubectl describe service <service_name>                                     //=======> to get detailed info about a specific service.
4. kubectl delete service <service_name>                                       //=======> to delete a specific service.
5. kubectl get endpoints <service_name>                                        //=======> to view the target pods/IPs connected to a service.
```
                              =========================================================

                                           #**Volumes & Storage**#
                                         ===========================
```bash
1. kubectl get pv                        //=======> to list all Persistent Volumes in the cluster.
2. kubectl get pvc                       //=======> to list all Persistent Volume Claims in the cluster.
3. kubectl describe pvc <pvc_name>       //=======> to get detailed info and status of a specific PVC.
4. kubectl delete pvc <pvc_name>         //=======> to delete a specific Persistent Volume Claim.
```
                              ==========================================================

                                                #**Namespaces**#
                                             ======================
```bash
1. kubectl get namespaces                 //=======> to list all namespaces in the cluster.
2. kubectl create namespace <ns_name>     //=======> to create a new namespace.
3. kubectl get pods -n <namespace_name>   //=======> to list pods inside a specific namespace.
4. kubectl delete namespace <ns_name>     //=======> to delete a specific namespace and its resources.
```

                         ================================================================================
                                           
                               ##Kubernetes Zero Trust Architecture - Network Policies 🛡️##
                                      ==============================================
                                      
هذا التوثيق يوضح سياسات الأمان العازلة (Network Policies) المطبقة لتأمين تطبيق مكون من ثلاث طبقات (Three-Tier Application): Frontend, Backend, و MySQL Database.

1. Default Deny All Policy

هذه السياسة تقوم بحظر جميع حركة المرور (Ingress و Egress) افتراضياً على مستوى الـ Namespace كخطوة أولى لتطبيق مبدأ "Zero Trust".

```bash

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
                                      =======================================
                                      
2. Frontend Egress Policy

تسمح هذه السياسة لطبقة الواجهة الأمامية (Frontend) بإرسال الترافيك حصرياً إلى طبقة الـ Backend على البورت 5000.

```bash

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-allow-egress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 5000
```

                          =====================================================
                          
3. Backend Ingress Policy

تسمح هذه السياسة لطبقة الـ Backend باستقبال الترافيك القادم فقط من الـ Frontend على البورت 5000.

```bash

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-ingress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 5000
```

                       ========================================================

                       
4. Backend Egress Policy

تسمح هذه السياسة لطبقة الـ Backend بإرسال الترافيك نحو قاعدة البيانات MySQL على البورت 3306.

```bash

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-egress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: mysql
    ports:
    - protocol: TCP
      port: 3306
```

                               ================================================
                               
5. MySQL Ingress Policy

تسمح هذه السياسة لقاعدة البيانات MySQL باستقبال الاتصالات حصرياً من الـ Backend على البورت 3306.
```bash

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mysql-allow-ingress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 3306
```
أوامر التطبيق (Deployment Commands):
لتطبيق هذه السياسات دفعة واحدة على الكلاستر:

```bash

kubectl apply -f network-policy-deny-all.yaml
kubectl apply -f network-policy-frontend-egress.yaml
kubectl apply -f network-policy-backend-ingress.yaml
kubectl apply -f network-policy-backend-egress.yaml
kubectl apply -f network-policy-mysql-ingress.yaml

```
