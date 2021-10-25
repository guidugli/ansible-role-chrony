Ansible Role: chrony
=========

An Ansible Role that install and configure chrony on RHEL/CentOS, Fedora and Debian/Ubuntu.

Requirements
------------

Operating system running on bare metal or on a hypervisor virtualization. Crony should not be set on containers.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    #chrony_servers:
    #  - 2.fedora.pool.ntp.org iburst

Specify individual servers to sync time. At least one server or pool must be informed.

    # Use public servers from the pool.ntp.org project.
    # Please consider joining the pool (https://www.pool.ntp.org/join.html).
    #chrony_pools:
    #  - pool.ntp.org          iburst maxsources 4
    #  - ntp.ubuntu.com        iburst maxsources 4
    #  - 0.ubuntu.pool.ntp.org iburst maxsources 1
    #  - 1.ubuntu.pool.ntp.org iburst maxsources 1
    #  - 2.ubuntu.pool.ntp.org iburst maxsources 2

Specify pool of servers to sync time. At least one server or pool must be informed.

    #chrony_sourcedir:
    #  - /run/chrony-dhcp

Use NTP servers from DHCP.
The sourcedir directive includes configuration files with the .sources suffix from a directory. They can only specify NTP sources i.e. use the server, pool, and peer directive), and can be reloaded by the reload sources command in chronyc. It is particularly useful with dynamic sources like NTP servers received from a DHCP server, which can be written to a file specific to the network interface by a networking script.

    chrony_driftfile: /var/lib/chrony/drift

Where to store the file that records the rate at which the system clock gains/losses time.


    chrony_makestep: 1.0 3

Allow the system clock to be stepped in the first three updates if its offset is larger than 1 second. This directive forces chronyd to step the system clock if the adjustment is larger than a threshold value, but only if there were no more clock updates since chronyd was started than a specified limit (a negative value can be used to disable the limit).
This is particularly useful when using reference clocks, because the initstepslew directive works only with NTP sources.
chrony_makestep: threshold limit

    #chrony_maxupdateskew: 100.0

Stop bad estimates upsetting machine clock. The maxupdateskew directive sets the threshold for determining whether an estimate might be so unreliable that it should not be used. By default, the threshold is 1000 ppm. Typical values for skew-in-ppm might be 100 for a dial-up connection to servers over a phone line, and 5 or 10 for a computer on a LAN.

    chrony_enable_rtcsync: true

Enable kernel synchronization of the real-time clock (RTC). The rtcsync directive enables a mode where the system time is periodically copied to the RTC and chronyd does not try to track its drift. This directive cannot be used with the rtcfile directive. On Linux, the RTC copy is performed by the kernel every 11 minutes.

    # chrony_hwtimestamp: *

Enable hardware timestamping on all interfaces that support it.

    # chrony_minsources: 2

Increase the minimum number of selectable sources required to adjust the system clock. The default value is 1.

    #chrony_allow:
    #  - 192.168.0.0/16

Allow NTP client access from local network. The default is that no clients are allowed access.

    #chrony_local: stratum 10

Serve time even if not synchronized to a time source.
This directive is normally used in an isolated network, where computers are required to be synchronised to one another, but not necessarily to real time. The server can be kept vaguely in line with real time by manual input.
Local directive has the following options:
  - stratum <stratum>: sets the stratum of the server
  - distance <distance>
  - orphan: enables a special ‘orphan’ mode, where sources with stratum equal to the local stratum are assumed to not serve real time.

.

    #chrony_authselectmode: require

Require authentication (nts or key option) for all NTP sources. Valid values: require, prefer, mix or ignore

    chrony_keyfile: "{{ chrony_etc_path }}/chrony.keys"

Specify file containing keys for NTP authentication.

    #chrony_ntsdumpdir: /var/lib/chrony

Save NTS keys and cookies. Only available on chrony version 4.

    #chrony_leapsecmode: slew

Insert/delete leap seconds by slewing instead of stepping. Valid values: system, step, slew or ignore.

    #chrony_leapsectz: right/UTC

Get TAI-UTC offset and leap seconds from the system tz database.

    chrony_logdir: /var/log/chrony

Specify directory for log files.

    #chrony_log: measurements statistics tracking

Select which information is logged

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    chrony_service: chronyd

Chrony service name.

    chrony_packages:

Packages to be installed to provide chrony functionality.

    chrony_cfg_mode: '0644'

Desired configuration files permission.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      vars:
        chrony_pools:
          - pool.ntp.org          iburst maxsources 4
          - ntp.ubuntu.com        iburst maxsources 4
          - 0.ubuntu.pool.ntp.org iburst maxsources 1
          - 1.ubuntu.pool.ntp.org iburst maxsources 1
          - 2.ubuntu.pool.ntp.org iburst maxsources 2
        chrony_driftfile: /var/lib/chrony/drift
        chrony_makestep: 1.0 3
        chrony_maxupdateskew: 100.0
        chrony_enable_rtcsync: true
        chrony_minsources: 2
        chrony_allow:
          - 192.168.0.0/16
        chrony_local: stratum 10
        chrony_keyfile: "{{ chrony_etc_path }}/chrony.keys"
        chrony_leapsecmode: slew
        chrony_logdir: /var/log/chrony
        chrony_log: measurements statistics tracking
      roles:
         - { role: guidugli.chrony }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
