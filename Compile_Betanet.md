# Build a Betanet node for Debian, Ubuntu, or MacOS

## Notice

These directions are for a quick build of a betanet server. Nothing is done here to harden the security of the server. Since the tezos code is complex and communicating with potentially malicious peers, consider that anything on the server could be exposed or exploited.

This procedure is thoroughly tested on **Debian** 9.4.  It is reported to work on **Ubuntu** 18.04 and 16.04 as well; look for special notes below though.  It also works on **MacOS** 10.13.5 if you skip directly to the step for installing opam.

## Steps

Login to new Debian or Ubuntu system and update its base packages.
```
ssh root@192.155.xxx.xxx
apt-get update
apt-get upgrade -y
```

Create a user account for building and running tezos. The `tezos` is name is arbitrary; pick your favorite -- just don't build and run services as root.

```
adduser tezos
adduser tezos sudo
su - tezos
```

If you are on **Ubuntu 16**, do this to be able to install bubblewrap and latest version of git. Everyone else, ignore this.

```
sudo add-apt-repository ppa:ansible/bubblewrap
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
```

Similarly, if you are on **Debian 9**, do this to upgrade git to version 2.18.0. 

```
sudo apt-get -t stretch-backports install git
```

Install the system packages needed to start building tezos binaries.  The actual build scripts will install more packages.

```
sudo apt-get install -y patch unzip make gcc m4 git g++ aspcud bubblewrap curl
```

(If you're on **MacOS**, you can start here.)

Install OPAM utility needed to build the OCaml code. Version 2.0.0~rc3 of opam is required. See https://opam.ocaml.org/blog/opam-2-0-0-rc3/ for alternative installation steps.

### Opam installation method 1:

If asked, just accept the default of installing to /usr/local/bin.

```
sh <(curl -sL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh)
```

### Opam installation method 2:
```
wget https://github.com/ocaml/opam/releases/download/2.0.0-rc3/opam-2.0.0-rc3-x86_64-linux
sudo mv opam-2.0.0-rc3-x86_64-linux /usr/local/bin/opam
sudo chmod a+x /usr/local/bin/opam
```

Now that opam is installed, initialize it.  When the following runs, allow it to update your .profile and, if asked, also allow to "add a hook to opam's init scripts". The init step will take quite a while to complete -- might be a good time to call your Mom, or someone else who deserves it.

```
opam init
eval $(opam env)
```

Note that the `make build-deps` step below builds a local opam environment within the build directory, so we no longer need to set up a switch as we did before.

Get the betanet source code.
```
git clone -b betanet https://gitlab.com/tezos/tezos.git
cd tezos
```

Install OCaml dependencies (and some system package dependencies too). If prompted about installing the `depext` package, agree. If this fails, see the Workaround section below. An error message about "No repository tezos found" is normal when running this step for the first time; you can ignore that error. If you see an error like "Could not update repository "tezos": Commit found, but unreachable: enable uploadpack.allowReachableSHA1InWant on server" then you need to start over and install a newer version of git. See above about upgrading git in Ubuntu 16 and Debian 9.
```
make build-deps
```

Compile the binaries. Since the build-deps step above created an _opam directory with an opam switch for tezos, update the environment again before compiling to be sure we've got the right opam configuration.
```
eval $(opam env)
make
```

Configure the node identity. In the betanet the "difficulty" used in generating the node identity must be at least 26. That is the default now in the `identity generate` command.

```
./tezos-node identity generate 26.
```

Run the node. (I also like to run the node inside a `screen(1)` session (without `nohup` or `&`) so that the process persists in the foreground and I can detach and come back to it later in another ssh session.)\

```
nohup ./tezos-node run --rpc-addr 127.0.0.1:8732 --connections 10 &
```

The node will take a while to sync, bringing in the blockchain data from the peers. To see its progress you can run the following command which shows the head block known so far:

```
./tezos-client rpc get /chains/main/blocks/head
```

Look for the `timestamp` value in the output from that. When that value gets to within a minute or so of the current date and time then your node is synced. The expected network/chain_id value is `NetXdQprcVkpaWU`.

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

## Baking in betanet

[TBD: this section is incomplete]

See http://doc.tzalpha.net/introduction/zeronet.html.

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

If you encounter an error about the opam depext plugin not being installed, you can install it manually and try again:

```
opam install depext
```

If you get 0 connections, verify that your node identity was built with sufficient difficulty:

```
./tezos-node identity check
```

## Rebuilding

To rebuild with the latest beta code you can move or remove the "tezos" directory and start again from the `git clone` step. As an alternative you can update in place as follows.

```
git fetch
git reset --hard origin/betanet
git clean -dxf
eval $(opam env)
make build-deps
make
```

## References

Baking info: https://gist.github.com/dakk/bdf6efe42ae920acc660b20080a506dd 

Docker images: https://hub.docker.com/r/tezos/tezos/tags/

Docker script: https://gitlab.com/tezos/tezos/raw/betanet/scripts/alphanet.sh
Copy to betanet.sh and run as usual.

Korean version : https://blog.naver.com/justmustone/221310459511 

Korean version : Raspberry Pi 3 https://blog.naver.com/justmustone/221310583691 

## About
Written by @fredcy with help from the [tech chat room](https://riot.im/app/#/room/#freenode_#tezos:matrix.org) and others.
