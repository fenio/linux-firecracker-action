# setup-firecracker

GitHub Action that runs commands inside a Linux [Firecracker](https://github.com/firecracker-microvm/firecracker) microVM.

## Usage

```yaml
- uses: fenio/setup-firecracker@main
  with:
    cmd: |
      echo "Hello from Firecracker VM"
      uname -a
```

### Run k3s with the [tns-csi](https://github.com/fenio/tns-csi) profile

```yaml
- uses: fenio/setup-firecracker@main
  with:
    kernel-url: https://github.com/fenio/linux-firecracker/releases/latest/download/vmlinux-tns-csi
    rootfs-url: https://github.com/fenio/linux-firecracker/releases/latest/download/rootfs-tns-csi.ext4.zst
    vcpus: 4
    memory: 4096
    disk-size: 5G
    cmd: |
      k3s server --disable=traefik &
      sleep 30
      kubectl get nodes
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `cmd` | *required* | Commands to run inside the VM |
| `kernel-url` | latest `vmlinux-base` | URL to vmlinux kernel binary |
| `rootfs-url` | latest `rootfs-base.ext4.zst` | URL to zstd-compressed rootfs |
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

## Profiles

Defaults use the `base` kernel and rootfs. For other workloads, override `kernel-url` and `rootfs-url` with a different profile from [linux-firecracker releases](https://github.com/fenio/linux-firecracker/releases):

**Kernel:** `vmlinux-minimal` | `vmlinux-base` (default) | [`vmlinux-tns-csi`](https://github.com/fenio/tns-csi)

**Rootfs:** `rootfs-base.ext4.zst` (default) | [`rootfs-tns-csi.ext4.zst`](https://github.com/fenio/tns-csi) (k3s + nvme-cli, open-iscsi, nfs-common, cifs-utils)
