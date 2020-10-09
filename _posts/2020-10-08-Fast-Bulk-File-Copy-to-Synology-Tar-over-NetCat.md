---
title: "Faster Bulk File Copy to Synology (Tar over Netcat)"
date: 2020-10-08
layout: "post"
categories: "nas, synology, netcat, nc, file transfer"
---

# Faster Bulk File Copy to Synology

The thing about moving a large amount of data around your network, is well, it is going to take some time. Sometimes, lots and lots and lots of time. One of the ways I have overcome this in the past was to use `tar` and `nc` to basically ship the files as fast as they could be sent. This gets to be interesting when the Synology NAS does not ship with the `nc` or [Netcat](http://netcat.sourceforge.net/) tool by default.

[![I am speed](https://c2.staticflickr.com/4/3583/3697064212_2306a8d02d_b.jpg "Pictured: A red Crock sandal on gravel. Sandal is stylized as the Lightning McQueen race car and has a frog peeking out of a window. I am speed. ")](https://c2.staticflickr.com/4/3583/3697064212_2306a8d02d_b.jpg)

> **Note:** This method of bulk file moving does not even pretend to be secure. We are basically dumping raw data onto the network from one node and dumping it back to disk on the other. ***GOTTA GO FAST***

## tl;dr

The TL;DR for receiving:

* SSH into the NAS and become root
* Run `synogear install`
* Move to the receiving folder: `cd /volume1/Data/destination/`
* Run `/var/packages/DiagnosisTool/target/tool/ncat -l -p 7000 | tar -xpf -`

The TL;DR for sending:

* SSH into the NAS and become root
* Run `synogear install`
* Move to the sending folder: `cd /volume1/Data/destination/`
* Run `tar -cf - * | /var/packages/DiagnosisTool/target/tool/ncat sendinghost 7000`

## Installing Ncat on Synology

The Synology NAS does not ship with the [Netcat](http://netcat.sourceforge.net/) installed, so we need to get that installed first. Thankfully, Synology provides a set of "Diagnostic" tools (`synogear`) that come with [`ncat`](https://nmap.org/ncat/), which is an improved version of the standard `nc`. To install the `synogear` utilities from Synology. First connect to your NAS over SSH then, as root check to see if the tools are installed:

```shell
# synogear list
```

If the tools have not been installed before you will see the following output:

```shell
Tools are not installed yet. You can run this command to install it:
   synogear install
```

If the `synogear` tools have been installed before, but this is your first time loading them this session you will see the following:

```shell
Tools are installed but not loaded yet. You can run this command to load it:
   synogear install
```

Next is to load or install the tools and check again:

```
# synogear install
# synogear list
All tools:
addr2line              eu-readelf    ld               pkill             strings
addr2name              eu-size       ld.bfd           pmap              strip
ar                     eu-stack      ldd              ps                sysctl
arping                 eu-strings    log-analyzer.sh  pstree            sysstat
as                     eu-strip      lsof             pwdx              tcpdump_wrapper
autojump               eu-unstrip    ltrace           ranlib            tcpspray
capsh                  file          mpstat           rarpd             tcpspray6
c++filt                fio           name2addr        rdisc             tcptraceroute6
cifsiostat             fix_idmap.sh  ncat             rdisc6            telnet
clockdiff              free          ndisc6           readelf           tload
dig                    gcore         nethogs          rltraceroute6     tmux
domain_test.sh         gdb           nm               sa1               top
elfedit                gdbserver     nmap             sa2               tracepath
eu-addr2line           getcap        nping            sadc              traceroute6
eu-ar                  getpcaps      nslookup         sadf              tracert6
eu-elfcmp              gprof         objcopy          sar               uptime
eu-elfcompress         iftop         objdump          setcap            vmstat
eu-elflint             iostat        perf-check.py    sid2ugid.sh       w
eu-findtextrel         iotop         pgrep            size              watch
eu-make-debug-archive  iperf         pidof            slabtop           zblacklist
eu-nm                  iperf3        pidstat          sockstat          zmap
eu-objdump             kill          ping             speedtest-cli.py  ztee
eu-ranlib              killall       ping6            strace
```

## Sending and Receiving using `tar` and `ncat` (`ncat`, etc)

Unix like systems allow us to pipeline the output of one command to the input of another, which is what makes this possible. We take the output of the `tar` archiving utility and push it into `netcat` to move over the network.

> **Note:** The following commands assume you have logged into your NAS over SSH and have [loaded the `synogear` tools](#installing-ncat-on-synology).

### Sending

The command for sending looks like this:

```shell
tar -cf - * | /var/packages/DiagnosisTool/target/tool/ncat destinationHost 7000
```

Which breaks down like this:

* Have `tar` create (`-c`) an archive file (`f`) to stdout (`-`) of all files (`*`)
* Send the output to the `ncat` utility
* Have `ncat` send the data to the remote host (`destinationHost`) over port 7000

> **Note:** If successful this command does not produce output. You can add `-v` to the `tar` command to get some feedback. However, doing so produces a lot of noise and is not a great indicator of progress.

### Receiving

The command for receiving looks like this:

```shell
/var/packages/DiagnosisTool/target/tool/ncat -l -p 7000 | tar -xpf -
```

Which breaks down like this:

* Run the `ncat` utility, telling it to listen (`-l`) on port 7000 (`-p 7000`)
* Take whatever output comes over port 7000 and pass it to the `tar` program
* Have `tar` extract (`-x`) what it receives, preserve (`-p`) permissions, from the 'file' (`-f`) stdin (`-`).

> **Note:** If successful this command does not produce output. You can add `-v` to the `tar` command to get some feedback. However, doing so produces a lot of noise and is not a great indicator of progress.

## Extra

One of the things I dislike about this method is there is not a *good* way to see progress. Depending on the source / destination you may be able to use the `pv` or Pipe Viewer utility. Like this:

Sending:

```shell
tar -cf - * | pv | nc destinationHost 7000
```

Receiving:

```shell
ncat -l -p 7000 | pv | tar -xpf -
```

Both cases will produce output that indicates how much data has been sent, the duration, and current speed:

```shell
ncat -l -p 7000 | pv | tar -xpf -
19.08 MB 0:00:04 5.15 MB/s] [<=>                                                                        ]
```

## Resources

* [Install `synogear` tools](https://github.com/SynoCommunity/spksrc/wiki/FAQ-synogear)
* [`tar` over `nc` syntax](http://toast.djw.org.uk/tarpipe.html)
