<h1> Task 1 Syscall Tracing </h1>

<h2> syscall.c </h2>

```void
syscall(void)
{
  int num;
  struct proc *curproc = myproc();

  num = curproc->tf->eax;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    curproc->tf->eax = syscalls[num]();
    #ifdef PRINT_SYSCALLS
      cprintf("%s -> %d \n",syscallnames[num], num);
    #endif
  } else {
    cprintf("%d %s: unknown sys call %d\n",
            curproc->pid, curproc->name, num);
    curproc->tf->eax = -1;
  }
}
```

<h1> Date System Call </h1>

<h2> makefile </h2>

```CS333_PROJECT ?= 1
PRINT_SYSCALLS ?= 0
CS333_CFLAGS ?= -DPDX_XV6
ifeq ($(CS333_CFLAGS), -DPDX_XV6)
CS333_UPROGS +=	_halt _uptime
endif

ifeq ($(PRINT_SYSCALLS), 1)
CS333_CFLAGS += -DPRINT_SYSCALLS
endif

ifeq ($(CS333_PROJECT), 1)
CS333_CFLAGS += -DCS333_P1
CS333_UPROGS += _date
endif
```

<h2>user. h</h2>
```#ifdef CS333_P1
int date(struct rtcdate*);
#endif
```

<h2> usys.S </h2>
```SYSCALL(date)
```

<h2> syscall.h </h2>
```#define SYS_date    SYS_halt+1
```

<h2> syscall.c </h2>
```#ifdef CS333_P1
extern int sys_date(void);
#endif // CS333_P1
#ifdef CS333_P1
[SYS_date]    sys_date,
#endif // PDX_XV6
};
```

<h2>sysproc.c</h2>
```int
sys_date(void)
{
  struct rtcdate *d;
  if (argptr(0 ,(void*)&d , sizeof(struct rtcdate)) < 0)
    return -1;
  cmostime(d);
  return 0;
}
```

<h1>Process Information</h1>
<h2>proc.h </h2>
```uint start_ticks;
```

<h2>proc.c</h2>
```p->start_ticks = ticks;

void procdumpP1(struct proc *p, char *state_string)
{
  int elapsed_s;
  int elapsed_ms;

  elapsed_ms = ticks - p->start_ticks;
  elapsed_s = elapsed_ms / 1000;
  elapsed_ms = elapsed_ms % 1000;

  char *nol = "";
  if (elapsed_ms < 100 && elapsed_ms >= 10)
    nol = "0";
  if (elapsed_ms < 10)
    nol = "00";

  cprintf("%d\t%s\t%s%d.%s%d\t%s\t%d\t",
          p->pid, p->name, "     ", elapsed_s, nol, elapsed_ms, states[p->state], p->sz);
  return;
}
#endif
```
