zfsnostalgia
============

tools to create many clone of a a  zfs'set to an distant host which will be based on different time

# use case

* on host host-src named host-src

* if you give a look to the source you will see

```bash
LIST_OF_SUBZFS_HIST="ctl_rdo data admin"
…
ZFS_ROOT_SOURCE="rpool/oracle/dbatst1"
ZFS_ROOT_HIST="rpool/oracle/ictst-hist"
SNAP_HIST_PATTERN="dolly"
…
HOST_TARGET="host-tgt"
HOST_SOURCE="host-src"

```

..* to send the data from the host-src to host-tgt do

```
zfsnostalgia send
  # starting send
  # -----
  # doing zfs(rpool/oracle/dbatst1/ctl_rdo) 
  # -----
  #   step (1)
  #     check if there is a zfs(rpool/oracle/ictst-hist/ctl_rdo) distant... x
  #      - test if there is a snap to send... v
  #        - do a full from (rpool/oracle/dbatst1/ctl_rdo@dolly_mnt::2014.02.11_19:00:03.879178::host-src::ONLINE) to (rpool/oracle/ictst-hist/ctl_rdo)
  #   step (2)
  #     check if there is a zfs(rpool/oracle/ictst-hist/ctl_rdo) distant... v
  #     copy the next one snap incrementally
  #      - check if there is some commom snap between from and to... v it us the snap(dolly_mnt::2014.02.12_19:00:06.723837::host-src::ONLINE)
  #      - send rpool/oracle/dbatst1/ctl_rdo@{dolly_mnt::2014.02.11_19:00:03.879178::host-src::ONLINE,dolly_mnt::2014.02.12_19:00:06.723837::host-src::ONLINE} | receive rpool/oracle/ictst-hist/ctl_rdo
  #   step (3)
  #     check if there is a zfs(rpool/oracle/ictst-hist/ctl_rdo) distant... v
  #     copy the next one snap incrementally
  #      - check if there is some commom snap between from and to... x
  #        - no other snapshot to send/receive
  # -----
  # doing zfs(rpool/oracle/dbatst1/data) 
  # -----
  #   step (1)
  #     check if there is a zfs(rpool/oracle/ictst-hist/data) distant... x
  #      - test if there is a snap to send... v
  #        - do a full from (rpool/oracle/dbatst1/data@dolly_mnt::2014.02.11_19:00:03.879178::host-src::ONLINE) to (rpool/oracle/ictst-hist/data)
  #   step (2)
  #     check if there is a zfs(rpool/oracle/ictst-hist/data) distant... v
  #     copy the next one snap incrementally
  #      - check if there is some commom snap between from and to... v it us the snap(dolly_mnt::2014.02.12_19:00:06.723837::host-src::ONLINE)
  #      - send rpool/oracle/dbatst1/data@{dolly_mnt::2014.02.11_19:00:03.879178::host-src::ONLINE,dolly_mnt::2014.02.12_19:00:06.723837::host-src::ONLINE} | receive rpool/oracle/ictst-hist/data
  #   step (3)
  #     check if there is a zfs(rpool/oracle/ictst-hist/data) distant... v
  #     copy the next one snap incrementally
  #      - check if there is some commom snap between from and to... x
  #        - no other snapshot to send/receive
  # -----
  # doing zfs(rpool/oracle/dbatst1/admin) 
  # -----
  #   step (1)
  #     check if there is a zfs(rpool/oracle/ictst-hist/admin) distant... x
  #      - test if there is a snap to send... v
  #        - do a full from (rpool/oracle/dbatst1/admin@dolly_mnt::2014.02.11_19:00:03.879178::host-src::ONLINE) to (rpool/oracle/ictst-hist/admin)
  #   step (2)
  #     check if there is a zfs(rpool/oracle/ictst-hist/admin) distant... v
  #     copy the next one snap incrementally
  #      - check if there is some commom snap between from and to... v it us the snap(dolly_mnt::2014.02.12_19:00:06.723837::host-src::ONLINE)
  #      - send rpool/oracle/dbatst1/admin@{dolly_mnt::2014.02.11_19:00:03.879178::host-src::ONLINE,dolly_mnt::2014.02.12_19:00:06.723837::host-src::ONLINE} | receive rpool/oracle/ictst-hist/admin
  #   step (3)
  #     check if there is a zfs(rpool/oracle/ictst-hist/admin) distant... v
  #     copy the next one snap incrementally
  #      - check if there is some commom snap between from and to... x
  #        - no other snapshot to send/receive
```

as you can see, if we follow for eg zfs(rpool/oracle/dbatst1/data), we do 3 steps
..1 first we see that there is no such target zfs, so let's send the oldest snapshot we have
..1 we see that we have a zfs on the target, but that this is not the latest one. So we'll send him the next one
..1 all the snap were send, so we skip this zfs and go to the next one (rpool/oracle/dbatst1/data)

* on host-tgt

 * check that we receive

zfsnostalgia 
  # available snap:
  #   dolly_mnt::2014.02.11_19:00:03.879178::mdba3::ONLINE
  #   dolly_mnt::2014.02.12_19:00:06.723837::mdba3::ONLINE
  # clone mounted :

 * we see that 
zfs list gives 3 new zfs …/admin  …/ctl_rdo …/data
  # …
  # rpool/oracle/ictst-hist/admin      969M  66.0G   949M  /oracle/ictst-hist/admin
  # rpool/oracle/ictst-hist/ctl_rdo    416M  66.0G   311M  /oracle/ictst-hist/ctl_rdo
  # rpool/oracle/ictst-hist/data      9.78G  66.0G  9.56G  /oracle/ictst-hist/data
  # …

we can see that the zfs were created on host-tgt, and also that 2 snapshots were sent.

* now we can mount the a clone 
