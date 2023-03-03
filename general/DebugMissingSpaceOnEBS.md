## Debug missing space on EBS ( Linux )

*	check disk usage using “df -h”
```
[root@ip-172-27-9-181 /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         60G   68K   60G   1% /dev
tmpfs            61G     0   61G   0% /dev/shm
/dev/xvda1      7.8G  7.4G  279M  97% /
```

* check in detail how the space is used ( 97%) 
```
[root@ip-172-27-9-181 /]# du -sh *
7.0M	bin
44M	boot
4.0K	cgroup
68K	dev
9.9M	etc
92M	home
102M	lib
20M	lib64
4.0K	local
16K	lost+found
8.0K	media
4.0K	mnt
106M	opt
0	proc
37M	root
8.0K	run
13M	sbin
4.0K	selinux
4.0K	srv
0	sys
72K	tmp
1.1G	usr
809M	var
```

**From what we see , the usage is less than 2GB, so where is the rest of the space ? Almost 8 GB is being used somehow.**

* Investigate further. Using “lsof | grep deleted” we are going to look for processes that are in deleted state 
```
[root@ip-172-27-9-181 /]#  lsof |grep deleted
python     19990 aerospike    3u      REG              202,1    5452255357     395534 /var/log/aerospike/aerospike.log-20210628 (deleted)
python     19990 aerospike    4w      REG              202,1        488279     395686 /var/log/aerospike/telemetry.log-20210701 (deleted)
```

Now we can see that over 5GB are in deleted state, but the space is still consumed . 

**How to fix the issue**  
Use the pid that we’ve found in the previous command and go to that folder . In our case the pid is 19990 and list files.
```
[root@ip-172-27-9-181 /]# cd /proc/19990/fd
[root@ip-172-27-9-181 fd]# ll
total 0
lr-x------ 1 root root 64 Sep 28 13:31 0 -> /dev/null
lrwx------ 1 root root 64 Sep 28 13:31 1 -> /dev/null
lr-x------ 1 root root 64 Sep 28 13:31 11 -> /dev/urandom
lrwx------ 1 root root 64 Sep 28 13:31 2 -> /dev/null
lrwx------ 1 root root 64 Sep 28 13:31 3 -> /var/log/aerospike/aerospike.log-20210628 (deleted)
l-wx------ 1 root root 64 Sep 28 13:31 4 -> /var/log/aerospike/telemetry.log-20210701 (deleted)
```

After we list the files we see those 2 pid with problems. The 3 and 4 are pointing to some other files that were “deleted”

Time to recover the space. Execute the following command to clear the file 3 and 4. 
```
echo -n > 3
echo -n > 4
```

Let’s check to see how it looks after executing the above command for both files
```
[root@ip-172-27-9-181 fd]#  lsof |grep deleted
python     19990 aerospike    3u      REG              202,1             0     395534 /var/log/aerospike/aerospike.log-20210628 (deleted)
python     19990 aerospike    4w      REG              202,1             0     395686 /var/log/aerospike/telemetry.log-20210701 (deleted)
```

The space seems to be recovered, and we can also confirm it with df -h
```
[root@ip-172-27-9-181 fd]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         60G   68K   60G   1% /dev
tmpfs            61G     0   61G   0% /dev/shm
/dev/xvda1      7.8G  2.3G  5.5G  29% /
```


> **Root Cause \
On Linux or Unix systems, deleting a file via rm or through a file manager application will unlink the file from the file system's directory structure; however, if the file is still open (in use by a running process) it will still be accessible to this process and will continue to occupy space on disk. Therefore such processes may need to be restarted before that file's space will be cleared up on the filesystem.**