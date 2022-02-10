# Web MIDI tools

The following tools are self-contained web applications which interact with
attached MIDI instruments using the Web MIDI API. Each tool runs entirely
client-side, depends on no resources outside the single static HTML file
which implements it, and is [MIT licensed](#copying) for ease of code reuse.


## System exclusive tool

- <https://arachsys.github.io/webmidi/sysex>
- <https://github.com/arachsys/webmidi/blob/master/sysex.html>

Choose MIDI input and output ports from the drop-down selectors in the
header bar, then enter complete MIDI commands in hexadecimal byte format at
the prompt to send them. Received messages will be logged to the screen in
green.

For example, `f0 7e 00 06 01 f7` will send a universal identity request,
which should trigger an identity reply from a compliant instrument.

The shell supports basic command-line editing including history, and the
`clear` command clears the log of received messages but leaves the command
history intact.

The display filter drop-down narrows the recorded data to system exclusive
messages, all system common and channel data, or all messages including real
time bytes. Note that if enabled, real time chatter from active sensing
bytes and time clock messages can result in a very large log in the browser.


## Montage performance scratchpad

- <https://arachsys.github.io/webmidi/montage/scratchpad>
- <https://github.com/arachsys/webmidi/blob/master/montage/scratchpad.html>

This tool is a simple instrument-specific patch librarian, demonstrating
automatic instrument detection and system exclusive dump/restore. It scans
for an attached Yamaha Montage or MODX synthesizer by sending universal
identity requests on all available outputs and monitoring for known replies.
When a suitable instrument is connected, the model and version are indicated
in the header bar.

Use the rightward arrow to fetch the current performance from the
instrument's edit buffer. Alternatively, use the upward arrow to load a
performance from a file in raw sysex dump format, or just drag the file onto
the page.

For each received or loaded performance, a scratchpad row will be inserted
with its name, a leftward arrow to return it to the instrument's edit
buffer, a downward arrow to save it to a file, and an X to delete the row
from the scratchpad.

For convenience, some simple keyboard shortcuts are defined. The `=` or `>`
key fetches the current performance from the instrument, equivalent to
clicking on the rightward arrow. The digits `1` to `9` send one of the last
nine rows back to the instrument, equivalent to clicking on the left arrow
for that entry, with `1` corresponding to the most recent performance.


## MIDI skeleton page

- <https://arachsys.github.io/webmidi/skeleton>
- <https://github.com/arachsys/webmidi/blob/master/skeleton.html>

Choose MIDI input and output ports from the drop-down selectors in the
header bar. Useful functions and other constants are exported in global
scope, accessible from the javascript console for easy hacking. This
skeleton code is extracted and generalised from the system exclusive tool
above.

Call `transmit` with an array of MIDI bytes to send that command to the
selected output. Call `receive` with a listener function and an optional
timeout in milliseconds to subscribe to inbound messages as arrays of MIDI
bytes. `receive` is asynchronous, returning a promise which resolves when
the timeout expires or the listener returns true to signify completion.
Alternatively, add a handler manually to `receivers` with `receivers.add` to
subscribe to all message events, then remove it with `receivers.delete` when
finished.

Convenience functions `hex` and `raw` are provided to convert hex strings
to/from raw byte arrays, as passed to `receive` handlers and expected by
`transmit`. Invalid byte representations like '??' are translated by `raw`
into null entries in the array, which is useful for constructing `match`
patterns.

`match(pattern, data)` returns true if the start of the message in `data`
matches the pattern array `pattern`, where each element of `pattern` is
either a literal byte to match, or null for a wildcard byte.

A higher-level interface building on these facilities is also provided as
async function `dump(request, start, filter, finish, timeout)`. This sends
`request`, waits for a response matching `start`, then logs all responses
matching `pattern` up to and including a terminator that matches `finish`,
resolving to the list of received messages after either the terminator or
`timeout` milliseconds have elapsed. If `start` is null, logging of messages
begins immediately. If `finish` is null, logging of messages continues until
reception times out.

For example,

    request = raw("f0 43 20 7f 1c 02 0e 25 00 f7");
    start = raw("f0 43 00 7f 1c ?? ?? 02 0e 25 00");
    filter = raw("f0 43 00 7f 1c ?? ?? 02");
    finish = raw("f0 43 00 7f 1c ?? ?? 02 0f 25 00");
    response = await dump(request, start, filter, finish, 1000);

will request a dump of the performance edit buffer from an attached Yamaha
Montage, giving up after one second if the dump hasn't completed. Use
`response.map(hex).join("\n")` to pretty-print a hex dump of the bulk data,
and `response.forEach(transmit)` to restore the dump to the instrument.


## Montage skeleton page

- <https://arachsys.github.io/webmidi/montage/skeleton>
- <https://github.com/arachsys/webmidi/blob/master/montage/skeleton.html>

A minimal demonstration of automatic instrument detection, this page sends
universal identity requests on each available output in turn, monitoring
every input for a corresponding reply from a Yamaha Montage or MODX
synthesizer. The object `montage` is exported in global scope, accessible
from the javascript console.

When an instrument is found, `montage.input` and `montage.output` are the
connected input and output port, `montage.model` is the model detected (e.g.
"Montage 8") and `montage.version` is the firmware version (e.g. "3.0"). The
page header is updated to show the model and version detected.

As above, call `montage.transmit` to send a command to the synthesizer, and
call asynchronous function `montage.receive` with a listener and timeout to
receive messages from it. The corresponding higher-level `montage.dump`
interface also exists; for example:

    request = raw("f0 43 20 7f 1c 02 0e 25 00 f7");
    start = raw("f0 43 00 7f 1c ?? ?? 02 0e 25 00");
    filter = raw("f0 43 00 7f 1c ?? ?? 02");
    finish = raw("f0 43 00 7f 1c ?? ?? 02 0f 25 00");
    response = await montage.dump(request, start, filter, finish, 1000);

will request the performance edit buffer from a Montage. (A MODX would need
group/model ID "7f 1c 07" rather than "7f 1c 02".)

If no instrument is found, this is indicated in the header and the page
continues to rescan once a second. In this case, `montage.transmit` and
`montage.receive` do nothing, and all of `montage.input`, `montage.output`,
`montage.model` and `montage.version` are set to null.

The page will automatically initiate a rescan when the instrument's ports
are disconnected. However, it does not continuously poll when the detected
ports remain up. In practice, this means that unplugging a USB MIDI cable
will be detected but unplugging a DIN MIDI cable will not. Click on the
status message in the header to forget the existing instrument and force a
rescan.


## Browser support

Web MIDI has been [fully supported][1] by Google Chrome/Chromium/Blink since
version 43 in May 2015, running on Android, Chrome OS, Linux, macOS and
Windows. It also works on derivative browsers such as recent Microsoft Edge,
Brave and Opera. Hot-plugging interfaces works correctly on all platforms.

Despite an [alarming RFP discussion][2] complete with hysterical claims that
hardware MIDI users are incapable of informed consent to sysex access,
Mozilla Firefox/Gecko introduced Web MIDI support on Linux, macOS and
Windows in the 99.0a1 nightly builds. Flip `dom.webmidi.enabled` to true in
`about:config` to enable it. Web MIDI is not available in the 97.0 release
nor the 98.0b2 beta, but might be in the final 98.0 release. Hot-plugging
interfaces after the browser is launched does not yet work correctly on
every platform.

Apple Safari/WebKit still has no support as of February 2022. Unfortunately,
WebKit lags behind other browser engines in implementing far more mainstream
web standards, suggesting the outlook for Web MIDI is bleak. Worse, all
alternative browsers on iOS must use the system-provided WebKit; Apple abuse
their monopoly position as platform gatekeeper to exclude competition from
better-maintained browser engines.

Plugins and extensions are not supported on iOS, so unless Mobile Safari
itself switches to Blink as rumoured, Web MIDI is likely to remain locked
out from Apple iDevices for the foreseeable future. The macOS build of
Safari is less restricted: in theory the [Jazz plugin][3] and [extension][4]
could be employed there, but other WebKit deficiencies are likely to break
these Web MIDI tools too.

[1]: https://www.chromestatus.com/feature/4923613069180928
[2]: https://github.com/mozilla/standards-positions/issues/58
[3]: https://jazz-soft.net/download/Jazz-Plugin/
[4]: https://jazz-soft.net/download/web-midi/


## Copying

The Web MIDI tools and this documentation are distributed as Free Software
under the terms of the following MIT licence:

> Copyright &copy; 2020 Chris Webb \<chris@arachsys.com>
>
> Permission is hereby granted, free of charge, to any person obtaining a
> copy of this software and associated documentation files (the "Software"),
> to deal in the Software without restriction, including without limitation
> the rights to use, copy, modify, merge, publish, distribute, sublicense,
> and/or sell copies of the Software, and to permit persons to whom the
> Software is furnished to do so, subject to the following conditions:
>
> The above copyright notice and this permission notice shall be included in
> all copies or substantial portions of the Software.
>
> The Software is provided "as is", without warranty of any kind, express or
> implied, including but not limited to the warranties of merchantability,
> fitness for a particular purpose and noninfringement. In no event shall
> the authors or copyright holders be liable for any claim,  damages or
> other liability, whether in an action of contract, tort or otherwise,
> arising from, out of or in connection with the Software or the use or
> other dealings in the Software.

Feel free to raid this code for useful spare parts when putting together
your own Web MIDI hacks.


## Contact

Please send comments, bug reports or proposed patches to Chris Webb
\<[chris@arachsys.com](mailto:chris@arachsys.com)>.
