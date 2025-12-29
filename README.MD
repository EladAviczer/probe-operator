# Readiness Controller (Probe Architecture)

![Version](https://img.shields.io/badge/version-0.1.0-blue.svg) ![Kubernetes](https://img.shields.io/badge/kubernetes-1.19+-green.svg)
[![CI](https://github.com/EladAviczer/readiness-controller/actions/workflows/ci.yaml/badge.svg)](https://github.com/EladAviczer/readiness-controller/actions/workflows/ci.yaml)

**Readiness Controller** is a Kubernetes controller that implements **Centralized Dependency Probing**.

It continuously checks the health of shared resources (Databases, External APIs) and publishes their status to native **Kubernetes Custom Resources (`Probe`)**.

## The Problem
When 50 microservices all depend on a single database, standard Kubernetes Probes create inefficiencies:

1.  **The "Thundering Herd":** 50 replicas checking the database every 5 seconds = **600 requests per minute** just for health checks.
2.  **Opacity:** If a dependency fails, it's hard to distinguish app errors from dependency errors.

## The Solution
**Readiness Controller** acts as a **Singleton Sentinel**.

1.  **Check Once:** The controller checks the dependency **one time**.
2.  **Publish Status:** It updates a `Probe` resource in Kubernetes with the result.
3.  **Consume Anywhere:** Your applications, dashboards, or other controllers can simply watch these resources to know the health of the system.

---

## Features
*   **Probe CRD:** Native accessible status via `kubectl get probes`.
*   **Pluggable Probers:**
    *   `http`: Check REST APIs or Webhooks.
    *   `tcp`: Check Databases, Queues, or raw sockets.
    *   `exec`: Run custom scripts (curl, grep, jq).
*   **Status Dashboard:** Built-in web UI (port 8080) to visualize all probes.

---

## Quick Start

### 1. Prerequisites
*   Kubernetes 1.19+
*   Helm 3+

### 2. Install
Clone the repository and install using Helm:

```bash
git clone https://github.com/EladAviczer/readiness-controller.git
cd readiness-controller

# Install the chart (includes CRD)
helm install rc ./charts/readiness-controller
```

### 3. Usage
Once installed and configured, the controller will automatically create `Probe` resources for your defined rules.

**Check Status:**
```bash
kubectl get probes
```

**Output:**
```
NAME            HEALTHY   MESSAGE        AGE
postgres-main   true      Check passed   5m
redis-cache     false     Check failed   5m
```

**Get Details:**
```bash
kubectl describe probe postgres-main
```