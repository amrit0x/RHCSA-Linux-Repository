### Linux System Logging

### 1\. What is logging in Linux?

Logging is the process of recording events that happen inside a Linux system.

For example:

* A user logs in or logs out.

* An SSH service starts or fails.

* A server encounters an error.

* A scheduled cron job executes.

* A system service is restarted.

* The kernel reports a hardware or networking issue.

Linux stores these events as log messages so administrators can investigate problems and understand what happened.

### Real-world example

Suppose an application stops working at 10:30 AM. You want to know:

* Did the application crash?

* Was the server running out of memory?

* Did someone restart the service?

* Was there a permission error?

System logs help you investigate these questions instead of guessing.

### 2\. Two major Linux logging mechanisms

Your whiteboard shows two important components:

### 1\. systemd-journald

Collects and stores journal entries from the kernel, services, and other sources.

Common command: `journalctl`

On many RHEL systems, journal entries can be forwarded

### 2\. rsyslog

Processes log messages according to configured rules and writes them to files or forwards them to other logging systems.

Common directory: `/var/log/`

Both mechanisms can work together. They are not mutually exclusive.

### A. systemd-journald

`systemd-journald` collects log messages from sources such as:

* The Linux kernel.

* Systemd services.

* Standard output and standard error of services managed by systemd.

* Applications that send messages through supported logging interfaces.

Its journal is often stored in a structured, binary format rather than ordinary readable text.

Default storage locations:

| Location          | Purpose                                                     |
| ----------------- | ----------------------------------------------------------- |
| /run/log/journal/ | Runtime journal data; generally lost after reboot           |
| /var/log/journal/ | Persistent journal data, when persistent storage is enabled |

If persistent journal storage is not enabled, journal data may be kept only in `/run/log/journal/`.

Useful commands:

```bash
systemctl status systemd-journald
journalctl
journalctl -b
journalctl -p err
```

* `journalctl` displays journal entries.

* `journalctl -b` displays entries from the current boot.

* `journalctl -p err` filters entries at error priority and higher severity.

### B. rsyslog

`rsyslog` is a logging service that receives, filters, and routes log messages based on its configuration.

On many RHEL systems, it writes readable text logs under `/var/log/`.

Examples include:

| Log file          | Typical purpose                                         |
| ----------------- | ------------------------------------------------------- |
| /var/log/messages | General system messages                                 |
| /var/log/secure   | Authentication and security-related messages            |
| /var/log/cron     | Cron-related messages                                   |
| /var/log/maillog  | Mail-related messages                                   |
| /var/log/boot.log | Boot-related logging, depending on system configuration |

The exact files and message routing depend on the operating system and `/etc/rsyslog.conf` configuration.

Useful commands:

```bash
systemctl status rsyslog
systemctl enable --now rsyslog
```

View the general system log:

```bash
tail -f /var/log/messages
```

On a system where `/var/log/messages` is configured, this displays new messages as they arrive.

### 3\. Difference between journald and rsyslog

The whiteboard compares their storage format and purpose.

| Feature       | systemd-journald                            | rsyslog                                       |
| ------------- | ------------------------------------------- | --------------------------------------------- |
| Main purpose  | Collects and stores journal entries         | Processes, routes, and stores log messages    |
| Format        | Structured journal format, generally binary | Commonly readable text files                  |
| Common access | journalctl                                  | cat, less, tail, grep                         |
| Storage       | /run/log/journal/ or /var/log/journal/      | Commonly /var/log/                            |
| Persistence   | Depends on journal configuration            | Depends on file configuration and storage     |
| Routing rules | Journal configuration and service settings  | Rules in /etc/rsyslog.conf and included files |

Remember: Journald does not simply store every log as a text file. Rsyslog can receive messages from journald and save selected messages to separate files.

### 4\. Understanding Linux log management flow

Linux kernel, services and applications

systemd-journald

Collects journal entries

Possible forwarding to rsyslog

rsyslog

Applies rules and routes messages

Log files under `/var/log/`

For example, messages, secure and cron

This is a common arrangement, not a mandatory flow for every message. Some applications write directly to files or send messages through other interfaces.

### 5\. What is a facility in rsyslog?

According to your second whiteboard, rsyslog uses two important terms:

* Facility — identifies the source or category of a log message.

* Priority — identifies the severity of the message.

Think of a college office receiving complaints.

* Facility tells you which department the complaint belongs to.

* Priority tells you how urgent the complaint is.

For example, a message may come from the `cron` facility with `crit` priority.

### Common facilities

| Facility        | Meaning                                      |
| --------------- | -------------------------------------------- |
| auth / authpriv | Authentication and security-related messages |
| cron            | Scheduled tasks and cron activity            |
| daemon          | Background system services                   |
| kern            | Kernel messages                              |
| mail            | Mail subsystem                               |
| user            | User-level messages                          |
| local0–local7   | Custom local logging categories              |

The exact facilities available and their usage depend on the logging system and configuration.

### 6\. Understanding log priorities (severity)

Priority indicates how serious a message is. The standard syslog severity levels range from `0` to `7`.

| Numeric level | Priority | Meaning                          |
| ------------- | -------- | -------------------------------- |
| 0             | emerg    | System is unusable               |
| 1             | alert    | Immediate action required        |
| 2             | crit     | Critical condition               |
| 3             | err      | Error condition                  |
| 4             | warning  | Warning condition                |
| 5             | notice   | Normal but significant condition |
| 6             | info     | Informational message            |
| 7             | debug    | Debugging details                |

Important rule: A lower number represents a more severe condition.

For example, priority `crit` (2) is more severe than `warning` (4).

In rsyslog rules, selecting a priority usually also selects messages of higher severity. For example:

```text
cron.crit    /var/log/cron-critical.log
```

This rule matches `cron` messages at `crit`, `alert`, and `emerg` severity. It does not mean “only critical messages.”

### 7\. Practical: Generate a log message using `logger`

The second whiteboard shows this command:

```bash
logger -p cron.crit "message"
```

The `logger` command allows an administrator to generate a log message from the command line.

### Command structure

```bash
logger -p FACILITY.PRIORITY "MESSAGE"
```

For example:

```bash
logger -p cron.crit "Critical cron test message"
```

Here:

* `logger` — generates a log message.

* `-p` — specifies the facility and priority.

* `cron` — facility.

* `crit` — priority.

* `"Critical cron test message"` — message content.

### Practical exercise

Step 1: Generate a message

```bash
logger -p user.info "My first Linux log message"
```

Step 2: Check the journal

```bash
journalctl -n 20
```

To search for your message:

```bash
journalctl -g "My first Linux log message"
```

Step 3: Generate a critical-priority message

```bash
logger -p cron.crit "Critical cron test message"
```

Step 4: Verify the message in the journal or configured log file

```bash
journalctl -n 50
```

If rsyslog is configured to route `cron.crit` messages into a particular file, check that file as well.

```bash
tail -n 20 /var/log/messages
```

The message may not appear in `/var/log/messages` if the active rsyslog rules route it elsewhere. The `logger` command sends a message; the logging configuration determines where it is stored.

### 8\. Configure a custom log file using rsyslog

Now let's understand how an administrator can separate messages into different log files.

Suppose you want all cron messages with `crit` severity or higher to be written to:

```text
/var/log/cron-critical.log
```

### Step 1: Edit the rsyslog configuration

```bash
vim /etc/rsyslog.conf
```

Alternatively, create a separate configuration file:

```bash
vim /etc/rsyslog.d/cron-critical.conf
```

Add this rule:

```text
cron.crit    /var/log/cron-critical.log
```

This tells rsyslog to route critical and more severe messages from the cron facility to that file.

### Step 2: Restart rsyslog

```bash
systemctl restart rsyslog
```

Verify that it is running:

```bash
systemctl status rsyslog
```

### Step 3: Generate a test message

```bash
logger -p cron.crit "Testing custom cron log"
```

### Step 4: Check the custom file

```bash
cat /var/log/cron-critical.log
```

If the file does not exist or the message is missing, check the rsyslog service status, configuration syntax, permissions, and active routing rules.

### 9\. More useful rsyslog configuration examples

These are examples of common rules you can use in a lab.

| Configuration rule                | Purpose                                                    |
| --------------------------------- | ---------------------------------------------------------- |
| cron.* /var/log/cron-custom.log   | Route all cron severities to a custom file                 |
| *.err /var/log/errors-custom.log  | Route error and more severe messages from all facilities   |
| user.info /var/log/user-info.log  | Route informational and more severe user-facility messages |
| *.debug /var/log/debug-custom.log | Route debug and more severe messages from all facilities   |

For example:

```text
cron.*       /var/log/cron-custom.log
*.err        /var/log/errors-custom.log
user.info    /var/log/user-info.log
```

Add these rules to a dedicated file under `/etc/rsyslog.d/`, restart rsyslog, and test each rule with `logger`.

Be careful when using broad rules such as `*.debug`, because they can produce a large volume of logs.

### 10\. Log rotation: Why are log files rotated?

The first whiteboard also appears to reference log files and older log entries.

Log files can grow continuously. Linux commonly uses logrotate to rotate, compress, retain, and eventually remove old log files according to configured policies.

For example, a directory may contain:

```text
/var/log/messages
/var/log/messages-20261001
/var/log/messages-20261008.gz
```

The exact filenames depend on the rotation configuration.

Useful commands:

```bash
cat /etc/logrotate.conf
ls /etc/logrotate.d/
```

To inspect the logrotate service or timer:

```bash
systemctl status logrotate.timer
```

The timer may not be configured on every distribution; some systems run logrotate through cron or another scheduler.

Note: Log rotation and log collection are different activities. Journald and rsyslog collect or process messages, while logrotate manages the lifecycle of text log files.

### 11\. Practical revision checklist

Hands-on practice

0 of 9 completed

Check systemd-journald status

View logs using journalctl

Check rsyslog status

Identify common files under /var/log/

Generate a message using logger

Understand facility versus priority

Configure a custom rsyslog rule

Restart rsyslog and test the rule

Inspect logrotate configuration

### 12\. Interview questions

* What is the difference between `systemd-journald` and `rsyslog`?

* Where does journald store runtime and persistent logs?

* What is the purpose of `/var/log/messages`?

* What is the difference between a facility and a priority?

* Which is more severe: `crit` or `warning`?

* What does `logger -p cron.crit "Test"` do?

* How do you configure rsyslog to write selected messages to a separate file?

* What is the purpose of logrotate?

* How do you view logs from the current boot?

* Why might a message appear in `journalctl` but not in `/var/log/messages`?

The key concept to remember: Journald collects structured log entries, rsyslog applies routing rules, facilities identify message categories, priorities indicate severity, and `logger` helps you test the logging configuration.
