# heketi-glusterfs-playbook


gfs1
  ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
  systemctl daemon-reload
  systemctl start heketi
  systemctl status heketi
  systemctl enable heketi
  
  
  
  
  k8s:
    echo -n "PASSWORD" | base64
    
    
