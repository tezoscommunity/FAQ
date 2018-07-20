To upgrade a Tezos betanet server to newly released code it is possible to build from scratch as always.

But to save time it is also possible to upgrade in place. Here is what I did:

Copy the entire Tezos build tree to a new place. That way we leave the current one around as a backup.
Also, the currently running binaries are not disturbed. This assumes that `~/tezos` is the current build directory.
You could ignore this and work right in the original build directory if you don't care about such a backup.

```
cd
rsync -av tezos/ tezos2
```

Move to the new build directory where we'll do the rebuild.

```
cd tezos2
eval $(opam env)
```

Clean up a bit, removing much of the code that we built before but leaving the opam info.

```
make clean
git clean -f
```

Get the new code and build it. This assumes that we are on the 'betanet' branch already.  Do `git checkout betanet` first if not.

```
git fetch
git pull
make
```
