

## Introduction

Créer cette architecture 

<figure markdown="1">
![](infra.svg)
</figure>

### Objectifs
  - Proxmox
    - Apprendre à déployer une stack k3s sur un serveur Proxmox via k3sup
  - Argo:
    - ArgoCD:
        - Apprendre à déployer ArgoCD HA via argocd-autopilot
        - Apprendre à importer des ressources AWS avec Crossplane automatiquement
        - Apprendre à créer des projects dans ArgoCD via argocd-autopilot
    - ArgoRollouts:

### Prérequis
  - Packages:
    - k3sup
    - kubectl
    - argocd-autopilot
    - kubectx

### Ma configuration
  - 3 cluster Kubernetes:
    - Management: 3 control-planes, 3 workers
    - Production: 3 control-planes, 3 workers
    - Development: 1 control-planes, 2 workers
  - 1 host WSL

---

## Préparation de l'environnement 

??? Example "Création des 3 clusters Kubernetes sous Proxmox via k3sup"
    ??? Info "Remove all PCT"
        ```shell 
        for id in 210 211 212 213 214 215 216 220 221 222 223 224 225 226 231 232 233; do
            pct stop $id
            pct destroy $id
        done
        ```

    ??? Info "Create all PCT"
        ```shell
        pct create 211 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname mgmt-cp1 --storage local-lvm --rootfs local-lvm:8 --memory 2048 --core 4 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.211/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 212 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname mgmt-cp2 --storage local-lvm --rootfs local-lvm:8 --memory 2048 --core 4 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.212/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 213 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname mgmt-cp3 --storage local-lvm --rootfs local-lvm:8 --memory 2048 --core 4 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.213/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 214 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname mgmt-w1  --storage local-lvm --rootfs local-lvm:8 --memory 1024 --core 2 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.214/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 215 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname mgmt-w2  --storage local-lvm --rootfs local-lvm:8 --memory 1024 --core 2 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.215/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 216 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname mgmt-w3  --storage local-lvm --rootfs local-lvm:8 --memory 1024 --core 2 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.216/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 221 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname prod-cp1 --storage local-lvm --rootfs local-lvm:8 --memory 2048 --core 4 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.221/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 222 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname prod-cp2 --storage local-lvm --rootfs local-lvm:8 --memory 2048 --core 4 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.222/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 223 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname prod-cp3 --storage local-lvm --rootfs local-lvm:8 --memory 2048 --core 4 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.223/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 224 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname prod-w1  --storage local-lvm --rootfs local-lvm:8 --memory 1024 --core 2 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.224/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 225 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname prod-w2  --storage local-lvm --rootfs local-lvm:8 --memory 1024 --core 2 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.225/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 226 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname prod-w3  --storage local-lvm --rootfs local-lvm:8 --memory 1024 --core 2 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.226/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 231 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname dev-cp1  --storage local-lvm --rootfs local-lvm:8 --memory 2048 --core 4 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.231/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 232 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname dev-w1   --storage local-lvm --rootfs local-lvm:8 --memory 1024 --core 1 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.232/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        pct create 233 local:vztmpl/ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst --hostname dev-w2   --storage local-lvm --rootfs local-lvm:8 --memory 1024 --core 1 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.233/24,gw=192.168.1.254 --unprivileged 0 --features nesting=1,keyctl=1,fuse=1 --ostype ubuntu --password="mathod"
        ```

    ??? Info "Add special arguments to PCT"
        ```shell
        for CONF in /etc/pve/lxc/*.conf; do
            CTID=$(basename "$CONF" .conf)
            echo "Modifying container $CTID ..."

            # Backup
            cp "$CONF" "$CONF.bak"

            # Apply changes
            grep -q '^lxc.apparmor.profile' "$CONF" && sed -i 's/^lxc.apparmor.profile.*/lxc.apparmor.profile: unconfined/' "$CONF" || echo "lxc.apparmor.profile: unconfined" >> "$CONF"
            grep -q '^lxc.cgroup2.devices.allow' "$CONF" && sed -i 's/^lxc.cgroup2.devices.allow.*/lxc.cgroup2.devices.allow: a/' "$CONF" || echo "lxc.cgroup2.devices.allow: a" >> "$CONF"
            grep -q '^lxc.cap.drop' "$CONF" && sed -i 's/^lxc.cap.drop.*/lxc.cap.drop:/' "$CONF" || echo "lxc.cap.drop:" >> "$CONF"
            grep -q '^lxc.mount.auto' "$CONF" && sed -i 's/^lxc.mount.auto.*/lxc.mount.auto: cgroup:rw proc:rw sys:rw/' "$CONF" || echo "lxc.mount.auto: cgroup:rw proc:rw sys:rw" >> "$CONF"

            echo "Done with container $CTID"
        done
        ```

    ??? Info "Installation de packages sur les containers LXC"
        ```shell
        for vmid in $(pct list | awk 'NR>1 {print $1}'); do
          echo "Mise à jour du conteneur $vmid..."
          pct exec $vmid -- bash -c "apt install -y curl openssh-server"
        done
        ```

    ??? Info "Déploiement de votre clé SSH sur les containers LXC"
        ```shell
        # Your SSH public key
        PUB_KEY="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOK3xRTG8HI6PFMM8eEqKd2bsqTXXP2wApKKdkrJBdYQ k3s"

        # Loop through all Proxmox LXC container IDs
        for CTID in $(pct list | awk 'NR>1 {print $1}'); do
            echo "Updating container $CTID"

            # Ensure ~/.ssh exists for root
            pct exec $CTID -- mkdir -p /root/.ssh
            pct exec $CTID -- chmod 700 /root/.ssh

            # Ensure authorized_keys exists
            pct exec $CTID -- touch /root/.ssh/authorized_keys
            pct exec $CTID -- chmod 600 /root/.ssh/authorized_keys

            # Append the key if not already present
            pct exec $CTID -- sh -c "grep -qxF '$PUB_KEY' /root/.ssh/authorized_keys || echo '$PUB_KEY' >> /root/.ssh/authorized_keys"

        done

        echo "All Proxmox LXC containers updated!"
        ```

    ??? Info "Commande magique pour que ça marche"
        ```shell
        for CTID in $(pct list | awk 'NR>1 {print $1}'); do
          echo "Applying K3s LXC fix to container $CTID..."
          pct exec $CTID -- bash -c 'echo "#!/bin/sh -e
        ln -s /dev/console /dev/kmsg
        mount --make-rshared /" > /etc/rc.local && chmod +x /etc/rc.local'
          pct reboot $CTID
        done
        ```

    ??? Info "Use kubeConfig"
        ```shell
        export KUBECONFIG=/root/kubeconfig
        kubectl config use-context default
        kubectl get node -o wide
        ```

    !!! Info "Déploiement k3s"
        ```shell
        # Management
        k3sup install --ip 192.168.1.211 --user root --cluster --ssh-key ~/.ssh/k3s --cluster --k3s-channel stable --k3s-extra-args "--disable servicelb --disable traefik" --context management
        k3sup join --ip 192.168.1.212 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.211 --server-user root --server --k3s-channel stable --k3s-extra-args '--disable traefik --disable servicelb'
        k3sup join --ip 192.168.1.213 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.211 --server-user root --server --k3s-channel stable --k3s-extra-args '--disable traefik --disable servicelb'
        k3sup join --ip 192.168.1.214 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.211 --server-user root --k3s-channel stable
        k3sup join --ip 192.168.1.215 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.211 --server-user root --k3s-channel stable
        k3sup join --ip 192.168.1.216 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.211 --server-user root --k3s-channel stable
        ```

        ```shell
        # Production
        k3sup install --ip 192.168.1.221 --user root --cluster --ssh-key ~/.ssh/k3s --cluster --k3s-channel stable --k3s-extra-args "--disable servicelb --disable traefik" --context production
        k3sup join --ip 192.168.1.222 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.221 --server-user root --server --k3s-channel stable --k3s-extra-args '--disable traefik --disable servicelb'
        k3sup join --ip 192.168.1.223 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.221 --server-user root --server --k3s-channel stable --k3s-extra-args '--disable traefik --disable servicelb'
        k3sup join --ip 192.168.1.224 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.221 --server-user root --k3s-channel stable
        k3sup join --ip 192.168.1.225 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.221 --server-user root --k3s-channel stable
        k3sup join --ip 192.168.1.226 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.221 --server-user root --k3s-channel stable
        ```

        ```shell
        # Development
        k3sup install --ip 192.168.1.231 --user root --cluster --ssh-key ~/.ssh/k3s --cluster --k3s-channel stable --k3s-extra-args "--disable servicelb --disable traefik" --context development
        k3sup join --ip 192.168.1.232 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.231 --server-user root --k3s-channel stable
        k3sup join --ip 192.168.1.233 --user root --ssh-key ~/.ssh/k3s --server-ip 192.168.1.231 --server-user root --k3s-channel stable
        ```

---

## Conclusion

### En rapport avec cet article

### Liens utile
- Argocd-autopilot:
    - [https://argocd-autopilot.readthedocs.io/en/stable/](https://argocd-autopilot.readthedocs.io/en/stable/)
    - [https://github.com/argoproj-labs/argocd-autopilot](https://github.com/argoproj-labs/argocd-autopilot)