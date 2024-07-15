# AdGuard DNS Proxy

Adding DNS-over-QUIC to Windows 11 using AdGuard's DNS Proxy.

## Preface

Microsoft's Windows 11 uses unencrypted DNS by default; however, does have
native support for DNS-over-HTTPS (DoH).
DoH is enabled via templates, with custom templates added via the command-line.

As an example:

```Powershell
# Add a IPv4 DoH Server Template
Add-DnsClientDohServerAddress -ServerAddress 192.168.1.2 -DohTemplate https://adguard.nickfedor.com/dns-query -AutoUpgrade $True

# Add a IPv6 DoH Server Template
Add-DnsClientDohServerAddress -ServerAddress 2600:3333:2222:1111:aaaa:bbbb:cccc:dddd -DohTemplate https://adguard.nickfedor.com/dns-query -AutoUpgrade $True
```

This is often more than sufficient for most use-cases; however, might not
satisfy those who prefer the option to use DNS-over-QUIC (DoQ), DNS-over-TLS
(DoT), etc.

## Forewarning

As with any third-party software, there are inherent risks.
While AdGuard is a reasonably trustworthy organization of developers that
specializes in DNS-related products, it's always beneficial to ensure you are
doing your due diligence regarding your own computing practices.

As such, DNS Proxy is an open-source project with publicly available source
code:
<https://github.com/AdguardTeam/dnsproxy/>

## Prerequisites

- A preferred upstream DNS provider that supports DoQ.  
  I am currently using AdGuard Home as my local DNS Resolver, which supports
  various options, including DoQ.

  - IPv4 Address: `192.168.1.2`
  - IPv6 Global Address: `2600:3333:2222:1111:aaaa:bbbb:cccc:dddd`
  - DoQ Address: `quic://adguard.nickfedor.com:853`

- Referenced software versions:
  - Windows 11 23H2 - Build 22631.3880
  - DNSProxy v0.72.0

## Setup AdGuard DNS Proxy

### Install the Binary

1. Download AdGuard's DNS Proxy from their [GitHub repository](https://github.com/AdguardTeam/dnsproxy/releases)

2. Extract the archive

3. Create a new folder in `C:\Program Files` called `AdGuard DNS Proxy`

4. Copy the contents of the extracted archive into the newly created folder.

### Create a New Task for Persistence and Run It

#### General Tab

1. Open Task Scheduler

2. Select: `Create Task`

3. Task Name: `AdGuard DNS Proxy`

4. Security Options:
   - Enable: `Run whether user is logged on or not`
   - Enable: `Do not store password.`
5. Configure for: `Windows 10`

#### Triggers Tab

1. Select: `New...`
2. Begin the task: `At startup`
3. Select: `OK`

#### Actions Tab

1. Select `New...`
2. Program/script: `"C:\Program Files\AdGuard DNS Proxy\dnsproxy.exe"`
3. Add arguments: `-l 127.0.0.100 -l ::1 -u quic://adguard.nickfedor.com:853 -b 192.168.1.2`
   > Note: I am using the IPv4 Loopback Address of `127.0.0.100`; however,
   > `127.0.0.1` works as well.  
   > Just remember to use `::1` as the IPv6 loopback address if you're
   > interested in enabling IPv6 DNS within Windows' settings.
4. Select: `OK`

#### Conditions Tab

Disable: `Start the task only if the computer is on AC power`

#### Settings Tab

1. Enable: `Run task as soon as possible after a scheduled start is missed`
2. Enable: `If the task fails, restart every:`
3. Disable: `Stop the task if it runs longer than:`

#### Create the task and Run it

1. Select: `OK`
2. Right-click the newly created task and select: `Run`

### Microsoft Windows Network Connectivity Status Indicator (NCSI) Configuration

Windows feature that is used to determine Internet connectivity.
It defaults to using interface-specific DNS settings.  
Enabling it to use "Global DNS" allows it to use the loopback interface.

1. Open `gpedit.msc`
2. Access the NCSI Policy Settings: `Computer Configuration > Administrative
Templates > Network > Network Connectivity Status Indicator`
3. Enable: `Specify global DNS`
4. Enable: `Use global DNS`
5. Select: `OK`

## Configure Windows to use AdGuard DNS Proxy

1. Open `Settings` > `Network & Internet`
2. Select `Ethernet`
3. Select your network interface
4. `DNS server assignment` > Select `Edit`
5. Enable: `IPv4`
6. Preferred DNS: `127.0.0.100`
7. Enable: `IPv6`
8. Preferred DNS: `::1`
9. Select: `Save`

## Maintenance

Updating the program is a simple matter of switching out the binary; however,
just remember that any changes to its name or path will require updates to the
scheduled task.

In addition, changes to the upstream DNS Resolver will require updating the argument variable
passed to the binary when launched via the scheduled task.

## Cleanup

1. Update the Network Interface to use either DHCP (automatically-assigned) DNS
   settings or manually assign other DNS resolvers.
2. Disable NCSI Group Policy that enables specifying it to use a `Global DNS`.
3. Stop and delete the `AdGuard DNS Proxy` task.
4. Delete the `AdGuard DNS Proxy` folder in `C:\Program Files`.

## Final Thoughts

This is a welcomed addition to my system, as it enables me to take advantage of
modern DNS protocols until Microsoft implements a native DoQ solution.
While the lack of a GUI for the program is possibly a non-starter for some, it's
largely unnecessary when using a local upstream solution.
This guide is not comprehensive and should be seen as a basic starting point for
enabling DoQ on Windows 11.
