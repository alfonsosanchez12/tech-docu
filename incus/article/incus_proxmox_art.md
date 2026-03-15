# Why Incus is the best starting point for your homelab, and your DevOps journey

*A personal take on why less is more in your DevOps Learning path.*

---

I'll be honest, this is not a neutral comparison. This is me telling you why I think Incus is one of the best tools you can pick up when you're starting a homelab and want to actually learn how infrastructure works, not just click through an UI until something runs.

About a year ago, a friend who became my unofficial Linux and DevOps mentor introduced me to Incus. Before that, I had tried Proxmox, gotten frustrated, and nearly given up on self-hosting. What changed wasn't just the tool, it was the philosophy behind it.

## The Proxmox experience

When I first decided to build my homelab a while ago, I did what most beginers do, followed the youtubers path. And YouTube had a pretty clear pathway: *Proxmox is the way to go. It's easy, it's powerful, it's what everyone uses.* So I installed it on my main server and got ready to learn.

By that moment I was already comfortable with Linux and the CLI, but new to infrastructure. I expected a learning curve. What I didn't expect was feeling unconfortable with the tool before I even got started. The web UI is smashed in your face, the subscription base repository is annoying, every VM starts with an ISO and an installer, and the whole thing feels overwhelming.

I just wanted to spin up a couple of environments and learn. Instead, I got an over stacked suit I wasn't ready for, and I walked away.

For some time after that I stuck with Podman as a way to keep experimenting without the hassle of dealing with all the extra features I didn't need. Podman taught me a lot about the containers. However, I couldn't simulate "real" infrastructure the way I wanted. That's when the same mentor pointed me toward Incus, and things started to click.

## The Linux philosophy analogy

Here's how I think about it. Proxmox is the "Windows" of hypervisors, it comes fully loaded out of the box. "Noisy" UI, unsolicited features, workflows, all decided for you. That's great if you're not in a learning journey. It's overwhelming if you're still figuring it out.

Incus is the "Linux" of hypervisors. It comes with the minimum needed to get going, and you build from there based on what you actually need. No web UI by default, but you can add one. No ISO management forced on you, but it's available when you need it. No annoying enterprise subscription; it's fully open source and community driven.

That minimalism isn't a weakness. It's a teaching tool, it leads you to learn and try. Every component you add to Incus, you add it intentionally. You understand why it's there because you put it there yourself. That's exactly how Linux works, and it's exactly how good DevOps engineers think.

## Getting started is really fast

Once Incus is installed, initialization takes one command:

```bash
incus admin init
```

It walks you through a guided setup: storage pool, networking, clustering (you can skip that), and you're ready to go. No ISO to download, no installer to click. Want to launch an Ubuntu container that pulls a minimal Ubuntu 24.04 image and starts a fully functional system in seconds?

```bash
incus launch images:ubuntu/24.04 my-container
```

 Want a VM instead?

```bash
incus launch images:ubuntu/24.04 my-vm --vm
```

Same command, one flag. The mental model is consistent across containers and VMs, which is something you really appreciate once you've worked with tools that treat them as completely separate things.

## Containers and VMs, The best of both worlds.

One thing Incus helps you learn through hands-on is the difference between system containers and virtual machines, and where each fits.

Incus system containers (LXC-based) are like lightweith VMs, they share the host kernel, boot with a full OS, run systemd, have their own network, behave like a real machine; but they're much lighter than a VM because there's no hardware emulation. Perfect to use in Old HW or small systems where you can spin lots of these.

VMs in Incus run via QEMU and give you full isolation. You can use them for things like Kubernetes nodes, where you need a proper isolated kernel, or for testing distros.

```bash
# Get a shell inside your container / VM
incus exec my-container -- bash

# or simplier
incus shell my-container
```

You are not limited to run LXC images repo that comes by default. If you prefer OCI Docker and Podman images, you can use them, and Incus system containers are a perfect host for them. You can add the Docker or Podman repos natively:

```bash
# Add Podman and Doker repo:
incus remote add quay/podman https://quay.io/podman --protocol=oci
incus remote add oci-docker https://docker.io --protocol=oci

# Spin a podman or docker container:
incus launch quay/podman:hello --ephemeral --console
incus launch oci-docker:hello-world --ephemeral --console
```

This pattern is clean and reusable trhough all the incus expirience.

## What my homelab actually looks like

I run Incus on Debian servers. After about a year, my homelab looks like this:

In containers: **Pi-hole** (network-wide ad blocking), **Caddy** (reverse proxy and automatic TLS), and **Step-CA** (a private certificate authority for internal services). Each lives in its own isolated container, easy to snapshot, easy to rebuild.

In VMs: dedicated nodes for Kubernetes lab clusters. Running Kubernetes in Incus VMs gives you real kernel isolation per node, which means you're practicing on something that actually resembles a production environment — not a nested hack.

Every one of those deployments taught me something. Networking between containers, storage profiles, cloud-init configuration. I learned all of it hands-on because Incus exposes those concepts directly instead of hiding them away behind a Web UI.

## Is Incus for everyone?

Yes, if you value deep learning and hands-on infrastructure, if you really want to make each experience in your homelab an opportunity to grow.

This is my take, if you're starting your homelab journey, are comfortable with Linux and a terminal, and want a tool that grows with your understanding, Incus is hard to beat.

> The best tool isn't the one with the most features. It's the one that makes you understand what you're doing.

Incus did that for me. A year in, I'm a better DevOps engineer for it, not because Incus is magic, but because it got out of my way and let me learn.

---

*If you're curious about Incus, the official documentation is at [linuxcontainers.org/incus](https://linuxcontainers.org/incus). The community on the forums and GitHub is also really helpful.*
