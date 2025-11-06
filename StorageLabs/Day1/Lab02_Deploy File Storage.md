# 🗂️ Lab 2 – Deploy File Storage (NFS File Share on EC2)

**Related Topics:** NAS, NFS, File Storage  

---

### 🎯 Objective
Simulate file-based network storage using **NFS** on a single EC2 instance.

---

### 🕓 Estimated Time
45–60 minutes

---

### 🧰 Resources Required
- 1 EC2 instance (reuse from Lab 1)

---

### 🪜 Steps

#### 1. Connect to Your Instance
```bash
ssh -i <your-key.pem> ec2-user@<public-ip>
```

#### 2. Install NFS Tools
```bash
sudo yum install -y nfs-utils
```

#### 3. Create a Shared Directory
```bash
sudo mkdir -p /srv/nfs/share
sudo chmod 777 /srv/nfs/share
```

#### 4. Configure NFS Exports
```bash
echo "/srv/nfs/share *(rw,sync,no_root_squash)" | sudo tee /etc/exports
sudo exportfs -rav
sudo systemctl enable --now nfs-server
```

#### 5. Mount the Share (as Client)
```bash
sudo mkdir /mnt/nfsclient
sudo mount -t nfs localhost:/srv/nfs/share /mnt/nfsclient
```

#### 6. Test File Access
```bash
touch /mnt/nfsclient/test.txt
ls /srv/nfs/share
```

---

### ✅ Expected Result
You’ve created and mounted an **NFS file share**, showing how file-based storage is shared between systems.

---

### 🧹 Cleanup
```bash
sudo umount /mnt/nfsclient
sudo systemctl stop nfs-server
```

---
