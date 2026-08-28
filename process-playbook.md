# Day 004 - Process Playbook

## What I Learned

Today I learned how Linux processes, jobs, signals, priorities, and process states work.

A process is a running program. Every process has a PID, and a process can also have a PPID that identifies its parent process.

## PID and PPID

- PID = Process ID
- PPID = Parent Process ID

Example:

```bash
ps -p PID -o pid,ppid,stat,cmd
```

This command shows information about a specific process.

## Foreground and Background

A foreground process uses the terminal until it finishes.

```bash
sleep 100
```

A background process runs without blocking the terminal.

```bash
sleep 100 &
```

The `&` symbol starts the process in the background.

## Jobs

```bash
jobs
```

Shows jobs managed by the current shell.

```bash
jobs -l
```

Shows jobs together with their PID.

```bash
fg %1
```

Moves Job 1 to the foreground.

```bash
bg %1
```

Continues a stopped Job 1 in the background.

`Ctrl + Z` stops a foreground process without terminating it.

`Ctrl + C` usually interrupts and terminates a foreground process.

## Signals

A signal is a message sent to a process.

### SIGTERM

```bash
kill PID
```

By default, `kill` sends SIGTERM.

SIGTERM asks a process to shut down cleanly.

### SIGKILL

```bash
kill -9 PID
```

SIGKILL forces the process to terminate immediately.

SIGKILL should normally be used only when SIGTERM does not work.

Before killing a process, I should verify that I am targeting the correct PID.

## Finding Processes

Find a process by name:

```bash
pgrep process_name
```

Show its PID and command:

```bash
pgrep -a process_name
```

Send SIGTERM by process name:

```bash
pkill process_name
```

## Monitoring Processes

```bash
top
```

`top` displays running processes and updates them continuously.

Important columns:

- PID = Process ID
- %CPU = CPU usage
- %MEM = memory usage
- COMMAND = running program

I tested high CPU usage with:

```bash
yes > /dev/null &
```

The `yes` process used approximately 100% CPU and was visible in `top`.

## Process Priority

Nice values affect process scheduling priority.

- Lower nice value = higher priority
- Higher nice value = lower priority
- Normal nice value is usually 0

Start a new process with a nice value:

```bash
nice -n 10 sleep 1000 &
```

Change the nice value of an existing process:

```bash
renice 10 -p PID
```

`nice` is used when starting a process.

`renice` is used for an already running process.

## Process States

Common process states:

- R = Running
- S = Sleeping
- T = Stopped
- Z = Zombie
- D = Uninterruptible Sleep

A zombie process has already finished, but its parent process has not collected its exit status yet.

A zombie is not actively running, so sending SIGKILL directly to the zombie does not solve the real problem.

## Nohup

`nohup` means "no hangup".

It can be used when I want a process to continue even if the terminal or SSH session closes.

```bash
nohup sleep 1000 &
```

`&` runs the process in the background.

`nohup` helps the process survive terminal disconnection.

Program output may be written to:

```text
nohup.out
```

## My First 5 Process Commands When a Service Is Slow

### 1. top

```bash
top
```

I use it first to check which processes are consuming CPU and memory.

### 2. ps

```bash
ps -p PID -o ppid,stat,cmd
```

I use it to inspect a suspicious process, its parent, state, and command.

### 3. pgrep

```bash
pgrep -a process_name
```

I use it when I know the process name but do not know its PID.

### 4. renice

```bash
renice 10 -p PID
```

I can lower the priority of a resource-heavy process without immediately terminating it.

### 5. kill

```bash
kill PID
```

I use SIGTERM first to request a clean shutdown.

If SIGTERM does not work and I have verified the correct PID, I can use:

```bash
kill -9 PID
```

as a last resort.

## Wrong PID Risk

Sending a signal to the wrong PID can stop the wrong process and cause a service outage.

PIDs can be reused by Linux after a process exits.

Therefore, I should verify the process before sending dangerous signals, especially SIGKILL.

```bash
ps -p PID -o pid,ppid,stat,cmd
```

## 10 Technical Terms

1. **Process** - A running instance of a program.
2. **PID (Process ID)** - A unique number used to identify a process.
3. **PPID (Parent Process ID)** - The PID of the process that created another process.
4. **Foreground Process** - A process currently using the terminal interactively.
5. **Background Process** - A process that runs without blocking the terminal.
6. **Signal** - A message sent to a process to control its behavior.
7. **SIGTERM** - A signal that requests a process to terminate cleanly.
8. **SIGKILL** - A signal that forces a process to terminate immediately.
9. **Zombie Process** - A finished process whose exit status has not yet been collected by its parent.
10. **Nice Value** - A value that influences the scheduling priority of a process.

## Daily Summary

Today, I learned how to inspect and manage Linux processes using tools such as `ps`, `top`, `pgrep`, `pkill`, and `kill`.
I practiced foreground and background jobs, signals, process priorities, process states, zombie processes, and `nohup`.
I can now perform a basic process investigation when a Linux service becomes slow or consumes too many system resources.
