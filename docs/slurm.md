# Slurm

[Slurm](https://slurm.schedmd.com/overview.html) is an open source job scheduling system for Linux clusters often used in the HPC space.

Most of SLURM related help can be obtained from the excellent wiki here: [https://wiki.fysik.dtu.dk/niflheim/SLURM](https://wiki.fysik.dtu.dk/niflheim/SLURM)

Here I note some tips/tricks/commands that I ended up discovering from my $dayjob as a sysadmin maanging a slurm cluster.

### Run my job in a specific node in the cluster
```bash
scontrol update jobid=734116 qos=high partition=ghpc_v2 nodelist=sky006
```