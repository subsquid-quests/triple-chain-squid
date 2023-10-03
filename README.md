<p align="center">
<picture>
    <source srcset="https://uploads-ssl.webflow.com/63b5a9958fccedcf67d716ac/64662df3a5a568fd99e3600c_Squid_Pose_1_White-transparent-slim%201.png" media="(prefers-color-scheme: dark)">
    <img src="https://uploads-ssl.webflow.com/63b5a9958fccedcf67d716ac/64662df3a5a568fd99e3600c_Squid_Pose_1_White-transparent-slim%201.png" alt="Subsquid Logo">
</picture>
</p>

[![docs.rs](https://docs.rs/leptos/badge.svg)](https://docs.subsquid.io/)
[![Discord](https://img.shields.io/discord/1031524867910148188?color=%237289DA&label=discord)](https://discord.gg/subsquid)

[Website](https://subsquid.io) | [Docs](https://docs.subsquid.io/) | [Discord](https://discord.gg/subsquid)

[Subsquid Network FAQ](https://docs.subsquid.io/subsquid-network/)

# Deploy a triple processor squid

This is a quest to run a squid with three processors. Here is how to run it:

### I. Install dependencies: Node.js, Docker, Git.

<details>
<summary>On Windows</summary>

1. Enable [Hyper-V](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v).
2. Install [Docker for Windows](https://docs.docker.com/desktop/install/windows-install/).
3. Install NodeJS LTS using the [official installer](https://nodejs.org/en/download).
4. Install [Git for Windows](https://git-scm.com/download/win).

In all installs it is OK to leave all the options at their default values. You will need a terminal to complete this tutorial - [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) bash is the preferred option.

</details>
<details>
<summary>On Mac</summary>

1. Install [Docker for Mac](https://docs.docker.com/desktop/install/mac-install/).
2. Install Git using the [installer](https://sourceforge.net/projects/git-osx-installer/) or by [other means](https://git-scm.com/download/mac).
3. Install NodeJS LTS using the [official installer](https://nodejs.org/en/download).

We recommend configuring NodeJS to install global packages to a folder owned by an unprivileged account. Create the folder by running
```bash
mkdir ~/global-node-packages
```
then configure NodeJS to use it
```bash
npm config set prefix ~/global-node-packages
```
Make sure that the folder `~/global-node-packages/bin` is in `PATH`. That allows running globally installed NodeJS executables from any terminal. Here is a one-liner that detects your shell and takes care of setting `PATH`:
```
CURSHELL=`ps -hp $$ | awk '{print $5}'`; case `basename $CURSHELL` in 'bash') DEST="$HOME/.bash_profile";; 'zsh') DEST="$HOME/.zshenv";; esac; echo 'export PATH="${HOME}/global-node-packages/bin:$PATH"' >> "$DEST"
```
Alternatively you can add the following line to `~/.zshenv` (if you are using zsh) or `~/.bash_profile` (if you are using bash) manually:
```
export PATH="${HOME}/global-node-packages/bin:$PATH"
```

Re-open the terminal to apply the changes.

</details>
<details>
<summary>On Linux</summary>

Install [NodeJS (v16 or newer)](https://nodejs.org/en/download/package-manager), Git and Docker using your distro's package manager.

We recommend configuring NodeJS to install global packages to a folder owned by an unprivileged account. Create the folder by running
```bash
mkdir ~/global-node-packages
```
then configure NodeJS to use it
```bash
npm config set prefix ~/global-node-packages
```
Make sure that any executables globally installed by NodeJS are in `PATH`. That allows running them from any terminal. Open the `~/.bashrc` file in a text editor and add the following line at the end:
```
export PATH="${HOME}/global-node-packages/bin:$PATH"
```
Re-open the terminal to apply the changes.

</details>

### II. Install Subsquid CLI

Open a terminal and run
```bash
npm install --global @subsquid/cli@latest
```
This adds the [`sqd` command](/squid-cli). Verify that the installation was successful by running
```bash
sqd --version
```
A healthy response should look similar to
```
@subsquid/cli/2.5.0 linux-x64 node-v20.5.1
```

### III. Run the squid

1. Open a terminal and run the following commands to create the squid and enter its folder:
   ```bash
   sqd init my-triple-proc-squid -t https://github.com/subsquid-quests/triple-chain-squid
   ```
   ```bash
   cd my-triple-proc-squid
   ```
   You can replace `my-triple-proc-squid` with any name you choose for your squid. If a squid with that name already exists in [Aquarium](https://docs.subsquid.io/deploy-squid/), the first command will throw an error; if that happens simply think of another name and repeat the commands.

2. Press "Get Key" button in the quest card to obtain the `tripleProc.key` key file. Save it to the `./query-gateway/keys` subfolder of the squid folder. The file will be used by the query gateway container.

3. The template squid uses a PostgreSQL database and a query gateway. Start Docker containers that run these with
   ```bash
   sqd up
   ```
   Wait for about a minute before proceeding to the next step.

   If you get an error message about `unknown shorthand flag: 'd' in -d`, that means that you're using an old version of `docker` that does not support the `compose` command yet. Update Docker or edit the `commands.json` file as follows:
   ```diff
            "up": {
            "deps": ["check-key"],
            "description": "Start a PG database",
   -        "cmd": ["docker", "compose", "up", "-d"]
   +        "cmd": ["docker-compose", "up", "-d"]
          },
          "down": {
            "description": "Drop a PG database",
   -        "cmd": ["docker", "compose", "down"]
   +        "cmd": ["docker-compose", "down"]
          },
   ```

4. Prepare the squid for running by installing dependencies, building the source code and creating all the necessary database tables:
   ```bash
   npm ci
   sqd build
   sqd migration:apply
   ```
5. Start your squid with
   ```bash
   sqd run .
   ```
   The command should output lines like these:
   ```
   [eth-processor] 22:54:19 INFO  sqd:processor processing blocks from 16000000
   [base-processor] 22:54:19 INFO  sqd:processor processing blocks from 3800000
   [bsc-processor] 22:54:19 INFO  sqd:processor processing blocks from 28000000
   [bsc-processor] 22:54:19 INFO  sqd:processor using archive data source
   [base-processor] 22:54:19 INFO  sqd:processor using archive data source
   [bsc-processor] 22:54:19 INFO  sqd:processor prometheus metrics are served at port 44341
   [base-processor] 22:54:19 INFO  sqd:processor prometheus metrics are served at port 43297
   [eth-processor] 22:54:19 INFO  sqd:processor using archive data source
   [eth-processor] 22:54:19 INFO  sqd:processor prometheus metrics are served at port 38937
   [api] 22:54:20 WARN  sqd:graphql-server enabling dumb in-memory cache (size: 100mb, ttl: 1000ms, max-age: 1000ms)
   [api] 22:54:20 INFO  sqd:graphql-server listening on port 4350
   [eth-processor] 22:54:23 INFO  sqd:processor 16005819 / 18248806, rate: 1631 blocks/sec, mapping: 656 blocks/sec, 1246 items/sec, eta: 23m
   [eth-processor] 22:54:28 INFO  sqd:processor 16011899 / 18248806, rate: 2210 blocks/sec, mapping: 673 blocks/sec, 1307 items/sec, eta: 17m
   [bsc-processor] 22:55:27 INFO  sqd:processor 28007279 / 32195252, rate: 108 blocks/sec, mapping: 570 blocks/sec, 1101 items/sec, eta: 10h 50m
   [bsc-processor] 22:55:32 INFO  sqd:processor 28011319 / 32195252, rate: 161 blocks/sec, mapping: 604 blocks/sec, 1170 items/sec, eta: 7h 14m
   ```
   The squid should sync in 30-35 minutes. When it's done, stop it with Ctrl-C, then stop and remove the auxiliary containers with
   ```bash
   sqd down
   ```

# Quest Info

| Category         | Skill Level                          | Time required (minutes) | Max Participants | Reward                              | Status |
| ---------------- | ------------------------------------ | ----------------------- | ---------------- | ----------------------------------- | ------ |
| Squid Deployment | $\textcolor{green}{\textsf{Simple}}$ | ~40                     | -                | $\textcolor{red}{\textsf{750tSQD}}$ | open   |

# Acceptance critera

Sync this squid using the key from the quest card. The syncing progress is tracked by the amount of data the squid has retrieved from [Subsquid Network](https://docs.subsquid.io/subsquid-network).

# About this squid

This [squid](https://docs.subsquid.io/) captures USDC Transfer events on ETH, BSC and Base, stores them in the same database and serves the data over a common GraphQL API.

Data ingester ("processor") code for each network is located at the corresponding `src/` subdirectory: `src/eth`, `src/bsc` or `src/base`. The scripts file `commands.json` contains commands for running each processor (`process:eth`, `process:bsc` and `process:base` correspondingly). GraphQL server runs as a separate process started by `sqd serve`. You can also use `sqd run` to run all the services at once.

The squid uses [Subsquid Network](https://docs.subsquid.io/subsquid-network) as its primary data source.
