# GCP SSH Access Configuration & Permissions 


# 1. Generate SSH Keys

-> Open Google SDK terminal/ Gcloud Terminal

-> Run the following command to generate an SSH key pair:

```
ssh-keygen -t rsa -b 2048 -C "your_email@example.com"
```

where: 
-t rsa: Specifies the key type (RSA).
-b 2048: Key size.
-C: Comment (usually your email).

-> press - enter – enter – enter – enter to save the keys in the default location (~/.ssh/id_rsa).

# 2. Setting File Permissions

-> Proper file permissions ensure security when using SSH keys.

1. For Private Key (id_rsa):

Set the permission to 600 (read and write for the owner only):

```
chmod 600 ~/.ssh/id_rsa
```
2. For Public Key (id_rsa.pub):

Set the permission to 644 (readable by everyone, but writable only by the owner):

```
chmod 644 ~/.ssh/id_rsa.pub
```
3. For SSH Directory (~/.ssh):

Set the permission to 700 (only the owner can access):

```
chmod 700 ~/.ssh
```
4. For Authorized Keys File (~/.ssh/authorized_keys):

Ensure it is only readable and writable by the owner:

```
chmod 600 ~/.ssh/authorized_keys
```
# 4. Adding SSH Key to GCP VM

-> To allow SSH access using the generated key, you need to add your public key to your GCP VM.

### 1. Add SSH Key to Specific VM:

-> Run the following command to add your public key to a specific VM instance:

```
gcloud compute instances add-metadata INSTANCE_NAME \
    --metadata ssh-keys="USERNAME:$(cat ~/.ssh/id_rsa.pub)"
```
- Replace INSTANCE_NAME with the name of your VM.
- Replace USERNAME with the username you'd like to use for SSH access (e.g., your GCP account or custom username)

### 2. Add SSH Key to All VMs in the Project:

-> If you want to add the key to all VMs in the project, use this command:
```
gcloud compute project-info add-metadata \
    --metadata ssh-keys="USERNAME:$(cat ~/.ssh/id_rsa.pub)"
```
# 5. Connecting to GCP VM Using SSH

-> Once the SSH key is added and file permissions are set, you can now connect to your VM.

Connect Using gcloud Command:

Run the following command to connect to your VM:
```
gcloud compute ssh USERNAME@INSTANCE_NAME --zone=ZONE
```
- Replace USERNAME with your Google account username (if using OS Login).
- Replace INSTANCE_NAME with your VM instance name.
- Replace ZONE with the GCP zone of your VM (e.g., us-central1-a).
