# Resolve ssh config Match blocks and Include files like OpenSSH

| | |
|---|---|
| **Project** | [apache/mina-sshd](https://github.com/apache/mina-sshd) (Apache-2.0) |
| **Base commit** | [`c4287faf84`](https://github.com/apache/mina-sshd/tree/c4287faf844a0250e4a097640a0442270b756d9f) |
| **Language** | Java |
| **Reference solution** | +1,083 / -91 lines across 5 files |
| **New tests** | +1,368 lines in 1 test file |

## Problem

`HostConfigEntry.readHostConfigEntries` throws when it hits `Match` and skips over `Include`, so `HostConfigEntry.toHostConfigEntryResolver` and `ConfigFileHostEntryResolver` can't resolve configs that use them. Both should resolve like OpenSSH 9.2 does. See the output of `ssh -G -F <file> [-l user] [-p port] <host>` in the image and `/usr/share/man/man5/ssh_config.5.gz`. `IdentityFile` and `CertificateFile` will accumulate multiple values but no duplicates. If ssh can't parse a `Match` statement the config should fail to resolve with an `IOException`.

When we hit a `final` statement we need to start over after the first pass. Even if it doesn't match. The second time around we know what the host name is, in lower case, and it doesn't change. `Host` statements and `Match host` will now use that.

`Include` should be resolved at the point of the statement, relative to the `.ssh` directory in `PathUtils.getUserHomeFolder()`. These should be resolved even if the `Include` statement isn't applicable. But their contents should only apply if it is. `Host` and `Match` statements inside of them shouldn't effect the current blocks.

Host names and host patterns are matched without regard to case, and a `HostName` keeps its case unless a final pass resolves it. Only `%h` and `%%` should be used in `HostName`. `exec`, `localnetwork`, `localuser` and `canonical` aren't supported, and a `Match` that uses one of them should fail with an `IOException`.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/apache/mina-sshd repo && cd repo
git checkout c4287faf844a0250e4a097640a0442270b756d9f
cp ../Dockerfile . && git apply ../test.patch
docker build -t mina-sshd-ssh-config-match .
docker run --rm mina-sshd-ssh-config-match ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t mina-sshd-ssh-config-match .
docker run --rm mina-sshd-ssh-config-match ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm mina-sshd-ssh-config-match ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
