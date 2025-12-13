# Manually Managing Local and Remote Podman connection sockets for use of Podman over LazyDocker

This guide will help you to do the manual configuration steps to create a podman-socket that can be used with Lazydocker.

## Prerequisites

**Required Tooling:**

| Tool             | Purpose                      |
|------------------|------------------------------|
| `docker`         | Optional: for other tooling  |
| `podman`         | Local container engine       |
| `podman-remote`  | Connect to remote Podman (Included with Podman 4+)    |
| `lazydocker`     | Docker/Podman TUI            |

## 1. The Hard Way: Manually Connecting Lazydocker to a Remote Podman Instance

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

#### For local linux-machine Podman instances

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

#### For local macos-machine Podman instances

First make sure the default `podman-machine` is initiated and started

```shell
podman machine init
podman machine start
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

#### For remote podman instances

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

Test the connection to confirm that your local client can successfully communicate with the remote Podman instance:

```shell
podman-remote --connection my-remote info
```
