## Std Storage

Std Storage is a library that makes manipulating storage easy.

To use Std Storage, import the following in your test contract:

```solidity
import {stdStorage, StdStorage} from "forge-std/Test.sol";              
```

Add the following line in your test contract:

```solidity
using stdStorage for StdStorage;
```

Then, access Std Storage via the `stdstore` instance.

### Functions

Query functions:

- [`target`](./target.md): Set the address of the target contract
- [`sig`](./sig.md): Set the 4-byte selector of the function to static call
- [`with_key`](./with_key.md): Pass an argument to the function (can be used multiple times)
- [`depth`](./depth.md): Set the position of the value in the `tuple` (e.g. inside a `struct`)

Terminator functions:

- [`find`](./find.md): Return the slot number
- [`checked_write`](./checked_write.md): Set the data to be written to the storage slot(s)
- [`read_<type>`](./read.md): Read the value from the storage slot as `<type>`

### Example

`playerToCharacter` tracks info about players' characters.

```solidity
// MetaRPG.sol

struct Character {
    string name;
    uint256 level;
}

mapping (address => Character) public playerToCharacter;
```

Let's say we want to set the level of our character to 120.

```solidity
// MetaRPG.t.sol
// Reads the entire content of file to string, (path) => (data)
function readFile(string calldata) external returns (string memory);
/// Reads the entire content of file as binary. `path` is relative to the project root.
function readFileBinary(string calldata path) external view returns (bytes memory data);
/// Reads the directory at the given path recursively, up to `maxDepth`.
/// `maxDepth` defaults to 1, meaning only the direct children of the given directory will be returned.
/// Follows symbolic links if `followLinks` is true.
function readDir(string calldata path) external view returns (DirEntry[] memory entries);
// Reads next line of file to string, (path) => (line)
function readLine(string calldata) external returns (string memory);
/// Reads a symbolic link, returning the path that the link points to.
/// This cheatcode will revert in the following situations, but is not limited to just these cases:
/// - `path` is not a symbolic link.
/// - `path` does not exist.
function readLink(string calldata linkPath) external view returns (string memory targetPath);
// Writes data to file, creating a file if it does not exist, and entirely replacing its contents if it does.
// (path, data) => ()
function writeFile(string calldata, string calldata) external;
// Writes line to file, creating a file if it does not exist.
// (path, data) => ()
function writeLine(string calldata, string calldata) external;
// Closes file for reading, resetting the offset and allowing to read it from beginning with readLine.
// (path) => ()
function closeFile(string calldata) external;
// Removes file. This cheatcode will revert in the following situations, but is not limited to just these cases:
// - Path points to a directory.
// - The file doesn't exist.
// - The user lacks permissions to remove the file.
// (path) => ()
function removeFile(string calldata) external;
// Returns true if the given path points to an existing entity, else returns false
// (path) => (bool)
function exists(string calldata) external returns (bool);
// Returns true if the path exists on disk and is pointing at a regular file, else returns false
// (path) => (bool)
function isFile(string calldata) external returns (bool);
// Returns true if the path exists on disk and is pointing at a directory, else returns false
// (path) => (bool)
function isDir(string calldata) external returns (bool);

stdstore
    .target(address(metaRpg))
    .sig("playerToCharacter(address)")
    .with_key(address(this))
    .depth(1)
    .checked_write(120);
```
// Verify that path 'foo/file/bar.txt' points to a file
string memory validFilePath = "foo/files/bar.txt";
assertTrue(vm.isFile(validFilePath));

// Verify that 'foo/file' points to a directory
string memory validDirPath = "foo/files";
assertTrue(vm.isDir(validDirPath));

### Limitations

- Accessing packed slots is not supported

### Known issues

- Slot(s) may not be found if the `tuple` contains types shorter than 32 bytes
