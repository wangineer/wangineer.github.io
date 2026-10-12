---
layout: post
title: "Getting pyATS MCP Running Quickly Without sudo Access"
description: "How I got the pyATS MCP server running on a Linux server without sudo, connected it to Cursor, and used it for pre- and post-upgrade checks."
date: 2026-10-10 14:00:00 -0400
categories: [Network Automation, pyATS]
tags: [pyats, genie, mcp, cursor, network automation, upgrades, linux, uv]
pin: false
toc: true
comments: true
---



## Scenario

Recently I was tasked with upgrading all of our routers in our environment. The easy part was typing the command that starts the upgrade. The time-consuming part was the pre- and post-check validation.

A device that boots on the new software is not necessarily a healthy device. Beyond the hardware, I had to confirm that everything the device does was still working. That means routing, interfaces, port-channels, high-availability pairing, licensing, CDP neighbors, ARP table entries, registered access points, and more. Our usual method is to run the same long list of `show` commands before and after, save the text, and compare the two copies by eye. Having done this a few hundred times, I need a repeatable more effeciant way of completing the health of the device and the network.

I had been thinking about using pyATS over the past few years. I knew there were practical ways to use it, but I never found a good way to make it easy to use for my team. Writing the test code was a barrier, because not everyone on the team codes... including myself. When the pyATS MCP server appeared, iI thought this could be a game changer. It removes that coding barrier. Anyone on the team can ask questions in plain English, and the tool does the work.

## What is pyATS?

pyATS is Cisco's Python-based network test framework. It logs in to devices over SSH and runs commands for you. Its companion library, Genie, includes parsers for thousands of `show` commands. A parser reads the text a device prints and turns it into structured data: named fields and values instead of a wall of text.

That structure is the important part. Here is the idea, simplified:

```text
Gi0/0/1   10.1.1.1   YES  manual  up   up
```

becomes something like this:

```json
{
  "interface": {
    "GigabitEthernet0/0/1": {
      "ip_address": "10.1.1.1",
      "status": "up",
      "protocol": "up"
    }
  }
}
```

Once the output is saved as a JSON file, you can query it. You can ask which interfaces are down, or compare a file from before a change with a file from after it, and see exactly which values changed. Nobody has to scroll through the text or scrape it with patterns.

pyATS also has a "learn" feature that gathers many related commands into one model of a feature, such as routing or interfaces. In this guide, I saved the parsed output of each command as its own JSON file

## What I wanted to use it for

My first use was pre- and post-upgrade checks. I wanted a tool that could efficiently find issues before and after an upgrade and tell me, for each device, what changed.

I also realized the same approach would help in other situations:

- **After a power outage.** Collect the state of a site and compare it to a known-good snapshot to see what did not come back.
- **When someone reports latency or another problem at a remote site.** Ask the devices there about interface errors, routing changes, and neighbors, instead of logging in and running the commands one at a time.
- **Before and after any change,** not only upgrades.

This post covers how I got the first version working in one evening. It is a first-steps guide, not a polished one.

## How the pieces fit together

The word "server" shows up a lot in this post, and it can mean two different things, so here are the terms I use:

- **pyATS MCP server** is the software. It is the program that connects to your network devices and answers Cursor's requests.
- **Linux server** is the remote machine that the pyATS MCP server is installed on. In my case, it is an AlmaLinux server.

Three machines are involved:

```text
My laptop                    Remote Linux server                 Network devices
(Cursor)  --- SSH tunnel --> (pyATS MCP server)  --- SSH --->    (routers, controllers)
```



## Read this first: this is not best practice

- **This is a temporary way to get started.** I installed the pyATS MCP server in my home directory on a remote Linux server because I didn't have sudo access to use `/opt`. A shared service should live in a system location, run under a service account, and start on its own.
- **It runs in a terminal window on the Linux server.** When that window closes, the pyATS MCP server stops.
- **Only I can reach it.** Cursor on my laptop connects through an SSH tunnel, so my teammates cannot use it.

Use this setup to prove the idea and show people results. Once it has proven its value, ask your server team for a proper installation on a shared Linux server.

## What you need

Too many Linux how-to guides leave out a package or a prerequisite, and you only find out when something fails. Here is everything this setup depended on.

**The remote Linux server**


| Item                      | What I had                                                                                   |
| ------------------------- | -------------------------------------------------------------------------------------------- |
| Operating system          | AlmaLinux 9.8, x86_64 (any RHEL 9 family distribution should behave the same)                |
| Account                   | A normal user account. No sudo, no root, no Docker                                           |
| `git`                     | Already installed. Check with `git --version`                                                |
| `curl`                    | Already installed. Used to download `uv`                                                     |
| System Python             | 3.9.25. Too old for this project, which needs 3.10 or newer. I left it alone                 |
| Network access out        | GitHub (for the clone), astral.sh (for `uv`), and the Python package index (for the install) |
| Network access to devices | The Linux server must already be able to SSH to the devices you want to test                 |


I did not install any system packages. If `git` or `curl` is missing on your Linux server, you will need someone with sudo access to run `sudo dnf install git curl`.

**Installed during setup, in your home directory on the Linux server (no sudo needed)**


| Item             | Version I got                                                                                    | Why                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| `uv`             | 0.12.23                                                                                          | Installs Python and Python packages without touching the system               |
| Python           | 3.12.15, installed by `uv`                                                                       | Runs the pyATS MCP server                                                     |
| pyATS MCP server | [automateyournetwork/pyATS_MCP](https://github.com/automateyournetwork/pyATS_MCP), `main` branch | The pyATS tools that Cursor calls                                             |
| Python packages  | About 150, from the project's `requirements.txt`                                                 | Includes pyATS, the Genie parsers (`genie-libs-parser`), and `genie-libs-sdk` |


**Your laptop**

- Cursor
- An SSH client (macOS and Linux include one)
- SSH access to the Linux server



## Set up a projects folder for Git repositories

Keep each Git repository in its own folder under one parent folder. When you add another project later, it sits next to this one and the two stay separate.

```text
/home/youruser/projects/
├── pyats-mcp/       <- the pyATS MCP server, cloned from GitHub
└── another-repo/    <- the next project
```

On the Linux server, your home directory is normally `/home/youruser`, and `~` is a shortcut for it. Some company Linux servers mount home directories in a different location, so run `echo $HOME` to see yours.

This project is public, so cloning it over HTTPS does not ask for credentials. If you plan to commit anything from this Linux server, set your name and email once:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

The `git clone` command creates the project folder for you. Run it from inside `projects`, not from inside a `pyats-mcp` folder you created yourself:

```bash
cd ~
mkdir -p projects
cd projects
git clone https://github.com/automateyournetwork/pyATS_MCP.git pyats-mcp
cd pyats-mcp
pwd
```

`pwd` should print `/home/youruser/projects/pyats-mcp`.

## Install Python 3.12 with uv

The Linux server's system Python was 3.9, which is too old for this project. Without sudo access, I could not upgrade it, and I did not want to change the system anyway. Instead, I used `uv`, a Python installer and package manager from Astral that installs entirely inside your home directory on the Linux server.

The first command uses `curl` to download the `uv` installer from astral.sh and run it. The second command adds `uv` to the current shell's `PATH`. The third command downloads a private copy of Python 3.12.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
uv python install 3.12
```

`uv` is installed in `~/.local/bin`, and the Python it downloads also stays in your home directory on the Linux server. Nothing system-wide changes.

## Install the pyATS MCP server's packages

Create a virtual environment inside the project, then install the project's package list into it:

```bash
cd ~/projects/pyats-mcp
source $HOME/.local/bin/env
uv venv --python 3.12
source .venv/bin/activate
uv pip install -r requirements.txt
```

When the environment is active, your prompt starts with `(.venv)`. Running `source .venv/bin/activate` again does not create a second environment. It simply activates the same one.

## Writing files: echo and cat, or vi

I normally edit files in Visual Studio Code as that has been my go to software in the past. For this first setup, Cursor was handing me the exact commands to run, and copying them into the Linux server's terminal was quicker than opening an editor. Pasting the same command twice also gave the same result, which made it easy to retry.

To keep the install log out of Git, I pasted this line:

```bash
grep -q '^install.log$' .gitignore || echo 'install.log' >> .gitignore
```

The `grep -q` part checks whether `.gitignore` already contains that exact line. If it does not, `echo` adds the line to the end of the file. If it does, nothing happens, so the command is safe to run more than once. A successful run prints nothing.

I used the same pattern to keep the device file, which should never be committed, out of Git:

```bash
grep -q '^testbed.yaml$' .gitignore || echo 'testbed.yaml' >> .gitignore
```

If you would rather edit the file by hand, run `vi .gitignore` and press `i` to start typing. Add `install.log` and `testbed.yaml` on their own lines, press Esc, then type `:wq` and press Enter. The result is the same.

## Create a basic testbed file

The testbed is the list of devices that pyATS can log in to. The clone does not include one, so you need to create it.

I made three decisions about this file:

- **Prompt for credentials instead of storing them.** The `%ASK{...}` setting makes pyATS ask for the username and password when it first loads the file. Because the credentials are set at the top of the file, one login covers every device. I like this as a starting point. While you are only testing, you do not have to store a clear-text password, and you do not have to set up a vault yet.
- **Keep the paging commands.** Every time pyATS logs in, it sends `terminal length 0` and `terminal width 511` so long output does not stop at `--More--`. These settings last only for that SSH session and are not saved to the device configuration, although your AAA logs will still record them.
- **Block configuration changes at login.** On some platforms, pyATS also enters configuration mode at login and sends commands such as `no logging console` and `exec-timeout 0`. Setting `init_config_commands: []` prevents that, so the connection stays read-only.

Replace the device name and address with a device your Linux server can already SSH to:

```bash
cd ~/projects/pyats-mcp
cat > testbed.yaml << 'EOF'
testbed:
  name: lab
  credentials:
    default:
      username: "%ASK{Username}"
      password: "%ASK{Password}"
    enable:
      password: "%ASK{Password}"

devices:
  LAB-RTR01:
    alias: LAB-RTR01
    os: iosxe
    connections:
      cli:
        protocol: ssh
        ip: 192.0.2.10
        port: 22
        arguments:
          connection_timeout: 360
          init_exec_commands:
            - terminal length 0
            - terminal width 511
          init_config_commands: []
EOF
```

The quotes around `'EOF'` matter. They stop the shell from changing anything inside the block, including the `%ASK{...}` text.

If you prefer `vi`, run `vi testbed.yaml`, paste in the YAML, then save and quit. YAML requires spaces, not tabs, and every indent has to line up exactly.

To add another device later, add a new entry under `devices:` with the same indentation as the first one, then restart the pyATS MCP server so it reloads the file.

## Start the pyATS MCP server

In your first terminal window, logged in to the Linux server:

```bash
cd ~/projects/pyats-mcp
source .venv/bin/activate
export PYATS_TESTBED_PATH=$HOME/projects/pyats-mcp/testbed.yaml
export PYATS_MCP_HTTP_HOST=127.0.0.1
export PYATS_MCP_HTTP_PORT=8080
export PYATS_MCP_TESTBED_CACHE_TTL=86400
python pyats_mcp_server.py
```

Here is what each setting does:

- `PYATS_TESTBED_PATH` tells the pyATS MCP server which testbed file to load.
- `PYATS_MCP_HTTP_HOST=127.0.0.1` makes the pyATS MCP server listen only on the Linux server itself, so it is not exposed to the network.
- `PYATS_MCP_HTTP_PORT=8080` sets the port that the tunnel will connect to.
- `PYATS_MCP_TESTBED_CACHE_TTL=86400` keeps the loaded testbed, including your credentials, for 24 hours or until the pyATS MCP server restarts.

A `pkg_resources` warning appears first. It is harmless. The pyATS MCP server is ready when it prints `Uvicorn running on http://127.0.0.1:8080`.

The first time Cursor uses a pyATS tool, the username and password prompts appear in this Linux server window, not in Cursor. The first time, I pressed Enter without typing anything, and the pyATS MCP server cached the empty login. If that happens to you, press Ctrl+C, start the pyATS MCP server again, and you will be prompted again.

Leave this window open while you work, and press Ctrl+C when you want to stop the pyATS MCP server.

## Connect Cursor through an SSH tunnel

The pyATS MCP server only listens on the Linux server's own loopback address, so your laptop needs a tunnel to reach it. Open a second terminal on your laptop, not on the Linux server, and forward port 8080 on your laptop to port 8080 on the Linux server:

```bash
ssh -L 8080:127.0.0.1:8080 youruser@mcp-server.example.com
```

Log in and leave this window open too. You now have two windows that must stay open: the one running the pyATS MCP server and the tunnel.

Add the pyATS MCP server to Cursor's MCP settings in `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "pyats": {
      "url": "http://yourpyatsserveriporfqdn:8080/mcp"
    }
  }
}
```

{: file="~/.cursor/mcp.json" }

To test the tunnel, run this from another terminal on your laptop:

```bash
curl -i http://yourpyatsserverniporfqdn:8080/mcp
```

Surprisingly, the result you want is an error:

```text
HTTP/1.1 400 Bad Request
```

The response body says `Missing session ID`. That means the pyATS MCP server is running and rejected `curl` only because `curl` is not an MCP client. If you see `Connection refused` instead, either the tunnel window or the pyATS MCP server window has closed.

Finally, open Cursor Settings and go to the MCP section. While I was setting this up, Cursor showed a red circle and the error `ECONNREFUSED 127.0.0.1:8080`, even though `curl` worked when I ran it on the Linux server itself. The tunnel on my laptop was not open yet. There was no on and off switch. Once both sessions were up, I clicked the `pyats` MCP server and clicked Reload. That gave Cursor access to the pyATS MCP server on the Linux server, and it showed as connected.

## First test: a lab router

I started with a single lab router and simple questions. I typed them the same way I would ask a teammate who was already logged in:

- What are the BGP neighbors on this device?
- What are its CDP and LLDP neighbors?
- What prefixes are we receiving from this BGP neighbor?
- Are any interfaces showing increasing errors? Are any err-disabled or administratively down?
- What are my BGP paths to `0.0.0.0`? Which one is best, and why?

Next, I gave it the list of `show` commands we run before and after every upgrade and asked it to save the output as a "before" snapshot. Each command was saved as its own JSON file in a `before` folder on my laptop. After the change, I collected the same commands into an `after` folder. Then I tested the idea by asking questions about the saved files, such as "Diff these and tell me what is broken." That comparison does not log in to the device again. It only reads the files I had already collected, and it worked better than I expected.

### A note on pipes "|" to filter show commands

Our original command list included a few commands that needed cleanup before pyATS could use them:

- **Commands with a pipe (**`|`**).** We often trim a command's output down to a few columns. pyATS does not recognize the pipe and passes over it, so I removed the pipe and ran the plain command. The trimming is no longer needed. Everything in the output is stored as an attribute in the JSON file, so you can query the values you care about instead of filtering the text by hand.



## Second test: a wireless controller upgrade

The router proved that the idea worked. A real wireless controller upgrade proved that it was useful.

First, I added the controller to the testbed as a second device. It used the same login, so I only had to add another entry under `devices:`. Before the upgrade, I asked a warm-up question: "How many access points from one of our sites are connected to this controller?" It ran the matching command, filtered the results by site, and returned the count and the names. This verified the connectivity was working as expected.

Then I asked it to take a precheck by issuing our standard show commands and save the output so I could compare it after the upgrade. 

Each command's output was saved in a dated pre-upgrade folder. Commands that have a parser were saved as JSON. Commands without one, such as running config and mobility outputs, were saved as plain text.

After the upgrade, I wrote one sentence: "The upgrade is complete. Compare all the same show command outputs and tell me about any differences and anything I need to troubleshoot or fix."

It collected the same commands into a post-upgrade folder. Some commands timed out when I sent them all at once, so it collected the rest in smaller batches. Then it compared the two folders and reported the following in a great graphical easy to interpret view:

- **The upgrade succeeded.** Both chassis were running the new image, the install was committed, and the HA pair was back in SSO with the standby in a hot state.
- **Four access point did not rejoin.** The controller had 236 access points registered instead of 240. It also gave me that access point's model, last IP address, location, and the switch port it had used before the upgrade.
- **The path MTU to one mobility peer dropped from 1385 to 576.** The tunnel was still up, but large packets might be dropped on that path.

It also separated expected changes from real problems. Lower uptime, reset ARP ages, shorter CDP hold times, and smaller client counts all came from the reload, so it flagged them as expected instead of listing them as failures.

Finally, I asked for an executive summary. It created a one-page PDF with the upgrade result, the access point counts, the follow-up list, and a chart of the MTU change. I was able to share this with my team. I created something similar using fictious for this post. 

## What I would keep

A few choices from this first setup are worth keeping after I move it out of a home directory.

- Prompt for the username and password while you are still testing. There is no clear-text password to store, and there is no vault to set up yet.
- Copy and paste is fine while Cursor is walking you through the first setup. I normally edit files in Visual Studio Code, and I will go back to that once the setup is settled.
- After the pyATS MCP server window and the SSH tunnel are both open, click the `pyats` entry in Cursor's MCP settings and click Reload.
- Run the full show command, without a pipe. The parsed JSON already has every field, so you can ask for the values you care about.
- Save a before snapshot and an after snapshot, then ask what changed. That is the part I want to keep using for upgrades, after an outage, and when someone reports a problem at a site.

This is still a get-started setup in a user home directory on a Linux server. Once it proves its value, move it somewhere the team can share and support.