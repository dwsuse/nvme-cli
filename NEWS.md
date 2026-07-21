<!-- SPDX-License-Identifier: GPL-2.0-only -->
# NEWS

> This file tracks user-facing changes for each release: new
> features, behavior changes, and anything that could break an
> existing setup. Loosely modeled on systemd's NEWS file
> (https://github.com/systemd/systemd/blob/main/NEWS): one prose
> paragraph per entry, "Feature removals and incompatible changes"
> listed first in each release section since that's what someone
> upgrading most needs to see, then changes grouped by component.
> Add an entry here alongside the change that introduces it, not
> after the fact at release time -- it's much easier to describe a
> change accurately while it's fresh than to reconstruct it later
> from a commit log.
>
> Lines in this file are wrapped at ~75 columns on purpose, unlike
> our other markdown docs: this file is meant to still read cleanly
> in a plain editor or if a section is copy-pasted into an email,
> not only when viewed through a renderer.

## Changes in 3.0 (unreleased)

### Feature removals and incompatible changes

* `nvme disconnect-all` without options no longer disconnects all fabric
  controllers. It now only disconnects controllers with no recorded owner
  in the new ownership registry. Use `--force` to restore the legacy
  disconnect-all behavior, or `--owner NAME` to filter by a specific owner.

* New configuration format. Users must convert `config.json` and
  `discovery.conf` to the new INI format. nvme-cli support migrating
  existing configuration with `nvme config convert`. If not done
  explicitly, nvme-cli will auto convert with the first fabric command
  execute. The existing configuration file will not be removed but it is
  strongly recommended to remove it when there is no need for it anymore,
  e.g. no downgrading of nvme-cli needed. See man pages for more details.

* Default installations no longer ship the stub `/etc/nvme/discovery.conf`
  template file.

### New: ownership registry and exclusion list

* Added an ownership registry (`nvme registry`, backed by
  `/run/nvme/registry/`) to track orchestrator and process ownership of
  connected controllers, preventing accidental teardowns by competing
  services.

* Added a system-wide exclusion list (`nvme exclusion`, backed by
  `/etc/nvme/exclusions.conf` and `exclusions.conf.d/` drop-ins) allowing
  administrators to block specific controllers by transport, address, or
  NQN from auto-connecting.

### New: Windows support

* Added Windows (MSYS2/UCRT64) build and execution support for `nvme-cli`,
  `libnvme`, and most vendor plugins (excluding fabrics support).

### nvme-cli

* Added `--idempotent` and `--devid-file` flags to `nvme connect`.
  `--idempotent` allows connections to existing controllers without
  throwing an error, while `--devid-file` outputs the assigned `nvmeX`
  device name upon success.

* Updated `nvme disconnect` to accept full connection-matching parameters
  (transport, address, NQN) consistent with `nvme connect`.

* Introduced `nvme top`, an interactive dashboard for real-time monitoring
  of NVMe subsystems, paths, multipath distribution, and diagnostic
  counters.

* Added `nvme utils dump-command-metadata` to export the live command tree
  and options in JSON format.

### libnvme

* Introduced an INI-format configuration parser (`nvme-fabrics.conf` +
  drop-ins) replacing legacy JSON/discovery connection formats.

* Refactored fabrics protocol logic out of `nvme-cli` and into `libnvme` to
  streamline implementation across external consumers.

* Unified 32-bit and 64-bit passthru command structures and functions into
  a single 64-bit layout (`struct nvme_passthru_cmd`).

* Added read-only Python bindings for parsing and validating the new INI
  configuration format.

* Reworked SWIG Python bindings and modernized internal accessor generation
  tools.

* Merged `libnvme-mi` into `libnvme` and renamed the unified package to
  **libnvme3** (`libnvme3.so.1`), with headers installed under
  `include/libnvme3/`.

* Split `<nvme/nvme-cmds.h>` and `<nvme/nvme-types.h>` into domain-specific
  sub-headers (e.g., base, fabrics, MI). The top-level headers remain
  intact for backward compatibility.

* Replaced `nvme_root_t` with `struct libnvme_global_ctx *` across the
  public API.

* Updated `libnvme_create_global_ctx()` to require explicit
  post-construction setter calls for logging configuration.

* Introduced `struct nvme_transport_handle` to abstract command execution
  across direct `ioctl` file descriptors and MI endpoints.

* Added dedicated asynchronous passthru APIs (`libnvme_submit_*`,
  `libnvme_wait_*`, `libnvme_reap_*`) alongside synchronous execution
  interfaces. Depends on io_uring support.

* Decoupled passthru command construction (`nvme_init_*`) from command
  submission (`libnvme_exec_*` / `libnvme_submit_*`).

* Added real-time, non-cached diagnostic counter accessors for sysfs
  statistics across controllers, namespaces, and paths.

* Promoted `libnvmf_host_get_ids()` to a public API for resolving host
  identity prior to connection setup.

* Removed implicit host identity lookups from fabrics connection routines.
  Callers must resolve or set `hostnqn` and `hostid` explicitly.

* Refactored internal fabrics routing through a context hook structure to
  allow custom caller policies.

* Updated `libnvmf_read_hostnqn()` and `libnvmf_read_hostid()` to require a
  `struct libnvme_global_ctx *` parameter.

* Added global context setters (`libnvme_global_ctx_set_hostnqn()`,
  `libnvme_global_ctx_set_hostid()`) to define fallback host identity defaults.

* Removed legacy `sizeof_args` compatibility macros and struct-based
  command shims.

* Removed platform-specific filter helpers (`nvme_filter_*`) from the
  public API in favor of `libnvme_scan_*` interfaces.

* Removed unused internal functions `libnvme_ctrl_match_config()` and
  `libnvmf_ctrl_get_fabrics_config()`.

* Removed `nvme_mi_ctrl_t` and MI-specific identify helpers in favor of the
  unified `struct nvme_transport_handle` interface.

* Removed legacy definitions, deprecated status code aliases, and telemetry
  function names.

* Removed environment variable configuration flags (`LIBNVME_*`) in favor
  of explicit context setter API calls.

> Open question, not yet decided: whether to also generate a
> mechanically-produced contributor list per release (roughly `git
> shortlog -sn <previous-tag>..HEAD`), the way systemd's own
> announcement emails do. Cheap to produce, credits everyone who
> touched the release.
