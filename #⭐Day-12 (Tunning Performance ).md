
# Day-12 Linux Performance Tuning with `tuned`

Performance tuning means changing or selecting system settings so that the Linux system can work better for a specific workload.

Different servers have different requirements. So, we may not use the same performance settings on every server.

For example:

```text
Database Server
Web Server
Virtual Machine
File Server
Application Server
Laptop
```

A database server may need high performance and fast data processing.

A laptop may need better battery saving.

A virtual machine may need settings suitable for a virtual environment.

So, Linux provides tuning profiles that can be selected according to the system requirement.

---

# 1. What Is Performance Tuning?

**Performance tuning** means adjusting the system according to its workload and requirement.

The basic idea is:

```text
Workload
   ↓
Performance Requirement
   ↓
Select Suitable Configuration
   ↓
Better System Behavior
```

For example:

If a server needs to process a large amount of data, we may choose a profile focused on **high throughput**.

If saving power is more important, we may choose a **power-saving** profile.

So:

> **Performance tuning means configuring the system according to its workload and requirement.**

---

# 2. Understanding Tuning with Real-Life Examples

## Example 1: Radio

Think about a radio.

A radio has a frequency dial.

If we want to listen to a particular station, we adjust the frequency.

```text
Radio
  ↓
Adjust Frequency
  ↓
Required Station
```

Linux tuning works with a similar idea.

Linux has many system settings, and these settings can be adjusted according to the workload.

---

## Example 2: Mobile Phone

A mobile phone can have:

```text
Power Saving Mode
```

When battery saving is important, the phone changes some of its behavior to reduce power usage.

Linux can also use different tuning profiles depending on the system requirement.

For example:

```text
Performance Requirement
        ↓
Suitable Tuning Profile
```

---

## Example 3: Motorcycle

Suppose we want:

```text
Maximum Mileage
```

The motorcycle can be tuned for better mileage.

But if our main requirement is:

```text
Maximum Performance
```

the tuning requirement can be different.

Linux works in a similar way.

> **Different requirements can need different tuning.**

---

# 3. What Is `tuned`?

`tuned` is a Linux **system tuning framework and service**.

It helps us apply suitable performance settings to the system.

Instead of manually changing many system parameters one by one, we can select a suitable tuning profile.

The basic working can be understood as:

```text
Workload
   ↓
Select Tuning Profile
   ↓
tuned
   ↓
System Settings
   ↓
Optimized System Behavior
```

The main benefit is that the administrator does not have to manually configure every setting.

---

# 4. Install `tuned`

On RHEL-based Linux systems, install the `tuned` package using:

```bash
dnf install tuned -y
```

After installation, start the service and enable it to start automatically when the system boots:

```bash
systemctl enable --now tuned
```

Check the service status:

```bash
systemctl status tuned
```

If the service is working correctly, we should see:

```text
active (running)
```

---

# 5. Check the Active Tuning Profile

To check which tuning profile is currently active:

```bash
tuned-adm active
```

Example:

```text
Current active profile: virtual-guest
```

This means that the `virtual-guest` profile is currently active.

---

# 6. List Available Tuning Profiles

To see the tuning profiles available on the system:

```bash
tuned-adm list
```

Depending on the Linux version and installed packages, we may see profiles such as:

```text
balanced
powersave
throughput-performance
latency-performance
virtual-guest
virtual-host
```

> **Note:** Available profiles can be different on different Linux versions. Always use `tuned-adm list` to check the profiles available on your system.

---

# 7. Get the Recommended Profile

`tuned` can recommend a profile based on the detected system environment.

Use:

```bash
tuned-adm recommend
```

Example:

```text
virtual-guest
```

This means `tuned` recommends the `virtual-guest` profile for the current system.

This is useful when we are not sure which profile should be selected.

---

# 8. Change the Tuning Profile

The general syntax is:

```bash
tuned-adm profile <profile-name>
```

For example:

```bash
tuned-adm profile virtual-guest
```

Or:

```bash
tuned-adm profile throughput-performance
```

After changing the profile, verify it:

```bash
tuned-adm active
```

Example:

```text
Current active profile: throughput-performance
```

Now the `throughput-performance` profile is active.

---

# 9. Important Tuning Profiles

Now let us understand some commonly used tuning profiles.

---

## 9.1 `virtual-guest`

The `virtual-guest` profile is designed for a Linux system running as a **virtual machine**.

Examples:

```text
VMware Virtual Machine
KVM Virtual Machine
Cloud Virtual Machine
```

If our Linux server is running inside a virtual environment, this profile can be suitable.

Example:

```bash
tuned-adm profile virtual-guest
```

---

## 9.2 `powersave`

The `powersave` profile focuses on reducing power consumption.

The main priority is:

```text
Power Saving
     >
Maximum Performance
```

It can be useful when power efficiency is more important than maximum performance.

Example:

```bash
tuned-adm profile powersave
```

---

## 9.3 `throughput-performance`

The `throughput-performance` profile focuses on **high throughput**.

Throughput means how much work or data a system can process in a given amount of time.

The main priority is:

```text
High Throughput
     >
Power Saving
```

This profile can be useful for workloads where processing a large amount of data efficiently is important.

Example:

```bash
tuned-adm profile throughput-performance
```

---

## 9.4 `latency-performance`

The `latency-performance` profile focuses on reducing system latency.

**Latency** means the time taken to respond to an operation or request.

Some applications need fast response time.

For such workloads, a latency-focused profile can be useful.

The basic idea is:

```text
Lower Latency
      ↓
Faster Response
```

Example:

```bash
tuned-adm profile latency-performance
```

---

# 10. How to Select the Right Profile?

Before selecting a profile, first understand the purpose of the server.

For example:

| Requirement     | Possible Profile         |
| --------------- | ------------------------ |
| Virtual Machine | `virtual-guest`          |
| Power Saving    | `powersave`              |
| High Throughput | `throughput-performance` |
| Low Latency     | `latency-performance`    |

This is a general guideline.

In a production environment, the actual workload and performance requirements should always be considered before selecting a profile.

---

# 11. Practical Workflow

As a Linux administrator, we can follow this basic workflow:

```text
1. Understand the Server Workload
             ↓
2. Understand the Performance Requirement
             ↓
3. Check Available Profiles
             ↓
4. Check Recommended Profile
             ↓
5. Select Suitable Profile
             ↓
6. Apply the Profile
             ↓
7. Verify the Active Profile
```

Commands:

```bash
tuned-adm list
```

Check available profiles.

```bash
tuned-adm recommend
```

Check the recommended profile.

```bash
tuned-adm active
```

Check the currently active profile.

```bash
tuned-adm profile <profile-name>
```

Apply a selected profile.

Then verify again:

```bash
tuned-adm active
```

---

# 12. Complete Practical Example

Suppose we have a Linux server running as a virtual machine.

First, check available profiles:

```bash
tuned-adm list
```

Check the recommended profile:

```bash
tuned-adm recommend
```

Suppose the output is:

```text
virtual-guest
```

Now apply the profile:

```bash
tuned-adm profile virtual-guest
```

Finally, verify:

```bash
tuned-adm active
```

Expected output:

```text
Current active profile: virtual-guest
```

Now the selected tuning profile is active.

---

# 13. Important Commands for Revision

### Install `tuned`

```bash
dnf install tuned -y
```

### Start and enable the service

```bash
systemctl enable --now tuned
```

### Check service status

```bash
systemctl status tuned
```

### List available profiles

```bash
tuned-adm list
```

### Check recommended profile

```bash
tuned-adm recommend
```

### Check active profile

```bash
tuned-adm active
```

### Apply a profile

```bash
tuned-adm profile <profile-name>
```

Example:

```bash
tuned-adm profile virtual-guest
```

---

# 14. Key Points to Remember

1. **Performance tuning** means configuring the system according to its workload and requirement.

2. Different servers can have different performance requirements.

3. **`tuned`** is a Linux system tuning framework and service.

4. `tuned` provides different tuning profiles.

5. Use this command to list available profiles:

```bash
tuned-adm list
```

6. Use this command to get a recommended profile:

```bash
tuned-adm recommend
```

7. Use this command to check the active profile:

```bash
tuned-adm active
```

8. Use this command to apply a profile:

```bash
tuned-adm profile <profile-name>
```

9. `virtual-guest` is useful for virtual machines.

10. `powersave` focuses on reducing power consumption.

11. `throughput-performance` focuses on high throughput.

12. `latency-performance` focuses on low latency.

---

# Final Revision

Remember the complete concept like this:

```text
Understand the Workload
        ↓
Understand the Requirement
        ↓
Check Tuning Profiles
        ↓
Select Suitable Profile
        ↓
Apply Profile
        ↓
Verify Profile
```

The most important commands are:

```bash
tuned-adm list
tuned-adm recommend
tuned-adm active
tuned-adm profile <profile-name>
```

The main idea is simple:

> **Choose the tuning profile according to what the Linux system is required to do.**
