# Troubleshooting

This page contains hints for troubleshooting general issues with the
Greenbone Community Edition.

- For source build specific troubleshooting, see
[Troubleshooting the source build](./22.4/source-build/troubleshooting.md).
- For community container specific troubleshooting see [Troubleshooting the community containers](./22.4/container/troubleshooting.md).

## My scan does not show any results

After a finished scan, your report does not contain any results or errors.

Some common issues if scans does not return any results are:

1. The targets are not answering to an **ICMP Echo Request** → Check the
   **Alive Test** setting of your target definition and try some of the other
   available methods. Further reading:

   - [Greenbone Enterprise Appliance documentation - Hosts not found](https://docs.greenbone.net/GSM-Manual/gos-22.04/en/scanning.html#hosts-not-found)
   - [Greenbone Enterprise Appliance documentation - Creating a target](https://docs.greenbone.net/GSM-Manual/gos-22.04/en/scanning.html#creating-a-target)
   - [Greenbone Enterprise Appliance documentation - Alive Test](https://docs.greenbone.net/GSM-Manual/gos-22.04/en/scanning.html#alive-test)

2. You are using a custom scan configuration which does not include the following
   two VTs from the **Port scanners** family.

   - [Nmap (NASL wrapper) - OID: 1.3.6.1.4.1.25623.1.0.14259](https://secinfo.greenbone.net/nvt/1.3.6.1.4.1.25623.1.0.14259)
   - [Ping Host - OID: 1.3.6.1.4.1.25623.1.0.100315](https://secinfo.greenbone.net/nvt/1.3.6.1.4.1.25623.1.0.100315)

    Further reading [here](https://community.greenbone.net/t/hint-self-created-scan-configs-copy-of-empty-scan-config-showing-no-results/331)

3. You are using a [port list](https://docs.greenbone.net/GSM-Manual/gos-22.04/en/performance.html#selecting-a-port-list-for-a-task)
   which is not optimal for your environment:

    e.g., a ``All TCP and All UDP`` port list might be responsible for your
    port scan timing out, causing your scan to not return any results at all.
    It is suggested to start with a smaller port list, e.g., ``All IANA TCP``.

4. **SELinux** is enabled and blocking the scanner from doing its job.

5. You do not have **nmap** installed or not available within your **PATH**.

For further debugging/logging, the mentioned **Nmap (NASL wrapper)** and
**Ping Host** VTs allow to configure various settings:

* Ping Host
    1. **Report about unreachable Hosts** configured to **yes**: include notes
      if a remote host is considered as dead/not reachable, and the reason why.
    2. **Log failed nmap calls** and **Log nmap output** configured to **yes**:
      Logs additional output if nmap was used.

* Nmap (NASL wrapper)
    1. **Log nmap output** configured to **yes**: Log additional output if nmap
      was used.

## OOM is killing Redis on large scans

During a larger scan, the machine is running out of memory. Therefore, the Linux
Out-of-Memory (OOM) killer is terminating the Redis database server and the scan
gets interrupted.

The problem described is not easy to solve as it can have several root causes,
from known issues to usage behavior. In particular, there can be problems with
vHosts and CGI caching.

In general, we recommend the following:

* Prevent overloading the system by adjusting the usage:
    * Do not start scan tasks all at once. Use schedules to start them at intervals.
    * Reconfigure scan targets to include less hosts. Split the hosts into more targets and tasks instead.
    * Do not run or schedule feed updates for times where scan tasks are running or scheduled to run.
    * Do not view or download large reports while scan tasks are running.

* Disable vHost expansion for scans that cause problems:
    * Clone and edit the used scan configuration.
    * Set the scanner preference `expand_vhosts` to `0` and save the change.

* Disable CGI caching for scans that cause problems:
    * Clone and edit the used scan configuration.
    * Navigate to the VT family *Settings*.
    * Edit the VT `Global Variable Settings` (OID: *1.3.6.1.4.1.25623.1.0.12288*).
    * Set the preference `Disable caching of web pages during CGI scanning` to `Yes` and save the change.

If you think that you can narrow the problem down to a specific host and/or
vulnerability test, please either open an issue for the scanner at
[https://github.com/greenbone/openvas-scanner/issues](https://github.com/greenbone/openvas-scanner/issues)
or the vulnerability test at [Vulnerability Tests - Greenbone Community Forum](https://forum.greenbone.net/c/vulnerability-tests/7).

## Vulnerabilities are not found

A scan report does not contain any results or some known vulnerabilities do not
show up in the report.

This may have several reasons. Most likely, the {term}`feed` synchronization is not
finished yet. The feed data has been downloaded to the file system of the local
machine but {command}`gvmd` and {command}`ospd-openvas` did not have enough resources
to load them into their databases and memory yet. This can be checked on the web interface
by opening the {menuselection}`SecInfo > NVTs` page to see whether {term}`VTs<VT>`
are listed there and the {menuselection}`Administration > Feed Status`
page to see whether a synchronization process is currently running.

If both pages are fine, you may take a look at the following sections in our
product manual:
- [20.2 Why Is a Service/Product Not Detected?](https://docs.greenbone.net/GSM-Manual/gos-22.04/en/faq.html#why-is-a-service-product-not-detected)
- [20.3 Why Is a Vulnerability Not Detected?](https://docs.greenbone.net/GSM-Manual/gos-22.04/en/faq.html#why-is-a-vulnerability-not-detected)
