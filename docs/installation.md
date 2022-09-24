# Installation

## Clone Source Code
```
git clone git@github.com:octopus-network/ChainBridge.git
# switch to oct-dev branch
git checkout oct-dev
```
## Dependencies

- [Subkey](https://github.com/paritytech/substrate): 
Used for substrate key management. Only required if connecting to a substrate chain.

```
make install-subkey
```


## Building from Source

To build `chainbridge` in `./build`.
```
make build
```

**or**

Use`go install` to add `chainbridge` to your GOBIN.

```
make install
```
