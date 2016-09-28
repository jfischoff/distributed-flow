# About

`distributive-flow`, or `dflow` for short, is a simple method for zero-downtime deployment of socket based `Haskell` applications.

`dflow` manages multiple versions of the same `Executable` on a clusters of `Node`s.

`dflow` is parameterized over the following variables.
- `Image` is an opaque handle. It cannot be inspected. It is passed to the `VMBackend`.
- `PublicKeys` is also an opaque.
- `VMBackend` provides the function `Image -> Node`
- `ContainerBackend` provides the `Executable -> Container`
- `Store` is define by the interface

`dflow` is "agent less" in the sense it relies on `sshd`. `dflow` assumes client applications respond to the following signals

- `SIGINT`  informs the `Executable` to begin graceful shutdown.
- `SIGTERM`  informs the `Executable` to shutdown immediantly.

# TODO Process watcher
How does `dflow` handle registering a new process with the process watcher?

# Commands

### Common Flags
- `--vm-backend` Vagrant
- `--container-backend` default is Docker
- `--store-backend` default is a json file.
- `--keys` authorized_keys file.

### Provisioning
- `add NAME` provision a `Node` named `NAME`.
- `remove NAME` the `Node` named `NAME`.
- `cycle NAME` recreate the `NAME` `Node`.
- `list` will list all the `Node`s.
- `chaos-monkey` randomly destroy `Node`s.

##### Executable Management

All commands can take the following flags
- Common flags
 - `--name`  is the name of a `Node`.
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
