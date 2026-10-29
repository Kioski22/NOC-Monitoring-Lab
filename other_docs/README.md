# NOC Monitoring Lab

A small home lab where I monitor three routers with **LibreNMS** and get **Discord alerts** when something goes down.

I built this after passing my CCNA. I wanted to practice what a NOC actually does every day: watch the network, catch problems fast, and get notified.

---

## What I built

- 3 virtual routers running in GNS3
- A LibreNMS server that checks them every 5 minutes
- Alerts that go to a Discord channel when a link goes down, and again when it comes back

## Tools I used

| What | Used for |
|---|---|
| GNS3 + VMware Workstation | Running the lab |
| MikroTik CHR 7.24.5 | Router 1 |
| VyOS (rolling) x2 | Router 2 and Router 3 |
| LibreNMS on Ubuntu | The monitoring server |
| SNMP v2c | How LibreNMS talks to the routers |
| Discord webhook | Where alerts get sent |

## How it's connected

The three routers are connected in a triangle. LibreNMS sits on its own network and reaches the lab through VyOS-1.

| Device | Monitored IP (loopback) |
|---|---|
| VyOS-1 | 1.1.1.1 |
| VyOS-2 | 2.2.2.2 |
| MikroTik CHR | 3.3.3.3 |

The LibreNMS server has two network cards:
- One for internet and my browser (NAT)
- One for the lab (`10.0.100.10`), which is the one that talks to the routers

I monitor each router by its loopback IP. A loopback stays up as long as the router is alive, so if one link breaks, LibreNMS doesn't think the whole router died.

![Topology](images/topology.png)

---

## How I set it up

### 1. Install LibreNMS
I installed LibreNMS on an Ubuntu VM with nginx, PHP, and MariaDB. Then I finished the setup in the web installer and ran `validate.php` to check everything. All checks passed.

### 2. Turn on SNMP on the routers
On each router I made a read-only SNMP community and allowed only the LibreNMS server to use it (`10.0.100.10`).

On the MikroTik I also disabled the default `public` community, because it was open to everyone.

### 3. Add the routers to LibreNMS
I added all three using their loopback IPs with SNMP v2c. After the first poll, LibreNMS showed the router type, version, ports, and uptime.

![Devices list](images/devices.png)

### 4. Set up Discord alerts
- Made a webhook in my Discord server
- Added it to LibreNMS as an alert transport
- Created an Operation (the delivery plan) and attached it to the rules I care about:
  - Port status up/down
  - Device down (SNMP)
  - Device down (no ping)

---

## The test

To prove it works, I cut a link on purpose.

1. I suspended the link between the MikroTik and VyOS-2 in GNS3 (like pulling the cable).
2. LibreNMS caught it and showed two faults, one for each end of the link.
3. Two alerts arrived in Discord.
4. I resumed the link.
5. Two recovery messages arrived in Discord.

It took about 4 minutes each way, because LibreNMS checks every 5 minutes.

**Link down:**

![Alerts in Discord](images/discord-alerts.png)

**Link back up:**

![Recovery in Discord](images/discord-recovery.png)

---

## Problems I ran into (and how I fixed them)

**1. LibreNMS refused to add my second VyOS router.**
It said "duplicate sysName". Both routers were named `vyos`, and LibreNMS uses the name to tell devices apart. I renamed them to `vyos-1` and `vyos-2` and it worked.

**2. My alerts weren't being sent anywhere.**
In this newer version of LibreNMS, a rule can't pick a transport directly. You have to create an Operation first, then attach it to the rule. Before that, the Operation dropdown was empty.

**3. Too many alerts at once.**
The first time I tested, LibreNMS alerted on every port without a cable too, so I couldn't see the real problem. I turned off the unused ports on the routers, and then only the real link showed up.

**4. Wrong IP in the SNMP allow-list.**
I first used the server's internet-side IP. The routers never see that IP, because the traffic comes from the lab-side card (`10.0.100.10`). Once I used the right one, polling worked.

---

## What I learned

- How SNMP polling works, and why the source IP matters
- Why monitoring a loopback is better than a physical interface
- How to read a LibreNMS alert and match it to the real problem
- How to test an alert chain from start to finish: fault, alert, recovery
- Troubleshooting step by step instead of guessing

## What I want to add next

- A network map in LibreNMS (needs LLDP on the routers)
- Alerts for high CPU and for interface traffic
- Email as a second alert option

---

## About me

I'm Randy. I passed the CCNA in October 2026 and I'm looking for a NOC role. I'm open to shift and night work.

- GitHub: [Kioski22](https://github.com/Kioski22)
- LinkedIn: [randy-flores-879140363](https://www.linkedin.com/in/randy-flores-879140363)
- CCNA labs: [CCNA-LABS-portfolio](https://github.com/Kioski22/CCNA-LABS-portfollio)
