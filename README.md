# Game-2408 Kubernetes (Fargate) Project with AWS Load Balancer Controller

This project demonstrates deploying a game application on **Amazon EKS** with an **ALB Ingress** setup. The Ingress Controller manages traffic routing to services running in different namespaces, making it accessible externally via AWS Application Load Balancer (ALB).


---

## Project Overview

The project showcases:
- Kubernetes deployment of a game application in namespace `game-2408`.
- ALB Ingress via **AWS Load Balancer Controller** installed in `kube-system` namespace.
- IAM role and policy configuration for the controller.
- Automatic creation of ALB target groups and routing rules.

---

## Architecture

       ┌────────────────────┐
       │   AWS ALB          │
       │   (Public)         │
       └─────────┬──────────┘
                 │
           Ingress Rules
                 │
       ┌─────────▼──────────┐
       │ AWS LB Controller   │ (kube-system)
       └─────────┬──────────┘
                 │
        ┌────────▼─────────┐
        │ Kubernetes Service│
        │ (game-2408 ns)   │
        └────────┬─────────┘
                 │
         ┌───────▼───────┐
         │ Game Pods      │
         └───────────────┘

- Ingress controller resides in `kube-system` but manages services across namespaces.
- Traffic flows from public ALB → Ingress → Service → Pods.
![65d5b018-2ca1-4b58-85ef-cfb9b96aa7b7](https://github.com/user-attachments/assets/85a2a9c9-4c4f-4f67-ad25-8abf7b879720)

---

## Prerequisites

- AWS Account with permission to create EKS clusters, IAM roles, and ALBs.
- `kubectl` installed and configured.
- `eksctl` installed.
- Helm 3+ installed.

---

