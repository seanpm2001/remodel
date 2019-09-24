<img src="builderman.png" alt="Remodel Logo" width="327" height="277" align="right" />
<h1 align="left">Remodel</h1>

[![Remodel on crates.io](https://img.shields.io/crates/v/remodel.svg?label=crates.io)](https://crates.io/crates/remodel)
[![Actions Status](https://github.com/rojo-rbx/remodel/workflows/CI/badge.svg)](https://github.com/rojo-rbx/remodel/actions)

Remodel is a command line tool for manipulating Roblox files and the instances contained within them. It's intended as a building block for Roblox automation tooling.

Remodel is still in early development, but much of its API is already fairly stable. Feedback is welcome!

## Installation

### From GitHub Releases
You can download pre-built Windows and macOS binaries from [Remodel's GitHub Releases page](https://github.com/rojo-rbx/remodel/releases).

### From crates.io
You'll need Rust 1.37+.

```bash
cargo install remodel
```

### Latest development changes (unstable!!)
You'll need Rust 1.37+.

```bash
cargo install --git https://github.com/rojo-rbx/remodel
```

## Quick Start
Most of Remodel's interface is its Lua API. Users write Lua 5.3 scripts that Remodel runs, providing them with a special set of APIs.

One use for Remodel is to break a place file apart into multiple model files. Imagine we have a place file named `my-place.rbxlx` that has some models stored in `ReplicatedStorage.Models`.

We want to take those models and save them to individual files in a folder named `models`.

```lua
local game = remodel.readPlaceFile("my-place.rbxlx")

-- If the directory does not exist yet, we'll create it.
remodel.createDirAll("models")

local Models = game.ReplicatedStorage.Models

for _, model in ipairs(Models:GetChildren()) do
	-- Save out each child as an rbxmx model
	remodel.writeModelFile(model, "models/" .. model.Name .. ".rbxmx")
end
```

For more examples, see the [`examples`](examples) folder.

## Supported Roblox API
Remodel supports some parts of Roblox's API in order to make code familiar to existing Roblox users.

* `Instance.new(className)` (0.5.0+)
	* The second argument (parent) is not supported by Remodel.
* `<Instance>.Name` (read + write)
* `<Instance>.ClassName` (read only)
* `<Instance>.Parent` (read + write)
* `<Instance>:Destroy()` (0.5.0+)
* `<Instance>:Clone()` (**unreleased**)
* `<Instance>:GetChildren()`
* `<Instance>:FindFirstChild(name)`
	* The second argument (recursive) is not supported by Remodel.
* `<DataModel>:GetService(name)` (**unreleased**)

## Remodel API
Remodel has its own API that goes beyond what can be done inside Roblox.

### `remodel.readPlaceFile`
```
remodel.readPlaceFile(path: string): Instance
```

Load an `rbxlx` file from the filesystem.

Returns a `DataModel` instance, equivalent to `game` from within Roblox.

Throws on error.

### `remodel.readModelFile`
```
remodel.readModelFile(path: string): List<Instance>
```

Load an `rbxmx` or `rbxm` (0.4.0+) file from the filesystem.

Note that this function returns a **list of instances** instead of a single instance! This is because models can contain mutliple top-level instances.

Throws on error.

### `remodel.readPlaceAsset` (0.5.0+)
```
remodel.readPlaceAsset(assetId: string): List<Instance>
```

Reads a place asset from Roblox.com, equivalent to `remodel.readPlaceFile`.

Most models and places uploaded to Roblox.com are in Roblox's binary model format, which tools like Remodel and Rojo have poor support for. Models and places uploaded by Remodel and Rojo are currently always in the XML format, which is less efficient but has better support.

**This method requires web authentication for private assets! See [Authentication](#authentication) for more information.**

Throws on error.

### `remodel.readModelAsset` (0.5.0+)
```
remodel.readModelAsset(assetId: string): List<Instance>
```

Reads a model asset from Roblox.com, equivalent to `remodel.readModelFile`.

Most models and places uploaded to Roblox.com are in Roblox's binary model format, which tools like Remodel and Rojo have poor support for. Models and places uploaded by Remodel and Rojo are currently always in the XML format, which is less efficient but has better support.

**This method requires web authentication for private assets! See [Authentication](#authentication) for more information.**

Throws on error.

### `remodel.writePlaceFile`
```
remodel.writePlaceFile(instance: DataModel, path: string)
```

Saves an `rbxlx` file out of the given `DataModel` instance.

If the instance is not a `DataModel`, this method will throw. Models should be saved with `writeModelFile` instead.

Throws on error.

### `remodel.writeModelFile`
```
remodel.writeModelFile(instance: Instance, path: string)
```

Saves an `rbxmx` or `rbxm` (0.4.0+) file out of the given `Instance`.

If the instance is a `DataModel`, this method will throw. Places should be saved with `writePlaceFile` instead.

Throws on error.

### `remodel.writeExistingPlaceAsset` (0.5.0+)
```
remodel.writeExistingPlaceAsset(instance: Instance, assetId: string)
```

Uploads the given `DataModel` instance to Roblox.com over an existing place.

If the instance is not a `DataModel`, this method will throw. Models should be uploaded with `writeExistingModelAsset` instead.

**This method always requires web authentication! See [Authentication](#authentication) for more information.**

Throws on error.

### `remodel.writeExistingModelAsset` (0.5.0+)
```
remodel.writeExistingModelAsset(instance: Instance, assetId: string)
```

Uploads the given instance to Roblox.com over an existing place.

If the instance is a `DataModel`, this method will throw. Places should be uploaded with `writeExistingPlaceAsset` instead.

**This method always requires web authentication! See [Authentication](#authentication) for more information.**

Throws on error.

### `remodel.readFile` (0.3.0+)
```
remodel.readFile(path: string): string
```

Reads the file at the given path.

Throws on error, like if the file did not exist.

### `remodel.readDir` (0.4.0+)
```
remodel.readDir(path: string): List<string>
```

Returns a list of all of the file names of the children in a directory.

Throws on error, like if the directory did not exist.

### `remodel.writeFile` (0.3.0+)
```
remodel.writeFile(path: string, contents: string)
```

Writes the file at the given path.

Throws on error.

### `remodel.createDirAll`
```
remodel.createDirAll(path: string)
```

Makes a directory at the given path, as well as all parent directories that do not yet exist.

This is a thin wrapper around Rust's [`fs::create_dir_all`](https://doc.rust-lang.org/std/fs/fn.create_dir_all.html) function. Similar to `mkdir -p` from Unix.

Throws on error.

## Authentication
Some of Remodel's APIs access the Roblox web API and need authentication in the form of a `.ROBLOSECURITY` cookie to access private assets. Auth cookies look like this:

```
_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|<actual cookie stuff here>
```

**Auth cookies are very sensitive information! If you're using Remodel on a remote server like Travis CI or GitHub Actions, you should create a throwaway account with limited permissions in a group to ensure your valuable accounts are not compromised!**

If you're on Windows, Remodel will attempt to use the cookie from a logged in Roblox Studio session to authenticate all requests.

To give a different auth cookie to Remodel, use the `--auth` argument:

```
remodel foo.lua --auth "$MY_AUTH_COOKIE"
```

## Remodel vs rbxmk
Remodel is similar to [rbxmk](https://github.com/Anaminus/rbxmk):
* Both Remodel and rbxmk use Lua
* Remodel and rbxmk have a similar feature set and use cases
* Remodel is imperative, while rbxmk is declarative
* Remodel emulates Roblox's API, while rbxmk has its own, very unique API

## License
Remodel is available under the terms of the MIT license. See [LICENSE.txt](LICENSE.txt) for details.

[Logo source](https://pixabay.com/illustrations/factory-worker-industry-2318026/).