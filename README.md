# fzf-ssh-manager

Simple TUI fuzzy finder for SSH servers found in ~/.ssh/config. Minimal setup with almost no dependency. Additional features like status display and servers/containers startup with proxy traversal.

> [!WARNING]
> This tool is not designed to replace a monitoring or a provisioning system. By default, at each launch it sends a ping to any server found in the configuration file. If used with many nodes behind a proxy server, it can produce huge load on it.

> [!CAUTION]
> Doesn't provide any security mechanism for credentials (ipmi, redfish…)

<picture>
    <source media="(prefers-color-scheme: dark)"
        srcset="assets/demo-dark.gif">
    <img alt="fzf-ssh-manager Demo" src="assets/demo-light.gif">
    <br>
    <b>FzF-SSH-Manager</b>
</picture>


## Features

- fuzzy find hosts in the ssh config file (*~/.ssh/config*)
- minimal configuration based on ssh_config
- node status display (ping response)
- ssh proxy traversal (man ssh_config -> ProxyCommand)
- node automatic power up
    - WakeOnLan
    - Ipmi
    - Redfish
    - Docker / K8S / NSpawn / Incus
    - or any custom command
- easy installation, pure bash/fzf, no extra dependency
- ~~[ ] filter by any criteria (up down, unknown, start method)~~
    - apparently pattern matching on hidden text will not be implemented in fzf


## Out of scope

- scalability over 100 nodes
- power management
- parallel shell
- node management / deployment
- credential security
- any persistence


## Requirements

### Mandatory

- bash
- fzf
- ping
- OpenSSH

### Optional

- unicode fonts like nerdfonts
- wakeonlan
- ipmitool
- incus
- docker
- k8s
- systemd/nspawn

## Installation example

Adapt the following at your convenience.

```bash
# Script installation
mkdir -p ~/.local/bin
cp ssh-manager ~/.local/bin
chmod +x ~/.local/bin/ssh-manager

# Bash keybind
cat > ~/.bashrc.d/ssh-manager.sh << 'EOF'
bind -x '"\C-bs": "~/.local/bin/ssh-manager"'
EOF

```
## Configuration keywords

ssh-manager will use look for the following keywords in your ssh
configuration file:

### Host

SSH hostname, documented in man ssh_config(5). Patterns not implemented/tested.

### Hostname

DNS hostname or IP address, documented in man ssh_config(5).

### ProxyCommand

SSH proxy management, documented in man ssh_config(5).

ssh-manager assumes the following format:
> ProxyCommand ssh -W %h:%p proxyNameOrIpAddress


### #DESC

Host description, free format.

### #WAKE

Used to start servers remotely:
> #WAKE <method> <timeout_s> wake cmdline

- **timeout_s**: ssh-manager will wait up to this duration before starting ssh,
- **wake cmdline**: custom command line to start servers. Can be any command
or custom script,
- **method**: boot method. Since there isn't any restriction on **cmdline**,
the main purpose of **method** is for icon management.
Can be any of this ones:
    - docker 🐳
    - podman 🦭
    - k8s ☸️
    - incus 🦴
    - nspawn 📦
    - lxd 📦
    - libvirt ✨
    - wol ⚡
    - ipmi 🌐
    - redfish 🐠
    - exec ⚙️


### #NOPING

Don't ping and don't display the node status. Useful to avoid overloading proxy
servers with too much SSH connections (1 per host entry).

### #HIDE

Don't display this host on the Fzf menu.

## Script customization



### MAXJOB

Maximum number of simultaneous bash jobs. Can be used to limit the number of parallel SSH connections on proxy servers. Unlimited by default.

### NO_UNICODE

Set this variable in your environment or in the script to use only ASCII
characters.

### PING_TIMEOUT

Change this variable in the script if you have latency issues on some servers.
Defaults to 1s.


### MAXNAMELENGTH

Host name length detection is not dynamic to avoid parsing the configuration twice. Change the hardcoded value if you have column alignment issues.

