---
title: "Clone Data Between EC2 Instances with EBS Snapshots"
datePublished: Fri Sep 05 2025 19:08:52 GMT+0000 (Coordinated Universal Time)
cuid: cmf77kplv000402ju3fj3falz
slug: clone-data-between-ec2-instances-with-ebs-snapshots
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1757099039633/9eb5147d-73d2-4ce5-8a71-de56ec297776.jpeg
tags: aws, cloud-computing, ebs-snapshots

---

Want to move a folder of files from one EC2 instance to another—without SCP or rsync? EBS snapshots make it easy to “copy” a whole data volume. In this tutorial, you’ll create an EBS volume on **Instance A**, put files on it, take a **snapshot**, create a new volume from that snapshot, attach it to **Instance B**, and verify the files using **MobaXterm**.

---

## What you’ll build

```plaintext
[Instance A] --(EBS Volume)--> [Snapshot] --(Create Volume)--> [Instance B]
```

* **A**: Create & mount a data EBS volume, add test files
    
* **B**: Snapshot that volume
    
* **C**: Create a new volume from the snapshot and attach to another EC2 instance
    
* **D**: Verify files via MobaXterm terminal & SFTP browser
    

---

## Prerequisites

* Two running **EC2** instances (Linux, any distro) in the **same region**
    
* Ability to SSH from your machine (use **MobaXterm** on Windows)
    
* IAM permissions to manage EC2 volumes/snapshots
    
* Security Group allows inbound SSH (port 22) from your IP
    

> ⚠️ **AZ rule:** You can only attach a volume to an instance in the **same Availability Zone (AZ)**. Make sure the volume you create is in the AZ of the target instance.

---

## Step 1 — Create a new EBS data volume (for Instance A)

1. Go to **AWS Console → EC2 → Volumes → Create volume**.
    
2. **Volume type:** gp3 (good default)
    
3. **Size:** e.g., 4–10 GiB for demo
    
4. **Availability Zone:** **must match** Instance A’s AZ (e.g., `ap-south-1a`)
    
5. Click **Create volume**.
    

**Attach it:**

1. Select the new volume → **Actions → Attach volume**.
    
2. Choose **Instance A**.
    
3. Device name suggestion: `/dev/sdf` (will appear as `/dev/xvdf` on Linux; on Nitro instances it often becomes `/dev/nvme1n1`).
    

---

## Step 2 — Connect to Instance A with MobaXterm

1. Open **MobaXterm** → **Session → SSH**.
    
2. Enter **Remote host** = public IP/DNS of Instance A, **Username** = `ec2-user` (Amazon Linux) or `ubuntu` (Ubuntu).
    
3. Add your key under **Advanced SSH settings** if needed → **OK**.
    

In the terminal, find the new disk:

```bash
lsblk
```

You should see an unmounted disk (e.g., `/dev/xvdf` or `/dev/nvme1n1`) with no filesystem.

> 🧭 **Device name tip:**
> 
> * Older/virtualized: likely `/dev/xvdf`
>     
> * Nitro-based: likely `/dev/nvme1n1`  
>     Substitute the correct device in the commands below.
>     

---

## Step 3 — Format & mount the volume on Instance A

> ⚠️ **Run these only once on a brand-new empty volume.** Don’t format a volume that already has data you care about.

```bash
# Pick the right device (change if your lsblk shows nvme1n1)
DEV=/dev/xvdf

# Create an ext4 filesystem
sudo mkfs -t ext4 $DEV

# Create a mount point and mount it
sudo mkdir -p /dataA
sudo mount $DEV /dataA

# Set basic permissions (optional)
sudo chown -R $USER:$USER /dataA
```

Create a few test files:

```bash
echo "Hello from Instance A" > /dataA/hello.txt
mkdir -p /dataA/reports
echo "Q3 report" > /dataA/reports/q3.txt
```

Verify:

```bash
ls -lah /dataA
ls -lah /dataA/reports
```

> (Optional) To auto-mount on reboot, get the UUID and add to `/etc/fstab`:
> 
> ```bash
> sudo blkid $DEV
> # Copy the UUID and add a line like below to /etc/fstab:
> # UUID=<that-uuid>  /dataA  ext4  defaults,nofail  0  2
> ```

---

## Step 4 — Create a snapshot of the volume

1. **EC2 → Volumes** → select the data volume you attached to Instance A.
    
2. **Actions → Create snapshot**.
    
3. Add a useful **name/description** (e.g., `dataA-snap-01`) → **Create snapshot**.
    
4. Wait for **Status = completed**.
    

> ✅ You can snapshot while the volume is in use; for perfect consistency, stop writes or unmount first. For this demo, our simple files are fine.

---

## Step 5 — Create a new volume from the snapshot (for Instance B)

1. Go to **EC2 → Snapshots** → select your snapshot.
    
2. **Actions → Create volume from snapshot**.
    
3. **Availability Zone:** choose **Instance B’s AZ** (critical!).
    
4. Keep size/type (or adjust) → **Create volume**.
    

---

## Step 6 — Attach the new volume to Instance B

1. **EC2 → Volumes** → select the volume you just created from the snapshot.
    
2. **Actions → Attach volume**.
    
3. Choose **Instance B** and confirm the device name (e.g., `/dev/sdf`).
    

---

## Step 7 — Mount & verify the data on Instance B with MobaXterm

SSH to **Instance B** in MobaXterm the same way as before.

Identify the device:

```bash
lsblk
```

> You should see the new device (e.g., `/dev/xvdf` or `/dev/nvme1n1`).  
> **Do not** run `mkfs` here—this volume already contains the filesystem from the snapshot.

Create a mount point and mount:

```bash
DEV=/dev/xvdf   # change if needed
sudo mkdir -p /dataFromA
sudo mount $DEV /dataFromA
sudo chown -R $USER:$USER /dataFromA   # optional for easy listing
```

List the files cloned from Instance A:

```bash
ls -lah /dataFromA
ls -lah /dataFromA/reports
cat /dataFromA/hello.txt
```

You should see `hello.txt` and the `reports` folder with `q3.txt`—exactly as created on Instance A.

---

## Step 8 — (Nice) Verify via MobaXterm’s SFTP side panel

1. In your **SSH tab to Instance B**, click the left **SFTP** panel (MobaXterm usually opens it automatically).
    
2. Navigate to `/dataFromA`.
    
3. You can view, download, or drag-and-drop files—handy visual confirmation.
    

---

## Step 9 — Cleanup (to avoid charges)

* On **Instance B**:
    
    ```bash
    sudo umount /dataFromA
    ```
    
* On **Instance A** (if you’re done):
    
    ```bash
    sudo umount /dataA
    ```
    
* In **EC2 Console**:
    
    * **Detach** any volumes you no longer need
        
    * **Delete** the extra volume created from snapshot (if finished)
        
    * **Delete** the snapshot (and original data volume if this was just a demo)
        

---

## Common gotchas & fixes

* **“Volume is in-use” when detaching:** make sure the filesystem is **unmounted** on the instance.
    
* **Device names differ:** `/dev/sdf` may appear as `/dev/xvdf` or `/dev/nvme1n1`. Always confirm with `lsblk`.
    
* **Mount fails (“wrong fs type”):** Make sure you didn’t accidentally run `mkfs` on the snapshot-derived volume.
    
* **Permissions denied when listing:** use `sudo ls -lah /mountpoint` or adjust ownership with `sudo chown`.
    

---

## Why this approach?

* **Fast cloning** of data volumes between instances
    
* **Immutable snapshot history** for rollback/backup
    
* **No data transfer tools** required—AWS handles it under the hood
    

---

### Final recap

1. Create & attach an **empty EBS volume** to Instance A → **format & mount** → add test files.
    
2. **Snapshot** the volume.
    
3. **Create a new volume from the snapshot** in Instance B’s **AZ**.
    
4. **Attach** to Instance B → **mount** → verify files with **MobaXterm** terminal and **SFTP**.
    

You’ve just moved a dataset between EC2s using pure EBS magic—clean, consistent, and quick!