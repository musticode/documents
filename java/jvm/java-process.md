# Listing working java applications

- List java processes with jps JDK Tool
```bash
jps -l
```
Output: Shows the PID (Process ID) and the main class/JAR name of Java processes.


- Use ps and grep to Find Java Processes

If you don’t have jps, use ps to search for Java processes:

```bash
ps -ef | grep -i '[j]ava'
```

  - ps -ef: Lists all processes.
  - grep -i '[j]ava': Filters for Java processes (case-insensitive) and excludes the grep command itself (using [j]ava trick).

### Stop a Java Process Gracefully

use jps or ps to get the PID of the java process

```bash
jps -l  # Example output: `12345 com.example.MainClass`
# OR
ps -ef | grep -i '[j]ava'
```

send termination signal

- Graceful shutdown
```bash
kill -15 <PID>  # SIGTERM (default)
```

- Forceful shutdown
```bash
kill -9 <PID>   # SIGKILL (last resort)
```


