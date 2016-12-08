# 电源管理



linux 3.10

# cat /sys/power/state 
freeze standby mem

freeze  这种Power State，并不涉及具体的Hardware或Driver，只是冻结所有的进程，包括用户空间进程及内核线程。

standby  

mem，即通常所讲的Sleep功能

