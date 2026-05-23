# Ansible Role: `chrony`

Install and configure **Chrony** on Linux distributions.

This role is designed for:

- package installation
- configuration management
- multi-distribution support across Ubuntu, Debian, and Fedora
- container-friendly testing (configuration-focused, not runtime-timekeeping validation)

> **Important:** Chrony itself is not meaningfully testable as a running time service inside standard containers because it depends on system/kernel time facilities. For this reason, the primary Molecule scenario for this role is the **default** scenario, which validates installation, rendering, and idempotency.

---

## What the role does

The role performs the following high-level steps:

1. Validates public inputs using `meta/argument_specs.yml` and additional cross-field checks in `tasks/validate_extra.yml`.
2. Refreshes the APT cache on Debian/Ubuntu hosts before installation.
3. Installs the Chrony package using the OS-native package manager.
4. Detects the target Chrony configuration layout:
   - classic single-file layout
   - `conf.d` layout
   - `sources.d` layout
5. Renders the main Chrony configuration and any supported drop-in files.
6. Optionally manages the Chrony service.

The role currently auto-detects container virtualization and skips service management in container contexts when appropriate.

---

## Requirements

- Ansible **2.14** or newer
- Supported operating systems are generated at release time from the Molecule matrix in `molecule/shared/vars.yml`

---

## Supported platforms

The `meta/main.yml` file is generated from the platform matrix stored in:

```yaml
molecule/shared/vars.yml
```

That file is the **source of truth** for:

- Molecule inventory generation
- tested OS matrix
- generated Galaxy metadata (`meta/main.yml`)

---

## Public role variables

The sections below are derived from the role defaults, comments, and validation rules.

### Required source configuration

At least one of the following must be provided with **one or more entries**:

```yaml
chrony_servers:
  - 2.fedora.pool.ntp.org iburst
```

or

```yaml
chrony_pools:
  - pool.ntp.org          iburst maxsources 4
  - ntp.ubuntu.com        iburst maxsources 4
  - 0.ubuntu.pool.ntp.org iburst maxsources 1
  - 1.ubuntu.pool.ntp.org iburst maxsources 1
  - 2.ubuntu.pool.ntp.org iburst maxsources 2
```

Validation rules enforce that at least one of `chrony_servers` or `chrony_pools` is provided.

---

### Common settings from `defaults/main.yml`

#### `chrony_debug`

Enable additional debug output.

```yaml
chrony_debug: false
```

#### `chrony_manage_service`

Controls whether the role attempts to manage the Chrony service.

```yaml
chrony_manage_service: true
```

> In container environments, the role may still skip service management automatically.

#### `chrony_dhcp_sourcedir`

Directory used for DHCP-provided Chrony source files.

```yaml
chrony_dhcp_sourcedir: /run/chrony-dhcp
```

#### `chrony_driftfile`

Path to the Chrony drift file.

```yaml
chrony_driftfile: /var/lib/chrony/drift
```

#### `chrony_makestep`

Controls when Chrony is allowed to step the system clock.

```yaml
chrony_makestep: 1.0 3
```

Expected format:

```text
<threshold> <limit>
```

Examples:

```yaml
chrony_makestep: 1.0 3
chrony_makestep: 0.5 5
```

#### `chrony_maxupdateskew`

Maximum allowed clock skew estimate.

```yaml
chrony_maxupdateskew: 100.0
```

#### `chrony_enable_rtcsync`

Enable kernel synchronization of the hardware RTC.

```yaml
chrony_enable_rtcsync: true
```

#### `chrony_keyfile`

Path to the Chrony keys file.

```yaml
chrony_keyfile: "{{ chrony_etc_path }}/chrony.keys"
```

#### `chrony_ntsdumpdir`

Directory used to persist NTS keys and cookies.

```yaml
chrony_ntsdumpdir: /var/lib/chrony
```

#### `chrony_leapsecmode`

How leap seconds should be handled.

```yaml
chrony_leapsecmode: slew
```

Allowed values:

- `system`
- `step`
- `slew`
- `ignore`

#### `chrony_leapsectz`

Timezone database source for TAI-UTC offset / leap second information.

```yaml
chrony_leapsectz: right/UTC
```

#### `chrony_logdir`

Directory where Chrony logs should be written.

```yaml
chrony_logdir: /var/log/chrony
```

---

### Optional variables documented in `defaults/main.yml` comments

These options are supported by the templates and validation logic when defined.

#### `chrony_sourcedir`

Optional sourcedir directives for additional NTP source files.

Example:

```yaml
chrony_sourcedir:
  - /run/chrony-dhcp
```

> Note: the role defaults define `chrony_dhcp_sourcedir`, while the templates also reference `chrony_sourcedir`. If you plan to use `chrony_sourcedir` explicitly, keep the distinction in mind.

#### `chrony_hwtimestamp`

Enable hardware timestamping on compatible interfaces.

Example:

```yaml
chrony_hwtimestamp: "*"
```

#### `chrony_minsources`

Increase the minimum number of selectable sources required to adjust the clock.

Example:

```yaml
chrony_minsources: 2
```

#### `chrony_allow`

Allow NTP client access from selected networks.

Example:

```yaml
chrony_allow:
  - 192.168.0.0/16
```

#### `chrony_local`

Serve time even if not synchronized to an external source.

Example:

```yaml
chrony_local: stratum 10
```

#### `chrony_authselectmode`

Require authentication for all NTP sources.

Allowed values:

- `require`
- `prefer`
- `mix`
- `ignore`

Example:

```yaml
chrony_authselectmode: require
```

#### `chrony_log`

Select which Chrony information should be logged.

Example:

```yaml
chrony_log: measurements statistics tracking
```

#### `chrony_generic_settings`

Generic escape hatch for supported directives that do not have dedicated variables.

Examples:

```yaml
chrony_generic_settings:
  - { key: manual }
  - { key: bindacqdevice, value: eth0, minversion: 0 }
  - { key: hwclockfile, value: /etc/adjtime }
```

Rules:

- `key` is required
- `value` is optional
- `minversion` is optional
- when `minversion` is defined, the directive is rendered only when the detected Chrony version is equal to or greater than that version

---

## Internal role variables (`vars/main.yml`)

These values are selected automatically and normally should **not** be overridden unless you have a very specific need.

### `chrony_service`

Internal service name used by the role.

```yaml
chrony_service: chronyd
```

### `chrony_packages`

OS-aware package mapping selected from `_chrony_packages`.

Current default mapping:

```yaml
_chrony_packages:
  default:
    - chrony
```

The role resolves package names in this order:

1. `<Distribution>-<Major>`
2. `<Distribution>`
3. `default`

### `chrony_etc_path`

OS-aware Chrony configuration base path selected from `_chrony_etc_path`.

Current mapping:

```yaml
_chrony_etc_path:
  default: /etc/chrony
  RedHat: /etc
  Fedora: /etc
  CentOS: /etc
  Rocky: /etc
```

### `chrony_dhcp_source_dir`

Internal runtime path for DHCP source files.

```yaml
chrony_dhcp_source_dir: "/run/chrony-dhcp"
```

### `chrony_cfg_mode`

File mode used for rendered configuration files.

```yaml
chrony_cfg_mode: "0644"
```

### `_container_types`

Container virtualization types used to decide whether service management should be skipped.

```yaml
_container_types:
  - docker
  - podman
  - lxc
  - containerd
  - container
```

---

## Validation behavior

The role validates inputs in two layers:

### 1. `meta/argument_specs.yml`
Provides type validation and choices for public variables such as:

- `chrony_servers`
- `chrony_pools`
- `chrony_driftfile`
- `chrony_logdir`
- `chrony_keyfile`
- `chrony_ntsdumpdir`
- `chrony_dhcp_sourcedir`
- `chrony_allow`
- `chrony_generic_settings`
- `chrony_makestep`
- `chrony_maxupdateskew`
- `chrony_enable_rtcsync`
- `chrony_minsources`
- `chrony_local`
- `chrony_leapsecmode`
- `chrony_authselectmode`
- `chrony_leapsectz`
- `chrony_log`

### 2. `tasks/validate_extra.yml`
Adds checks that are awkward to express in argument specs, including:

- requiring at least one of `chrony_servers` or `chrony_pools`
- validating `chrony_makestep` format
- validating path-like variables with a regex

---

## Example playbooks

### Basic example using pools

```yaml
- name: Configure Chrony
  hosts: all
  become: true
  roles:
    - role: guidugli.chrony
      vars:
        chrony_pools:
          - pool.ntp.org iburst maxsources 4
```

### Example using explicit servers and disabling service management

```yaml
- name: Configure Chrony without managing the service
  hosts: all
  become: true
  roles:
    - role: guidugli.chrony
      vars:
        chrony_manage_service: false
        chrony_servers:
          - time.google.com iburst
          - time.cloudflare.com iburst
```

### Example with additional generic directives

```yaml
- name: Configure Chrony with extra directives
  hosts: all
  become: true
  roles:
    - role: guidugli.chrony
      vars:
        chrony_pools:
          - pool.ntp.org iburst maxsources 4
        chrony_generic_settings:
          - key: manual
          - key: hwclockfile
            value: /etc/adjtime
```

---

## Templates and layout behavior

The role supports more than one Chrony layout.

It will automatically detect whether the target system supports:

- `chrony.conf` only
- `conf.d`
- `sources.d`

Then it renders the appropriate files:

- main config: `chrony.conf` or `chrony_with_conf_d.conf`
- `conf.d/01-ansible.conf` when `conf.d` exists
- `sources.d/01-ansible.sources` when `sources.d` exists

---

## Testing

This role uses **Molecule with Podman**.

### Primary scenario: `default`

```bash
molecule test -s default
```

This is the recommended test scenario for this role.

It validates:

- package installation
- configuration rendering
- idempotency

### Optional scenario: `systemd`

A `systemd` scenario exists in the repo, but it is optional for this role.

Because Chrony is not meaningfully runtime-testable inside regular containers, the **default** scenario is the authoritative one for release validation.

---

## Release metadata workflow

This repo uses a generated metadata approach.

### Source of truth

```text
molecule/shared/vars.yml
```

This file drives:

- tested platform matrix
- Molecule inventories
- generated `meta/main.yml`

### Refresh matrix and regenerate metadata

```bash
./scripts/update_release_metadata.sh
```

That script:

1. runs `scripts/update_matrix.py`
2. refreshes `molecule/shared/vars.yml`
3. regenerates Molecule inventories
4. renders `meta/main.yml` from `templates/meta_main.yml.j2`

### Recommended pre-release checks

```bash
git diff -- meta/main.yml molecule/
molecule test -s default
```

> `meta/main.yml` is generated. Do **not** edit it manually.

---

## Design principles

This role follows these principles:

- single source of truth for the platform matrix
- container-friendly test approach
- configuration-first validation
- minimal complexity where systemd adds no real value
- release-time metadata generation instead of manual drift

---

## License

MIT

---

## Author

Carlos Guidugli (`guidugli`)
