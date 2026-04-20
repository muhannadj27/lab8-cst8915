# Algonquin Pet Store (On Steroids) — Lab 8 Deployment

## CST8915 — Full Stack Cloud Development

**Student:** Muhannad J  
**Course:** CST8915 — Winter 2026  
**Professor:** Ramy Mohamed

---

## Demo Video

[![Watch the Demo](https://img.shields.io/badge/YouTube-Demo_Video-red?style=for-the-badge&logo=youtube)](https://youtu.be/MWMXBwMM1lE)

---

## Overview

This lab demonstrates the deployment of **Algonquin Pet Store (On Steroids)** — a microservices-based e-commerce application — to **Azure Kubernetes Service (AKS)**. The app showcases a polyglot architecture, event-driven design, and common open-source backend services.

## Architecture

```
                         ┌──────────────┐
                    ┌───▸│ order-service │───────┐
                    │    │   (Node.js)   │       │
┌───────────┐       │    └──────────────┘       ▼
│ store-front│───────┤                     ┌───────────┐
│  (Vue.js)  │       │    ┌──────────────┐ │order queue│
└───────────┘       ├───▸│product-service│ │(RabbitMQ) │
                    │    │    (Rust)     │ └─────┬─────┘
                    │    └──────────────┘       │
                    │                           ▼
┌───────────┐       │    ┌────────────────┐  ┌──────────┐
│store-admin │───────┘───▸│makeline-service│─▸│ MongoDB  │
│  (Vue.js)  │            │     (Go)       │  │(Database)│
└───────────┘            └───────┬────────┘  └──────────┘
                                 │
                          ┌──────▼──────┐
                          │ ai-service  │──▸ OpenAI (GPT-4 + DALL·E)
                          │  (Python)   │
                          └─────────────┘
```

## Services

| Service | Description | Technology |
|---------|-------------|------------|
| **store-front** | Customer-facing web app for browsing and ordering | Vue.js |
| **store-admin** | Employee dashboard for managing orders and products | Vue.js |
| **order-service** | Handles order placement | Node.js |
| **product-service** | CRUD operations for products | Rust |
| **makeline-service** | Processes orders from the queue | Go |
| **ai-service** | Generates product descriptions and images (optional) | Python |
| **rabbitmq** | Message queue for order processing | RabbitMQ |
| **mongodb** | Persistent data storage | MongoDB |
| **virtual-customer** | Simulates customer order creation | Rust |
| **virtual-worker** | Simulates order completion | Rust |

## Deployment Steps

### Prerequisites

- Azure CLI installed and configured
- `kubectl` installed
- An active Azure subscription

### 1. Login to Azure

```bash
az login
```

### 2. Create Resource Group

```bash
az group create --name myPetStoreRG --location westus3
```

### 3. Create AKS Cluster

```bash
az aks create \
  --resource-group myPetStoreRG \
  --name myPetStoreAKS \
  --node-count 2 \
  --generate-ssh-keys \
  --location westus3 \
  --node-vm-size Standard_B2s_v2
```

> **Note:** The default `Standard_DS2_v2` VM size is not available on Azure for Students subscriptions in `westus3`. Using `Standard_B2s_v2` as an alternative.

### 4. Connect to the Cluster

```bash
az aks get-credentials --resource-group myPetStoreRG --name myPetStoreAKS
kubectl get nodes
```

### 5. Deploy the Application

```bash
cd "Deployment Files"
kubectl apply -f secrets.yaml
kubectl apply -f config-maps.yaml
kubectl apply -f aps-all-in-one.yaml
kubectl apply -f admin-tasks.yaml
```

### 6. Verify Deployment

```bash
kubectl get pods
kubectl get services
```

## Accessing the Application

| App | URL | Type |
|-----|-----|------|
| **Store Front** | `http://<STORE_FRONT_EXTERNAL_IP>` | Customer-facing shop |
| **Store Admin** | `http://<STORE_ADMIN_EXTERNAL_IP>` | Employee admin dashboard |

External IPs are assigned automatically via Azure Load Balancer.

## Challenges Encountered

- **VM Size Restriction:** The default `Standard_DS2_v2` is not permitted on Azure for Students in `westus3`. Resolved by specifying `Standard_B2s_v2`.
- **AI Service:** The `ai-service` pod shows `CreateContainerConfigError` without a valid OpenAI API key in `secrets.yaml`. The rest of the application functions fully without it.

## Cleanup

To avoid ongoing Azure charges, delete all resources when done:

```bash
az group delete --name myPetStoreRG --yes --no-wait
```

## Acknowledgments

- Original application inspired by the [AKS Store Demo](https://github.com/Azure-Samples/aks-store-demo)
- Instructor: Ramy Mohamed — Algonquin College
