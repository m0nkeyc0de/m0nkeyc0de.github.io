---
title: "Tailscale"
date: 2026-10-25
lastmod: 2026-10-25
slug: tailscale
type: posts
draft: false
categories:
  - Networking
tags:
  - vpn
---
Frictionless VPN network : who could imagine a such thing would exist ?

## Setting up the tenant

Create an account on [Tailscale](https://login.tailscale.com), then:
* Create [Tags](https://console.tailscale.com/admin/acls/visual/tags) that will be assigned to your devices. They can be considered as "Device Groups".
* Delete the default "Allow all traffic" [Access Control Policy](https://console.tailscale.com/admin/acls/visual/general-access-rules).
* Create specific [Access Control Policies](https://console.tailscale.com/admin/acls/visual/general-access-rules) based on your previously created tags.
* Take note of your unique **Tailnet DNS name** which looks like `tailXXXXXX.ts.net`.


### ACL rule examples
Allow machines with Tag `roaming-clients` to reach internet through **Exit Nodes**.
```json
{
	"src": ["tag:roaming-clients"],
	"dst": ["autogroup:internet"],
	"ip":  ["*"],
}
```

Allow machines with Tag `lab-clients` to reach a network through a **Subnet router**
```json
{
	"src": ["tag:lab-client"],
	"dst": ["192.0.2.0/24"],
	"ip":  ["*"],
}
```

## Linux

### Installation

Run the [provided installtion command](https://tailscale.com/download/linux).

```bash
# Run those commands as root

# Running scripts straight from the Internet isn't generally a great idea.
# Check its content first !
curl -fsSL https://tailscale.com/install.sh | sh

# Enable Tailscale
# The first time an activation link is generated and must be opened in the browser that is logged-in to your tenant.
sudo tailscale up
```

After accepting the new machine:
* Add the required Tags
* Disable key expiry

### (En|Dis)abling

Commands to enable/disable Tailscale on a node.

```bash
# Enable
sudo tailscale up

# Disable
sudo tailscale down
```

### Displaying status and other nodes

With `tailscale status` you can see your own status, and also other nodes you're allowed to reach according to Access Control Policies.

### Route traffic through your machines
A machine can be used as an [Exit Node]() and/or as a [Subnet Router](https://console.tailscale.com/admin/machines).

Here is the basic setup for routing traffic.
```bash
# Check if IP forwarding is already enabled
sysctl net.ipv4.ip_forward

# Enable IP forward
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf

# Firewalld configuration on RHEL
# https://github.com/tailscale/tailscale/issues/3416
sudo firewall-cmd --permanent --add-masquerade
```

#### Exit Node
Useful to access internet through your Tailnet.

```bash
# Enable exit node advertisement
# Note: change is applied without restart
sudo tailscale set --advertise-exit-node
```

Then in the [admin console](https://console.tailscale.com/admin/machines), allow the machine to be an Exit Node in the **Route Settings**. 

#### Subnet Router
Useful to access specific networks through specific nodes.

```bash
# Advertise subnets (comma separated) you want to make available through this node
sudo tailscale set --advertise-routes=192.0.2.0/24
```

Then in the [admin console](https://console.tailscale.com/admin/machines), approve the new Subnet Route in **Route Settings**.

If your client machine is allowed to access this network, a route will automatically be added.
