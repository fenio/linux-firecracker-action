# linux-firecracker-action

GitHub Action that runs commands inside a Linux [Firecracker](https://github.com/firecracker-microvm/firecracker) microVM.

## Usage

```yaml
- uses: fenio/linux-firecracker-action@main
  with:
    cmd: |
      echo "Hello from Firecracker VM"
      uname -a
```

### Run k3s with a custom kernel

```yaml
- uses: fenio/linux-firecracker-action@main
  with:
    kernel-url: https://github.com/fenio/linux-firecracker/releases/latest/download/vmlinux-tns-csi
    vcpus: 4
    memory: 4096
    disk-size: 5G
    cmd: |
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -
      kubectl get nodes
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `cmd` | *required* | Commands to run inside the VM |
| `kernel-url` | latest `vmlinux-base` | URL to vmlinux kernel binary |
| `rootfs-url` | latest `rootfs.ext4.zst` | URL to zstd-compressed rootfs |
| `ssh-private-key-url` | latest `id_rsa` | URL to SSH private key |
| `firecracker-version` | `v1.15.0` | Firecracker release version |
| `vcpus` | `2` | Number of vCPUs |
| `memory` | `4096` | Memory in MiB |
| `disk-size` | `5G` | Rootfs disk size |
| `verbose` | `false` | Enable debug output |

## How it works

1. Downloads Firecracker, kernel, and rootfs from [linux-firecracker](https://github.com/fenio/linux-firecracker/releases) releases
2. Sets up TAP networking with NAT (VM at `172.16.0.2`, host at `172.16.0.1`)
3. Starts the microVM with PCI VirtIO transport
4. Waits for SSH, copies the script into the VM, and runs it
5. Cleans up the VM on completion

## Kernel profiles

The default kernel (`vmlinux-base`) supports networking. For workloads that need more (k3s, storage protocols), use the `kernel-url` input to select a different profile from [linux-firecracker releases](https://github.com/fenio/linux-firecracker/releases):

- `vmlinux-minimal` — boot + shell only
- `vmlinux-base` — networking (default)
- `vmlinux-tns-csi` — k3s + NVMe-oF, iSCSI, NFS, SMB
