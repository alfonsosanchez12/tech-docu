## Enable USB-tether interface (minimal Debian, no NetworkManager)

**Goal:** Bring up the Android USB tether network interface and get an IP via DHCP on a minimal Debian install.

### Steps

1. **Identify the interface name** created by USB tethering:
    
    ```bash
    ip -br link
    ```
    
    Example interface detected:
    
    ```bash
    INTF=<name_of_USB-Tether_interface>
    ```
    
2. **Check current IP status** (usually empty before DHCP):
    
    ```bash
    ip -br addr show $INTF
    ```
    
3. **Configure the interface for DHCP via ifupdown**  
    Append a stanza to `/etc/network/interfaces`:
    
    ```bash
    printf "\nallow-hotplug $INTF\niface $INTF inet dhcp\n" >> /etc/network/interfaces
    ```
    
4. **Apply the configuration**
    
    ```bash
    ifdown $INTF 2>/dev/null || true
    ifup $INTF
    ```
    
5. **Verify**
    
    ```bash
    ip -br addr show $INTF
    ```
    
    You should now see an IP address assigned to `$INTF`.