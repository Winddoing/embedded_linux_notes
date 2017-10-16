# SMP

```
struct plat_smp_ops {                                                               
	void (*send_ipi_single)(int cpu, unsigned int action);                      
	void (*send_ipi_mask)(const struct cpumask *mask, unsigned int action);     
	void (*init_secondary)(void);                                               
	void (*smp_finish)(void);                                                   
	void (*cpus_done)(void);                                                    
	void (*boot_secondary)(int cpu, struct task_struct *idle);                  
	void (*smp_setup)(void);                                                    
	void (*prepare_cpus)(unsigned int max_cpus);                                
#ifdef CONFIG_HOTPLUG_CPU                                                       
	int (*cpu_disable)(void);                                                   
	void (*cpu_die)(unsigned int cpu);                                          
#endif                                                                          
};                                                                              
```
> include/asm/smp-ops.h


## smp_init

```
smp_init
	|-> 
```
