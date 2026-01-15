## Debian Installer: internal storage not detected (MacBook Air 2013) — fix via GRUB boot param

**Problem:** During Debian install, the installer did not detect the internal SSD/storage.

**Fix:** Boot the installer with Intel IOMMU disabled.

### Steps

1. Boot from the Debian USB installer.
    
2. At the GRUB menu (Debian installer boot screen), highlight your install option (e.g. **Install** or **Graphical install**).
    
3. Press **`e`** to edit the boot entry.
    
4. Find the line that starts with `linux` (it contains `vmlinuz` and other kernel parameters), for example:
    
    ```text
    linux /install.amd/vmlinuz vga=788 --- quiet
    ```
    
5. Append the parameter **`intel_iommu=off`** to the end of that same `linux` line:
    
    ```text
    linux /install.amd/vmlinuz vga=788 --- quiet intel_iommu=off
    ```
    
    (Important: it must be on the **linux line**, not on a separate line.)
    
6. Boot with the edited parameters using **Ctrl+X** (or **F10**, depending on the screen).
    
7. Continue the installation — the internal storage should now be detected.
    

### Make it permanent after install (optional)

After Debian is installed and booting, persist it in GRUB:

1. Edit:
    
    ```bash
    sudo nano /etc/default/grub
    ```
    
2. Add it to the kernel command line, e.g.:
    
    ```text
    GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=off"
    ```
    
3. Apply:
    
    ```bash
    sudo update-grub
    ```
    
