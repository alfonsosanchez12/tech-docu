## Install and configure `tuned` on Debian (MacBook Air 2013, 4GB RAM)

### What is tuned?

`tuned` is a system tuning daemon that applies a **profile** (a set of kernel/sysctl/power/IO parameters) to balance **performance vs power usage vs latency**, depending on your goal.

---

## 1) Install packages

```bash
sudo apt update
sudo apt install -y tuned tuned-utils
```

> On Debian, the main CLI is **`tuned-adm`** (note: it’s **not a service**).

---

## 2) `tuned-adm` command availability (PATH note)

On minimal Debian, `tuned-adm` may live in `/usr/sbin`, which is sometimes **not** in a regular user’s PATH.

### Option A (simple): use it with `sudo`

This often works without changing PATH because `sudo` typically includes `/usr/sbin` in its secure path:

```bash
sudo tuned-adm active
sudo tuned-adm list
```

### Option B: add `/usr/sbin` to your user PATH

Add to `~/.bashrc`:

```bash
echo 'export PATH="$PATH:/usr/sbin:/sbin"' >> ~/.bashrc
source ~/.bashrc
```

Check where it is:

```bash
command -v tuned-adm || ls -l /usr/sbin/tuned-adm
```

---

## 3) Enable and start the tuned service

```bash
sudo systemctl enable --now tuned
```

Verify:

```bash
sudo systemctl status tuned --no-pager
```

---

## 4) Pick a profile (recommendation for your MacBook Air 2013 + 4GB)

### Recommended: `balanced`

For a light server on older laptop hardware, **`balanced`** is usually the best default:

- avoids aggressive “max performance” behavior (less heat / less fan / less power)
    
- still keeps the system responsive enough for DNS + small services + container workloads
    
- good “set and forget” choice
    

### Alternatives

- `powersave`: if your priority is **lowest power draw** and you don’t care about responsiveness spikes
    
- Avoid `throughput-performance` / `virtual-host` for your case (Pi-hole in an Incus container is **not** heavy virtualization)
    

---

## 5) Apply the profile

List profiles:

```bash
sudo tuned-adm list
```

Apply `balanced`:

```bash
sudo tuned-adm profile balanced
```

> This applies immediately and should persist across reboots while `tuned` is enabled.

---

## 6) Check what’s active

```bash
sudo tuned-adm active
```

Also useful:

```bash
sudo tuned-adm recommend
```

And confirm the daemon is running:

```bash
systemctl is-active tuned
```

---

## What the `balanced` profile does (high-level)

`balanced` aims to be a **middle ground**:

- it won’t force “always max performance”
    
- it typically applies reasonable defaults for CPU/power and a few kernel tunables
    
- the **exact** knobs can vary by distro/version and hardware
    

To inspect what it’s actually changing on your system:

```bash
sudo tuned-adm profile_info balanced
```

And you can view profile files here (read-only system profiles):

```bash
ls /usr/lib/tuned/
```

---

## Basic tuned usage cheat sheet

```bash
# Show active profile
sudo tuned-adm active

# List available profiles
sudo tuned-adm list

# See details of a profile
sudo tuned-adm profile_info balanced

# Switch profiles
sudo tuned-adm profile powersave
sudo tuned-adm profile balanced

# Let tuned suggest a profile
sudo tuned-adm recommend

# Service control
sudo systemctl status tuned --no-pager
sudo systemctl restart tuned
sudo systemctl disable --now tuned
```
