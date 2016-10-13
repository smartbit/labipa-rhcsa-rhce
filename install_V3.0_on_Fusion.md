
## VMware fusion
- update ipadress to 192.168.4.0/24 subnet


## labipa
- append `/home   *(rw)` to /etc/exports & `systemctl restart nfs-server`
- append `\tguest ok = yes` to /etc/samba/smb.conf & `for i in smb nmb; do systemctl restart $i; done`
- login with root (via other), set keyboard etc and logout again
- mkdir /home/ldap
- `for i in ldapuser1 ldapuser2 ldapuser3 ldapuser4 ldapuser5 isabelle; do mkdir /home/ldap/$i; chown $i:$i /home/ldap/$i; done`

## server1
- ``

## server2
- `yum clean all`
