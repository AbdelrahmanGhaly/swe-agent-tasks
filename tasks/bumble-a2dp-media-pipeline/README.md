# Fix media packetization across the AVDTP and A2DP audio path

| | |
|---|---|
| **Project** | [google/bumble](https://github.com/google/bumble) (Apache-2.0) |
| **Base commit** | [`18dd21eec3`](https://github.com/google/bumble/tree/18dd21eec3fbec96c3151682314caa0d5f3ec17d) |
| **Language** | Python |
| **Reference solution** | +403 / -87 lines across 3 files |
| **New tests** | 75 test functions, +876 lines in 2 test files |

## Problem

A rejection raises `InvalidPacketError` when the fault is in bytes that were received, and
`InvalidArgumentError` when it is in a caller's argument.

A packet carrying several contributing source identifiers must recover them all. When the extension
bit is set, an extension header follows the source list with a profile value and a length counted
in 32-bit words, and neither it nor its data is payload. When the padding bit is set, the packet's
last octet counts the padding octets including itself, and those are not payload either. Add
trailing `MediaPacket` arguments `extension_profile`, `extension_data` and `padding_size`, in that
order, so a parsed packet re-serializes to its original bytes. Reject a packet shorter than the
header it declares, a cut-short extension, extension data that is not a whole number of words, a
padding count that is zero or wider than one octet when padding is claimed, and one reaching past
the header.

An SBC frame is a whole number of octets, so a frame length that does not divide evenly rounds up.
Expose it as `sbc_frame_length`, taking subband count, channel count, block count, bitpool and
channel mode. Add `sbc_frame_parameters`, decoding a frame header into sampling frequency, block
count, channel mode, allocation method, subband count, bitpool and frame length, in that order.

Media payloads carry a one octet header built by `sbc_media_payload_header(frame_count,
fragmented=False, start=False, last=False)`: a fragmented flag, a start flag, a last flag, one
reserved bit, then a four bit frame counter, from the most significant bit down. An unfragmented
payload clears all three flags and counts the complete frames it carries, at most fifteen. A frame
too large for one payload is split; every fragment sets fragmented, the first sets start, the last
sets last, and the counter gives the fragments still to come including the current one, so the last
reads one. A frame needing more than fifteen fragments is rejected as a fault in the given
transport unit, as is a transport unit with no room for frame bytes.

Anything already accumulated goes out before a split frame. Timestamps count samples at the stream
sampling rate, a payload is stamped with its first frame, and every fragment of a frame shares that
stamp. The same sample based stamping applies to AAC. Every buffered frame must reach a packet,
including the last.

Add `SbcPayloadAssembler` to turn payloads back into frames, built with a callback that receives
each completed group of frames as one sequence, and fed through `on_payload`. It rejects an empty
payload, a declared frame count that disagrees with the payload and a counter that fails to count
down, and ignores a continuation with no start.

An AVDTP single packet header is two octets, a start header three, a continuation or end header
one, so each leaves different room. A fragmented message declares how many packets it spans,
correctly, and continuation packets fill the transport unit. Reject a message needing more
fragments than the count can express, and a transport unit too small to carry anything.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/google/bumble repo && cd repo
git checkout 18dd21eec3fbec96c3151682314caa0d5f3ec17d
cp ../Dockerfile . && git apply ../test.patch
docker build -t bumble-a2dp-media-pipeline .
docker run --rm bumble-a2dp-media-pipeline ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t bumble-a2dp-media-pipeline .
docker run --rm bumble-a2dp-media-pipeline ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm bumble-a2dp-media-pipeline ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
