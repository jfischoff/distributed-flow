# About

`distributive-flow`, or `dflow` for short, is a simple method for zero-downtime deployment of `Haskell` applications.

`dflow` manages multiple versions of the same `Target`'s `Process`es on a clusters of `Node`s.

`dflow` reads a package file and produces a `Target`. The `dflow` backends can deploy one or more of a executable, a library and source ... or something else. It is an opaque type from `dflow`'s perspective.

# Commands

### Plugins

Plugins are installed automatically when listed in config or set on the command
line. They can also be `install`ed explicitly and `uninstalled`
 - `plugin` this is the top level plugin command. The subcommands do the work.
   - `install Identifier`
   - `uninstall Identifier`
   - `list PATTERN`
     - `--vm` Filter to just `VMPlugin`s.
     - `--container` Filter to just `ContainerPlugin`s.
     - `--builder` Filter to just builder plugins.
     - `--package` Filter to just package plugins.
     - `--store` Filter to just `Store` plugins.

### Images
  - `image`
    - `list PATTERN` list `Image`s. This functionality is provided by
    the `VMPlugin`'s `listImages` method.

### Common Flags

- `--image PATH_OR_ID` e.g. `ubuntu-trusty`. The value is not interpreted by `dlow`
  it is passed to the `loadImage` function of the `VMPlugin`.

- #### Plugins

  - `--vm PATH_OR_ID` e.g. `vagrant` or `aws`. `VMPlugin` provides the interface
    ```haskell
    class ( NodeInterface (Node m)
          , ImageInterface (Image m)
          ) => VMPlugin m where
      type Node m
      type Image m
      loadImage        :: String -> m (Image m)
      listImages       :: Cursor (Image m) -> m (Cursor (Image m))
      create           :: Image m -> Keys -> m (Node m)
      createFromString :: String -> Keys -> m (Node m)

    class NodeInterface n where
      address :: n -> Address

    class ImageInterface i where
      imageIdentifier :: i -> String
    ```

  - `--container PATH_OR_ID` e.g.`docker`. `ContainerPlugin` provides the interface
    ```haskell
    class ContainerPlugin m where
      type Container m  
      type Process m
      containerize :: Target -> m (Container m)

      start :: Container m -> Node -> m (Process m)
      stop  :: Process m -> m ()
    ```
  - `--builder PATH_OR_ID` Build the `Target` essentially `TargetDesc -> Target` where `Target` is opaque.
  - `--package PATH_OR_ID` Parse a `TargetDesc`
  - `--store PATH_OR_ID` e.g.`etcd` or a JSON file. `Store`s implement
  ```haskell
  class Store m where
      type StoreHandle m
      load   :: m (StoreHandle m)
      save   :: StoreHandle m -> m ()

      add    :: Node         -> m Int
      remove :: Int  -> Node -> m ()
  ```
  - `--keys PATH` authorized_keys file. `Keys` are opaque but used by the
`VM`.

### Provisioning
- `set COUNT` where `COUNT` is the number of `Node`s.
- `get` the `COUNT`
- `address Index` get the address of the `Node` at Index.
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

- `start` the `Target` listed in the *dflow.yaml* file.
- `stop`  the current version of `Target`.
  - `--all` stop every version of the `Target`.
  - `--now` as in immediately.
- `restart` the service by `stop`ping and `start`ing.
- `versions` get the `git` hashes of the `Target` processes

#### Health

- `compare-checksums` compare the checksums of the current version of the `Target` with what is on the nodes.
- `health` Returns the health amount, e.g. the first part of `verify`'s result.
  - `--stats` Show statistics of the `health` portion of `verify`'s result.
- `repair` Attempt to bring the cluster to a healthy state by destroying the `Node`s that fail `verify`, and recopying the `Target` and `restart`ing. Additionally repair will create `Node`s that are missing.
 - `--watch` continuously watch the cluster and repair it if something is wrong.
- `verify` a `Node` using the a `verify` function defined in the `Target`. `verify` is a function that produces a health value and a threshold. `verify :: Node -> ({- health -} Double, {- threshold -} Double)`. If the value surpasses the thresold the `Node` is unhealthy.
  - `--thresold` see just the thresold of verify.
  - `--watch` continuously monitor the cluster.

#### Deployment

- `build` the `Target`. This should include any tests.
- `copy` the `Target` to the `Node`s
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

# Config File
```yaml
image:            PATH_OR_ID
vm-plugin:        PATH_OR_ID
container-plugin: PATH_OR_ID
project-plugin:   PATH_OR_ID
builder-plugin:   PATH_OR_ID
store-plugin:     PATH_OR_ID
count:            1

```
