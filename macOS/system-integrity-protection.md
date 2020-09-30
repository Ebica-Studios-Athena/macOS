# macOS System Integrity Protection

## Relevant Links

### Overviews

- [Official SIP Overview from Apple Support](https://support.apple.com/en-us/HT204899)
- [SIP Wikipedia Entry](https://en.wikipedia.org/wiki/System_Integrity_Protection)

### Articles

- [System Integrity Protection – Adding another layer to Apple’s security
  model](https://derflounder.wordpress.com/2015/10/01/system-integrity-protection-adding-another-layer-to-apples-security-model/)
: An excellent, in-depth article about SIP and the various facets of it's operation.
- [A short trip with rootless: what you can and cannot
  do](https://eclecticlight.co/2018/01/05/a-short-trip-with-rootless-what-you-can-and-cannot-do/):
  details using `xattr` to determine if a file is protected by SIP, as well as
  how, in some limited cases, a normal user can flag a file for protection by
  SIP.
  
## Overview

The System Integrity Protection (SIP) system is a recent addition to macOS that provides
special protections for certain files and directories considered critical to
correct operation of the machine. It was introduced in 10.11 "El Capitan."

## Protecting The Filesystem

When installing software in macOS, it is often required to provide the installer
access to certain protected directories and files. This process requires the
installer to be granted temporary `root permissions`, allowing them access to
such files and directories.

Unfortunately, this process is prone to abuse by more ill-intentioned
programmers. In the early 2010's, many everyday users were tricked by the famous
[`Mac Defender`](https://en.wikipedia.org/wiki/Mac_Defender) scam, in which
users had their personal information stolen by a program that presented itself
as a virus protection solution. Because they thought they were installing
legitimate software, they granted root access to the installer without a second
thought, allowing it to secretly install malware into protected portions of the
system.

The SIP system is an additional layer of protection that is impervious to scams
such as Mac Defender. The SIP has the final say in what a running process is
allowed to do when it comes to these protected files and directories. Even if
the installer has been granted root permissions by the user, the SIP simply will
not allow certain files and directories to be modified. It runs in the kernel,
rather than the OS, which provides even more protection as it is nearly
impossible to directly access from within the OS itself.

While SIP blocks most installers from operating on these protected sections, it
does allow certain ones access. If the installer has been code-signed by an
appropriate trusted entity, and said signature can be properly verified, then
the SIP will allow it to continue. This is how Apple can install OS updates
automatically without any form of interaction with the user.

## Enabling and Disabling SIP

SIP is enabled by default, and 99% of the time there is no good reason to turn
it off. Their are, however, certain edge cases that may require it to be
disabled. One such case is when attempting to dual-boot modern Macbook Pros
with Linux. Another popular one is attempting to remove the `Finder` program
from the Dock, which is usually not allowed by SIP.

The tiny amount of control over SIP provided to the user is done through the
`csrutil` program, which as of macOS 10.15 "Catalina", is located in
`/usr/bin/csrutil`. This program provides a couple of other features, but the
one we are concerned with is toggling SIP on and off.

Enabling or disabling SIP must be done in the Recovery OS, and attempting to do
so in the normal OS will result in an error message being displayed, with no
changes being made. This means that toggling SIP requires you to reboot the
machine into recovery mode by holding down `⌘ + R` while the machine starts back
up. In this mode, you will need to have access to an administrative account to
continue. After logging in, you will need to open a terminal by selecting
`Utilities > Terminal` in the top bar. Once open, just run `csrutil enable` to
turn SIP on, or `csrutil disable` to turn it off. After the command has been
run, simply reboot the computer and the changes will take effect.

Please note that it is generally a bad idea to leave SIP turned off for extended
periods of time. If you have disabled it for some reason, please make sure you
enable it again before you finish your work flow.

## Protected Sections

By default, SIP protects the following from unauthorized modification:

- /System
- /bin
- /sbin
- /usr (but not /usr/local)
- Most preinstalled Apple applications in /Applications

The following files are [symlinks](https://en.wikipedia.org/wiki/Symbolic_link)
to other directories. While the symlinks themselves are protected, the target directories
are not. This prevents hackers from changing the symlink target to some
directory they have secretly installed in an unprotected section of the
filesystem that contains malicious programs, an attack called `symlink hacking`:

- /etc -> /private/etc
- /tmp -> /private/tmp
- /var -> /private/var

You can see what files and directories in your current working directory are
protected by SIP by running the `ls -lO` command. Files protected by SIP will
have the `restricted` flag set,

A complete list of files and directories protected by SIP can be found in the
primary SIP configuration file, located at
`/System/Library/Sandbox/rootless.conf`. This file may itself be protected by
SIP. Apple has confirmed that even if SIP is disabled in the kernel, allowing
the user to freely edit it's contents, any modifications to this file will be
automatically overwritten by Apple, meaning this file is essentially immutable.