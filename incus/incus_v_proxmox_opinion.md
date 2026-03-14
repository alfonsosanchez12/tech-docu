# Why Incus is the best starting point for your homelab — and your DevOps journey

*A personal take on why less is more in your DevOps Learning path.*

---

I want to be upfront, this is not a neutral comparison. This is me telling you why I think Incus is one of the best tools you can pick up when you're starting a homelab and want to actually learn how infrastructure works — not just click through a UI until something runs.

About a year ago, a friend who became my unofficial DevOps mentor introduced me to Incus. Before that, I had tried Proxmox, gotten frustrated, and nearly given up on self-hosting altogether. What changed wasn't just the tool — it was the philosophy behind it.

## The Proxmox experience

When I first decided to build a homelab, I did what most people do — I went to YouTube. And YouTube had a pretty clear consensus: *Proxmox is the way to go. It's easy, it's powerful, it's what everyone uses.* So I installed it on my main server and got ready to learn.

What followed was not easy.

I was comfortable with Linux but new to infrastructure. I expected a learning curve. What I didn't expect was to feel like I was fighting the tool before I even got started. The web UI is front and center, the subscription nag screen greets you on every login, every VM starts with an ISO and an installer, and the whole thing feels like it assumes you already know what you're doing at an enterprise scale.

I didn't need HA clustering. I didn't need corosync. I just wanted to spin up a couple of environments and learn. Instead I was buried under concepts I wasn't ready for, and I walked away.

For a while after that I stuck with Podman — which my mentor had introduced me to — as a way to keep experimenting without the overhead. Podman was great for running apps and services, and it taught me a lot about the container model. But it had a ceiling: it's an application container tool, not a machine management platform. I could run services, but I couldn't simulate real infrastructure. That's when the same mentor pointed me toward Incus, and things started to click.

## The Linux philosophy analogy

Here's how I think about it. Proxmox is the Windows of hypervisors — it comes fully loaded out of the box. UI, features, workflows, all decided for you. That's great if you already know what you need. It's overwhelming if you're still figuring it out.

Incus is the Linux of hypervisors. It comes with the minimum needed to get going, and you build from there based on what you actually need. No web UI by default — but you can add one. No ISO workflow forced on you — but it's available when you need it. No enterprise subscription nags — it's genuinely community-driven with no monetization layer pushing you toward anything.

That minimalism isn't a weakness. It's a teaching tool. Every component you add to Incus, you add intentionally. You understand why it's there because you put it there yourself. That's exactly how Linux works, and it's exactly how good infrastructure engineers think.

## Getting started is surprisingly fast

Once Incus is installed, initialization takes one command:

```bash
incus admin init
```

It walks you through a guided setup — storage pool, networking, clustering (you can skip that) — and you're ready to go. No ISO to download, no installer to click through. Want to launch an Ubuntu container?

```bash
incus launch images:ubuntu/24.04 my-container
```

That pulls a minimal Ubuntu 24.04 image and starts a fully functional system container in seconds. Want a VM instead?

```bash
incus launch images:ubuntu/24.04 my-vm --vm
```

Same command, one flag. The mental model is consistent across containers and VMs, which is something you really appreciate once you've worked with tools that treat them as completely separate workflows.

## Containers vs VMs — and why Incus teaches you the difference properly

One thing Incus makes very clear through hands-on use is the difference between system containers and virtual machines — and where each fits.

Incus system containers (LXC-based) share the host kernel. They boot with a full OS userspace, run systemd, have their own network stack, behave like a real machine — but they're much lighter than a VM because there's no hardware emulation. Think of them as "a server, without the VM overhead."

VMs in Incus run via QEMU and give you full kernel isolation. You'd use them for things like Kubernetes nodes, where you need a proper isolated kernel, or for testing distros at a lower level.

Speaking of Docker and Podman — Incus system containers are a perfect host for them. You can add the Docker or Podman repos inside a container just like you would on a bare-metal server:

```bash
# Get a shell inside your container
incus exec my-container -- bash

# Add Docker's repo (Debian/Ubuntu)
apt install -y ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg \
  -o /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
  | tee /etc/apt/sources.list.d/docker.list
apt update && apt install -y docker-ce docker-ce-cli containerd.io
```

This pattern — machine isolation from Incus, application containers from Docker or Podman — is clean and composable. You get the best of both worlds without them stepping on each other.

## What my homelab actually looks like

I run Incus on Debian servers. After about a year, my homelab looks like this:

In containers: **Pi-hole** (network-wide ad blocking), **Caddy** (reverse proxy and automatic TLS), and **Step-CA** (a private certificate authority for internal services). Each lives in its own isolated container, easy to snapshot, easy to rebuild.

In VMs: dedicated nodes for Kubernetes lab clusters. Running Kubernetes in Incus VMs gives you real kernel isolation per node, which means you're practicing on something that actually resembles a production environment — not a nested hack.

Every one of those deployments taught me something. Networking between containers, storage profiles, cloud-init configuration, snapshot and restore — I learned all of it hands-on because Incus exposes those concepts directly rather than abstracting them away behind a GUI.

## Is Incus for everyone?

Honestly, no. If you need a team-friendly UI out of the box, formal enterprise support contracts, or a mature HA story for production, Proxmox has real advantages. It's a solid product for the right context.

But if you're starting your homelab journey, are comfortable with Linux and a terminal, and want a tool that grows with your understanding rather than front-loading complexity you're not ready for — Incus is hard to beat.

> The best infrastructure tool isn't the one with the most features. It's the one that makes you understand what you're building.

Incus did that for me. A year in, I'm a better DevOps engineer for it — not because Incus is magic, but because it got out of my way and let me learn.

---

*If you're curious about Incus, the official documentation is at [linuxcontainers.org/incus](https://linuxcontainers.org/incus). The community on the forums and GitHub is also genuinely helpful for getting started.*
