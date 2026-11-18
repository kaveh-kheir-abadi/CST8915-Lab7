# CST8915 – Lab 7: Introduction to Kubernetes Basics

**Student:** Kaveh Kheir Abadi  
**Course:** CST8915 – Full-stack Cloud-native Development  
**Semester:** Fall 2026

## 1. Demo Video

YouTube Demo: https://youtu.be/J0jB_ndu1q4

## 2. Lab Overview

In this lab, I deployed the Algonquin Pet Store application on Azure Kubernetes Service (AKS) using a single YAML configuration file.

The application includes four microservices:

- RabbitMQ
- Order Service
- Product Service
- Store Front

I used `kubectl` to deploy the application, check the Pods and Services, and access the Store Front through its external IP address.

## 3. RabbitMQ Configuration Issue

RabbitMQ is a **stateful application** because it manages message queues and stores messages that may need to survive restarts.

In the provided YAML file, RabbitMQ runs as a Kubernetes Deployment without a Persistent Volume or Persistent Volume Claim (PVC).

This means its data is stored inside the container's writable filesystem rather than on persistent storage.

When the RabbitMQ Pod is deleted and replaced, its stored data can be lost.

## 4. Testing the RabbitMQ Problem

During the lab, I performed the following steps:

1. Deployed the Algonquin Pet Store application using `kubectl apply`.
2. Verified that all four Pods were running.
3. Accessed the Store Front using the external IP address and submitted an order.
4. Deleted the RabbitMQ Pod and observed that Kubernetes automatically created a replacement Pod.
5. Created a durable RabbitMQ queue named `lab7-test`.
6. Verified that the queue existed with `durable=true`.
7. Deleted the RabbitMQ Pod again.
8. Checked the queues in the replacement Pod.

**Result:** The `lab7-test` queue disappeared after the Pod was replaced.

This demonstrated that Kubernetes can automatically recreate failed Pods, but it does not automatically preserve application data.

## 5. Potential Solutions

The main solution is to configure persistent storage for RabbitMQ.

A **Persistent Volume Claim (PVC)** can be used to provide storage that survives Pod replacement. The volume should be mounted to RabbitMQ's data directory.

A **StatefulSet** is also a better choice for RabbitMQ because it provides stable Pod identities and supports persistent storage for stateful workloads.

For a production environment, RabbitMQ should use properly configured persistent volumes, durable queues, persistent messages, and appropriate backup or replication strategies.

## 6. Does Azure Service Bus Solve This Problem?

Azure Service Bus can address the persistence and infrastructure-management problems demonstrated in this lab.

Unlike our RabbitMQ Deployment, Azure Service Bus is a fully managed messaging service. Azure manages the underlying messaging infrastructure and provides durable message storage.

This means we would not need to manage RabbitMQ Pods or configure Kubernetes Persistent Volumes for the messaging service.

However, Azure Service Bus is not a direct replacement without changes. The Order Service would need to use the Azure Service Bus SDK or supported protocol integration, and the messaging configuration would need to be updated.

Therefore, Azure Service Bus is a possible alternative that reduces the operational responsibilities associated with running RabbitMQ inside Kubernetes.

## 7. Conclusion

This lab helped me understand how Kubernetes manages Pods, Deployments, and Services using YAML files.

I also learned the difference between Kubernetes self-healing and data persistence.

Although Kubernetes automatically recreated the RabbitMQ Pod, the durable queue was lost because persistent storage was not configured.

For stateful applications such as RabbitMQ, persistent storage is essential to prevent data loss during Pod replacement.
