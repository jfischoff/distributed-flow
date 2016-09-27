# About

`distributive-flow` is simple method for zero-downtime deployment of socket based `Haskell` applications. Notably, it can bootstrap itself, facilating interactive development.

`distributive-flow` is a library that used to make hot-reloadable executables.  

`df` is a command for orchastrating `distributive-flow` applications. `df` is meant to manage multiple versions of the same executable on a clusters of machines. 

`df` is configured with an `Image`, `[PublicKey]`, a `Backend`,

`df` maintains a durable store of an `[Address]` and `(Map String Address)` or `Node`s is deploy to.

Additionally it assumes that multiple versions of the same executable can bind to a single port, so probably `SO_REUSEPORT` must be used ... or a load balancer.

# Commands

### Provisioning
`df` can provision a `Node` for an `Executable` for a given backend using an Image. It cannot install any software or dependencies. 

Backend plugins handle dependency management and can 
```haskell
containerize :: Executable -> Container
```
allowing for deployment. 

- ###### Common Flags

 - '--credentials' IaaS credentials
 - '--backend' Vagrant
 - '--authorized-users' authorized_keys file.
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
- `rollback` revert the last deploy and start the old version.
- `repair` Attempt to bring the cluster to a healthy state by killing the `Node`s that fail verification, and recopying the `Executable` and `restart`ing
 - `--full-copy` Do a full copy of the executable.
 - `--watch`
- `gc` collect old versions
 - `count` keep this number
 - `older` a relative or absolute date.

##### Optional Cache Management
Executables are cached to speed up delivery. Unless there is a bug caching commands are not needed
- `cache` commands all take the `--address` flag.
 - `flush` clear the cache.
 - `put` add to cache.
 - `get` get from the cache.
 - `diff` compare the cache to local src and artifacts. See what is different.
 - `update` make the cache have the new stuff.
 - `repair` make the cache look like the local would be if built from scratch.
 - `persist` send the data to the address.
   - `--s3-bucket` persist the cache to the S3 bucket.  
 - `gc` remove the old stuff.
   - `count` keep this number
   - `older` a relative or absolute date.
