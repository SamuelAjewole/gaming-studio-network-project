# Tier 1 Runbook: Workstation Can't Reach the Network

Studio: 15-person gaming studio, network built and tested in Cisco Packet Tracer
Scope: first-response triage for a "my computer won't connect" ticket
Escalation path: Tier 1 to Network Admin to ISP/vendor
Reference topology: Router0 to Multilayer Switch0 (VLANs 10/20/30/90, DHCP, ACL) to Switch0-3, one per VLAN, to end devices. Full diagram in README.md.

## Step 1: Physical layer

Check the link light on the NIC and the switch port. No light usually means a bad cable, a bad port, or the device is just off. Swap in a known-good cable or port to rule those out. Also confirm the switch port isn't administratively shut down with `show interfaces status`.

## Step 2: IP addressing and DHCP

Run `ipconfig` (Windows) or `ip a` (Linux/Mac).

No IP, or a 169.254.x.x address, means DHCP failed. Check whether the port is on the right VLAN (`show running-config interface <port>`), whether the trunk to the Multilayer Switch is actually carrying that VLAN (`show interfaces trunk`), and whether the DHCP pool for that VLAN is full (`show ip dhcp binding`).

If it has an IP but on the wrong subnet, the port is probably assigned to the wrong VLAN. Fix with `switchport access vlan <correct ID>`.

## Step 3: Same-VLAN connectivity

Ping the device's default gateway (for example 10.10.20.1 for VLAN 20). If that fails, the problem is upstream, go to Step 5. If it works, move on.

## Step 4: Inter-VLAN and external connectivity

Ping a device on a different VLAN to check inter-VLAN routing, then ping something external like 8.8.8.8 to check the path through Router0. If internal pings work but external ones don't, check Router0's uplink interface and its routing/NAT config.

One thing to keep in mind: if the device is on the Guest VLAN, it's supposed to fail when pinging Corp, Dev, or QA. That's the isolation ACL doing its job, not a fault.

## Step 5: When to escalate

- Trunk link is down and a reboot/reseat doesn't fix it
- DHCP pool for a VLAN is exhausted and needs re-scoping
- The ACL is blocking traffic it shouldn't, or letting through traffic it shouldn't
- Router0's uplink to the ISP is down, since that's outside Tier 1's scope

## Commands used above

```
show interfaces status
show running-config interface <if>
show interfaces trunk
show vlan brief
show ip dhcp binding
show ip interface brief
ping <gateway-ip>
tracert <destination-ip>
```
