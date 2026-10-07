# Add sample-accurate setting automation to Sequencer events

| | |
|---|---|
| **Project** | [SamiPerttu/fundsp](https://github.com/SamiPerttu/fundsp) (Apache-2.0) |
| **Base commit** | [`559584043a`](https://github.com/SamiPerttu/fundsp/tree/559584043aa3369e92d5aad12a5e3b003cfacfd5) |
| **Language** | Rust |
| **Reference solution** | +501 / -260 lines across 6 files |
| **New tests** | 65 test functions, +1,721 lines in 1 test file |

## Problem

You can now set `Setting`s on a sequencer while it is playing. Using `Setting::event(id)` will create an `Address::Event(EventId)` which selects an event from the sequencer just like `node(id)` selects a `Net` node. The sequencer pops this level and passes the rest of the `Setting`, including any leftover address, to the event's unit. `Sequencer::set_at(time, setting)` will set a `Setting` at the absolute time `time` in seconds. `set_at_relative(time, setting)` sets a `Setting` at `time` seconds from the current time. Using `set` on a `Sequencer` or `SequencerBackend` will set it at the current time. A setting that does not specify an event in the sequencer is discarded.

A setting that is scheduled at time `t` will reach the unit before the sample that time `t` falls on is rendered by the event. Time will be rounded just like the start time of events. Any setting at or before the start of an event will reach it before its first sample is rendered. Any setting that happens at or after the end of an event will not reach it. Any setting that has already occurred will reach the unit before its next sample is rendered. Settings that occur at the same sample will reach the unit in the order they were set.

On each replay of a sequencer (after a reset or when looping) it will begin at the state of the unit that was pushed and replay all settings at their corresponding points during the event's playback. If an event continues past the loop point it will continue to receive settings. `Sequencer::remove(id, fade_out_time)` will remove an event. If the event is sounding and has not reached its end, it will be faded out over `fade_out_time` seconds from now, even past its planned end, as if you edited the event to end at that time. Any other events will be removed immediately. Once an event is removed it will not be replayed.

Rendering using `tick` or `process` at any block size should produce the same result for all `ReplayMode`s. Including when looping, where an event can end and start again inside a single block. An event whose start and end are equal never sounds. With a backend, changes made to set or remove events take effect after `commit`. Relative times will be from the backend's time when the change is received.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/SamiPerttu/fundsp repo && cd repo
git checkout 559584043aa3369e92d5aad12a5e3b003cfacfd5
cp ../Dockerfile . && git apply ../test.patch
docker build -t fundsp-sequencer-automation .
docker run --rm fundsp-sequencer-automation ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t fundsp-sequencer-automation .
docker run --rm fundsp-sequencer-automation ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm fundsp-sequencer-automation ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
