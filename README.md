# About

`distributive-flow`, or `dflow` for short, is a simple method for zero-downtime deployment of `Haskell` applications.

`dflow` manages multiple versions of the same `Executable` on a clusters of `Node`s.

`dflow` is parameterized with following options.
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
- `set COUNT` where `COUNT` is the number of `Node`.
- `get` the `COUNT`
- `cycle Index` recreate the `Index` `Node`.
- `list` will list all the `Node`s and their `Index`.
- `chaos-monkey` randomly destroy `Node`s.

### Executable Management

All commands can take the following flags
- Common flags
 - `--index`  is the `Index` of a `Node`.
- `start` the `Executable` list in the *dflow.yaml* file.
- `stop`  the current version of `Executable`.
  - `--all` stop every version of the `Executable`.
  - `--now` as in immediantly.
- `restart` the service by `stop`ping and `start`ing.
- `compare-checksums` compare the checksums of the current version of the `Executable` with what is on the nodes.
- `verify` the current version using the verification target in the *dflow.yaml* file.
- `copy` the Executable the `Node`s
 - `--version` Specify a version. Default is the latest.
 - `--through-cache` Force the value through the cache and update on the `Node`.
- `build` the executable.
- `deploy` e.g. `build`, `copy`, and `start` the new version. `verify` the new
  version tell the other versions to `shutdown`
 - `--commit-hash` deploy a specific `git` hash.
 - `--rollout PERCENT` Deploys to PERCENTAGE of the cluster
- `rollback` revert the last deploy and start the old version.
- `repair` Attempt to bring the cluster to a healthy state by killing the `Node`s that fail verification, and recopying the `Executable` and `restart`ing. Additionally repair will create `Node`s that are missing.
 - `--watch` continuously watch the cluster and repair it if something is wrong.
- `gc` collect old versions
 - `count` keep this number
 - `older` a relative or absolute date.
