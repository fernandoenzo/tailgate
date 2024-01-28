# TailGate

![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/fernandoenzo/tailgate)
![GitHub - License](https://img.shields.io/github/license/fernandoenzo/tailgate)

Greetings, fellow technophile. Let’s take a moment to appreciate the marvel that is Tailscale. In the grand tradition of Unix philosophy, it does one thing and does it well: it establishes 
point-to-point WireGuard tunnels with a level of finesse in UDP hole punching that is truly commendable.

However, as we all know, no software is without its quirks. Tailscale has been grappling with a particular [issue since 2021](https://github.com/tailscale/tailscale/issues/1552).

In essence, Tailscale attempts to establish connections using all available network interfaces on your device. While this might seem like a good idea on the surface, it can lead to some unintended 
consequences. For instance, if you’re like me (and it's likely you are, since you've landed here), you might already have a VPN interface (in my case, OpenVPN) connecting you to devices that you also 
want to interconnect with Tailscale. Unfortunately, Tailscale sometimes routes traffic through the VPN interface instead of the open internet, which can hamper speed optimization.

Fear not, for this project offers a solution as elegant as it is simple. It provides two systemd units: `tailgate-allow@` and `tailgate-deny@`. These allow you to operate in either whitelist or 
blacklist mode, depending on your preference. With these units, you can control Tailscale’s routing behavior, either preventing it from routing traffic through certain network interfaces on your 
device, or forcing it to use only the interfaces you select, discarding all others.

The power, as they say, is in your hands. Happy hacking!

## Installation

The installation process for this project is refreshingly straightforward. All you need to do is copy one or both of the systemd units to the `/etc/systemd/system` directory on your Linux device. 
Here’s how you can do it:

```commandline
sudo cp tailgate-allow@.service /etc/systemd/system/
sudo cp tailgate-deny@.service /etc/systemd/system/
```

Once you’ve done that, you’ll need to reload the systemd daemon to ensure it recognizes the new units. You can do this with the following command:

```commandline
sudo systemctl daemon-reload
```

And that’s all there is to it!

## How to use it

In the realm of network routing, precision is key. It’s crucial to understand that using both the `tailgate-allow@` and `tailgate-deny@` units simultaneously can lead to unexpected behavior. This 
might seem obvious, as no program in the world operates with both whitelists and blacklists at the same time, but it bears repeating.  
It’s like trying to use a joystick and a paddle at the same 
time on your Atari - it just doesn’t work. So, pick one.

Personally, I’d go with the whitelist mode. Why, you ask? Well, most devices have a limited number of interfaces that connect to the internet (ethernet and WiFi, at most), while the number of virtual 
interfaces can change faster than you can say “Moore’s Law”.

Now, these systemd units are not your run-of-the-mill units. They don’t have an `[Install]` section and they’re not meant to boot with your system. They’re special. They’re designed to be linked to 
the `tailscaled` service. So, instead of fiddling with the default unit provided by Tailscale, you can just run:

```commandline
sudo systemctl edit tailscaled.service
```

Then, add your chosen network interfaces to the `Wants=` option (and don't forget to add the default `network-pre.target`), like so:

```commandline
[Unit]
Wants=network-pre.target tailgate-allow@eth0.service tailgate-allow@wifi0.service
```

Once you’ve saved your changes, a new file will be created at `/etc/systemd/system/tailscaled.service.d/override.conf`

Don't forget to reload the systemd daemon:

```commandline
sudo systemctl daemon-reload
```

Now, whenever `tailscaled` starts, stops, or restarts, our systemd units will 
follow suit, controlling Tailscale traffic through the desired network interfaces. No further user interaction required. It’s as simple as PEEK and POKE in BASIC!
