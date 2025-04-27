# install-CVMFS-server-on-Rocky-Linux-8
Install and configure CVMFS server on Rocky Linux 8, and CVMFS client test mount

Test drive the CVMFS server, and client, on a same node. Just for test.

1. First intall the CVMFS repo
```   
yum install -y https://cvmrepo.s3.cern.ch/cvmrepo/yum/cvmfs-release-latest.noarch.rpm
```

2. Instll and configure server
```
yum install cvmfs-server
cvmfs_server mkfs my.repo.name
cvmfs_server  transaction
...
#Install software here, I just simply copy ones to the repo folder:
cd /cvmfs/my.repo.name
rsync -av /opt/ohpc/pub ./
cd ~/

...
cvmfs_server  publish 
```

Now you should see the mounted folders:

```
df -h
...
my.repo.name          4.0G  5.8M  4.0G   1% /var/spool/cvmfs/my.repo.name/rdonly
overlay_my.repo.name  100G   56G   45G  56% /cvmfs/my.repo.name
```

and

```
mount|grep my.repo
...
my.repo.name on /var/spool/cvmfs/my.repo.name/rdonly type fuse (ro,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other)
overlay_my.repo.name on /cvmfs/my.repo.name type overlay (ro,nodev,relatime,seclabel,lowerdir=/var/spool/cvmfs/my.repo.name/rdonly,upperdir=/var/spool/cvmfs/my.repo.name/scratch/current,workdir=/var/spool/cvmfs/my.repo.name/ofs_workdir)

```

3. On the same node, install CVMFS client and test mount

```
yum install cvmfs
```

Then update the local config file as below:
```
cat /etc/cvmfs/default.local


CVMFS_REPOSITORIES="my.repo.name"
CVMFS_SERVER_URL=http://localhost/cvmfs/my.repo.name
CVMFS_QUOTA_LIMIT=10000
CVMFS_HTTP_PROXY="DIRECT"
```

And check the setup:

```
cvmfs_config  setup
cvmfs_config  chksetup
```

If everything goes fine, then we can manually to mount the publish repo on the same local node(so to avoid any config conflicts, since 
both server and client will try to use /cvmfs folder, etc).

```
mount -t cvmfs my.repo.name /test/

df -h
...
overlay_my.repo.name  100G   56G   45G  56% /cvmfs/my.repo.name
cvmfs2                9.8G  273M  9.5G   3% /test
```


