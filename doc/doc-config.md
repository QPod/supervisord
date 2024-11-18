# `supervisord` Configuration Guidelines

## Supervisord daemon settings

Following parameters configured in "supervisord" section:

- **logfile**. Where to put log of supervisord itself.
- **logfile_maxbytes**. Rotate log-file after it exceeds this length.
- **logfile_backups**. Number of rotated log-files to preserve.
- **loglevel**. Logging verbosity, can be trace, debug, info, warning, error, fatal and panic (according to documentation of module used for this feature). Defaults to info.
- **pidfile**. Full path to file containing process id of current supervisord instance.
- **minfds**. Reserve at least this amount of file descriptors on supervisord startup. (Rlimit nofiles).
- **minprocs**. Reserve at least this amount of processes resource on supervisord startup. (Rlimit noproc).
- **identifier**. Identifier of this supervisord instance. Required if there is more than one supervisord run on one machine in same namespace.

## Supervised program settings

Supervised program settings configured in [program:programName] section and include these options:

- **command**. Command for supervision. It can be given as full path to executable or can be calculated via PATH variable. Command line parameters also should be supplied in this string.
- **process_name**. the process name
- **numprocs**. number of process
- **numprocs_start**. ?? (not supported)
- **autostart**. Should be supervised command run on supervisord start? Defaults to **true**.
- **startsecs**. The total number of seconds which the program needs to stay running after a startup to consider the start successful (moving the process from the STARTING state to the RUNNING state). Set to 0 to indicate that the program needn’t stay running for any particular amount of time.
- **startretries**. The number of serial failure attempts that supervisord will allow when attempting to start the program before giving up and putting the process into an FATAL state. See Process States for explanation of the FATAL state.
- **autorestart**. Automatically re-run supervised command if it dies.
- **exitcodes**. The list of “expected” exit codes for this program used with autorestart. If the autorestart parameter is set to unexpected, and the process exits in any other way than as a result of a supervisor stop request, supervisord will restart the process if it exits with an exit code that is not defined in this list.
- **stopsignal**. Signal to send to command to gracefully stop it. If more than one stopsignal is configured, when stoping the program, the supervisor will send the signals to the program one by one with interval "stopwaitsecs". If the program does not exit after all the signals sent to the program, supervisord will kill the program.
- **stopwaitsecs**. Amount of time to wait before sending SIGKILL to supervised command to make it stop ungracefully.
- **stdout_logfile**. Where STDOUT of supervised command should be redirected. (Particular values described lower in this file).
- **stdout_logfile_maxbytes**. Log size after exceed which log will be rotated.
- **stdout_logfile_backups**. Number of rotated log-files to preserve.
- **redirect_stderr**. Should STDERR be redirected to STDOUT.
- **stderr_logfile**. Where STDERR of supervised command should be redirected. (Particular values described lower in this file).
- **stderr_logfile_maxbytes**. Log size after exceed which log will be rotated.
- **stderr_logfile_backups**. Number of rotated log-files to preserve.
- **environment**. List of VARIABLE=value to be passed to supervised program. It has higher priority than `envFiles`.
- **envFiles**. List of .env files to be loaded and passed to supervised program.
- **priority**. The relative priority of the program in the start and shutdown ordering
- **user**. Sudo to this USER or USER:GROUP right before exec supervised command.
- **directory**. Jump to this path and exec supervised command there.
- **stopasgroup**. Also stop this program when stopping group of programs where this program is listed.
- **killasgroup**. Also kill this program when stopping group of programs where this program is listed.
- **restartpause**. Wait (at least) this amount of seconds after stopping suprevised program before start it again.
- **restart_when_binary_changed**. Boolean value (false or true) to control if the supervised command should be restarted when its executable binary changes. Defaults to false.
- **restart_cmd_when_binary_changed**. The command to restart the program if the program binary itself is changed.
- **restart_signal_when_binary_changed**. The signal sent to the program for restarting if the program binary is changed.
- **restart_directory_monitor**. Path to be monitored for restarting purpose.
- **restart_file_pattern**. If a file changes under restart_directory_monitor and filename matches this pattern, the supervised command will be restarted.
- **restart_cmd_when_file_changed**. The command to restart the program if any monitored files under **restart_directory_monitor** with pattern **restart_file_pattern** are changed.
- **restart_signal_when_file_changed**. The signal will be sent to the proram, such as Nginx, for restarting if any monitored files under **restart_directory_monitor** with pattern **restart_file_pattern** are changed.
- **depends_on**. Define supervised command start dependency. If program A depends on program B, C, the program B, C will be started before program A. Example:

```ini
[program:A]
depends_on = B, C

[program:B]
...
[program:C]
...
```

## Set default parameters for all supervised programs

All common parameters that are identical for all supervised programs can be defined once in "program-default" section and omitted in all other program sections.

In example below the VAR1 and VAR2 environment variables apply to both test1 and test2 supervised programs:

```ini
[program-default]
environment=VAR1="value1",VAR2="value2"
envFiles=global.env,prod.env

[program:test1]
...

[program:test2]
...

```

## Group

Section "group" is supported and you can set "programs" item

## Events

Supervisord 3.x defined events are supported partially. Now it supports following events:

- all process state related events
- process communication event
- remote communication event
- tick related events
- process log related events

## Logs

Supervisord can redirect stdout and stderr ( fields stdout_logfile, stderr_logfile ) of supervised programs to:

- **/dev/null**. Ignore the log - send it to /dev/null.
- **/dev/stdout**. Write log to STDOUT.
- **/dev/stderr**. Write log to STDERR.
- **syslog**. Send the log to local syslog service.
- **syslog @[protocol:]host[:port]**. Send log events to remote syslog server. Protocol must be "tcp" or "udp", if missing, "udp" assumed. If port is missing, for "udp" protocol, it's defaults to 514 and for "tcp" protocol, it's value is 6514.
- **file name**. Write log to specified file.

Multiple log files can be configured for the stdout_logfile and stderr_logfile with ',' as delimiter. For example:

```ini
stdout_logfile = test.log, /dev/stdout
```

### syslog settings

To write the log to syslog, following additional parameter can be set like:

```ini
syslog_facility=local0
syslog_tag=test
syslog_stdout_priority=info
syslog_stderr_priority=err
```

- **syslog_facility**, can be one of(case insensitive): KERNEL, USER, MAIL, DAEMON, AUTH, SYSLOG, LPR, NEWS, UUCP, CRON, AUTHPRIV, FTP, LOCAL0~LOCAL7
- **syslog_stdout_priority**, can be one of(case insensitive): EMERG, ALERT, CRIT, ERR, WARN, NOTICE, INFO, DEBUG
- **syslog_stderr_priority**, can be one of(case insensitive): EMERG, ALERT, CRIT, ERR, WARN, NOTICE, INFO, DEBUG

## Web GUI

Supervisord has builtin web GUI: you can start, stop & check the status of program from the GUI. Following picture shows the default web GUI:

![screenshot of webui](https://raw.githubusercontent.com/QPod/supervisord/main/doc/go_supervisord_gui.png)

Please note that in order to see|use Web GUI you should configure it in /etc/supervisord.conf both in [inet_http_server] (and|or [unix_http_server] if you prefer unix domain socket) and [supervisorctl]:

```ini
[inet_http_server]
port=127.0.0.1:9001
;username=test1
;password=thepassword

[supervisorctl]
serverurl=http://127.0.0.1:9001
```

## Usage from a Docker container

supervisord is compiled inside a Docker image to be used directly inside another image, from the Docker Hub version.

```Dockerfile
FROM debian:latest
COPY --from=qpod/supervisord:ubuntu /opt/supervisord /opt/supervisord
CMD ["/opt/supervisord/supervisord"]
```

## Integrate with Prometheus

The Prometheus node exporter supported supervisord metrics are now integrated into the supervisor. So there is no need to deploy an extra node_exporter to collect the supervisord metrics. To collect the metrics, the port parameter in section "inet_http_server" must be configured and the metrics server is started on the path /metrics of the supervisor http server.

For example, if the port parameter in "inet_http_server" is "127.0.0.1:9001" and then the metrics server should be accessed in url "http://127.0.0.1:9001/metrics"

## Register service

Autostart supervisord after os started. Look up supported platforms at [kardianos/service](https://github.com/kardianos/service).

```Shell
# install
sudo supervisord service install -c full_path_to_conf_file
# uninstall
sudo supervisord service uninstall
# start
supervisord service start
# stop
supervisord service stop
```