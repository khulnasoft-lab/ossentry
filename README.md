# ossentry

[![Actions Status](https://github.com/khulnasoft-lab/ossentry/workflows/Go/badge.svg)](https://github.com/khulnasoft-lab/ossentry/actions)
[![Go Report](https://goreportcard.com/badge/github.com/khulnasoft-lab/ossentry)](https://goreportcard.com/badge/github.com/khulnasoft-lab/ossentry)
[![Latest Release](https://img.shields.io/github/v/release/khulnasoft-lab/ossentry?include_prereleases)](https://github.com/khulnasoft-lab/ossentry/releases/latest)
[![stable](http://badges.github.io/stability-badges/dist/stable.svg)](http://github.com/badges/stability-badges)

A swiss-army tool for testing, creating, and manipulating [osquery](https://osquery.io/) query packs.

![ossentry logo](images/logo-small.png?raw=true "ossentry logo")

## Installation

With [Go](https://go.dev/) v1.20+ installed, run:

```shell
go install github.com/khulnasoft-lab/ossentry/cmd/ossentry@latest
```

## Usage

ossentry supports 4 commands:

* `apply` - programatically manipulate an osquery query pack, for instance, adjusting intervals
* `pack` - create a JSON pack file from a directory of raw SQL files
* `unpack` - extract raw SQL files from a JSON query pack file
* `run` - run an osquery pack file or directory of SQL queries with human and diff-friendly output
* `verify` - verify that the queries in a query pack, directory, or raw SQL file are valid and test well

### apply

Want to take an osquery pack from the internet, but make changes to it programatically? ossentry has you covered:

```shell
curl https://raw.githubusercontent.com/osquery/osquery/master/packs/it-compliance.conf \
  | ossentry --min-interval=8h --platforms linux,posix --exclude os_version apply -
```

This will set all queries to an 8-hour interval, remove Windows-specific queries, and exclude a query named `os_version`.

### Pack

Create an osquery pack configuration from a recursive directory of SQL files:

```shell
ossentry pack /tmp/osx-attacks
```

Here's the example output:

```json
{
  "queries": {
    "Aobo_Keylogger": {
      "query": "select * from launchd where name like 'com.ab.kl%.plist';",
      "interval": "3600",
      "version": "1.4.5",
      "description": "(http://aobo.cc/aobo-mac-os-x-keylogger.html)",
      "value": "Artifact used by this malware"
    },
    "Backdoor_MAC_Eleanor": {
      "query": "SELECT * FROM launchd WHERE name IN ('com.getdropbox.dropbox.integritycheck.plist','com.getdropbox.dropbox.timegrabber.plist','com.getdropbox.dropbox.usercontent.plist');",
      "interval": "3600",
      "version": "1.4.5",
      "description": "(https://blog.malwarebytes.com/cybercrime/2016/07/new-mac-backdoor-malware-eleanor/)",
      "value": "Artifact used by this malware"
    },
...
```

The `pack` command supports the same flags as the `apply` command. In particular, you may find `--exclude`, `--exclude-tags`, and `--verify` useful.

### Run

Run a set of queries!

```shell
ossentry run incident-response.conf
```

Example output:

```log
block_devices (7 rows)
----------------------
block_size:512 label: model: name:/dev/nvme0n1 parent: size:488397168 type: uuid: vendor:
block_size: label: model: name:/dev/nvme0n1p1 parent:/dev/nvme0n1 size:614400 type: uuid: vendor:
block_size: label: model: name:/dev/nvme0n1p2 parent:/dev/nvme0n1 size:415929892 type: uuid: vendor:
block_size: label: model: name:/dev/nvme0n1p3 parent:/dev/nvme0n1 size:71843677 type: uuid: vendor:
block_size:512 label: model:'SD Card Reader' name:/dev/sda parent: size:0 type: uuid: vendor:CalDigit
block_size:512 label: model: name:/dev/dm-0 parent: size:415925796 type: uuid: vendor:
block_size:512 label: model: name:/dev/dm-1 parent: size:71839581 type: uuid: vendor:

crontab (1 rows)
----------------
command:'root run-parts /etc/cron.hourly' day_of_month:* day_of_week:* event: hour:* minute:01 month:* path:/etc/cron.d/0hourly

disk_encryption (0 rows)
```

### Unpack

Extract an osquery pack into a directory of SQL files:

```shell
ossentry --output=/tmp/osx-attacks unpack osx-attacks.conf
```

Here is example output:

```log
Writing 745 bytes to /tmp/out/OceanLotus_dropped_file_1.sql ...
Writing 268 bytes to /tmp/out/OSX_MaMi_DNS_Servers.sql ...
Writing 328 bytes to /tmp/out/OSX_ColdRoot_RAT_Files.sql ...
Writing 209 bytes to /tmp/out/iWorm.sql ...
74 queries saved to /tmp/out
```

The `unpack` command supports the same flags as the `apply` command.


### Verify

Verify that the queries are valid in a pack, SQL file, or directory of SQL files

```shell
ossentry verify /tmp/detect
```

Example output:

```log
Verifying "high-disk-bytes-written" ...
high-disk-bytes-written" returned 0 rows within 264.361831ms
Verifying "unexpected-shell-parents" ...
"unexpected-shell-parents" failed validation: /sbin/osqueryi --json [exit status 1]: Error: near line 1: near "sh": syntax error
78 queries found: 55 verified, 10 errored, 13 skipped
"verify" failed: 10 errors occurred:
 * xprotect-reports: /sbin/osqueryi --json [exit status 1]: Error: near line 1: no such table: xprotect_reports
```

You can set limits on the number of rows returned, amount of runtime per query, per day, or across the pack, see `--help` for more information.

### Common Flags

Here are the options that are available to `apply`, `unpack`, `pack`, and `verify`

```
  -default-interval duration
    	Interval to use for queries which do not specify one (default 1h0m0s)
  -exclude string
    	Comma-separated list of queries to exclude
  -exclude-tags string
    	Comma-separated list of tags to exclude (default "disabled")
  -max-interval duration
    	Queries can't be scheduled more often than this (default 15s)
  -max-query-daily-duration duration
    	Maximum duration for a single query multiplied by how many times it runs daily (checked during --verify) (default 1h0m0s)
  -max-query-duration duration
    	Maximum query duration (checked during --verify) (default 4s)
  -max-results int
    	Maximum number of results a query may return during verify (default 1000)
  -max-total-daily-duration duration
    	Maximum total query-duration per day across all queries (default 6h0m0s)
  -min-interval duration
    	Queries cant be scheduled less often than this (default 24h0m0s)
  -multi-line
    	output queries is multi-line form. This is accepted by osquery, but technically is invalid JSON.
  -output string
    	Location of output
  -platforms string
    	Comma-separated list of platforms to include
  -single-quotes
    	Render double quotes as single quotes (may corrupt queries)
  -skip_headers
    	If true, avoid header prefixes in the log messages
  -tag-intervals string
    	modifiers to the default-interval based on query tags (default "transient=5m,postmortem=6h,rapid=15s,often=x/4,seldom=2x")
  -verify
    	Verify the output
  -workers int
      Number of workers to use when verifying results (0 for automatic)
```

At the moment, flags must be declared before the subcommand. `¯\_(ツ)_/¯`
