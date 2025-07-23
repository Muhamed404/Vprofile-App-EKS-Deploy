#  Vprofile App – Kubernetes Deployment on AWS EKS

This repository contains Kubernetes manifests to deploy the **Vprofile application** (multi‑tier) on an **EKS cluster** with persistent storage and ingress access.

---

## 📂 **Kubernetes Manifests**

| File                | Purpose                                                                   |
| ------------------- | ------------------------------------------------------------------------- |
| `appdeploy.yaml`    | Deployment for the application (Tomcat app)                               |
| `appservice.yaml`   | Service (ClusterIP) to expose the application internally |
| `appingress.yaml`   | Ingress resource to expose the app externally via AWS ALB                 |
| `dbdeploy.yaml`     | Deployment for MySQL database                                             |
| `dbservice.yaml`    | Service to access MySQL internally                                        |
| `dbpvc.yaml`        | PersistentVolumeClaim for database storage                                |
| `storageclass.yaml` | StorageClass for EBS dynamic provisioning                                 |
| `mcservice.yaml`    | Service for Memcached                                                     |
| `rmqdeploy.yaml`    | Deployment for RabbitMQ                                                   |
| `rmqservice.yaml`   | Service for RabbitMQ                                                      |
| `secret.yaml`       | Kubernetes Secret for sensitive data (RabbitMQ password, DB password)                  |

---

## **Architecture**

The application consists of:

* **Frontend/App Tier** (Tomcat) 
* **Database Tier** (MySQL) with persistent storage 
* **Caching Layer** (Memcached)
* **Messaging Layer** (RabbitMQ)
* **Ingress Controller** (AWS Load Balancer Controller) exposing the app externally

---

##  **Deployment Order**

Apply the manifests in the following order:

```bash
# 1. Create StorageClass
kubectl apply -f storageclass.yaml

# 2. Create Secrets
kubectl apply -f secret.yaml

# 3. Create Database PVC
kubectl apply -f dbpvc.yaml

# 4. Deploy Database
kubectl apply -f dbdeploy.yaml
kubectl apply -f dbservice.yaml

# 5. Deploy Caching Layer
kubectl apply -f mcservice.yaml

# 6. Deploy Messaging Layer
kubectl apply -f rmqdeploy.yaml
kubectl apply -f rmqservice.yaml

# 7. Deploy Application
kubectl apply -f appdeploy.yaml
kubectl apply -f appservice.yaml

# 8. Deploy Ingress
kubectl apply -f appingress.yaml
```

---

## **Ingress**

The `appingress.yaml` exposes the application using **AWS ALB**:

* Make sure AWS Load Balancer Controller is installed and has the proper IAM role.
* Example annotation:

  ```yaml
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
  ```

---

## **Storage**

* `storageclass.yaml` uses **EBS CSI driver** with `WaitForFirstConsumer`.
* `dbpvc.yaml` claims persistent storage for MySQL.
* Ensure that **OIDC** is enabled on the cluster and the service account for the EBS CSI driver has the proper IAM role (AmazonEBSCSIDriverPolicy).

---

##  **Prerequisites**

* An **EKS cluster** with OIDC enabled.
* AWS Load Balancer Controller installed.
* EBS CSI driver installed with a ServiceAccount that has `AmazonEBSCSIDriverPolicy`.

### Driver Installation Documentation

For storage and ingress to work properly, you need to install the following drivers:

- **AWS EBS CSI Driver:** Documentation: [Amazon EBS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)

- **AWS Load Balancer Controller (for ALB):** Documentation: ([AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html)
---

##  **Deployment**

After applying all manifests, verify resources:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
kubectl get pvc
```

Your application should be accessible through the **Ingress ALB DNS**.

---

## **Troubleshooting**

* **PVC Pending** → Check EBS CSI driver & IAM permissions.
* **Ingress Failing** → Ensure Service ports are correct and ALB Controller IAM role is configured.
