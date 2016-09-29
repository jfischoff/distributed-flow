# About

`distributive-flow`, or `dflow` for short, is a simple method for zero-downtime deployment of `Haskell` applications.

`dflow` manages multiple versions of the same `Executable` on a clusters of `Node`s.

`dflow` is parameterized with following options.
- `Image` is an opaque handle. It cannot be inspected. It is passed to the `VMBackend`.
- `PublicKeys` is also an opaque.
- `VMBackend` provides the function `Image -> Node`
- `ContainerBackend` provides the `Executable -> Container`
- `Store` which implements this interface
```haskell
class IStore m where
  load   :: m Store
  save   :: Store -> m ()

  add    :: Node         -> m Int
  remove :: Int  -> Node -> m ()
```

# Commands

### Common Flags
- `--vm-backend` Vagrant
- `--container-backend` default is `nspawn`
- `--store-backend` default is a json file.
- `--keys` authorized_keys file.

### Provisioning
- `set COUNT` where `COUNT` is the number of `Node`s.
- `get` the `COUNT`
- `cycle Index` recreate the `Index` `Node`.
- `list` will list all the `Node`s and their `Index`.
- `outplace Index` remove a `Node`.
- `chaos-monkey` randomly destroy `Node`s.
 - `--rampage` continuously release the `chaos-monkey`

### Management

All commands can take the following flags
##### Common flags
- `--index`  is the `Index` of a `Node`.

#### Process

- `start` the `Executable` list in the *dflow.yaml* file.
- `stop`  the current version of `Executable`.
  - `--all` stop every version of the `Executable`.
  - `--now` as in immediantly.
- `restart` the service by `stop`ping and `start`ing.
- `versions` get the `git` hashes of the `Executable` processes

#### Health

- `compare-checksums` compare the checksums of the current version of the `Executable` with what is on the nodes.
- `health` Show statistics of the `health` portion of `verify`'s result.
- `repair` Attempt to bring the cluster to a healthy state by destroying the `Node`s that fail `verify`, and recopying the `Executable` and `restart`ing. Additionally repair will create `Node`s that are missing.
 - `--watch` continuously watch the cluster and repair it if something is wrong.
- `verify` the a `Node` using the source file in the `dflow.yaml`. `verify` relies on a function that produces a health value and a threshold.
```haskell
verify :: Node -> ({- health -} Double, {- threshold -} Double)
```
  - `--watch` continuously monitor the cluster.

#### Deployment

- `build` the `Executable`.
- `copy` the `Executable` to the `Node`s
 - `--version` Specify a version. Default is the latest.
 - `--latest` copy the latest.
 - `--through-cache` Force the value through the cache and update on the `Node`s.
- `deploy` e.g. `build`, `copy`, and `start` the new version. `verify` the new version and then tell the other versions to `shutdown`.
 - `--commit-hash` deploy a specific `git` hash.
 - `--rollout PERCENT` Deploys to PERCENT of the cluster.
- `rollback` revert the last deploy and start the old version. Calling rollback twice goes back two versions(should this require a flag?).
  - `--version` pass in the `git` hash.

#### Other

- `gc` collect old versions
 - `count` keep this number
 - `older` a relative or absolute date.
