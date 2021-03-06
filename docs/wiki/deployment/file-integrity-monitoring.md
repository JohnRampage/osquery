As of osquery version 1.4.2, file integrity monitoring support was introduced
for the Linux (using inotify) and Darwin (using FSEvents) platforms.  This module reads a list of directories to
monitor from the osquery config and details changes and hashes to those
selected files in the [`file_events`](https://osquery.io/docs/tables/#file_events) table.

To get started with FIM (file integrity monitoring), you must first identify
which files and directories you wish to monitor. Then use *fnmatch*-style, or filesystem globbing, patterns to represent the target paths. You may use standard wildcards "*\**" or SQL-style wildcards "*%*":

**Matching wildcard rules**

* `%`: Match all files and folders for one level.
* `%%`: Match all files and folders recursively.
* `%abc`: Match all within-level ending in "abc".
* `abc%`: Match all within-level starting with "abc".

**Matching examples**

* `/Users/%/Library`: Monitor for changes to every user's Library folder.
* `/Users/%/Library/`: Monitor for changes to files within each Library folder.
* `/Users/%/Library/%`: Same, changes to files within each Library folder.
* `/Users/%/Library/%%`: Monitor changes recursively within each Library.
* `/bin/%sh`: Monitor the *bin* directory for changes ending in *sh*.

For example, you may want to monitor `/etc` along with other files on a Linux
system. After you identify your target files and directories you wish to monitor, add them to a new section in the config *file_paths*.

## Example FIM Config

```json
{
  "schedule": {
    "crontab": {
      "query": "select * from crontab;",
      "interval": 300
    },
    "file_events": {
      "query": "select * from file_events;",
      "removed": false,
      "interval": 300
    }
  },
  "file_paths": {
    "homes": [
      "/root/.ssh/%%",
      "/home/%/.ssh/%%"
    ],
    "etc": [
      "/etc/%%"
    ],
    "tmp": [
      "/tmp/%%"
    ]
  }
}
```

## Sample Event Output

As file changes happen, events will appear in the [**file_events**](https://osquery.io/docs/tables/#file_events) table.  During
a file change event, the md5, sha1, and sha256 for the file will be calculated
if possible.  A sample event looks like this:

```json
{
  "action":"ATTRIBUTES_MODIFIED",
  "category":"homes",
  "md5":"bf3c734e1e161d739d5bf436572c32bf",
  "sha1":"9773cf934440b7f121344c253a25ae6eac3e3182",
  "sha256":"d0d3bf53d6ae228122136f11414baabcdc3d52a7db9736dd256ad81229c8bfac",
  "target_path":"\/root\/.ssh\/authorized_keys",
  "time":"1429208712",
  "transaction_id":"0"
}
```

## Tuning Linux inotify limits

For Linux, osquery uses inotify to subscribe to file changes at the kernel
level for performance.  This introduces some limitations on the number of files
that can be monitored since each inotify watch takes up memory in kernel space
(non-swappable memory).  Adjusting your limits accordingly can help increase
the file limit at a cost of kernel memory.

### Example sysctl.conf modifications

```
#/proc/sys/fs/inotify/max_user_watches = 8192
fs.inotify.max_user_watches = 524288

#/proc/sys/fs/inotify/max_user_instances = 128
fs.inotify.max_user_instances = 256

#/proc/sys/fs/inotify/max_queued_events = 16384
fs.inotify.max_queued_events = 32768
```
