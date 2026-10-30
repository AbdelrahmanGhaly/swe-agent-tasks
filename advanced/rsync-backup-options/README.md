# Add -R, --files-from, --link-dest and -H to gokr-rsync

| | |
|---|---|
| **Project** | [gokrazy/rsync](https://github.com/gokrazy/rsync) (BSD-3-Clause) |
| **Base commit** | [`076039b65f`](https://github.com/gokrazy/rsync/tree/076039b65f53338107e0678dc46a749e5a674b77) |
| **Language** | Go |
| **Reference solution** | +1,600 / -132 lines across 32 files |
| **New tests** | +1,464 lines in 1 test file |

## Problem

gokr-rsync does not currently support snapshot backups. It will not follow -H and doesn't have any of -R, --files-from or --link-dest. #68 mentions some of these as required for a backup. Need to implement all 4 for gokr-rsync to be a client or server, sending or receiving data, across a remote shell or local or as a daemon. No need to implement them in combination with --delete.

Using rsync 3.2.7 to rsync 3.2.7 with --protocol=27 which is what gokr-rsync supports and the same options as a reference. Any time gokr-rsync is used the destination should look just like it would if rsync was used on both ends. It should have the same items with the same content and attributes. It should have the same files hardlinked together and to files in the --link-dest. Anything rsync leaves alone should be left alone. Setuid/setgid permissions are also an attribute even if its on a current file just changing ownership.

-R will maintain the source path under the destination, starting after the /./ if it exists. All directories above it should be included with their attributes. --files-from allows you to pass source paths from a file on the client side. --link-dest will take one directory path relative to the destination or absolute.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/gokrazy/rsync repo && cd repo
git checkout 076039b65f53338107e0678dc46a749e5a674b77
cp ../Dockerfile . && git apply ../test.patch
docker build -t rsync-backup-options .
docker run --rm rsync-backup-options ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t rsync-backup-options .
docker run --rm rsync-backup-options ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm rsync-backup-options ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
