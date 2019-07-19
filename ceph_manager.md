#### 删除 ceph创建的vg  
vgs |grep ceph |awk '{print $1}' |xargs vgremove -y  
