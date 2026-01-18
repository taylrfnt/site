+++
title = "Modern SSH Hardening"
date = "2026-01-17T18:08:27-06:00"
author = "Taylor Font"
cover = ""
coverCaption = ""
tags = ["ssh", "security", "yubikey", "fido2"]
keywords = ["ssh", "security", "hardening"]
description = "A short and sweet guide on best practices to harden SSH and increase security for your public-facing resources."
showFullContent = false
readingTime = true
hideComments = false
color = "" #color from the theme settings
+++

# Disclaimer

**Nothing I write about in this blog post is sponsored. I do, on occasion,
recommend a few specific tools and platforms that I personally use, but those
are not paid endorsements or sponsored content. Just me liking them based on my
personal usage.**

# Setting the stage

It's 2026. Hackers are more skilled than ever, zero-days are being published at
what feels like an ever-increasing pace, quantum computing is now a concern for
breaking once faithful encryption algorithms.

You might SSH into a server nowadays and see something like the following
warning:

```bash
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
```

Between the above warning, other warnings about insecure or unsupported key
types, and the general state of the world, you might be interested in hardening
SSH for your servers. If you are, look no further - this blog post is meant to
provide you with a no-nonsense guide to hardning SSH for the modern age, and
provide you with a few additional ways you can further secure your servers and
identities.

# Low-hanging fruit

This section will cover the basics that everyone should be doing, no matter
what. This section is going to be the oft-repeated-but-not-actioned advice that
you've seen a million times, unless this is your first day on the internet.

## Updating OpenSSH

Before you do anything, you should always make sure your version of OpenSSH is
as modern as it can be. For folks using rolling distributions or package
managers (Arch, Gentoo, etc.) this will often be latest mainline releases. For
folks using point/fixed distros (Ubuntu LTS, Debian, Fedora, etc.), you'll be
using the version your distro supports, which should include security patches
for said version via official repos.

Making sure you keep OpenSSH updated ensures you're not leaving the door open to
any CVEs that may be disclosed and exploited.

## Default SSH configurations

This next one is oft-repeated-but-not-actioned. Change the default SSH
configuration. Tools like [`ssh-audit`](https://github.com/jtesta/ssh-audit) are
a fantatsic way to audit your SSH configurations and harden them.

In case you don't want to run the tool, there's a few things that you will want
to specifically update:

- Disabling SSH logins via password. Use SSH keys instead.
- Disable root login via SSH. Properly configure your sudo rules to allow users
  access to run commands as `root` where needed.
- Disable weak/vulnerable key algorithms. The most well-known offender is the
  [Diffie-Helman key exchange](https://medium.com/@lydia.cao26/diffie-hellman-key-exchange-basics-and-vulnerabilities-afc51342988e),
  but there are others. Most modern OpenSSH defaults should disable many of
  these, but usually not all.
- Limit authentication attempts. Set the amount of times a login attempt can
  fail authentication before being blocked or having to reconnect later. Leave
  yourself some room for mistakes - 3 to 5 attempts is usually enough to balance
  bouncing bad actors and not breaking your login attempt any time you make a
  mistake.

# Modern hardening methods

In addition to the tried-and-true classics, there's several additional steps
that can be taken to enhance your security posture. While the aforementioned
items focused specifically on OpenSSH configuration and patches, this next
section will expand our horizons - security is NOT just editing a config file or
running an `apt update && apt upgrade`.

In today's day and age, security extends into all sorts of things - scams are
more common than ever, passwords are breached all the time, and online systems
are just as vulnerable, if not more, than physical ones.

## Hardware-authenticator enabled keys for SSH

WebAuthn has had the concept of passkeys for a while now. You probably know
passkeys from the popup that appears on pretty much every wbesite that you log
into where you don't have one. You can store passkeys in your password manager,
or you can even store them on hardware keys[^1], like a Yubikey for additional
layer - user presence.

[^1]: I did a bit of hand-waving there when talking about storing passkeys on
    hardware authenticators. A lot of passkeys end up being non-resident keys,
    meaning the content is not stored directly on the device, and instead simply
    requires the device (and by extent, the user's presence) to use the passkey.
    There are certain sites which use resident keys, though, which do live
    on-device.

Little do most people know, but hardware-based authenticators (well, those that
support FIDO2) can also be used for SSH! Starting with
[OpenSSH 8.2p1](https://www.openssh.org/txt/release-8.2), there are new key
types that support the use of hardware authenticators:

> This release adds support for FIDO/U2F hardware authenticators to OpenSSH.
> U2F/FIDO are open standards for inexpensive two-factor authentication hardware
> that are widely used for website authentication. In OpenSSH FIDO devices are
> supported by new public key types "ecdsa-sk" and "ed25519-sk", along with
> corresponding certificate types.

_**NOTE:** OpenSSH 8.2p1 released in 2020, so even though this article is an
attempt to get your SSH security into the modern age, just know this is by no
means bleeding-edge stuff._

The
[Yubico documentation](https://developers.yubico.com/SSH/Securing_SSH_with_FIDO2.html)
goes over how to generate both kinds of keys - resident and non-resident keys.
Either one is a boon to your security posture, since it requires hardware
presence to use the private key.

My personal prefence is to use a resident key, since they are portable - you can
use the following command to download the key pairs onto any server you connect
your hardware authenticator to:

```bash
# download resident SSH key pairs to your device
ssh-keygen -K
```

This is purely for convenience. If you prefer to have a key per server, you can
use non-resident keys instead.

If you're using OpenSSH 8.3 or higher, which you should be, you can also
generate keys that require BOTH a PIN entry and physical touch for every use of
the key via the `-O verify-required` option. I highly recommend this option. The
few seconds you lose to entering your PIN and touching your device are still one
extra layer of security for your keys, especially if you are using resident keys
that can be exported to untrusted servers should a bad actor manage to get their
hands on your hardware authenticator.

Either way you slice it, the FIDO2-enabled keys are a step up from the standard
SSH keys, especially considering more people don't put passphrases on their keys
than anyone would like to admit.

## Take your SSH off the internet

This final item in this article I will discuss is likely the most niche. If
you're hosting servers on a private network, this may not be for you, unless
you've got some weird port-forwarding situation going on.

Anyway - the vast majority of people host their applications on a Virtual
Private Server (VPS) or some other cloud provider's Virtual Machines (VMs). Most
VPSes or VMs default to being accessible over the public internet on port 22,
the standard SSH port. Even if with all the hardening in the world, most SSH
services are **still accessible to the internet**, which means that your server
could be just one more zero-day or CVE away from being vulnerable.

In case you've never done it, take a few seconds to check your VPS/VM and many
scans and see just how many automated bot login attempts come in every hour.
Spoiler alert: it's a lot.

```bash
# view auth log for (most) Linux servers
sudo less /var/log/auth.log
```

[Tailscale](https://tailscale.com/) provides you another way. You don't have to
expose your SSH sessions over the internet. Instead, you can join your servers
and devices into a _tailnet_. This provides you with private IPs and MagicDNS
(it's literally called that) entries that can be used to access the server over
a private connection. Just install Tailscale, authenticate, and you're off to
the races. It's literally [that simple](https://tailscale.com/kb/1017/install).

Just using Tailscale will send your SSH sessions over the private network mesh
for your tailnet; however, it will not prohibit any access from the internet. We
will want to fix that. You can use a tool like `ufw` to
[block all connections
_except_ those coming over your tailnet](https://tailscale.com/kb/1077/secure-server-ubuntu),
which makes it so that no one can SSH to your servers over the internet. Say
hello to the peace of mind that comes with knowing bots and automated scans will
no longer be able to connect to your server - it's effectively been removed from
the internet!

# Wrap-up

This blog post is not meant to be a deep, in-the-weeds guide for how to harden
your security posture line-by-line. That documentation is out there in the wild,
written by people much smarter than me. Over the last few years, I have observed
a surprisingly large amount of carelessness when it comes to some of the basic
steps that can be taken to secure a server or secrets, and felt compelled to
write this. Just a short and sweet mention of the basic concepts that can be
applied with minimal effort that reap significant benefits.It was cathartic.

I've implemented everything I have mentioned in the post in one way or another
on all of my environments, and I have noticed minimal (if any) interruption to
my workflows. I have noticed that I have much more peace of mind with these
hardening steps in place.

This post may not resonate with everyone, but I hope it was informative, at the
very least.
