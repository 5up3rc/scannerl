# Scannerl

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)
[![AUR](https://img.shields.io/aur/version/scannerl.svg)](https://aur.archlinux.org/packages/scannerl)

[Scannerl](https://github.com/kudelskisecurity/scannerl) is a modular distributed fingerprinting engine
implemented by [Kudelski Security](https://www.kudelskisecurity.com/).
Scannerl can fingerprint thousands of targets on a single host, but can just as easily be distributed
across multiple hosts. Scannerl is to fingerprinting what zmap is to port scanning.

Scannerl works on Debian/Ubuntu/Arch (but will probably work on
other distributions as well). It uses a master/slave architecture where
the master node will distribute the work (host(s) to fingerprint) to
its slaves (local or remote). The entire deployment is transparent to
the user.

# Why use Scannerl

When using conventional fingerprinting tools for large-scale analysis,
security researchers will often hit two limitations: first, these tools are typically built
for scanning comparatively few hosts at a time and are inappropriate for large
ranges of IP addresses. Second, if large range of IP addresses
protected by IPS devices are being fingerprinted, the probability of being
blacklisted is higher what could lead to an incomplete set of information.
Scannerl is designed to circumvent these limitations, not only by providing the
ability to fingerprint multiple hosts simultaneously, but also by distributing
the load across an arbitrary number of hosts.
Scannerl also makes the distribution of these tasks completely transparent,
which makes setup and maintenance of large-scale fingerprinting projects
trivial; this allows to focus on the analyses rather than the herculean
task of managing and distributing fingerprinting processes by hand.
In addition to the speed factor, scannerl has been designed to allow to
easily set up specific fingerprinting analyses in a few lines of code.
Not only is the creation of a fingerprinting cluster easy to set up, but it can be tweaked
by adding fine-tuned scans to your fingerprinting campaigns.

It is the fastest tool to perform large scale fingerprinting campaigns.

For more:

* [Fingerprint all the things with scannerl at BlackAlps](https://youtu.be/xHF2T5E7OFQ)
* [Fingerprinting MySQL with scannerl](https://research.kudelskisecurity.com/2017/12/11/fingerprinting-mysql-with-scannerl/)
* [Fingerprint ICS/Scada with scannerl](https://research.kudelskisecurity.com/2017/12/13/scannerl-ics-modules-open-source/)
* [Distributed fingerprinting with scannerl](https://research.kudelskisecurity.com/2017/06/06/distributed-fingerprinting-with-scannerl/)
* [6 months of ICS scanning](https://research.kudelskisecurity.com/2017/10/24/6-months-of-ics-scanning/)

---

**Table of Contents**

* [Installation](#installation)
* [Usage](#usage)

  * [Standalone](#standalone-usage)
  * [Distributed](#distributed-usage)
  * [Available modules](#list-available-modules)
  * [Module arguments](#modules-arguments)
  * [Result format](#result-format)

* [Extending Scannerl](#extending-scannerl)
* [Contributing](#contributing)
* [License and Copyright](#license-and-copyright)

See the [wiki](https://github.com/kudelskisecurity/scannerl/wiki) for more.

# Installation

See the different installation options under [wiki installation page](https://github.com/kudelskisecurity/scannerl/wiki/Installation)

To install from source, first install Erlang (at least v.18) by choosing the right packaging for your
platform: [Erlang downloads](https://www.erlang-solutions.com/resources/download.html)

Install the required packages:
```bash
# on debian
$ sudo apt install erlang erlang-src rebar

# on arch
$ sudo pacman -S erlang-nox rebar
```

Then build scannerl:

```bash
$ git clone https://github.com/kudelskisecurity/scannerl.git
$ cd scannerl
$ ./build.sh
```

Get the usage by running
```bash
$ ./scannerl -h
```

Scannerl is available on aur for arch linux users
* [scannerl](https://aur.archlinux.org/packages/scannerl/)
* [scannerl-git](https://aur.archlinux.org/packages/scannerl-git/)

DEBs (Ubuntu, Debian) are available in the [releases](https://github.com/kudelskisecurity/scannerl/releases).

RPMs (Opensuse, Centos, Redhat) are available under https://build.opensuse.org/package/show/home:chapeaurouge/scannerl.

## Distributed setup

Two types of nodes are needed to perform a distributed scan:

* **Master node**: this is where scannerl's binary is run
* **Slave node(s)**: this is where scannerl will connect to
  distribute all its work

The  master node needs to have scannerl installed and compiled while the
slave node(s) only needs Erlang to be installed. The entire setup is
transparent and done automatically by the master node.

Requirements for a distributed scan:

* All hosts have the same version of Erlang installed
* All hosts are able to connect to each other using SSH public key
* All hosts' names resolve (use */etc/hosts* if no proper DNS is setup)
* All hosts have the same [Erlang security cookie](http://erlang.org/doc/reference_manual/distributed.html)
* All hosts must allow connection to Erlang EPMD port (TCP/4369)
* All hosts have the following range of ports opened: TCP/11100 to TCP/11100 + *number-of-slaves*

# Usage

```
$ ./scannerl -h
   ____   ____    _    _   _ _   _ _____ ____  _
  / ___| / ___|  / \  | \ | | \ | | ____|  _ \| |
  \___ \| |     / _ \ |  \| |  \| |  _| | |_) | |
   ___) | |___ / ___ \| |\  | |\  | |___|  _ <| |___
  |____/ \____/_/   \_\_| \_|_| \_|_____|_| \_\_____|

USAGE
  scannerl MODULE TARGETS [NODES] [OPTIONS]

  MODULE:
    -m <mod> --module <mod>
      mod: the fingerprinting module to use.
           arguments are separated with a colon.

  TARGETS:
    -f <target> --target <target>
      target: a list of target separated by a comma.
    -F <path> --target-file <path>
      path: the path of the file containing one target per line.
    -d <domain> --domain <domain>
      domain: a list of domains separated by a comma.
    -D <path> --domain-file <path>
      path: the path of the file containing one domain per line.

  NODES:
    -s <node> --slave <node>
      node: a list of node (hostnames not IPs) separated by a comma.
    -S <path> --slave-file <path>
      path: the path of the file containing one node per line.
            a node can also be supplied with a multiplier (<node>*<nb>).

  OPTIONS:
    -o <mod> --output <mod>     comma separated list of output module(s) to use.
    -p <port> --port <port>     the port to fingerprint.
    -t <sec> --timeout <sec>    the fingerprinting process timeout.
    -T <sec> --stimeout <sec>   slave connection timeout (default: 10).
    -j <nb> --max-pkt <nb>      max pkt to receive (int or "infinity").
    -r <nb> --retry <nb>        retry counter (default: 0).
    -c <cidr> --prefix <cidr>   sub-divide range with prefix > cidr (default: 24).
    -M <port> --message <port>  port to listen for message (default: 57005).
    -P <nb> --process <nb>      max simultaneous process per node (default: 28232).
    -Q <nb> --queue <nb>        max nb unprocessed results in queue (default: infinity).
    -C <path> --config <path>   read arguments from file, one per line.
    -O <mode> --outmode <mode>  0: on Master, 1: on slave, >1: on broker (default: 0).
    -v <val> --verbose <val>    be verbose (0 <= int <= 255).
    -K <opt> --socket <opt>     comma separated socket option (key[:value]).
    -l --list-modules           list available fp/out modules.
    -V --list-debug             list available debug options.
    -A --print-args             Output the args record.
    -X --priv-ports             use only source port between 1 and 1024.
    -N --nosafe                 keep going even if some slaves fail to start.
    -w --www                    DNS will try for www.<domain>.
    -b --progress               show progress.
    -x --dryrun                 dry run.
```

See the [wiki](https://github.com/kudelskisecurity/scannerl/wiki) for more.

## Standalone usage

Scannerl can be used on the local host without any other host.
However, it will still create a slave node on the same host it is run from.
Therefore, the requirements described in [Distributed setup](#distributed-setup)
must also be met.

A quick way to do this is to make sure your host is able to resolve itself with
```bash
grep -q "127.0.1.1\s*`hostname`" /etc/hosts || echo "127.0.1.1 `hostname`" | sudo tee -a /etc/hosts
```

and create an SSH key (if not yet present) and add it to the `authorized_keys` (you need
an SSH server running):
```bash
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
```

The following example runs an HTTP banner grabing on *google.com* from localhost
```bash
./scannerl -m httpbg -d google.com
```

## Distributed usage

In order to perform a distributed scan, one need to pre-setup the hosts
that will be used by scannerl to distribute the work.
See [Distributed setup](#distributed-setup) for more information.

Scannerl expects a list of slaves to use (provided by the **-s** or
**-S** switches).

```bash
./scannerl -m httpbg -d google.com -s host1,host2,host3
```

## List available modules

Scannerl will list the available modules (output modules as well as
fingerprinting modules) with the **-l** switch:

```bash
$ ./scannerl -l

Fingerprinting modules available
================================

bacnet             UDP/47808: Bacnet identification
chargen            UDP/19: Chargen amplification factor identification
fox                TCP/1911: FOX identification
httpbg             TCP/80: HTTP Server header identification
                     - Arg1: [true|false] follow redirection [Default:false]
httpsbg            SSL/443: HTTPS Server header identification
modbus             TCP/502: Modbus identification
mqtt               TCP/1883: MQTT identification
mqtts              TCP/8883: MQTT over SSL identification
mysql_greeting     TCP/3306: Mysql version identification

Output modules available
========================

csv                output to csv
                     - Arg1: [true|false] save everything [Default:true]
csvfile            output to csv file
                     - Arg1: [true|false] save everything [Default:false]
                     - Arg2: File path
file               output to file
                     - Arg1: File path
file_ip            output to stdout (only ip)
                     - Arg1: File path
file_mini          output to file (only ip and result)
file_resultonly    output to file (only result)
stdout             output to stdout
stdout_ip          output to stdout (only IP)
stdout_mini        output to stdout (only ip and result)
```

## Modules arguments

Arguments can be provided to modules with a colon. For
example for the *file* output module:
```bash
./scannerl -m httpbg -d google.com -o file:/tmp/result
```

## Result format

The result returned by scannerl to the output modules
has the following form:

```
{module, target, port, result}
```

Where

* `module`: the module used (Erlang atom)
* `target`: IP or hostname (string or IPv4 address)
* `port`: the port (integer)
* `result`: see below

The `result` part is of the form:

```
{{status, type},Value}
```

Where `{status, type}` is one of the following tuples:

* `{ok, result}`: fingerprinting the target succeeded
* `{error, up}`: fingerprinting didn't succeed but the target responded
* `{error, unknown}`: fingerprinting failed

`Value` is the returned value - it is either an atom or a list of element

# Extending Scannerl

Scannerl has been designed and implemented with modularity in mind.
It is easy to add new modules to it:

* **Fingerprinting module**: to query a specific protocol or service.
  As an example, the *fp_httpbg.erl* module allows to retrieve the *server*
  entry in the HTTP response.
* **Output module**: to output to a specific database/filesystem or output the
  result in a specific format.
  For example, the *out_file.erl* and *out_stdout.erl* modules allow
  respectively to output to a file or to stdout (default behavior if not specified).

To create new modules, simply follow the behavior (*fp_module.erl* for fingerprinting
modules and *out_behavior.erl* for output module) and implement your modules.

New modules can either be added at compile time or dynamically as an external file.

See the [wiki page](https://github.com/kudelskisecurity/scannerl/wiki/Module-Implementation) for more.

# Contributing

Feel free to open an issue or a PR.

# License and Copyright

Copyright(c) 2017 Nagravision SA.

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License version 3 as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

