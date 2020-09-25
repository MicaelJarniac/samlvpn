# SamlVPN

Some VPN providers allow you to log into their service using SAML.
Unfortunately, this is not a standard process and requires a custom client.
Furthermore, some providers don't distribute clients for all the operating
systems commonly used by their users. This program aims to allow you to
connect to SAML authorized VPNs from a Linux client.

## Quick Start

#### Prerequisites

- You must have a working C toolchain installed to compile OpenVPN
	- OpenVPN requires some libraries to compile. Check their docs for details.
- You must have a working Go toolchain installed to compile SamlVPN

### Compile OpenVPN

You will need to be able to compile the OpenVPN client, as it needs to be
patched. More info on this here: [OpenVPN's INSTALL
file](https://github.com/OpenVPN/openvpn/blob/master/INSTALL).

Once you are able to compile OpenVPN, checkout the `release/2.4` branch and
apply the `openvpn-v2.4.9.diff` patch to it:

```bash
git checkout release/2.4
git apply openvpn-v2.4.9.diff
```

Then, compile it again. This patch just changes some buffer sizes to allow the
much bigger SAML payloads.

Finally, you can choose to install it (`sudo make install`) or to move this
patched binary somewhere of your liking.

### Configure and compile SamlVPN

For the sake of simplicity, SamlVPN is configured by modifying its source code.

Run `make config.go` to generate a sample configuration file with documentation
on what each setting does. Then, using your favourite text editor, change this
file to suit your needs: `vim config.go`.

Once you're happy with the configuration, you can then compile and/or install SamlVPN:

```bash
# just compile:
make bin

# install to ~/go/bin
make install
```

### Usage

Once installed, simply run the program.

```bash
# if just compiled
./bin/samlvpn

# or, if installed
samlvpn
```

### How it works

SAML VPN providers work slightly differently than regular ones.

On the first attempt to connect to the VPN, instead of using the usual
authentication procedure, the client will send `N/A` as the username, and
`ACS::{PORT}`, where `{PORT}` is a port in which the localhost will be
listening for a SAML callback. The server will then return an URL that the
localhost will need to open. This URL will start a SAML authentication
procedure, that when successful, will redirect to `localhost:{PORT}`.

This program automates the first part, then starts a server that will receive
the callback. The callback will contain the required authentication details.
At this point, a connection to the VPN is possible by using the username `N/A`
and crafting a password containing the SAML payload and some metadata from the
previous call.

### Credit

This is based on [Alex Samorukov's work](github.com/samm-git/aws-vpn-client).
