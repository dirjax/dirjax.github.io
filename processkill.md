
Ways to kill parent and child processes in one command

On linux, kill a process is simple, but sometimes, when something goes wrong, a process could fork hundreds and thousands child processes, you can either create a script to kill them all, or find some quick ways to kill them all in one command.

Here is the ways I use, some of them may not work on all linux distributions.

1. kill a group of processes with negative PID(Process ID)
> kill  -TERM -PID
Is to kill PID and all its child processes.

2. kill a group of processes with their PGID(Process Group ID)
> kill -- -$PGID   Kill using the default signal (TERM = 15)
 kill -9 -$PGID   Kill using the KILL signal (9)

3. kill a group processes with only PID info
> kill -- -$(ps -o pgid= $PID | grep -o [0-9]\*)
Actually, you may notice that it's just the way from #2

4. Using pkill, kill processes by PGID(Proess Group ID)
> pkill -9 -g $PGID

5.Using pkill, kill processes by GID(Group ID)
> pkill -9 -G $GID

6. Using pkill, kill processes by PPID(Parent Process ID)*
> pkill -9 -P $PPID

7. Using pkill, kill processes by terminal
> pkill -9 -t $terminal
Note: without /dev/ prefix

8. Using pkill, kill processes by process name
> pkill -9 -x $process_name

9. Using pkill, kill processes by session
> pkill -9 -s $sess

0. How to get PID,PGID,sessionid etc?
> \# ps -o pid,ppid,pgid,gid,sess,cmd -U root
>   PID  PPID  PGID   GID  SESS CMD

source: http://fibrevillage.com/sysadmin
