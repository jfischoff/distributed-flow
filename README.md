# About

`distributive-flow`, or `dflow` for short, is a simple method for zero-downtime deployment of `Haskell` applications.

`dflow` manages multiple versions of the same `Target`'s `Process`es on a clusters of `Node`s.

`dflow` reads the cabal file, and by convention looks for an executable named the same as the library. The `dflow` backends can deploy one or more of a executable, a library and source. 

```haskell
type Target = OneOrMoreOf Executable Library Source
```

# Commands
### Common Flags
- `--image` e.g. `ubuntu`. `Image` is an opaque handle. It cannot be inspected. It is passed to the `VMBackend`.
- `--vm-backend` e.g. `vagrant`. `VMBackend` provides the function `Image -> Keys -> Node`
- `--container-backend` e.g.`docker`.
  `ContainerBackend` provides the interface 
- `--builder` Build the `Target` essentially `TargetDesc -> Target` where
```haskell
type Target     = OneOrMoreOf Executable     Library     Source
type TargetDesc = OneOrMoreOf ExecutableDesc LibraryDesc SourceDesc
```
- `--package-reader` Parse a `TargetDesc`

```haskell 
class ContainerBackend m where
  type Container m  
  type Process m
  containerize :: Target -> m (Container m)
  
  start :: Container m -> Node -> m (Process m)
  stop  :: Process m -> m ()
```

- `--store` e.g.`etcd`. `Store`s implement
```haskell
class MStore m where
    type StoreHandle m
    load   :: m (StoreHandle m)
    save   :: StoreHandle m -> m ()

    add    :: Node         -> m Int
    remove :: Int  -> Node -> m ()
```
- `--keys` authorized_keys file. `Keys` are opaque but used by the
`VMBackend`.

### Provisioning
- `set COUNT` where `COUNT` is the number of `Node`s.
- `get` the `COUNT`
- `cycle Index` recreate the `Node` at `Index`.
- `list` will list all the `Node`s and their `Index`.
- `destroy Index` remove a `Node`.
- `chaos-monkey` randomly destroy `Node`s.
 - `--rampage Interval` continuously destroy `Node`s at the given `Interval`.

### Management

All commands can take the following flags
##### Common flags
- `--index`  is the `Index` of a `Node`.

#### Process

- `start` the `Executable` listed in the *dflow.yaml* file.
- `stop`  the current version of `Executable`.
  - `--all` stop every version of the `Executable`.
  - `--now` as in immediately.
- `restart` the service by `stop`ping and `start`ing.
- `versions` get the `git` hashes of the `Executable` processes

#### Health

- `compare-checksums` compare the checksums of the current version of the `Executable` with what is on the nodes.
- `health` Show statistics of the `health` portion of `verify`'s result.
- `repair` Attempt to bring the cluster to a healthy state by destroying the `Node`s that fail `verify`, and recopying the `Executable` and `restart`ing. Additionally repair will create `Node`s that are missing.
 - `--watch` continuously watch the cluster and repair it if something is wrong.
- `verify` a `Node` using the source file specified in the `dflow.yaml`. `verify` relies on a function that produces a health value and a threshold. `verify :: Node -> ({- health -} Double, {- threshold -} Double)`
  - `--watch` continuously monitor the cluster.

#### Deployment

- `build` the `Executable`. This should include any tests.
- `copy` the `Executable` to the `Node`s
 - `--version` Specify a version. Default is the latest.
 - `--latest` copy the latest.
 - `--through-cache` Force the value through the cache and update on the `Node`s.
- `deploy` e.g. `build`, `copy`, and `start` the new version. `verify` the new version and then tell the other versions to `shutdown`.
 - `--watch` deploy when files change.
 - `--commit-hash` deploy a specific `git` hash.
 - `--rollout PERCENT` Deploys to PERCENT of the cluster.
 - `--rolling-timed` time for a rolling deploy.
 - `--auto-rollout Tolerance` rollback automatically if the health decrease more than `Tolerance`.
- `rollback` revert the last deploy and start the old version. This can be called multiple times to go further back in history.
  - `--version` pass in the `git` hash.

#### Other

- `gc` collect old versions
 - `count` keep this number
 - `older` a relative or absolute date.
