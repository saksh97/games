# Automated Dynamic Multi-Tenant Web Deployment Pipeline 🚀

An automated CI/CD pipeline built with **Jenkins** and **Docker** on **AWS EC2** that handles dynamic, multi-tenant static web applications with zero downtime and asset-caching prevention.

This pipeline dynamically fetches open-source HTML5 games from distinct public repositories (**2048 Puzzle Game** & **Tic-Tac-Toe**) and hot-swaps them inside an isolated Nginx container environment on-demand.

---

## 🛠️ Step-by-Step Implementation Guide

### Step 1: AWS EC2 Storage Expansion & Optimization
During initial pipeline executions, the agent triggered a `Free Temp Space` threshold warning due to restricted system storage limits.
1. Expanded the **AWS EBS Volume** capacity from the AWS Management Console.
2. Allocated the unpartitioned hard drive blocks to the Linux root partition via SSH:
   ```bash
   sudo growpart /dev/nvme0n1 1
   sudo resize2fs /dev/nvme0n1p1
   ```
3. Added a permanent **2GB Swap file** onto the expanded space to handle continuous integration load stabilities:
   ```bash
   sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
   sudo mkswap /swapfile && sudo swapon /swapfile
   echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
   ```

### Step 2: Labeled Jenkins Distributed Architecture
To protect the local controller container from system layer resource exhaustion, workloads were moved onto a dedicated remote runner node.
1. Configured a standalone target node with the label string: `agent1`.
2. Granted the agent execution shell passwordless `sudo` rights so Jenkins could run configuration installations flawlessly:
   ```bash
   echo "ubuntu ALL=(ALL) NOPASSWD:ALL" | sudo tee -a /etc/sudoers
   ```

### Step 3: Container Host-Swapping & Cache Flushing Mechanics
When static files inside an active container update via raw `nginx -s reload` signals, Nginx can aggressively cache static layout trees. To achieve a reliable state transformation, the pipeline lifecycle destroys old containers on every build run:
1. Validates and updates the core systems utilities (`unzip`, `docker.io`) dynamically at runtime.
2. Erases old stale container stacks entirely: `sudo docker rm -f my-nginx-container`.
3. Provisions a clean, micro-isolated image instance: `sudo docker run -d -p 81:80 --name my-nginx-container nginx:alpine`.
4. Uses `docker cp` streams to inject the chosen game assets directly into the internal Nginx routing path (`/usr/share/nginx/html/`).

---

## 🎮 How to Run and Deploy the Project

1. Log into your **Jenkins Dashboard** and click on your pipeline project.
2. Click **Build with Parameters** in the left sidebar menu.
3. Open the **CHOSEN_GAME** dropdown menu parameter selection window and choose your deployment:
   * 🕹️ `2048_Puzzle_Game`
   * ❌ `Tic_Tac_Toe`
4. Click the blue **Build** button.
5. Once the pipeline finishes with a green **SUCCESS** banner, access your application live in your web browser:
   ```text
   http://13.201.193.8:81
   ```

---

## 🧰 Technology Stack Used
* **CI/CD Orchestration:** Jenkins (Declarative Pipelines)
* **Containerization:** Docker Engine (`nginx:alpine`)
* **Cloud Infrastructure:** AWS EC2 (Ubuntu Linux)
* **Source Repositories:** Git & GitHub
