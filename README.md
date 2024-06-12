# Dynamic-VFIO-Passthrough
Basic guide on VFIO passthrough for GPUs


# WARNING
This guide is provided with out any liablity and you use it at your own risk.

I have tested and use these steps on my own system with the specs listed below

### Test Specs
* OS: Debain 12
* Graphic Cards:
  - Driver:
    - Nvidia driver from Nvidia manually installed (https://www.nvidia.com/download/index.aspx)
  - Nvidia GeForce GTX 1060
  - Nvida GeFroce GTX 960
* CPU: Intel i7-8700K
* Motherboard: Asus ROG STRIX Z390-F Gaming

# IOMMU 
Please ensure that iommu is enabled on your system. Steps will be provided soon

# X Server
Set XServer to only use one GPU when it starts up. Otherwise it will cause issues when attempting to bind the card to the vfio-pci 
1. Open /etc/X11/xorg.conf with you favorite editor
   - Set not to automaticall add GPUs
     ```
      Section "ServerFlags"
        Option          "AutoAddGpu" "false"
      EndSection
     ```
   - Create a device section for your main GPU
     ```
     Section "Device"
       Identifier     "Device0"
       Driver         "nvidia"
       VendorName     "NVIDIA Corporation"
       BusID          "PCI:YOUR_PCI_ID"
       Option         "ProbeAllGpus" "false"
     EndSection
     ```
2. Restart your system
3. Run `sudo nvidia-smi` to ensure that xorg is not on the second GPU
   - You should only see one occurance of /usr/lib/xorg/Xorg which should be on the main GPU
   - Correct output
     ```
      +-----------------------------------------------------------------------------------------+
      | NVIDIA-SMI 550.90.07              Driver Version: 550.90.07      CUDA Version: 12.4     |
      |-----------------------------------------+------------------------+----------------------+
      | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
      | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
      |                                         |                        |               MIG M. |
      |=========================================+========================+======================|
      |   0  NVIDIA GeForce GTX 1060 3GB    Off |   00000000:01:00.0  On |                  N/A |
      | 18%   54C    P0             32W /  120W |     490MiB /   3072MiB |      0%      Default |
      |                                         |                        |                  N/A |
      +-----------------------------------------+------------------------+----------------------+
      |   1  NVIDIA GeForce GTX 960         Off |   00000000:02:00.0 Off |                  N/A |
      | 29%   30C    P0             30W /  160W |       0MiB /   4096MiB |      1%      Default |
      |                                         |                        |                  N/A |
      +-----------------------------------------+------------------------+----------------------+
                                                                                               
      +-----------------------------------------------------------------------------------------+
      | Processes:                                                                              |
      |  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
      |        ID   ID                                                               Usage      |
      |=========================================================================================|
      |    0   N/A  N/A      2147      G   /usr/lib/xorg/Xorg                            153MiB |
      |    0   N/A  N/A      2397      G   /usr/bin/gnome-shell                           60MiB |
      +-----------------------------------------------------------------------------------------+
     ```

# QEMU Hooks
Please ensure that you have installed the [hook helper tool](https://passthroughpo.st/simple-per-vm-libvirt-hooks-with-the-vfio-tools-hook-helper/) from Passthrough POST and that it is executable
```
sudo mkdir -p /etc/libvirt/hooks/qemu.d
sudo wget 'https://raw.githubusercontent.com/PassthroughPOST/VFIO-Tools/master/libvirt_hooks/qemu' -O /etc/libvirt/hooks/qemu
sudo chmod +x /etc/libvirt/hooks/qemu
```
