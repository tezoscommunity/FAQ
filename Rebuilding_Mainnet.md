To upgrade a Tezos Mainnet server to newly released code it is possible to build from scratch as always.

But to save time it is also possible to upgrade in place. Here is what I did:

# Re-compiling Mainnet without a full rebuild

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

Get the new code and build it. This assumes that we are on the 'mainnet' branch already.  Do `git checkout mainnet` first if not.

```
git fetch
git pull
make
```

# Notes

Several old binaries are gone:  tezos-alpha-baker, tezos-alpha-endoser, and tezos-alpha-accuser. Each now has a pair of replacements, one for the first protocol and one for the new second protocol: tezos-accuser-001-PtCJ7pwo, tezos-baker-001-PtCJ7pwo, tezos-endorser-001-PtCJ7pwo, tezos-accuser-002-PsYLVpVv, tezos-baker-002-PsYLVpVv, tezos-endorser-002-PsYLVpVv.

The new binaries make use of the same node and client state data as before, in ~/.tezos-node and ~/.tezos-client (by default).
