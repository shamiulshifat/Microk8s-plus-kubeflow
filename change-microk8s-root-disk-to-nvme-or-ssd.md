# Move MicroK8s Storage from Root Disk to `/data` SSD

This guide explains how to move MicroK8s runtime storage from the root filesystem `/` to a larger SSD mounted at `/data`.

## Problem

MicroK8s stores most of its runtime data under:

```bash
/var/snap/microk8s/common
```

On this machine, that path is on the root disk:

```bash
/dev/nvme0n1p4  /
```

The root disk is nearly full, while a second NVMe SSD is available:

```bash
/dev/nvme1n1p1  /data
```

Example disk usage:

```bash
df -h
```

Output:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p4  492G  408G   59G  88% /
/dev/nvme1n1p1  1.9T   28K  1.8T   1% /data
```

MicroK8s usage check showed:

```text
263G  /var/snap/microk8s/common/var/lib/containerd
46G   /var/snap/microk8s/common/var/lib/kubelet
307G  /var/snap/microk8s/common
```

This means the main issue is **container runtime storage**, not only PVC storage.

`containerd` contains pulled images, container layers, workflow images, model images, and writable container data.

## Goal

Move this directory:

```bash
/var/snap/microk8s/common
```

To:

```bash
/data/microk8s/common
```

Then bind-mount it back to the original MicroK8s path so MicroK8s continues working normally.

Final result:

```bash
/var/snap/microk8s/common -> /data/microk8s/common
```

MicroK8s will still use the same path internally, but the actual data will live on the larger SSD.

---

# 1. Check Current Disk Usage

```bash
df -h
```

Check MicroK8s storage usage:

```bash
sudo du -xh --max-depth=1 /var/snap/microk8s/common | sort -h
sudo du -xh --max-depth=1 /var/snap/microk8s/common/var/lib | sort -h
sudo du -xh --max-depth=1 /var/snap/microk8s/common/default-storage 2>/dev/null | sort -h
```

Typical large directories:

```bash
/var/snap/microk8s/common/var/lib/containerd
/var/snap/microk8s/common/var/lib/kubelet
/var/snap/microk8s/common/default-storage
```

Meaning:

| Path                                           | Purpose                                                 |
| ---------------------------------------------- | ------------------------------------------------------- |
| `/var/snap/microk8s/common/var/lib/containerd` | Container images, container layers, pulled model images |
| `/var/snap/microk8s/common/var/lib/kubelet`    | Pod runtime data, mounted volumes, kubelet state        |
| `/var/snap/microk8s/common/default-storage`    | Default MicroK8s hostpath PVC data                      |

---

# 2. Confirm `/data` Is Persistent

Run:

```bash
findmnt /data
lsblk -f
grep /data /etc/fstab
```

Expected:

```text
/data  /dev/nvme1n1p1 ext4
```

And `/etc/fstab` should contain something like:

```text
UUID=e42db93a-21be-4336-9208-d1eb99dd8bbd  /data  ext4  defaults  0  2
```

This confirms `/data` will mount automatically after reboot.

Do **not** continue unless `/data` is properly mounted and present in `/etc/fstab`.

---

# 3. Stop MicroK8s

First cache sudo credentials:

```bash
sudo -v
```

Stop MicroK8s:

```bash
sudo microk8s stop
```

Check status:

```bash
sudo microk8s status
```

Check whether MicroK8s pod mounts are still active:

```bash
findmnt -R /var/snap/microk8s/common
```

If you still see many paths like this:

```text
/var/snap/microk8s/common/var/lib/kubelet/pods/...
```

Then stop the snap service directly:

```bash
sudo snap stop microk8s
```

Check again:

```bash
findmnt -R /var/snap/microk8s/common
```

Ideally, there should be no active pod bind mounts before moving the directory.

---

# 4. Copy MicroK8s Data to `/data`

Create the target directory:

```bash
sudo mkdir -p /data/microk8s
```

Copy MicroK8s data using `rsync`:

```bash
sudo rsync -aHAX --numeric-ids \
  /var/snap/microk8s/common/ \
  /data/microk8s/common/
```

Why `rsync`?

* Preserves ownership
* Preserves permissions
* Preserves hard links
* Preserves extended attributes
* Safer than `cp` for runtime data directories

---

# 5. Move Old MicroK8s Directory Aside

Do **not** delete the old directory immediately.

Move it to a backup path:

```bash
sudo mv /var/snap/microk8s/common /var/snap/microk8s/common.bak.$(date +%F)
```

Create a new empty mount point:

```bash
sudo mkdir -p /var/snap/microk8s/common
```

---

# 6. Bind-Mount the New Location

Add a bind mount to `/etc/fstab`:

```bash
echo '/data/microk8s/common /var/snap/microk8s/common none bind 0 0' | sudo tee -a /etc/fstab
```

Mount it immediately:

```bash
sudo mount /var/snap/microk8s/common
```

Verify:

```bash
findmnt /var/snap/microk8s/common
df -h /var/snap/microk8s/common
```

Expected result:

```text
TARGET                       SOURCE
/var/snap/microk8s/common    /data/microk8s/common
```

And the filesystem should show the large `/data` disk, for example:

```text
/dev/nvme1n1p1  1.9T  ...  /var/snap/microk8s/common
```

---

# 7. Start MicroK8s Again

Start MicroK8s:

```bash
sudo microk8s start
```

Wait until ready:

```bash
sudo microk8s status --wait-ready
```

Check node status:

```bash
microk8s kubectl get nodes
```

Check all pods:

```bash
microk8s kubectl get pods -A
```

---

# 8. Confirm MicroK8s Is Now Using `/data`

Run:

```bash
findmnt /var/snap/microk8s/common
df -h /var/snap/microk8s/common
df -h /
df -h /data
```

Check MicroK8s storage again:

```bash
sudo du -xh --max-depth=1 /var/snap/microk8s/common/var/lib | sort -h
```

The large `containerd` and `kubelet` directories should now consume space from `/data`, not from `/`.

---

# 9. Keep Backup Until Everything Works

Check the backup directory:

```bash
ls -lh /var/snap/microk8s/
```

You should see something like:

```text
common.bak.2026-07-06
```

Keep this backup until:

* MicroK8s starts successfully
* Nodes are ready
* Pods are running
* Workflows run normally
* Reboot test passes

---

# 10. Reboot Test

Reboot the machine:

```bash
sudo reboot
```

After reboot, verify:

```bash
findmnt /data
findmnt /var/snap/microk8s/common
df -h /var/snap/microk8s/common
sudo microk8s status --wait-ready
microk8s kubectl get nodes
microk8s kubectl get pods -A
```

Expected:

```text
/var/snap/microk8s/common -> /data/microk8s/common
```

---

# 11. Delete the Old Backup

Only do this after confirming everything works.

Replace the date with the actual backup folder name:

```bash
sudo rm -rf /var/snap/microk8s/common.bak.2026-07-06
```

Then check root disk usage:

```bash
df -h /
df -h /data
```

The root disk should now have significantly more free space.

---

# Optional: Create an SSD-Backed StorageClass for Future PVCs

Moving `/var/snap/microk8s/common` solves the main problem because it moves `containerd`, `kubelet`, and MicroK8s runtime storage to `/data`.

However, if workloads use PVCs, it is also useful to create a dedicated StorageClass that explicitly provisions volumes under `/data`.

Create a directory:

```bash
sudo mkdir -p /data/microk8s-volumes
sudo chmod 700 /data/microk8s-volumes
```

Create a StorageClass:

```bash
cat <<'EOF' > ssd-hostpath-sc.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: ssd-hostpath
provisioner: microk8s.io/hostpath
reclaimPolicy: Delete
parameters:
  pvDir: /data/microk8s-volumes
volumeBindingMode: WaitForFirstConsumer
EOF
```

Apply it:

```bash
microk8s kubectl apply -f ssd-hostpath-sc.yaml
```

Check StorageClasses:

```bash
microk8s kubectl get storageclass
```

Make the new StorageClass default:

```bash
microk8s kubectl annotate storageclass microk8s-hostpath \
  storageclass.kubernetes.io/is-default-class="false" --overwrite

microk8s kubectl annotate storageclass ssd-hostpath \
  storageclass.kubernetes.io/is-default-class="true" --overwrite
```

New PVCs will now be created under:

```bash
/data/microk8s-volumes
```

Important: existing PVCs do **not** automatically move. Existing PVs must be migrated separately if needed.

Check existing PVs:

```bash
microk8s kubectl get pv
microk8s kubectl describe pv <pv-name>
```

---

# Optional: Prune Unused Container Images

This can reduce disk usage, but it does not solve the main issue if MicroK8s remains on `/`.

List images:

```bash
sudo microk8s crictl images
```

Prune unused images:

```bash
sudo microk8s crictl rmi --prune
```

Check containerd size:

```bash
sudo du -xh --max-depth=1 /var/snap/microk8s/common/var/lib/containerd | sort -h
```

---

# Rollback Plan

If MicroK8s fails to start after the move, stop MicroK8s:

```bash
sudo microk8s stop
```

Unmount the bind mount:

```bash
sudo umount /var/snap/microk8s/common
```

Remove the bind mount line from `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Delete or comment out this line:

```text
/data/microk8s/common /var/snap/microk8s/common none bind 0 0
```

Remove the empty mount point:

```bash
sudo rm -rf /var/snap/microk8s/common
```

Restore the backup:

```bash
sudo mv /var/snap/microk8s/common.bak.YYYY-MM-DD /var/snap/microk8s/common
```

Start MicroK8s again:

```bash
sudo microk8s start
sudo microk8s status --wait-ready
microk8s kubectl get nodes
microk8s kubectl get pods -A
```

---

# Summary

Recommended fix for this machine:

1. Stop MicroK8s.
2. Copy `/var/snap/microk8s/common` to `/data/microk8s/common`.
3. Move the old directory aside as backup.
4. Bind-mount `/data/microk8s/common` back to `/var/snap/microk8s/common`.
5. Start MicroK8s.
6. Verify pods and node status.
7. Reboot and verify again.
8. Delete the backup only after everything works.

This makes MicroK8s use the 1.8T SSD while preserving the original path MicroK8s expects.
