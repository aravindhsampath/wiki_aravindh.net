# linuxadmin

A collection of commands, one-liners and scripts that I have come across related to Linux administration.

## List all MAC addresses in a network

I needed a list of all MAC addresses in my private network(of compute nodes) so that a software can use it for licensing.

```bash
nmap -sn 172.16.0.0/17
```

```bash
Starting Nmap 7.70 ( https://nmap.org ) at 2020-04-16 11:54 CEST
Nmap scan report for 172.16.1.10
Host is up (0.00018s latency).
MAC Address: 00:0E:1C:A2:27:40 (QLogic)
Nmap scan report for 172.16.1.11
Host is up (-0.10s latency).
MAC Address: 00:0E:1E:A1:A7:B0 (QLogic)
Nmap scan report for 172.16.1.12
Host is up (-0.10s latency).
MAC Address: 00:0E:1E:12:31:C0 (QLogic)
Nmap scan report for 172.16.1.13
Host is up (-0.10s latency).
MAC Address: 00:0E:1E:13:33:D0 (QLogic)
--------- Snip ----------
```

## Turn OFF all CPU vulnerability mitigations in Linux

!!! danger
    It would not be prudent to turn these mitigations off at the OS for any server that could run untrusted code and/or facing the wrath of public internet.

 However, on a HPC cluster where compute nodes are humming along running custom built in-house software it is justifiable to choose performance over security - if someone were to run a payload on these machines, I have bigger problems to deal with.

``` bash
echo 'Backing up grub files'
cp /etc/default/grub /etc/default/grub-backup
cp /boot/efi/EFI/centos/grub.cfg /boot/efi/EFI/centos/grub.cfg-backup
## Add this to the kernel parameters
nano /etc/default/grub
noibrs noibpb nopti nospectre_v2 nospectre_v1 l1tf=off nospec_store_bypass_disable no_stf_barrier mds=off mitigations=off
## Make grub again..
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
reboot
```

## Invalidate sssd_cache

SSSD or System Security Services daemon enables access to remote directories and authentication mecahanisms. In particular SSSD is used by FreeIPA/RedHat IDM to talk to the identity server. In a cluster where user identity is managed by FreeIPA/IDM, the clients use SSSD to talk to the FreeIPA/IDM server(s). While testing, or even in real-life odd situations, if you need to invalidate the local cache of SSSD to reflect the change that happened in the server, use the following command.

```bash
sss_cache -E
```

To invalidate the cache of a specific user,
```bash
sss_cache -u user1
```

## Tunnel a local port through a bastion/jumphost to a private server via SSH

<img class="pt3" src="https://aravindh.net/wiki/images/tunnel.png" class="w-100 f5 " alt="SSH tunnel idea"></img>
```bash
ssh -L 25263:dest.server:443 -A username@bastion.server
```
