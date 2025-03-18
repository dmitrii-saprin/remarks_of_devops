# 🚀 Running GitLab Runner and Docker in RAM

## 🎯 Introduction
To accelerate CI/CD processes, you can run **GitLab Runner** and **Docker** in RAM using `tmpfs`. This reduces disk I/O load and boosts performance, though it demands significant RAM (e.g., 10-20 GB depending on workload). This guide explains how to configure `tmpfs` for these services.

---

## 🔥 What is tmpfs?
`tmpfs` is a **temporary file system** that resides in RAM instead of on disk. Its key benefits are:
- ⚡ **Exceptional speed** due to RAM-based storage.
- 💾 **Reduced wear** on physical disks, especially SSDs.
- 📏 **Dynamic size adjustment** based on available memory (up to a specified limit).
- ⚠️ **Primary limitation**: All data is erased on system reboot.

---

## 🛠️ Running GitLab Runner in RAM

### ✅ 1. Creating the `/builds` directory
Before mounting, ensure the `/builds` directory exists:

```bash
sudo mkdir -p /builds
```

### 🏗️ 2. Temporarily mounting `/builds`
GitLab Runner uses `/builds` for temporary files. Mount this directory as `tmpfs`:

```bash
sudo mount -t tmpfs -o size=15G,mode=1777 tmpfs /builds
```

Verify the mount:
```bash
mount | grep /builds
```

### 🔄 3. Automating mount via `/etc/fstab`
Add the following line to `/etc/fstab`:

```fstab
tmpfs /builds tmpfs defaults,size=15G,mode=1777 0 0
```

Apply the changes:
```bash
sudo mount -a
```

### 📝 4. Configuring GitLab Runner
Open the GitLab Runner configuration file:
```bash
sudo vi /etc/gitlab-runner/config.toml
```
Modify or add the following parameters:

```toml
[[runners]]
  name = "your_runner_name"
  url = "https://your-gitlab-instance.com/"
  clean_runner_cache = true
  builds_dir = "/builds"
```

Restart GitLab Runner:
```bash
sudo systemctl restart gitlab-runner
sudo systemctl status gitlab-runner
```

---

## 🐳 Running Docker Fully in RAM
Docker stores its data in `/var/lib/docker`. Moving this directory to RAM enhances container performance.

### 📌 1. Adding mount points in `/etc/fstab`
Add the following lines:

```fstab
tmpfs /var/lib/docker         tmpfs defaults,size=5G,mode=1777 0 0
tmpfs /var/lib/docker/volumes tmpfs defaults,size=5G,mode=1777 0 0
tmpfs /var/lib/docker/containerd tmpfs defaults,size=5G,mode=1777 0 0
```

Apply the changes:
```bash
sudo mount -a
```

Verify the mount:
```bash
mount | grep /var/lib/docker
```

### 🔄 2. Restarting Docker
```bash
sudo systemctl stop docker docker.socket
sudo systemctl start docker.socket
sudo systemctl start docker
docker info
```

---

## 🛑 Possible Issues and Solutions

### 🚨 1. Insufficient memory
Check available RAM:
```bash
free -m
```

### 🐳 2. Docker fails to start
Ensure it is using `tmpfs`:
```bash
docker info | grep "Docker Root Dir"
```

### ❗ 3. Data loss after reboot
This method should be used **only for temporary files and cache**.

---

## 🎉 Conclusion
Running **GitLab Runner** and **Docker** in RAM significantly accelerates CI/CD processes but requires sufficient memory. This approach is ideal for **temporary data** and **faster builds** but is not suited for **persistent container storage**.

By following this guide, you can create a **high-performance environment** for working with containers and builds. 🚀🔥

