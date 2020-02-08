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


## Browser support

Web MIDI has been [fully supported][1] by Google Chrome/Chromium/Blink since
version 43 in May 2015, running on Android, Chrome OS, Linux, macOS and
Windows. It also works on derivative browsers such as recent Microsoft Edge,
Brave and Opera.

Mozilla Firefox/Gecko does not support Web MIDI as of February 2020. A
[third-party extension][2] is available, enabling the API using the [Jazz
plugin][3]. Work towards native Web MIDI in Firefox is slowly progressing,
despite an [alarming RFP discussion][4] complete with hysterical claims that
hardware MIDI users are incapable of informed consent to sysex access.

Apple Safari/WebKit also has no support as of February 2020. Unfortunately,
WebKit lags behind other browser engines in implementing far more mainstream
web standards, suggesting the outlook for Web MIDI is bleak. Worse, all
alternative browsers on iOS must use the system-provided WebKit; Apple abuse
their monopoly position as platform gatekeeper to exclude competition from
better-maintained browser engines.

Plugins and extensions are not supported on iOS, so unless Mobile Safari
itself switches to Blink as rumoured, Web MIDI is likely to remain locked
out from Apple iDevices for the foreseeable future. The macOS build of
Safari is less restricted: in theory the [Jazz plugin][3] and [extension][2]
could be employed there as with Firefox, but other current WebKit
deficiencies are likely to break these Web MIDI tools too.

[1]: https://www.chromestatus.com/feature/4923613069180928
[2]: https://jazz-soft.net/download/web-midi/
[3]: https://jazz-soft.net/download/Jazz-Plugin/
[4]: https://github.com/mozilla/standards-positions/issues/58


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
