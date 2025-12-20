## Issue

> [!WARNING] Issue
> Error when trying to do shell on VM that was deployed on an incus instance on Fedora 42/43

```
incus shell <vm>
.
.
.
Error: websocket: bad handshake>
```

> [!NOTE]
> Solution - incus-agent needs to be static build and no dynamic.

[git issue](https://github.com/lxc/incus/issues/2513)

## Fix

**Clone source**
```
mkdir -p ~/src/github.com/lxc/
cd ~/src/github.com/lxc/
git clone https://github.com/lxc/incus.git
cd incus/
```

**Build:**
```
CGO_LDFLAGS_ALLOW="-Wl,-z,now" CGO_ENABLED=0 go build -trimpath -tags=agent,netgo ./cmd/incus-agent/
```

**Backup current agent:**
```
cp /bin/incus-agent /tmp/incus-agent-bk
```

**Copy new static build agent:**
```
sudo mv incus-agent /bin/incus-agent
```

**Restart vm and service:**
```
incus restart <vm>
sudo systemctl restart incus.service
sudo systemctl restart incus.socket
```

**Test:**
```
incus exec <vm> bash
-or-
incus shell <vm>
```
