# ZFS

All things ZFS. 

I use ZFS in production at work. Here are some notes on this wonderful piece of software. 

## Simple view of ZFS
```bash
root@ironbank01:~ # zpool list
NAME        SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
iron_bank   587T   107T   481T        -         -     0%    18%  1.00x  ONLINE  -
```
CAP is short for capacity. This filer is 18% full.

## Show all datasets and how full they are
```bash
root@ironbank01:~ # zfs list
NAME                              USED  AVAIL  REFER  MOUNTPOINT
iron_bank                        71.0T   308T   192K  /iron_bank
~~~~~~~~~~~~~~~~~~~~~ Trimmmed for brevity ~~~~~~~~~~~~~~~~~~~~~
```

Datasets are cheap with ZFS and provide granular control over tunables specific to the workload. For example a dataset with research data can have a record size of 1 MiB resulting in better seq.read and seq.write performance, whereas a dataset with home directory of a user can be left with a default recordsize of 128 KiB for better overall performance.

## List all snapshots and show how much space they occupy :
```bash
zfs list -t snapshot
```

## Show status of all disks in ZPOOLs

```bash
root@ironbank01:~ # zpool status
  pool: iron_bank
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	iron_bank   ONLINE       0     0     0
	  raidz2-0  ONLINE       0     0     0
	    da0     ONLINE       0     0     0
	    da1     ONLINE       0     0     0
	    da2     ONLINE       0     0     0
	    da90    ONLINE       0     0     0
	    da91    ONLINE       0     0     0
	    da92    ONLINE       0     0     0
	  raidz2-2  ONLINE       0     0     0
	    da3     ONLINE       0     0     0
~~~~~~~~~~~~~~~~~~~~~ Trimmmed for brevity ~~~~~~~~~~~~~~~~~~~~~
	logs
	  nvd0      ONLINE       0     0     0
	spares
	  da27      AVAIL   
	  da117     AVAIL   

errors: No known data errors
```

## Check compression efficiency on a dataset

```bash
zfs get used,compressratio,compression,logicalused iron_bank/qgg
NAME           PROPERTY       VALUE     SOURCE
iron_bank/qgg  used           60.2T     -
iron_bank/qgg  compressratio  1.19x     -
iron_bank/qgg  compression    lz4       local
iron_bank/qgg  logicalused    71.0T     -
```

Ofcourse, the compression ratio depends on the type of data ZFS is dealt with. But, ZFS is smart enough to not burn CPU cycles when the data is already compressed and it knows when to give up. So, keep compression ON everywhere.

## View of all ZFS related data in FreeBSD system

Install ZFS-stats:

```bash
pkg install zfs-stats
```

Get info:

```bash
root@ironbank01:~ # zfs-stats -a

------------------------------------------------------------------------
ZFS Subsystem Report				Wed Jul 17 17:09:29 2019
------------------------------------------------------------------------

System Information:

	Kernel Version:				1200086 (osreldate)
	Hardware Platform:			amd64
	Processor Architecture:			amd64

	ZFS Storage pool Version:		5000
	ZFS Filesystem Version:			5

FreeBSD 12.0-RELEASE-p3 GENERIC
 5:09PM  up 27 days,  5:53, 1 user, load averages: 0.12, 0.17, 0.09

------------------------------------------------------------------------

System Memory:

	0.00%	2.14	MiB Active,	0.07%	275.17	MiB Inact
	97.55%	364.91	GiB Wired,	0.00%	0 Cache
	2.32%	8.70	GiB Free,	0.05%	206.81	MiB Gap

	Real Installed:				384.00	GiB
	Real Available:			99.91%	383.65	GiB
	Real Managed:			97.50%	374.07	GiB

	Logical Total:				384.00	GiB
	Logical Used:			97.67%	375.04	GiB
	Logical Free:			2.33%	8.96	GiB

Kernel Memory:					6.02	GiB
	Data:				99.36%	5.98	GiB
	Text:				0.64%	39.24	MiB

Kernel Memory Map:				374.07	GiB
	Size:				97.33%	364.09	GiB
	Free:				2.67%	9.98	GiB

------------------------------------------------------------------------

ARC Summary: (HEALTHY)
	Memory Throttle Count:			0

ARC Misc:
	Deleted:				86.17m
	Recycle Misses:				0
	Mutex Misses:				9.47k
	Evict Skips:				0

ARC Size:				90.67%	335.47	GiB
	Target Size: (Adaptive)		90.72%	335.66	GiB
	Min Size (Hard Limit):		54.05%	200.00	GiB
	Max Size (High Water):		1:1	370.00	GiB

ARC Size Breakdown:
	Recently Used Cache Size:	92.44%	310.29	GiB
	Frequently Used Cache Size:	7.56%	25.37	GiB

ARC Hash Breakdown:
	Elements Max:				5.97m
	Elements Current:		98.82%	5.90m
	Collisions:				8.31m
	Chain Max:				5
	Chains:					243.32k

------------------------------------------------------------------------

ARC Efficiency:					2.82b
	Cache Hit Ratio:		98.39%	2.78b
	Cache Miss Ratio:		1.61%	45.39m
	Actual Hit Ratio:		98.32%	2.78b

	Data Demand Efficiency:		100.00%	51.29m

	CACHE HITS BY CACHE LIST:
	  Anonymously Used:		0.06%	1.57m
	  Most Recently Used:		9.05%	251.42m
	  Most Frequently Used:		90.88%	2.53b
	  Most Recently Used Ghost:	0.00%	0
	  Most Frequently Used Ghost:	0.01%	342.08k

	CACHE HITS BY DATA TYPE:
	  Demand Data:			1.85%	51.29m
	  Prefetch Data:		0.00%	0
	  Demand Metadata:		98.06%	2.72b
	  Prefetch Metadata:		0.10%	2.75m

	CACHE MISSES BY DATA TYPE:
	  Demand Data:			0.00%	0
	  Prefetch Data:		0.00%	0
	  Demand Metadata:		99.03%	44.95m
	  Prefetch Metadata:		0.97%	441.19k

------------------------------------------------------------------------

L2ARC is disabled

------------------------------------------------------------------------

File-Level Prefetch: (HEALTHY)

DMU Efficiency:					1.29b
	Hit Ratio:			5.97%	77.29m
	Miss Ratio:			94.03%	1.22b

	Colinear:				0
	  Hit Ratio:			100.00%	0
	  Miss Ratio:			100.00%	0

	Stride:					0
	  Hit Ratio:			100.00%	0
	  Miss Ratio:			100.00%	0

DMU Misc:
	Reclaim:				0
	  Successes:			100.00%	0
	  Failures:			100.00%	0

	Streams:				0
	  +Resets:			100.00%	0
	  -Resets:			100.00%	0
	  Bogus:				0

------------------------------------------------------------------------

VDEV Cache Summary:				1.01m
	Hit Ratio:			14.99%	150.83k
	Miss Ratio:			82.75%	832.62k
	Delegations:			2.25%	22.69k

------------------------------------------------------------------------

ZFS Tunables (sysctl):
	kern.maxusers                           24889
	vm.kmem_size                            401658576896
	vm.kmem_size_scale                      1
	vm.kmem_size_min                        0
	vm.kmem_size_max                        1319413950874
	vfs.zfs.trim.max_interval               1
	vfs.zfs.trim.timeout                    30
	vfs.zfs.trim.txg_delay                  32
	vfs.zfs.trim.enabled                    1
	vfs.zfs.vol.immediate_write_sz          32768
	vfs.zfs.vol.unmap_sync_enabled          0
	vfs.zfs.vol.unmap_enabled               1
	vfs.zfs.vol.recursive                   0
	vfs.zfs.vol.mode                        1
	vfs.zfs.version.zpl                     5
	vfs.zfs.version.spa                     5000
	vfs.zfs.version.acl                     1
	vfs.zfs.version.ioctl                   7
	vfs.zfs.debug                           0
	vfs.zfs.super_owner                     0
	vfs.zfs.immediate_write_sz              32768
	vfs.zfs.sync_pass_rewrite               2
	vfs.zfs.sync_pass_dont_compress         5
	vfs.zfs.sync_pass_deferred_free         2
	vfs.zfs.zio.dva_throttle_enabled        1
	vfs.zfs.zio.exclude_metadata            0
	vfs.zfs.zio.use_uma                     1
	vfs.zfs.zil_slog_bulk                   786432
	vfs.zfs.cache_flush_disable             0
	vfs.zfs.zil_replay_disable              0
	vfs.zfs.standard_sm_blksz               131072
	vfs.zfs.dtl_sm_blksz                    4096
	vfs.zfs.min_auto_ashift                 9
	vfs.zfs.max_auto_ashift                 13
	vfs.zfs.vdev.trim_max_pending           10000
	vfs.zfs.vdev.bio_delete_disable         0
	vfs.zfs.vdev.bio_flush_disable          0
	vfs.zfs.vdev.def_queue_depth            32
	vfs.zfs.vdev.queue_depth_pct            1000
	vfs.zfs.vdev.write_gap_limit            4096
	vfs.zfs.vdev.read_gap_limit             32768
	vfs.zfs.vdev.aggregation_limit          1048576
	vfs.zfs.vdev.initializing_max_active    1
	vfs.zfs.vdev.initializing_min_active    1
	vfs.zfs.vdev.removal_max_active         2
	vfs.zfs.vdev.removal_min_active         1
	vfs.zfs.vdev.trim_max_active            64
	vfs.zfs.vdev.trim_min_active            1
	vfs.zfs.vdev.scrub_max_active           2
	vfs.zfs.vdev.scrub_min_active           1
	vfs.zfs.vdev.async_write_max_active     10
	vfs.zfs.vdev.async_write_min_active     1
	vfs.zfs.vdev.async_read_max_active      3
	vfs.zfs.vdev.async_read_min_active      1
	vfs.zfs.vdev.sync_write_max_active      10
	vfs.zfs.vdev.sync_write_min_active      10
	vfs.zfs.vdev.sync_read_max_active       10
	vfs.zfs.vdev.sync_read_min_active       10
	vfs.zfs.vdev.max_active                 1000
	vfs.zfs.vdev.async_write_active_max_dirty_percent60
	vfs.zfs.vdev.async_write_active_min_dirty_percent30
	vfs.zfs.vdev.mirror.non_rotating_seek_inc1
	vfs.zfs.vdev.mirror.non_rotating_inc    0
	vfs.zfs.vdev.mirror.rotating_seek_offset1048576
	vfs.zfs.vdev.mirror.rotating_seek_inc   5
	vfs.zfs.vdev.mirror.rotating_inc        0
	vfs.zfs.vdev.trim_on_init               1
	vfs.zfs.vdev.cache.bshift               16
	vfs.zfs.vdev.cache.size                 16777216
	vfs.zfs.vdev.cache.max                  16384
	vfs.zfs.vdev.default_ms_shift           29
	vfs.zfs.vdev.min_ms_count               16
	vfs.zfs.vdev.max_ms_count               200
	vfs.zfs.txg.timeout                     5
	vfs.zfs.space_map_ibs                   14
	vfs.zfs.spa_allocators                  4
	vfs.zfs.spa_min_slop                    134217728
	vfs.zfs.spa_slop_shift                  5
	vfs.zfs.spa_asize_inflation             24
	vfs.zfs.deadman_enabled                 0
	vfs.zfs.deadman_checktime_ms            5000
	vfs.zfs.deadman_synctime_ms             1000000
	vfs.zfs.debugflags                      0
	vfs.zfs.recover                         0
	vfs.zfs.spa_load_verify_data            1
	vfs.zfs.spa_load_verify_metadata        1
	vfs.zfs.spa_load_verify_maxinflight     10000
	vfs.zfs.max_missing_tvds_scan           0
	vfs.zfs.max_missing_tvds_cachefile      2
	vfs.zfs.max_missing_tvds                0
	vfs.zfs.spa_load_print_vdev_tree        0
	vfs.zfs.ccw_retry_interval              300
	vfs.zfs.check_hostid                    1
	vfs.zfs.mg_fragmentation_threshold      85
	vfs.zfs.mg_noalloc_threshold            0
	vfs.zfs.condense_pct                    200
	vfs.zfs.metaslab_sm_blksz               4096
	vfs.zfs.metaslab.bias_enabled           1
	vfs.zfs.metaslab.lba_weighting_enabled  1
	vfs.zfs.metaslab.fragmentation_factor_enabled1
	vfs.zfs.metaslab.preload_enabled        1
	vfs.zfs.metaslab.preload_limit          3
	vfs.zfs.metaslab.unload_delay           8
	vfs.zfs.metaslab.load_pct               50
	vfs.zfs.metaslab.min_alloc_size         33554432
	vfs.zfs.metaslab.df_free_pct            4
	vfs.zfs.metaslab.df_alloc_threshold     131072
	vfs.zfs.metaslab.debug_unload           0
	vfs.zfs.metaslab.debug_load             0
	vfs.zfs.metaslab.fragmentation_threshold70
	vfs.zfs.metaslab.force_ganging          16777217
	vfs.zfs.free_bpobj_enabled              1
	vfs.zfs.free_max_blocks                 -1
	vfs.zfs.zfs_scan_checkpoint_interval    7200
	vfs.zfs.zfs_scan_legacy                 0
	vfs.zfs.no_scrub_prefetch               0
	vfs.zfs.no_scrub_io                     0
	vfs.zfs.resilver_min_time_ms            3000
	vfs.zfs.free_min_time_ms                1000
	vfs.zfs.scan_min_time_ms                1000
	vfs.zfs.scan_idle                       50
	vfs.zfs.scrub_delay                     4
	vfs.zfs.resilver_delay                  2
	vfs.zfs.top_maxinflight                 32
	vfs.zfs.zfetch.array_rd_sz              1048576
	vfs.zfs.zfetch.max_idistance            67108864
	vfs.zfs.zfetch.max_distance             8388608
	vfs.zfs.zfetch.min_sec_reap             2
	vfs.zfs.zfetch.max_streams              8
	vfs.zfs.prefetch_disable                0
	vfs.zfs.delay_scale                     500000
	vfs.zfs.delay_min_dirty_percent         60
	vfs.zfs.dirty_data_sync                 67108864
	vfs.zfs.dirty_data_max_percent          10
	vfs.zfs.dirty_data_max_max              4294967296
	vfs.zfs.dirty_data_max                  4294967296
	vfs.zfs.max_recordsize                  1048576
	vfs.zfs.default_ibs                     17
	vfs.zfs.default_bs                      9
	vfs.zfs.send_holes_without_birth_time   1
	vfs.zfs.mdcomp_disable                  0
	vfs.zfs.per_txg_dirty_frees_percent     30
	vfs.zfs.nopwrite_enabled                1
	vfs.zfs.dedup.prefetch                  1
	vfs.zfs.dbuf_cache_lowater_pct          10
	vfs.zfs.dbuf_cache_hiwater_pct          10
	vfs.zfs.dbuf_metadata_cache_overflow    0
	vfs.zfs.dbuf_metadata_cache_shift       6
	vfs.zfs.dbuf_cache_shift                5
	vfs.zfs.dbuf_metadata_cache_max_bytes   6259138048
	vfs.zfs.dbuf_cache_max_bytes            12518276096
	vfs.zfs.arc_min_prescient_prefetch_ms   6
	vfs.zfs.arc_min_prefetch_ms             1
	vfs.zfs.l2c_only_size                   0
	vfs.zfs.mfu_ghost_data_esize            307525451776
	vfs.zfs.mfu_ghost_metadata_esize        120832
	vfs.zfs.mfu_ghost_size                  307525572608
	vfs.zfs.mfu_data_esize                  28770021376
	vfs.zfs.mfu_metadata_esize              1982464
	vfs.zfs.mfu_size                        40105465856
	vfs.zfs.mru_ghost_data_esize            52882492416
	vfs.zfs.mru_ghost_metadata_esize        0
	vfs.zfs.mru_ghost_size                  52882492416
	vfs.zfs.mru_data_esize                  287148002304
	vfs.zfs.mru_metadata_esize              4036173312
	vfs.zfs.mru_size                        304771647488
	vfs.zfs.anon_data_esize                 0
	vfs.zfs.anon_metadata_esize             0
	vfs.zfs.anon_size                       32768
	vfs.zfs.l2arc_norw                      1
	vfs.zfs.l2arc_feed_again                1
	vfs.zfs.l2arc_noprefetch                1
	vfs.zfs.l2arc_feed_min_ms               200
	vfs.zfs.l2arc_feed_secs                 1
	vfs.zfs.l2arc_headroom                  2
	vfs.zfs.l2arc_write_boost               8388608
	vfs.zfs.l2arc_write_max                 8388608
	vfs.zfs.arc_meta_strategy               0
	vfs.zfs.arc_meta_limit                  99321118720
	vfs.zfs.arc_free_target                 2088965
	vfs.zfs.arc_kmem_cache_reap_retry_ms    0
	vfs.zfs.compressed_arc_enabled          1
	vfs.zfs.arc_grow_retry                  60
	vfs.zfs.arc_shrink_shift                7
	vfs.zfs.arc_average_blocksize           8192
	vfs.zfs.arc_no_grow_shift               5
	vfs.zfs.arc_min                         214748364800
	vfs.zfs.arc_max                         397284474880
	vfs.zfs.abd_chunk_size                  4096
	vfs.zfs.abd_scatter_enabled             1

------------------------------------------------------------------------
```

## View the current utilisation of disk IO bandwidth

Show live updating view of R/W operations and bandwidth for a pool

```bash
zpool iostat 1 
```

Keep an eye on one disk. In my case the SLOG device - Intel Optane disk identified by nvd0

```bash
zpool iostat -v 1 |grep /dev/nvd0 
```

## ZFS and NFS

To share a ZFS dataset over NFS and prevent a user from getting root privileges over NFS, 

```bash
zfs set sharenfs="-network 172.16.0.0/17 -maproot=nfsroot" iron_bank/researchdata/aus_data
service nfsd restart
```
Where, the NFS share can be mounted only on 172.16.0.0/17 and a root user will be mapped to a dummy nfsroot account on this file server.

Alternatively, if you need to be able to access a dataset as root over NFS,

```bash
zfs set sharenfs="-network 172.16.0.0/17 -maproot=root" iron_bank/researchdata/aus_data
service nfsd restart
```

## Creating a ZFS dataset
```bash
zfs create -o compress=lz4 -o snapdir=visible iron_bank/researchdata
```

## Crucial datasets where data integrity takes double priority over performance
```bash
zfs create -o compress=lz4 -o snapdir=visible iron_bank/researchdata
zfs set sync=always iron_bank/researchdata
```

## ZFS Tunables

```bash
root@ironbank01:~ # grep vfs.zfs /boot/loader.conf
vfs.zfs.deadman_enabled=0
vfs.zfs.vdev.cache.size=16M
```

On /etc/sysctl.conf
```bash
# 200GiB of ARC minimum
vfs.zfs.arc_min=214748364800
# 150 GiB of ARC metadata limit
vfs.zfs.arc_meta_limit=161061273600
# 370 GiB of max ARC to prevent system from losing NFS memory
vfs.zfs.arc_max=397284474880
```

## Testing disks primitively

```bash
root@ironbank01:~ # diskinfo -ctv /dev/da1
/dev/da1
	512         	# sectorsize
	12000138625024	# mediasize in bytes (11T)
	23437770752 	# mediasize in sectors
	4096        	# stripesize
	0           	# stripeoffset
	1458933     	# Cylinders according to firmware.
	255         	# Heads according to firmware.
	63          	# Sectors according to firmware.
	HGST HUH721212AL5200	# Disk descr.
	8HKX1R0H    	# Disk ident.
	id1,enc@n5000ccab04075dbc/type@0/slot@2/elmdesc@SLOT_01,8HKX1R0H____________	# Physical path
	No          	# TRIM/UNMAP support
	7200        	# Rotation rate in RPM
	Not_Zoned   	# Zone Mode

I/O command overhead:
	time to read 10MB block      0.063105 sec	=    0.003 msec/sector
	time to read 20480 sectors   1.243264 sec	=    0.061 msec/sector
	calculated command overhead			=    0.058 msec/sector

Seek times:
	Full stroke:	  250 iter in   4.591331 sec =   18.365 msec
	Half stroke:	  250 iter in   3.427104 sec =   13.708 msec
	Quarter stroke:	  500 iter in   5.592419 sec =   11.185 msec
	Short forward:	  400 iter in   1.777049 sec =    4.443 msec
	Short backward:	  400 iter in   2.028514 sec =    5.071 msec
	Seq outer:	 2048 iter in   0.085151 sec =    0.042 msec
	Seq inner:	 2048 iter in   0.132923 sec =    0.065 msec

Transfer rates:
	outside:       102400 kbytes in   0.439125 sec =   233191 kbytes/sec
	middle:        102400 kbytes in   0.519029 sec =   197291 kbytes/sec
	inside:        102400 kbytes in   0.819735 sec =   124918 kbytes/sec

```