# sshlatex
A collection of hacks to efficiently run LaTeX via ssh

Tired of long LaTeX compile times on your old, slow computer? Do you want a
tool which monitors your source file for changes, copies it to a fast remote
server, runs LaTeX there, and downloads the resulting PDF file?

Then look no further. Actually, you should; this repository is a collection of
horrible hacks to make this possible. It works for me.


## Usage

1. Run `sshlatex ssh.example.org foo.tex` in the background. (Just as with
   `pdflatex`, you can drop the `.tex` if you want to.)
2. Work on your file `foo.tex`.
3. Enjoy an updated PDF each time you save your changes.

Optionally, put a program `beepy` in your `$PATH` (or change the corresponding
line in the source code) to audibly inform you of a completed run. Useful for
when you're switching virtual desktops and don't see the output of sshlatex.


## Dependencies

None on the remote host (well, LaTeX and Perl 5), inotifywait from the
inotify-tools package on the local side.

You should use the multiplexing capabilities of ssh, so that new connections
are piggybacked over an already established connection avoiding the TCP and
cryptography handshake. Add the following to two lines to your local
`~/.ssh/config`:

    ControlMaster auto
    ControlPath ~/.ssh/control:%h:%p:%r


## Features

* No installation on the remote host is necessary.
* Images and sources included with `\includegraphics`, `\input`, and `\include`
  are automatically copied to the compile server.
* Even before the source file is changed, LaTeX gets already started on the
  remote host with the preamble of the last run. This way all the required
  packages are already loaded when the new version is ready to be compiled,
  shaving off a couple of hundreds of milliseconds.
* Downloading the resulting PDF file starts before each run of `pdflatex`
  has finished. Since `pdflatex` does not simply append to the output file, but
  seeks and overwrites earlier parts, some blocks are transferred multiple times.
  This is still faster than waiting for `pdflatex` to end and then doing a
  proper `rsync`.
* Temporary files on the server are properly cleaned up on `^C`.


## Shortcomings

* Written in haste, will contain bugs. Is a grave assortment of hacks, the
  least of which is using the `LC_*` environment variables to pass data to the
  remote host.
* Doesn't contain any code comments at all. (This will be fixed soon.)
* Doesn't synchronize changes in the included images and sources. You have to
  restart `sshlatex` to update the server's copies.
* Not customizable.
* Doesn't support fancy reruns of LaTeX for making sure that references are
  correct and so on. This tool is for rapidly iterating.
