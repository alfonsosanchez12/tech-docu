# Managing Local and Remote Podman instances over LazyDocker

<img width="1708" height="1080" alt="image" src="https://github.com/user-attachments/assets/bb66b70b-ea43-44ba-8046-be2f23f80c87" />

Image of Lazydocker running over a Podman socket. Podman logo by [podman](https://raw.githubusercontent.com/containers/common/main/logos/podman-logo-full-vert.png)

If you're like me, you love the power of Podman for rootless containers and the efficiency of Lazydocker's TUI. But getting them to play nicely, especially across a local machine and remote servers could be difficult, especially when handling SSH commands and socket forwarding for configuring Lazydocker to manage multiple Podman instances.

This guide will help you with that process by introducing `ezpodman`, a wrapper script designed to automate and simplify this entire workflow.

The Official guthub repo for this could is [alfonsosanchez12/ezpodman](https://github.com/alfonsosanchez12/ezpodman)

> If you are also interested in knowing how to manage this process manually, you can read it here -> [alfonsosanchez12/podman_on_lazydocker_The-Hard-Way.md](https://github.com/alfonsosanchez12/tech-docu/blob/main/podman_on_lazydocker_The-Hard-Way.md.md)

## Setting the Stage: Essential Prerequisites

Before we get our hands dirty with configs, let's do a quick tool check. The table below outlines the required tooling.

### Tooling requirements

**Required Tooling:**

| Tool             | Purpose                      |
|------------------|------------------------------|
| `docker`         | Optional: for other tooling  |
| `podman`         | Local container engine       |
| `podman-remote`  | Connect to remote Podman (Included with Podman 4+)    |
| `lazydocker`     | Docker/Podman TUI            |
| `jq`             | Parses `podman-remote` JSON  (needed for ezpodman) |

**Optional (but strongly recommended):**

| Tool      | Purpose                     |
|-----------|-----------------------------|
| `fzf`     | Fuzzy remote picker         |

With these tools in place, it's time to tackle the manual connection process. Trust me, learning this once will save you hours of debugging later.

### Environment requirements

#### Remote podman instance management

If your intension is to manage a **remote** Podman instance with Lazydocker, make sure you have (aside a `podman` instance) enabled lingering and a `podman-socket`  

```shell
loginctl enable-linger $USER
```

```shell
systemctl --user enable --now podman.socket
```

To verify that the socket is active and listening, you can run a curl command directly against the Unix socket:

```shell
curl --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock http://d/v3.0.0/libpod/_ping
```

> A successful response (typically **OK**) confirms the remote side is ready.

#### local Podman instance management

If your intension is to manage a **local** Podman instance with Lazydocker, verify you have:

**For a local linux-machine** a `podman-socket` enabled

```shell
$ systemctl --user list-sockets podman.socket
LISTEN                            UNIT          ACTIVATES
/run/user/1000/podman/podman.sock podman.socket podman.service
```

**For a local linux-machine** a `podman-machine` initiated

```shell
$ podman system connection list
Name                         URI                                                           Identity                                                     Default     ReadWrite
podman-machine-default       ssh://core@127.0.0.1:50637/run/user/501/podman/podman.sock    /Users/<USER>/.local/share/containers/podman/machine/machine  true        true
podman-machine-default-root  ssh://root@127.0.0.1:50637/run/podman/podman.sock             /Users/<USER>/.local/share/containers/podman/machine/machine  false       true
```

### 2. Use ezpodman for Simplified Management

`ezpodman` is a **Bash wrapper** specifically designed to eliminate the difficulty of managing `lazydocker` connections for both local and remote Podman instances. It automates the work of tunnel creation and environment configuration.

**`ezpodman` feature set:**

- Interactive menu for easily selecting and managing your local and remote connections.
- Built-in setup wizard to automates the remote setup process, including SSH key generation, copying keys to the remote host, and podman-remote configuration.
- Automatic and persistent SSH tunnel management.

### 3. Getting `ezpodman` Up and Running

Installation is a straightforward process. Because the tool is a self-contained script, setup is quick, easy, and requires no complex dependencies or daemons.

#### Installation

**Recommended Installation** This installs the script to your user's local binary directory and doesn't require sudo privileges.

```shell
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/alfonsosanchez12/ezpodman/main/ezpodman -o ~/.local/bin/ezpodman
chmod +x ~/.local/bin/ezpodman
```

You can now test it with a local podman instance by executing it

```shell
ezpodman
```

For the complete set of options you can run `ezpodman -h`

>For in-depth information you can check the github repo notes

#### The Automated Setup Wizard

The `ezpodman` setup wizard helps you handling the podman socket management steps for you, from key generation to final connection configuration. This wizard is mainly useful for handling remote connections.

To launch the wizard, simply run:

```shell
ezpodman --setup
```

**The script will guide you through the following steps:**

- Naming the connection for easy future reference (e.g., prod-server).
- Specifying the remote destination in the format user@host.
- Generating a new ed25519 SSH key if you choose to, saving it for the connection.
- Copying the new public key to the remote host using the standard ssh-copy-id utility.
- Adding the fully configured connection to podman-remote for immediate use.

### 4. A Practical Guide to Using ezpodman

#### Managing the Local Podman Instances

The simplest use case is managing your local podman containers. Running ezpodman with no arguments defaults to local mode.
On Linux, it connects directly to the local rootless Podman socket.
On macOS, it automatically detects and creates a tunnel to the default `podman-machine`, treating it as the local instance.

```shell
ezpodman
```

#### Connecting to Remote Instances

1. Use the interactive picker to select from your list of configured remotes. Having `fzf` installed makes this option more enjoyable.

```shell
ezpodman -r
```

2. Connect directly to a specific, named remote instance:
If you know the name on the podman connection, you can specify directly the name as follows

```shell
ezpodman -r <connection-name>
```

#### Advanced Tunnel Control

`ezpodman` can also manage SSH tunnels independently. This makes it useful for any other Docker-compatible tools you might use, such as custom scripts.

```shell
ezpodman -m
```

- Open a tunnel only and print the DOCKER_HOST variable for use in your current shell
- List all active tunnels managed by `ezpodman`
- Stop a specific tunnel by its connection name
- Restart a specific tunnel
- Kill all `ezpodman` tunnels at once
- Remove a connection configuration entirely

> I hope you found useful this information, as well as the ezpodman tool for your day-to-day podman container management.
