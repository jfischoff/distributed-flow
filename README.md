# About

`distributive-flow` or `dflow` for short is simple method for zero-downtime deployment of socket based `Haskell` applications.

`dflow` is manages multiple versions of the same executable on a clusters of machines.

`dflow` is configured with an `Image`, `[PublicKey]`. Additionally it is parameterized over a data store, vm provider and containerizer backends.

`dflow` maintains a store of an `[Address]` and `(Map String Address)` or `Node`s is deploy to.

`dflow` is "agent less" in the sense it relies on `sshd`. `dflow` assumes client applications respond to the following signals

- `SIGINT`  informs the `Executable` to begin graceful shutdown.
- `SIGTERM`  informs the `Executable` to shutdown immediantly.

# TODO Process watcher
How does `dflow` handle registering a new process with the process watcher?

# Commands

### Provisioning
`dflow` can provision a `Node` for an `Executable` for a given backend using an Image. It cannot install any software or dependencies.

Container plugins handle dependency management and can
```haskell
containerize :: Executable -> Container
```

- ###### Common Flags

 - `--credentials` IaaS credentials
 - `--vm-backend` Vagrant
 - `--container-backend` default is Docker
 - `--store-backend` default is SQLite.
 - `--authorized-users` authorized_keys file.
- `add` the `Address` to the stored list of addresses.
 - `name` the `Address` to `add`.
- `remove` or `rm` the `Address` from the stored list of addresses.
 - `name` the `Address` to `rm`.
- `list` or `ls`
 - `nodes` the unnamed `[Address]` and  the named `Map String Address`
  - `unnamed` are only shown in the order they were added
  - `name` are only shown in the order they were added
 - `versions` the versions of the `Executable`

##### Context
- `with` push the `Address` on top of the context stack. The context defaults to all `Node`s

##### Executable Management

All commands can take the following flags
- Common flags
 - `--address`  is the `Address` or `Name` of a `Node`.
- `start` the `Executable` list in the *df.yaml* file.
- `stop`  the current version of `Executable`.
  - `--all` stop every version of the `Executable`.
  - `--now` as in immediantly.
- `restart` the service by `stop`ping and `start`ing.
- `compare-checksums` compare the checksums of the current version of the `Executable` with what is on the nodes.
- `verify` the current version using the verification target in the *df.yaml* file.
- `cp` the Executable the `Node`s of given by the `Context`
 - `--full` A full copy, not the optimized diff.
 - `--version` Specify a version. Default is the latest.
 - `--through-cache` Force the value through the cache and update on the `Node`.
- `build` the executable.
- `deploy` e.g. `build`, `copy`, and `start` the new version. `verify` the new
  version tell the other versions to `shutdown`
 - `--commit-hash` deploy a specific `git` hash.
 - `--rollout PERCENT` Deploys to PERCENTAGE of the cluster
- `rollback` revert the last deploy and start the old version.
- `repair` Attempt to bring the cluster to a healthy state by killing the `Node`s that fail verification, and recopying the `Executable` and `restart`ing
 - `--full-copy` Do a full copy of the executable.
 - `--watch`
- `gc` collect old versions
 - `count` keep this number
 - `older` a relative or absolute date.
