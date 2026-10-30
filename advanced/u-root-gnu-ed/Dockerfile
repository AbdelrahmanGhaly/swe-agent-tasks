FROM public.ecr.aws/d3j8x8q7/olympus-base-go:latest

# u-root's CI runs its tests with -timeout=20m: TestUrootCmdline alone builds every
# command for ten architectures and can outlast go test's 10-minute default.
ENV GOFLAGS="-buildvcs=false -timeout=20m"
ENV GOTOOLCHAIN=local

# The slim base image drops /usr/share/info and /usr/share/man; keep GNU ed's own manual.
RUN printf 'path-include /usr/share/info/ed.info.gz\npath-include /usr/share/man/man1/ed.1.gz\n' \
        >/etc/dpkg/dpkg.cfg.d/ed-manual \
    && apt-get update \
    && apt-get install -y --no-install-recommends ed=1.19-1 \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -g 1000 model && useradd -u 1000 -g model -m model

WORKDIR /app

COPY . .

# These tests cannot pass in an offline container: the integration tests and pox's VM tests
# boot QEMU virtual machines or mount /sys as root, findpkg resolves modules over the network,
# netstat reads /proc/net/udplite, du compares block counts that depend on the filesystem, and
# vpdbootmanager's TestInvalidCommand still expects the error text that commit 17804e0a2
# changed (upstream CI does not run tools/). The grader's sandbox runs the tests as root and
# lacks network bridges, IPv6, TCP metrics, block devices, a closed file descriptor 5 and the
# usual permission checks, so these fail there too: brctl, fusermount, ip's TestTcpMetrics,
# netcat's TestEstablishConnection, efivarfs's TestSet and TestRemove, pkg/netstat's TestIPv4,
# and securelaunch's TestGetBlkInfo and TestGetStorageDevice. gosh's interactive tests drive the
# shell through a pseudo-terminal with 2-second expectations and can hang under load until the
# test timeout.
RUN rm -f integration/generic-tests/*_test.go integration/generic-tests-cpu/*_test.go \
        integration/gotests/*_test.go cmds/exp/pox/pox_vm_test.go cmds/exp/cmd2pkg/findpkg/bb_test.go \
        cmds/core/netstat/netstat_test.go cmds/core/du/du_test.go \
        cmds/core/brctl/brctl_linux_test.go cmds/core/fusermount/fusermount_linux_test.go \
    && sed -i '/^func TestInvalidCommand(/,/^}/d' tools/vpdbootmanager/main_test.go \
    && sed -i '/^func TestTcpMetrics(/,/^}/d' cmds/core/ip/tcp_metrics_linux_test.go \
    && sed -i -e '/^func TestEstablishConnection(/,/^}/d' -e '/^\t"log"$/d' cmds/core/netcat/connect_test.go \
    && sed -i -e '/^func TestSet(/,/^}/d' -e '/^func TestRemove(/,/^}/d' pkg/efivarfs/varfs_test.go \
    && sed -i '/^func TestIPv4(/,/^}/d' pkg/netstat/netstat_test.go \
    && sed -i -e '/^func TestGetBlkInfo(/,/^}/d' -e '/^func TestGetStorageDevice(/,/^}/d' pkg/securelaunch/helpers_test.go \
    && sed -i -e '/^func TestInteractiveBubbline(/,/^}/d' -e '/^func TestInteractiveLiner(/,/^}/d' \
        -e '/^func TestInteractiveSimple(/,/^}/d' cmds/core/gosh/gosh_test.go

RUN go build ./... \
    && (go vet ./... > /dev/null 2>&1 || true) \
    && chmod -R a+rwX /app /opt/go

CMD ["/bin/bash"]
