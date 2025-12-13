# Managing Local and Remote Podman instances over LazyDocker

Image of Lazydocker running over a Podman socket. Podman logo by [podman](https://raw.githubusercontent.com/containers/common/main/logos/podman-logo-full-vert.png)
If you're like me, you love the power of Podman for rootless containers and the efficiency of Lazydocker's TUI. But getting them to play nicely, especially across a local machine and remote servers, can feel like a chore of endless SSH commands and socket forwarding. The core problem many of us face is the complexity of configuring Lazydocker to manage multiple Podman instances, a process that often involves manually establishing SSH tunnels and configuring connections.

This guide will help you with that process. First we will walk through the manual configuration steps to provide a solid understanding of the underlying mechanics. Then, I will introduce `ezpodman`, a wrapper script designed to automate and simplify this entire workflow, turning a complex chore into a single, effortless command.

## Setting the Stage: Essential Prerequisites
Before we get our hands dirty with configs, let's do a quick tool check. Nothing halts momentum faster than a command not found error midway through a setup. The table below outlines the required tooling.
Required Tooling:

| Tool             | Purpose                      |
|------------------|------------------------------|
| `docker`         | Optional: for other tooling  |
| `podman`         | Local container engine       |
| `podman-remote`  | Connect to remote Podman (Included with Podman 4+)    | 
| `lazydocker`     | Docker/Podman TUI            |
| `jq`             | Parses `podman-remote` JSON  (needed for ezpodman) | 

Optional:

| Tool      | Purpose                     |
|-----------|-----------------------------|
| `fzf`     | Fuzzy remote picker         |

With these tools in place, it's time to tackle the manual connection process. Trust me, learning this once will save you hours of debugging later.

## 1. The Hard Way: Manually Connecting Lazydocker to a Remote Podman Instance
Understanding the manual setup process is strategically important. While it's more complex than the automated approach, knowing these steps provides valuable insight into what tools like `ezpodman` automate under the hood.

### Step 1: Configuring the Remote Machine
If the intended use is to manage remote Podman instances on a linux server/machine do the following steps.
These commands must be run on the remote server as the user whose Podman instance you intend to manage.

First, you must enable user lingering. This ensures that the user's services (like the Podman socket) continue to run even after they have logged out.
```shell
loginctl enable-linger $USER
```

Next, enable and start the Podman socket for your user. This allows remote clients to connect to the Podman API.
```shell
systemctl --user enable --now podman.socket
```

To verify that the socket is active and listening, you can run a curl command directly against the Unix socket:
```shell
curl --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock http://d/v3.0.0/libpod/_ping
```

A successful response (typically **OK**) confirms the remote side is ready.

### Step 2: Configuring Your Local Machine
With the remote server configured (if intended), the next step is to configure your local podman or podman-remote client to establish the local socket or a secure connection to the newly enabled remote socket.

#### For local linux-machine Podman instances:
First verify if the local podman socket is enabled
```shell
$ systemctl --user list-sockets podman.socket
LISTEN                            UNIT          ACTIVATES
/run/user/1000/podman/podman.sock podman.socket podman.service
```

If no `podman.socket` is enabled, execute the following command
```shell
systemctl --user enable --now podman.socket
```

After the local Socket is ready you can start `lazydocker` by setting first the environment variable **DOCKER_HOST**
```shell
DOCKER_HOST=unix:///run/user/1000/podman/podman.sock DOCKER_API_VERSION="1.41" lazydocker
```

You can also create an alias to make things easy
```shell
alias <MY_ALIAS>='DOCKER_HOST=unix:///run/user/1000/podman/podman.sock DOCKER_API_VERSION="1.41" lazydocker'
```

#### For local macos-machine Podman instances:
First make sure the default `podman-machine` is initiated and started
```shell
$ podman machine init
$ podman machine start
```
With the podman machine initiated, you should see available the local connections for the default podaman machine. You can check with the following command
```shell
$ podman system connection list
Name                         URI                                                           Identity                                                     Default     ReadWrite
podman-machine-default       ssh://core@127.0.0.1:50637/run/user/501/podman/podman.sock    /Users/<USER>/.local/share/containers/podman/machine/machine  true        true
podman-machine-default-root  ssh://root@127.0.0.1:50637/run/podman/podman.sock             /Users/<USER>/.local/share/containers/podman/machine/machine  false       true
```
Then you will need to find the PodmanSocket for the default podman machine and use it to launch lazydocker
```shell
$ podman machine inspect | grep -i sock

               "PodmanSocket": {
                    "Path": "/var/folders/*/******/T/podman/podman-machine-default-api.sock"

$ DOCKER_HOST="unix:///var/folders/*/******/T/podman/podman-machine-default-api.sock" DOCKER_API_VERSION="1.41" lazydocker
```

#### For remote podman instances:
First, ensure you have secure SSH access using public key authentication. If you haven't already set this up, generate a new key and copy it to the remote host.
**Generate a new ed25519 SSH key**
```shell
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
```
**Copy the public key to the remote server**
```shell
ssh-copy-id user@remote-host
```

Next, create the `podman-remote` connection. This command tells your local client how to find and connect to the remote Podman socket via SSH.
> Note: Pay close attention to the socket path. The 1000 corresponds to the user's UID on the remote machine. You can find the correct UID by running id -u on the remote host.
```shell
podman-remote system connection add my-remote \
--identity ~/.ssh/id_ed25519 \
ssh://user@remote-host/run/user/1000/podman/podman.sock
```

Finally, test the connection to confirm that your local client can successfully communicate with the remote Podman instance:
```shell
podman-remote --connection my-remote info
```

>Takeaway: While this manual processes are effective, it is a multi-step, error-prone procedure that must be repeated for every new remote host. This perfectly introduces the need for a simpler approach.

### 2. The e-z Way: Use ezpodman for Simplified Management
>Official GitHub repository:
https://github.com/alfonsosanchez12/ezpodman

`ezpodman` is the easy solution to the manual process described above. It is a **Bash wrapper** specifically designed to eliminate the friction of managing `lazydocker` connections for both local and remote rootless Podman instances. It automates the tedious work of tunnel creation and environment configuration, allowing you to focus on your containers.

**The core value of `ezpodman` comes from its feature set:**
- An interactive menu for easily selecting and managing your local and remote connections.
- A built-in setup wizard that completely automates the remote setup process, including SSH key generation, copying keys to the remote host, and podman-remote configuration.
- Automatic and persistent SSH tunnel management, ensuring a stable connection without manual intervention.
- Full Docker compatibility by automatically setting the **DOCKER_HOST** environment variable, allowing other Docker-aware tools to use the established tunnels seamlessly.

Now that you understand what ezpodman can do, let's get it installed and running on your system.

### 3. Getting `ezpodman` Up and Running
Installation is a straightforward process. Because the tool is a self-contained script, setup is quick, easy, and requires no complex dependencies or daemons.

#### Installation
You can choose between a user-local or system-wide installation.

**Option 1: Recommended for Single-User Systems** This method installs the script to your user's local binary directory and doesn't require sudo privileges.
```shell
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/alfonsosanchez12/ezpodman/main/ezpodman -o ~/.local/bin/ezpodman
chmod +x ~/.local/bin/ezpodman
```

**Option 2: System-wide Installation** This approach makes the ezpodman command available to all users on the system.
```shell
sudo curl -fsSL https://raw.githubusercontent.com/alfonsosanchez12/ezpodman/main/ezpodman -o /usr/local/bin/ezpodman
sudo chmod +x /usr/local/bin/ezpodman
```

You can now test it with a local podman instance by executing it
```shell
ezpodman
```

#### The Automated Setup Wizard
The `ezpodman` setup wizard is the automated counterpart to the manual process detailed in Section 2. It handles every step for you, from key generation to final connection configuration. This wizard is mainly useful for handling remote connections.

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

With setup complete, you can now seamlessly manage your Podman instances.

### 4. A Practical Guide to Using ezpodman
You will see how to use ezpodman to connect to both local and remote Podman instances and showcase some of its command-line flags for advanced control.
> For the complete set of options you can run `ezpodman -h`

Managing the Local Podman Instance

The simplest use case is managing your local podman containers. Running ezpodman with no arguments defaults to local mode.
On Linux, it connects directly to the local rootless Podman socket.
On macOS, it automatically detects and creates a tunnel to the default `podman-machine`, treating it as the local instance.
```shell
ezpodman
```

#### Connecting to Remote Instances
Connecting to a remote host is just as simple. You have two main options:
1. Use the interactive picker to select from your list of configured remotes. Having `fzf` installed makes this option more enjoyable.
```shell
ezpodman -r
```

2. Connect directly to a specific, named remote instance:
For a common practical example, you can connect to a production server and instruct the script to keep the SSH tunnel running even after you exit Lazydocker (`--persist` or `-p`). This is invaluable when you need to run a series of podman-remotecommands or perhaps attach a VS Code remote container session to the same endpoint without having to manage the tunnel manually.
```shell
ezpodman -r prod-server -p
```

#### Advanced Tunnel Control
`ezpodman` is not just for `Lazydocker`; it can also manage SSH tunnels independently. This makes it useful utility for any other Docker-compatible tools you might use, such as custom scripts.
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
