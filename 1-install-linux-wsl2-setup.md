# WSL2 - The Windows hack to run Linux workloads

Using WSL2 is the easiest and lighter way to run Linux within Windows OS. It’s pretty straight forward to use and all images you can download from Windows Store.

# Disable `swap` - we will need that later

Kubernetes does not like swap files in Linux. We will need to disable it otherwise `kubeadm` will fail to setup a cluster. Even if you disable using `swapoff -a` , if you restart your machine, swap will come back to haunt you.

Reference: https://learn.microsoft.com/en-us/windows/wsl/wsl-config#wslconfig

- Open `%UserProfile%` on your explorer
- Create `.wslconfig` file
- Add the following contents on it

```powershell
# Settings apply across all Linux distros running on WSL 2
[wsl2]

# How much swap space to add to the WSL 2 VM, 0 for no swap file. Swap storage is disk-based RAM used when memory demand exceeds limit on hardware device.
swap=0
```

- If you have running instance already, you need to reboot WSL2 distribution

```powershell
# List all distros that are running
wsl.exe -l -v

# Terminate a specific distribution
wsl.exe -t Ubuntu-24.04

# Boot up a specific distribution
wsl.exe -d Ubuntu-24.04
```

## Create WSL instance

- Download from Windows Store - Ubuntu 24.04

![Windows Store](/img/install-linux-wsl2-windows-store.png)

- Setup user and password
- Open Terminal and connect to the distribution

```powershell
# Launch the instance via wsl command
# Replace the -u USER with the user you had setup on the installation steps of the distribution
wsl -d Ubuntu-24.04 -u tiago
```