---
dg-publish: true
---
- What is it?
    
    A baseboard management controller (BMC) is a small, specialized processor used for remote monitoring and management of a host system. Usually an ARM-based SoC (System on Chip) with graphics and control logic built in, it is commonly located on the main motherboard of the computer, server, network or storage device (the "baseboard"), can be accessed remotely either via a dedicated or shared network connection, and has multiple connections to the host system, giving it an ability to monitor hardware via sensors, flash BIOS/ UEFI firmware, give console access via serial or physical / virtual KVM, power cycle the host, and log events.
    
- Why do you need it?
    
    The key advantage of a BMC is that allows a system administrator to perform many different monitoring and management tasks remotely without having to be physically located next to and connected to the system – such as power cycling, installing BIOS or firmware updates, and monitoring fan speeds and temperatures. The BMC will also notify the administrator (via email or text message) if there is hardware failure (such as a hard drive, fan or PSU that needs replacing) or if there is another kind of error or fault. 
    
      
    
    The BMC is an extremely efficient labor and time saving feature – the administrator no longer needs to physically connect with each server in the rack to perform maintenance tasks. In modern [data centers](https://www.gigabyte.com/Glossary/data-center) which could have hundreds of racks and thousands of servers, it would be impossible to live without it. As a result, all modern servers and other devices used in a data center (such as switches, storage devices, power supply devices etc.) now have a BMC.

check the bmc ip
```bash
ipmitool lan print
```
显示bmc日志

```bash
# system event log
ipmitool sel list
ipmitool sel elist
# 搜集制定event id的bmc日志
ipmitool sel get event_id
# 清除bmc日志
ipmitool sel clear

```
