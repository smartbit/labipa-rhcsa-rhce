
## VMware fusion
- update ipadress to 192.168.4.0/24 subnet


## labipa
- login in GUI with root (via other), set keyboard etc and logout again
- append `/home   *(rw)` to /etc/exports & `systemctl restart nfs-server`
- append `\tguest ok = yes` to /etc/samba/smb.conf & `for i in smb nmb; do systemctl restart $i; done`
`tet`

- execute this:
```bash
# labipa has an error in the FreeIPA configuration
grep preauth /var/log/krb5kdc.log
echo $'password\npassword\npassword' | kinit admin # password expired
ipa config-mod --homedirectory /home/ldap --defaultshell /bin/bash
for i in `ipa user-find | grep "User login: " | sed 's/User login: //g'`; do ipa user-mod $i --homedir=/home/ldap/$i --shell=/bin/bash; done
for i in ldapuser1 ldapuser2 ldapuser3 ldapuser4 ldapuser5 isabelle; do mkdir /home/ldap/$i; chown $i:$i /home/ldap/$i; done
```
## server1
- ``

## server2
- `yum clean all`
