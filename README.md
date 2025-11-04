# Migrate-Laptop-Devops-Style

A step-by-step guide for DevOps enthusiasts and Linux power users to **migrate personal files, SSH keys, dotfiles, and credentials from an old laptop to a new Ubuntu laptop over Wi-Fi**. Based on real-world experience, designed for safety, security, and clarity for beginners.

---

## 💡 When Should You Use This?
- Moving to a new laptop/workstation
- Upgrading your main DevOps machine
- Migrating secure credentials (SSH, cloud, dotfiles)
- Ensuring minimal downtime and data loss

---

## 📋 Prerequisites

### On both laptops:
- Ubuntu (or other Linux) installed
- Both laptops **connected to the same Wi-Fi network**
- SSH enabled on both laptops
- Usernames and passwords available for both devices

---

## 1️⃣ Discover Hostnames and IP Addresses

**On your old laptop (_source_):**
```bash
whoami           # Get your username (e.g. bs00794)
hostname         # Get your hostname (e.g. BS00794)
ip addr show     # Find your IP address (look for wlp... or wlan... entry, e.g. 192.168.0.105)
```

**On your new laptop (_destination_):**
```bash
whoami           # Get your username (e.g. bs01699)
hostname         # Get your hostname (e.g. bs-support)
ip addr show     # Find your IP address (e.g. 192.168.0.107)
```

---

## 2️⃣ Confirm SSH is Running

**On both laptops (run separately):**
```bash
sudo systemctl status ssh
# If not active, run:
sudo apt update && sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh
```

**Check firewall status:**
```bash
sudo ufw status
# If active and SSH blocked:
sudo ufw allow ssh
```

---

## 3️⃣ Test SSH Connectivity

**On your new laptop, connect to your old laptop:**
```bash
ssh <olduser>@<oldip> 'echo "SSH works: $(whoami)@$(hostname)"'
# Example:
ssh bs00794@192.168.0.105 'echo "SSH works: $(whoami)@$(hostname)"'
```

- If prompted about authenticity, type `yes`.
- Enter your old laptop's password.

---

## 4️⃣ Create a Review Folder for Transfer

**On your new laptop:**
```bash
mkdir -p ~/from-old
```

---

## 5️⃣ Run a Dry-Run of Full Home Transfer (SAFE)**

**On your new laptop:**
```bash
rsync -avzh --progress --exclude='Downloads/' --exclude='.cache/' --dry-run <olduser>@<oldip>:/home/<olduser>/ ~/from-old/
# Example:
rsync -avzh --progress --exclude='Downloads/' --exclude='.cache/' --dry-run bs00794@192.168.0.105:/home/bs00794/ ~/from-old/
```
- Review output. No files are overwritten or changed in this step!

---

## 6️⃣ Run the Real Transfer

**On your new laptop:**
```bash
rsync -avzh --progress --exclude='Downloads/' --exclude='.cache/' <olduser>@<oldip>:/home/<olduser>/ ~/from-old/
```
- Enter password if prompted.
- Watch the progress.

---

## 7️⃣ Review and Verify Transferred Data

**On your new laptop:**
```bash
ls -la ~/from-old/
ls -la ~/from-old/.ssh
ls -la ~/from-old/.aws
ls -la ~/from-old/.config/
ls -la ~/from-old/.kube
cat ~/from-old/.gitconfig
```

- Check contents, sizes, and permissions before merging with your real home directory.

---

## 8️⃣ Fix Permissions for Credentials

**On your new laptop:**
```bash
sudo chown -R $(whoami):$(whoami) ~/from-old/
chmod 700 ~/from-old/.ssh ~/from-old/.gnupg 2>/dev/null
chmod 600 ~/from-old/.ssh/* ~/from-old/.gnupg/* 2>/dev/null
chmod -R go-rwx ~/from-old/.aws ~/from-old/.kube ~/from-old/.azure ~/from-old/.config/gcloud 2>/dev/null
```

---

## 9️⃣ Selective Copy: Only Move Critical Files

**Tip:**  
Don’t blindly overwrite everything in your new home (`~`).  
Move only the dotfiles, credential folders, and config directories you need—one at a time.

**Example:**
```bash
cp -r ~/from-old/.ssh ~/.ssh
cp -r ~/from-old/.aws ~/.aws
cp -r ~/from-old/.config/gcloud ~/.config/gcloud
cp ~/from-old/.gitconfig ~/.gitconfig
# Repeat for other essentials as needed
```

---

## 🔟 Reinstall DevOps Tools and Packages

1. **Export installed package list on old laptop:**
    ```bash
    dpkg --get-selections > ~/package-list.txt
    # Copy to new laptop:
    scp ~/package-list.txt <newuser>@<newip>:~/from-old/
    ```

2. **On your new laptop:**
    ```bash
    sudo apt update
    sudo dpkg --set-selections < ~/from-old/package-list.txt
    sudo apt-get dselect-upgrade
    ```

3. **Individually reinstall key tools with:**
    ```bash
    sudo apt install docker.io terraform ansible kubectl awscli gcloud ... # as needed
    ```

---

## 1️⃣1️⃣ (Optional) Clean Up Large Cache or Unneeded Data

After migration and verification, clean unnecessary files (e.g. browser cache, temp):
```bash
rm -rf ~/from-old/.config/google-chrome/Default/File\ System/
rm -rf ~/from-old/.cache
```

---

## 1️⃣2️⃣ Common Gotchas + Safety Tips

- Never overwrite your new home blindly—always use a review folder first!
- When moving browser profiles or desktop configs, some settings may not port perfectly.
- Always run transfers over trusted Wi-Fi or a secure cable.
- SSH and cloud credentials are sensitive—**set permissions tightly!**
- Reinstall apps from package manager for reliability. Don’t copy binaries.

---

## 1️⃣3️⃣ Troubleshooting

- Slow transfer? Try moving closer to the router, or use Ethernet.
- Permissions errors? Always check `chmod` and `chown` after transfer.
- App issue after copying config? Try resetting from template or reinstalling.

---

## 📚 References

- [Ubuntu Documentation: Home Directory](https://help.ubuntu.com/community/UbuntuUsersFolderStructure)
- [rsync Manual](https://linux.die.net/man/1/rsync)
- [SSH Overview](https://www.ssh.com/academy/ssh)

