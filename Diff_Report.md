<h1>CODE MODIFICATION REPORT</h1>
<hr>
<h2>makefile</h2> 

```CS333_PROJECT ?= 2```

```CS333_UPROGS += #_date _time _ps```

<h2>p2-test.c</h2> 

```
#define CPUTIME_TEST
#define GETPROCS_TEST
#define TIME_TEST
```

<h2>proc.c</h2> 

```
#ifdef CS333_P2
#include "uproc.h"
#endif
```

```
#ifdef CS333_P2
  p->cpu_ticks_total = 0;
  p->cpu_ticks_in = 0;
#endif
```

```
#ifdef CS333_P2
  p->uid = DEFAULT_UID;
  p->gid = DEFAULT_GID;
  #endif
``` 

```
#ifdef CS333_P2
  np->uid = curproc->uid;
  np->gid = curproc->gid;
#endif
```


```
#ifdef CS333_P2
p->cpu_ticks_in = ticks;
#endif // CS333_P2
```

```
#ifdef CS333_P2
  p->cpu_ticks_total += (ticks - p->cpu_ticks_in);
  #endif // CS333_P2
```

```
  uint elapsed_s;
  uint elapsed_ms;

  elapsed_ms = ticks - p->start_ticks;
  elapsed_s = elapsed_ms / 1000;
  elapsed_ms = elapsed_ms % 1000;

  uint elapsed_cpu_s;
  uint elapsed_cpu_ms;
  uint ppid;
  if(p->parent){
    ppid = p->parent->pid;
  }
  else{
    ppid = p->pid;
  }

  elapsed_cpu_ms = p->cpu_ticks_total;
  elapsed_cpu_s = elapsed_cpu_ms / 1000;
  elapsed_cpu_ms = elapsed_cpu_ms % 1000;

  char* zero = "";
  if(elapsed_ms < 100 && elapsed_ms >= 10)
    zero = "0";
  if(elapsed_ms < 10)
    zero = "00";

  char* cpu_zero = "";
  if(elapsed_cpu_ms < 100 && elapsed_cpu_ms >= 10)
    cpu_zero = "0";
  if(elapsed_cpu_ms < 10)
    cpu_zero = "00";

  cprintf(
    "\n%d\t%s\t%s%d\t%s%d\t%s%d\t%d.%s%d\t%d.%s%d\t%s\t%d\t", 
    p->pid, 
    p->name, "     ",
    p->uid, "        ", 
    p->gid, "",
    ppid, 
    elapsed_s, zero, elapsed_ms, 
    elapsed_cpu_s, cpu_zero, elapsed_cpu_ms, 
    state_string, 
    p->sz
```
<h2>proc.h</h2> 

```
uint uid;
uint gid;
```

<h2>syscall.c</h2>

```
#ifdef CS333_P2
extern int sys_getuid(void);
extern int sys_getgid(void);
extern int sys_getppid(void);
extern int sys_setuid(void);
extern int sys_setgid(void);
#endif
```
<h2>ps.c</h2>

```#ifdef CS333_P2
#include "types.h"
#include "user.h"
#include "uproc.h"

#define MAX 16

int
main(void)
{
  struct uproc *proc = malloc(sizeof(struct uproc)*MAX);
  int proc_num = getprocs(MAX, proc);
  printf(1,"PID\tName\t\tUID\tGID\tPPID\tElapsed\tCPU\tState\tSize\n");

  int i;
  for(i = 0; i<proc_num; i++){
    struct uproc current_proc = proc[i];
    uint elapsed_ticks = current_proc.elapsed_ticks;
    uint elapsed_s = elapsed_ticks/1000;
    uint elapsed_ms = elapsed_ticks%1000;

    uint elapsed_cpu_ticks = current_proc.CPU_total_ticks;
    uint elapsed_cpu_s = elapsed_cpu_ticks/1000;
    uint elapsed_cpu_ms = elapsed_cpu_ticks % 1000;
    
    char* zero = "";
    if(elapsed_ms < 100 && elapsed_ms >= 10)
      zero = "0";
    if(elapsed_ms < 10)
      zero = "00";

    char* cpu_zero = "";
    if(elapsed_cpu_ms < 100 && elapsed_cpu_ms >= 10)
      cpu_zero = "0";
    if(elapsed_cpu_ms < 10)
      cpu_zero = "00";

    printf(
      1,
      "%d\t%s\t\t%d\t%d\t%d\t%d.%s%d\t%d.%s%d\t%s\t%d\n",
      current_proc.pid,
      current_proc.name,
      current_proc.uid,
      current_proc.gid,
      current_proc.ppid,
      elapsed_s, zero, elapsed_ms,
      elapsed_cpu_s, cpu_zero, elapsed_cpu_ms,
      current_proc.state,
      current_proc.size
    );
  }

  free(proc);
  exit();
}
#endif
```
<h2>syscall.c</h2>
```
#ifdef CS333_P2
extern int sys_getuid(void);
extern int sys_getgid(void);
extern int sys_getppid(void);
extern int sys_setuid(void);
extern int sys_setgid(void);
extern int sys_getprocs(void);
#endif
```

```
#ifdef CS333_P2
[SYS_getuid]  sys_getuid,
[SYS_getgid]  sys_getgid,
[SYS_getppid] sys_getppid,
[SYS_setuid]  sys_setuid,
[SYS_setgid]  sys_setgid,
[SYS_getprocs]  sys_getprocs,
#endif
```

```
#ifdef CS333_P2
[SYS_getuid]  "getuid",
[SYS_getgid]  "getgid",
[SYS_getppid] "getppid",
[SYS_setuid]  "setuid",
[SYS_setgid]  "setgid",
[SYS_getprocs]  "getprocs",
#endif
```
<h2>syscall.h</h2>
```
#define SYS_getuid   SYS_date+1
#define SYS_getgid   SYS_getuid+1
#define SYS_getppid  SYS_getgid+1
#define SYS_setuid   SYS_getppid+1
#define SYS_setgid   SYS_setuid+1
```
<h2>sysproc.c</h2>
```
#ifdef CS333_P2
int
sys_getuid(void)
{
  return myproc()->uid;
}
int
sys_getgid(void)
{
  return myproc()->gid;
}
int
sys_getppid(void)
{
  if(myproc()->pid == 1)
    return myproc()->pid;
  return myproc()->parent->pid;
}
int
sys_setuid(void)
{
  int tmp;
  if(argint(0,&tmp) < 0 || tmp > 32767 || tmp < 0)
    return -1;
  myproc()->uid = (uint)tmp;
  return 0;
}
int
sys_setgid(void)
{
  int tmp;
  if(argint(0,&tmp) < 0 || tmp > 32767 || tmp < 0)
    return -1;
  myproc()->gid = (uint)tmp;
  return 0;
}
#endif //CS333_P2
```
<h2>time.c</h2>

```
#ifdef CS333_P2
#include "types.h"
#include "user.h"

int main(int argc, char *argv[]){
    if(argc == 1) {
      printf(1, "(null) ran in 0.00\n");
    } else {
      int start = uptime();
      int pid = fork();

      if (pid > 0) {
        pid = wait();
      } else if (pid == 0) {
        exec(argv[1], argv+1);
        printf(1, "ERROR: Unknown Command\n");
        kill(getppid());
        exit();
      } else {
        printf(1, "ERROR: Fork error return -1\n");
      }

      int end = uptime();
      int timelapse = end - start;
      int seconds = timelapse/1000;
      int ms = timelapse%1000;
      char *msZeros = "";

      if (ms < 10) {
        msZeros = "00";
      } else if (ms < 100) {
        msZeros = "0";
      }

      printf(
        1,
        "%s ran in %d.%s%d\n",
        argv[1],
        seconds,
        msZeros,
        ms
      );
    }
    exit();
}
#endif
```

<h2>user.h</h2>

```
#ifdef CS333_P2
uint getuid(void); // UID of the parent process
uint getgid(void); // GID of the parent process
uint getppid(void); // process ID of the parent process
int setuid(uint); // set UID
int setgid(uint); // set GID
int getprocs(uint max, struct uproc* table);
#endif
```

<h2>usys.S</h2>

```
SYSCALL(getuid)
SYSCALL(getgid)
SYSCALL(getppid)
SYSCALL(setuid)
SYSCALL(setgid)
```

<h2>defs.h</h2>

```
#ifdef CS333_P2
#include "uproc.h"
#endif
```

```
#ifdef CS333_P2
int getprocs(uint max, struct uproc* upTable);
#endif //CS333_P2
```

