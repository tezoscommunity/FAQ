# Build a Mainnet node for Debian, Ubuntu, or MacOS

## Notice

These directions are for a quick build of a mainnet server. Nothing is done here to harden the security of the server. Since the tezos code is complex and communicating with potentially malicious peers, consider that anything on the server could be exposed or exploited.

This procedure is thoroughly tested on **Debian** 9.4.  It is reported to work on **Ubuntu** 18.04 and 16.04 as well; look for special notes below though.  It also works on **MacOS** 10.13.5 if you skip directly to the step for installing opam.

These steps are reported to work on **Raspberry Pi** 3B and 3B+ running Ubuntu 18.04 as well. An external hard drive is required.

### Changelog

+ 2020-01-13:  Make Opam installation directions more general, allowing for Opam 2.0.5
+ 2018-08-31:  Added libhidapi-dev as a system package dependency.

## Steps

Login to new Debian or Ubuntu system and update its base packages.  (In the following, replace "192.155.xxx.xxx" with the actual IP address of your server).

```
ssh root@192.155.xxx.xxx
apt-get update
apt-get dist-upgrade -y
```

Create a user account for building and running tezos. The `tezos` is name is arbitrary; pick your favorite -- just don't build and run services as root. If you already have a user account you can use that instead. Note that the `make build-deps` step requires sudo rights when it first runs, to install some system packages via apt-get.

```
adduser tezos
adduser tezos sudo
reboot
ssh tezos@192.155.xxx.xxx
```

If you are on **Ubuntu 16**, do this to be able to install bubblewrap and latest version of git. Everyone else, ignore this.

```
sudo add-apt-repository ppa:ansible/bubblewrap
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
```

Similarly, if you are on **Debian 9**, do this to upgrade git to version 2.18.0.  You need the debian "stretch-backports" source enabled to do this. If the following command fails, see the "Add backports to your sources.list" section at https://backports.debian.org/Instructions/ and follow the steps there; then try this again.

```
sudo apt-get -t stretch-backports install git
```

If you are on **Centos 7.5** some of the system package names are different. Use this instead of the `apt-get` just below:

```
sudo yum install epel-release
sudo yum install bzip2 screen wget rsync gcc m4 make unzip patch libev-devel libev hidapi-devel gmp-devel bubblewrap git
```

If you are on **Arch Linux/Manjaro** the above applies with `pacman`:

```
sudo pacman -Syu
sudo pacman -S patch unzip make gcc m4 git g++ aspcud bubblewrap curl bzip2 rsync libevdev gmp pkgconf hidapi
```


Install the system packages needed to start building tezos binaries.  The actual build scripts will install more packages.

```
sudo apt-get install -y patch unzip make gcc m4 git g++ aspcud bubblewrap curl bzip2 rsync libev-dev libgmp-dev pkg-config libhidapi-dev
```

(If you're on **MacOS**, you can start here.)

Install OPAM utility needed to build the OCaml code. Version 2.0.0 or later of opam is required.
See https://opam.ocaml.org/doc/Install.html for alternative installation steps.

### Opam installation:

If asked, just accept the default of installing to /usr/local/bin.
Installing there depends on the 'tezos' user having sudo rights as we arranged above.
 ```
sh <(curl -sL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh)
```

### Opam installation, alternative manual method for MacOS and other:

Visit https://github.com/ocaml/opam/releases/.
Find the newest stable 2.0.x release (2.0.5 works at the time of writing, January 2020) and download the binary executable for your architecture.
Move that executable file to /usr/local/bin/opam and use `chmod a+x` to make it executable.

### Opam setup

Now that opam is installed, initialize it. (Allow it to update .profile, etc, at your own discretion).

```
opam init --bare
eval $(opam env)
```

If you are running in the **Windows 10 Linux Subsystem** you may need to add `--disable-sandboxing` to the call to `opam init` above. Otherwise you may be blocked by `brwap` errors as bubblewrap does not currently work in that environment.

Note that the `make build-deps` step below builds a local opam environment within the build directory, so we no longer need to set up a switch as we did before.

Get the mainnet source code.
```
git clone -b mainnet https://gitlab.com/tezos/tezos.git
cd tezos
```

Install OCaml dependencies (and some system package dependencies too). An error message about "No repository tezos found" is normal when running this step for the first time; you can ignore that error. If you see an error like "Could not update repository "tezos": Commit found, but unreachable: enable uploadpack.allowReachableSHA1InWant on server" then you need to start over and install a newer version of git. See above about upgrading git in Ubuntu 16 and Debian 9.
```
make build-deps
```

Compile the binaries. Since the build-deps step above created an \_opam directory with an opam switch for tezos, update the environment again before compiling to be sure we've got the right opam configuration.
```
eval $(opam env)
make
```

Configure the node identity. In the mainnet the "difficulty" used in generating the node identity must be at least 26. That is the default now in the `identity generate` command.

```
./tezos-node identity generate 26.
```

Run the node. (I also like to run the node inside a `screen(1)` session (without `nohup` or `&`) so that the process persists in the foreground and I can detach and come back to it later in another ssh session.)\

```
nohup ./tezos-node run --rpc-addr 127.0.0.1:8732 &
```

The node will take a while to sync, bringing in the blockchain data from the peers. To see its progress you can run the following command which shows the head block known so far and will exit when the node is fully synced:

```
./tezos-client bootstrapped
```

Look for the `timestamp` value in the output from that. When that value gets to within a minute or so of the current date and time then your node is synced. The expected network/chain_id value is `NetXdQprcVkpaWU`.

### Faster bootstrap

It's possible to start up your node with a copy of the chain data that is (typically) just a few days old.
See the directions at https://tezosshots.com/.

## To activate a donation account

```
./tezos-client add address my_account <public key hash from donation PDF>
./tezos-client activate fundraiser account my_account with <activation key from verification site>
```

The `my_account` name/alias above is arbitrary, but conventional.

Note: you can also import your private key info, but that is not necessary in order to claim the donation account: 

```
./tezos-client import fundraiser secret key my_account
```

Also: if your goal is only to activate your Tezos account and claim the tez from the fundraiser, it’s much easier to just use stephenandrews online utility at https://stephenandrews.github.io/activatez/ .

## Baking on Mainnet

See [the Tezos baking howto](https://gist.github.com/dakk/bdf6efe42ae920acc660b20080a506dd) by @dakk.

## Workarounds when things won't build or run

In the top code directory, ~/tezos, run `eval $(opam env)`. Then try again.

After doing `eval $(opam env)`, do `opam update && opam upgrade`. Then try again.

Sometimes it helps to remove the entire ~/.opam directory and start over again from the `opam init` step. Slow, but it gives a clean start for OPAM packages. Note that if you are building alphanet or other from the same user, this also wipes out the OPAM context for that work too.

```
rm -rf ~/.opam
```

The `make build-deps` step pins many packages to particular versions. If you want a clean start short of removing ~/.opam, you can remove all those pins:

```
opam pin list -s | xargs opam pin remove
```

If you get 0 connections, verify that your node identity was built with sufficient difficulty:

```
./tezos-node identity check
```

### Fix issue in macOS
On macOS `make build-deps` sometimes fails since it cannot build OCaml compiler. You may get the following error
```bash
<><> Gathering sources ><><><><><><><><><><><><><><><><><><><><><><><><><><>  🐫
[ocaml-base-compiler.4.08.1] found in cache

<><> Processing actions <><><><><><><><><><><><><><><><><><><><><><><><><><>  🐫
[ERROR] The compilation of ocaml-base-compiler failed at
        "/Users/ameten/.opam/opam-init/hooks/sandbox.sh build ./configure
        --prefix=/Users/ameten/.opam/4.08.1 CC=cc ASPP=cc -c".
∗ installed base-bigarray.base
∗ installed base-threads.base
∗ installed base-unix.base

#=== ERROR while compiling ocaml-base-compiler.4.08.1 =========================#
# context     2.0.5 | macos/x86_64 |  | https://opam.ocaml.org#b679cc38
# path        ~/.opam/4.08.1/.opam-switch/build/ocaml-base-compiler.4.08.1
# command     ~/.opam/opam-init/hooks/sandbox.sh build ./configure --prefix=/Users/ameten/.opam/4.08.1 CC=cc ASPP=cc -c
# exit-code   65
# env-file    ~/.opam/log/ocaml-base-compiler-5420-ffb3fd.env
# output-file ~/.opam/log/ocaml-base-compiler-5420-ffb3fd.out
### output ###
# /Users/ameten/.opam/opam-init/hooks/sandbox.sh: line 9: cd: /Users/ameten/.ccache: No such file or directory
# sandbox-exec: empty subpath pattern



<><> Error report <><><><><><><><><><><><><><><><><><><><><><><><><><><><><>  🐫
┌─ The following actions failed
│ λ build ocaml-base-compiler 4.08.1
└─
┌─ The following changes have been performed (the rest was aborted)
│ ∗ install base-bigarray base
│ ∗ install base-threads  base
│ ∗ install base-unix     base
└─
# Run eval $(opam env) to update the current shell environment
Switch initialisation failed: clean up? ('n' will leave the switch partially
installed) [Y/n] Y
```     
Sandbox executable complains that there is not place to store compiler cache. You don't have `/Users/username/.ccache` on your system.
Install `ccache` using your favourite package manager. Command for Homebrew:
```bash
brew install ccache
```
If `ccache` is installed, but you don't have `/Users/username/.ccache`, run the following command to create the directory:
```bash
ccache -C
```
When you have `/Users/username/.ccache` you should be able to install OCaml compiler using `opam`.
```bash
opam switch create 4.08.1
```
Once you have OCaml compiler in `opam`, you should be able to run `make build-deps` although it will install its own OCaml compiler.

## Rebuilding

To rebuild with the latest Mainnet code you can move or remove the "tezos" directory and start again from the `git clone` step. As an alternative you can update in place as follows.

```
git fetch
git reset --hard origin/mainnet
git clean -dxf
eval $(opam env)
make build-deps
make
```

Details here:https://github.com/Blindripper/FAQ/blob/patch-1/Rebuilding_Mainnet.md

## References

Baking info: https://gist.github.com/dakk/bdf6efe42ae920acc660b20080a506dd 

Docker images: https://hub.docker.com/r/tezos/tezos/tags/

Docker script: https://gitlab.com/tezos/tezos/blob/mainnet/docs/introduction/howtoget.rst
Copy to mainnet.sh and run as usual.

Korean version : https://blog.naver.com/justmustone/221310459511 

Korean version : Raspberry Pi 3 https://blog.naver.com/justmustone/221310583691 

## About
Written by @fredcy with help from the [tech chat room](https://riot.im/app/#/room/#freenode_#tezos:matrix.org) and others.
