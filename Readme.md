# EC2 Key Recovery Project - README

## Project Overview

This project documents the process of regaining SSH access to an EC2 instance (Server1) after losing its private key, by using a new key pair and a temporary EC2 instance (Server2).

---
![](/img/archi-1.png)

##  Problem

* The private key for Server1 was lost.
* EC2 instances store the public key in the instance filesystem (`~/.ssh/authorized_keys`), not in the cloud.
* Without the private key, SSH access to Server1 is not possible.

---

##  Solution Overview

1. Create a new key pair (`key-2.pem`).
2. Launch Server2 using the new key.
3. Detach Server1's root EBS volume.
4. Attach Server1's volume to Server2.
5. Mount the volume with `nouuid` to avoid UUID conflicts.
6. Copy the new public key into Server1's volume.
7. Reattach the volume back to Server1.
8. SSH into Server1 with the new key.

---

##  Steps in Detail

### 1. Create and Launch

* Create a new key pair: `key-2.pem`
* Launch a new EC2 instance (Server2) with this key.


### 2. Detach & Attach Volumes

* Detach root EBS from Server1.
* Attach it to Server2 as `/dev/xvdb`.


### 3. Mount the Volume

```bash
sudo mkdir /mnt/server1
sudo mount -o nouuid -t xfs /dev/xvdb1 /mnt/server1
```

> We used `-o nouuid` because both Server1 and Server2 volumes had the same UUID, which would normally prevent mounting.

![Mount command](/img/mounting-2.png)

### 4. Copy the Public Key

* Copy from `/home/ec2-user/.ssh/authorized_keys` of Server2.
* Paste the copied public key into:

  ```
  /mnt/server1/home/ec2-user/.ssh/authorized_keys
  ```

![Update authorized\_keys](/img/copyingkey-3.png)

### 5. Cleanup and Reattach

* Detach Server1's volume from Server2.
* Reattach it to Server1 as the root volume.

![](/img/umounting-4.png)

### 6. SSH Into Server1

```bash
ssh -i key-2.pem ec2-user@<Server1-IP>
```

![SSH success](/img/successful-login-5.png)

You should now have access.

---

##  Key Takeaways

* Before detaching EBS of server1 make sure it is stopped or else EBS may get corrupted 
* Public key is stored at `~/.ssh/authorized_keys` on the instance