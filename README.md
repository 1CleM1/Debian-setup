# Debian homelab

My first headless server — a Debian box on my local network, set up from scratch and documented as I go so I can rebuild it if I ever have to.

> [!NOTE]
> This is a personal setup guide, not production documentation. It's written for a
> **local network** — nothing here is hardened for exposing a server to the internet.

<!-- SCREENSHOT SLOT: a photo of the machine itself, or the login prompt on the monitor.
     Nice to open with something visual. Remember to crop out the hostname if it's on screen. -->

## What it runs

Built from reused parts I had lying around, the notable parts would be 
 - i7-6700k
 - 16 gb ddr4 ram
 - 4 tb RED NAS drive
 - rest was just some random, psu, motherboard, case.
 - also had a gtx 970 on hand, but would just increase my cost for no upside. 

| | |
|---|---|
| **OS** | Debian 13 |
| **Partitioning** | LVM, whole disk |
| **Firewall** | UFW |
| **Services** | Samba share · AdGuard Home · Tailscale · Debian VM for schoolwork |

The VM is the part I use most day to day. I SSH into it from anywhere over Tailscale to do
university work like "operating systems". As i do the rest of my work on an M3 macbook air that (ofc) runs ARM.
I can then ssh into a machine that runs x64. 

<!-- Add a line about why i picked the parts or something -->

## Prerequisites

- A PC or server that boots (UEFI or BIOS — it matters in step 1)
- A USB stick for the installation media
- A wired ethernet connection — the installer needs it for network setup

---

## 1. Installation media

Download the Debian ISO from [debian.org](https://www.debian.org/).

Download and format the installation media. Depending on whether your system is UEFI or BIOS,
you need to choose the GPT or MBR partition scheme.

I do this with a tool called [RUFUS](https://rufus.ie/en/) on Windows.

> [!TIP]
> Download the portable version of RUFUS, then you only need the .exe and don't have to install anything.

In RUFUS, choose GPT or MBR, keep all other settings default, and press ready. Close RUFUS
afterwards and eject your drive.

<!-- SCREENSHOT SLOT: RUFUS with your settings filled in. This is a good one to include —
     the GPT/MBR choice is where people get it wrong. -->

Go into the BIOS of your PC/server, select the USB (or other boot media), and set the boot order
so it boots into it.

## 2. Debian graphical install

When Debian 13 or newer launches, choose **Graphical install**.

Go through the first couple of steps and fill out your region, language and keyboard.

You will need an ethernet connection to proceed with network setup.

<!-- SCREENSHOT SLOT: the Debian installer boot menu. Optional — this one is fairly self-explanatory. -->

## 3. System configuration

Now we go through setting up the more crucial components.

**Hostname** — this setup is for local network use, so the naming scheme follows an easy-to-follow
pattern that matches my other machines on the network.

**Domain name** — this is a local network so the domain name isn't important, but fill out
`.yourowndomain` or whatever else.

**Root password** — advised to set up if you plan on deploying the server.

**Full name** — yes, fill out your name.

**Username** — something that matches your other machines on the network.

**Timezone** — configure it in the next step.

> [!WARNING]
> If you screenshot this section, blank out your hostname and username before committing it.

## 4. Partitioning the disk

If you have encrypted drives this isn't necessary. What many people recommend is either
*use entire disk and set up LVM* or *use entire disk and set up encrypted LVM*.

We're choosing LVM for this setup.

1. Choose your operating system drive
2. You may choose to make separate partitions, but it's really a hassle and I chose not to
3. Check **[yes]** for partition disk
4. Check that it has selected your whole drive in the next step
5. It will prompt you to accept the partitions it has created

<!-- SCREENSHOT SLOT: the partitioning screen with LVM selected. Worth including —
     this is the step that's hardest to undo if you get it wrong. -->

You will now install the base system after partitioning.

## 5. Package mirror

Next, Debian asks where the package manager should download its packages from — both after
installation and for your `apt` commands.

Choose your own country or one close by.

<!-- The original notes trail off here. Anything between the mirror step and a
     working install worth writing down? SSH setup, sudo, static IP? -->

## 6. Firewall (UFW)

```bash
# install ufw
sudo apt install ufw

# allow the ports you actually need before enabling — otherwise you can lock
# yourself out of SSH on a headless machine
sudo ufw allow [your port]

sudo ufw enable
sudo systemctl enable --now ufw
```

Check what's currently allowed:

```bash
sudo ufw status numbered
```

The numbered output is what you want, because it gives you the rule numbers you need to delete
a rule later:

```bash
sudo ufw delete [number]
```

<!-- TODO: your note said "delete ipv6 if you dont use it" — add the actual steps here.
     (It's the IPV6=yes line in /etc/default/ufw, plus a ufw reload.) -->

## 7. SSH with keys instead of passwords

This is a headless machine, so SSH is the only way in. That makes the login the weakest part of
the whole setup, and a password is a lot weaker than a key.

A key is really two files. The private one stays on your own machine and never moves anywhere,
the public one goes on the server. When you log in the server sends a challenge that only the
matching private key can answer, so nothing secret is ever sent over the network. A password gets
sent every single time you log in, which is the difference.

### Generate the key

Do this on the machine you connect FROM, not on the server. For me that is the macbook.

```bash
ssh-keygen -t ed25519 -C "macbook"
```

ed25519 is the current default and a lot shorter than the old RSA keys, there is no real reason
to pick anything else now.

It asks where to save it (just press enter, the default is fine) and then for a passphrase. Set
one. The passphrase encrypts the private key file on your own disk, so if someone ever gets hold
of the file it is useless to them without it. You only type it once per session, the ssh agent
remembers it after that.

> [!NOTE]
> The bit after -C is only a label so you can tell your keys apart later. I name it after the
> machine instead of an email.

### Copy the public key over

```bash
ssh-copy-id yourusername@yourserver
```

That appends the public key to `~/.ssh/authorized_keys` on the server and sets the file
permissions correctly, which is the part that goes wrong when you do it by hand.

Test it right away, before changing anything else:

```bash
ssh yourusername@yourserver
```

It should let you straight in, or ask for your key passphrase. If it still asks for the server
password then the key did not land properly, and you want to sort that out before going further.

### Turn off password login

> [!WARNING]
> Keep your current SSH session open the whole time you do this. If you lock yourself out you
> need a monitor and a keyboard on the server to fix it, which is exactly what headless was
> supposed to avoid.

```bash
sudo nano /etc/ssh/sshd_config
```

Set these three:

```
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
```

`PermitRootLogin no` means nobody can ssh straight in as root, they have to log in as you and
then sudo. Bots try root constantly, so this one line alone cuts a lot of noise out of your logs.

### Change the port

Same file, find the Port line and set whatever you want:

```
Port 2222
```

Anything above 1024 is fine. Be honest about what this does though, it is not real security.
Anyone actually scanning you finds the new port in seconds. What it does do is stop the endless
automated bots that only ever knock on port 22, and that keeps your logs readable.

**On Debian 13 there is a catch here.** A fresh install uses socket activation, which means
`ssh.socket` is holding the port and not `ssh.service`. So you change Port in sshd_config,
restart, and nothing happens, or you get "fatal: Cannot bind any address" because the socket is
still sitting on 22.

Turn the socket off:

```bash
sudo systemctl disable --now ssh.socket
```

If you upgraded from Debian 12 rather than installing fresh, the socket is already disabled and
you can skip this.

### Open the new port before you restart

Back to ufw from section 6, and do this first, not after.

```bash
sudo ufw allow 2222/tcp
sudo ufw status numbered
```

Once the new port is confirmed working you can remove the old rule for 22 with
`sudo ufw delete [number]`.

### Check the config, then restart

Always check before restarting. It catches the typo that would otherwise lock you out.

```bash
sudo sshd -t
```

No output means it is happy. Then:

```bash
sudo systemctl restart ssh
```

Now open a NEW terminal, leave the old session running, and try the new port:

```bash
ssh -p 2222 yourusername@yourserver
```

If that works you are done. If it does not, you still have the old session sitting there to
undo it with.

### Stop typing the port every time

On your own machine, put this in `~/.ssh/config`:

```
Host homelab
    HostName 192.168.x.x
    User yourusername
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

Then it is just:

```bash
ssh homelab
```

> [!TIP]
> Since the server is on Tailscale anyway, you can put the tailscale hostname in as HostName
> instead. Then you reach it the same way from anywhere without opening a single port to the
> internet, which is better than any port number you could pick.

<!-- SCREENSHOT SLOT: your ufw status output after all this, with the ports blanked out.
     Shows the end state without giving away your actual setup. -->

---

## Services

<!-- Each of these deserves a short section: what it does, why you run it,
     and anything that tripped you up. That's the part nobody else's guide has. -->

### Samba share

<!-- What you share, and what you access it from. -->

### AdGuard Home

<!-- Network-wide DNS filtering. Note how you pointed clients at it. -->

### Tailscale

<!-- How you reach the machine from outside the house. -->

### Schoolwork VM

<!-- Debian VM you SSH into for university work. Why a VM rather than the host? -->

---

## Notes / things I'd do differently

- Looking at replacing the self-built i7-6700K box with something smaller, quieter and much more cost effective —
  a Lenovo ThinkCentre or similar small-office machine.
  a immich image share, for me and family members.

<!-- Add to this as you go. This section tends to become the most useful part of a
     setup guide, because it's the only bit that isn't in the official docs. -->
