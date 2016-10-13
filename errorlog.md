# Unauthorized Errorlog [Red Hat RHCSA ®/ RHCE® 7 Cert Guide](http://www.sandervanvugt.com/book-red-hat-rhcsa-rhce-7-cert-guide/)
## Chap 6
- p137 step 6. `for i in lucy, lori, bob;do useradd $i; done` --> `for i in lucy lori bob;do useradd $i; done`
- p139 wrong **groupmems -g sales -l** "This shows users who are a member of this group as a secondary group assignment~~, but also users who are a member of this group as the primary group assignment~~."
<br />**lid -g <groupname>** shows both primary and secondary group memberships eg try:

  ```bash
  for i in lucy lori bob; do useradd $i; done
  usermod -g lucy bob     #make group lucy the primary group of bob
  usermod -aG lucy lori   #add lori to the group lucy
  groupmems -g lucy -l
  lid -g lucy
  lid lori
  ```
- p142 `nslcd service is configured and started when using autconfig-tui` —> `sssd service is configured and started when using authconfig-tui` (with an h). Maybe a change from rhel6? Try the exercise and then run `find / -name nslc` or `systemctl list-unit-files | grep -e sssd -e nslcd` and you'll find nothing related to nslcd.
<br /> see `man authconfig | grep -A2 nslcd` and notice **Used to**
- p145 Step 5 first create the directory `mkdir /etc/openldap/cacerts` then get the certificate with `scp labipa.example.com:/root/cacert.p12 /etc/openldap/cacerts` otherwise scp will create a file cacerts not a file /etc/openldap/cacerts/ca.crt.

- p142 **authconfig-tui** command is _deprecated_, see `man authconfig-tui | grep depr`. [CertDepot](https://www.certdepot.net/ldap-client-configuration-authconfig/) has two alternative exercises:
  #### the nslcd option
  see https://arthurdejong.org/nss-pam-ldapd/setup
  ```bash
  yum install -y openldap-clients nss-pam-ldapd
  mkdir /etc/openldap/cacerts
  scp labipa.example.com:/etc/ipa/ca.crt /etc/openldap/cacerts/
  authconfig --enableldap --enableldapauth \
             --ldapserver="labipa.example.com" \
             --ldapbasedn="dc=example,dc=com" \
             --enableldaptls --update
  systemctl list-unit-files | grep -e sssd -e nslcd
  nslcd -V
  grep -v -e "#" -e "^$" /etc/nslcd.conf
  authconfig --test | grep SSSD # SSSD ... *disabled*
  grep -e pam_sss -e -pam_ldap /etc/pam.d/system-auth  
  getent passwd ldapuser1
  su - ldapuser1
  whoami
  exit
  authconfig --enablemkhomedir --update
  su - ldapuser1
  pwd
  exit
  ```

  #### the System Security Services Daemon option
  With CentOS 7.0 (images V2.0) you might try sssd on Server2 instead (/usr/lib/systemd/systemd-logind crashed on Server1 with CentOS 7.0)
  ```bash
  yum install -y sssd # on Server2
  mkdir /etc/openldap/cacerts
  scp labipa.example.com:/etc/ipa/ca.crt /etc/openldap/cacerts/
  authconfig --enableldap --enableldapauth \
             --ldapserver="labipa.example.com" \
             --ldapbasedn="dc=example,dc=com" \
             --enableldaptls --enablemkhomedir --update
   systemctl list-unit-files | grep -e sssd -e nslcd
   grep -v -e "#" -e "^$" -e "^\[" /etc/sssd/sssd.conf
   authconfig --test | grep SSSD # SSSD ... *enabled*
   grep -e pam_sss -e -pam_ldap /etc/pam.d/system-auth
   getent passwd ldapuser1
   su - ldapuser1
   whoami
   pwd
   exit
  ```
  #### compare nslcd and sssd
  ```txt
  NSLCD / yum install -y openldap-clients nss-pam-ldapd                     SSSD / yum install -y sssd
  #%PAM-1.0                                                                 #%PAM-1.0
  # This file is auto-generated.                                            # This file is auto-generated.
  # User changes will be destroyed the next time authconfig is run.         # User changes will be destroyed the next time authconfig is run.
  auth        required      pam_env.so                                      auth        required      pam_env.so
  auth        sufficient    pam_fprintd.so                                <
  auth        sufficient    pam_unix.so nullok try_first_pass               auth        sufficient    pam_unix.so nullok try_first_pass
  auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success     auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
  auth        sufficient    pam_ldap.so use_first_pass                    | auth        sufficient    pam_sss.so use_first_pass
  auth        required      pam_deny.so                                     auth        required      pam_deny.so

  account     required      pam_unix.so broken_shadow                       account     required      pam_unix.so broken_shadow
  account     sufficient    pam_localuser.so                                account     sufficient    pam_localuser.so
  account     sufficient    pam_succeed_if.so uid < 1000 quiet              account     sufficient    pam_succeed_if.so uid < 1000 quiet
  account     [default=bad success=ok user_unknown=ignore] pam_ldap.so    | account     [default=bad success=ok user_unknown=ignore] pam_sss.so
  account     required      pam_permit.so                                   account     required      pam_permit.so

  password    requisite     pam_pwquality.so try_first_pass local_users_o   password    requisite     pam_pwquality.so try_first_pass local_users_o
  password    sufficient    pam_unix.so sha512 shadow nullok try_first_pa   password    sufficient    pam_unix.so sha512 shadow nullok try_first_pa
  password    sufficient    pam_ldap.so use_authtok                       | password    sufficient    pam_sss.so use_authtok
  password    required      pam_deny.so                                     password    required      pam_deny.so

  session     optional      pam_keyinit.so revoke                           session     optional      pam_keyinit.so revoke
  session     required      pam_limits.so                                   session     required      pam_limits.so
  -session     optional      pam_systemd.so                                 -session     optional      pam_systemd.so
  session     optional      pam_oddjob_mkhomedir.so umask=0077            | session     optional      pam_mkhomedir.so umask=0077
  session     [success=1 default=ignore] pam_succeed_if.so service in cro   session     [success=1 default=ignore] pam_succeed_if.so service in cro
  session     required      pam_unix.so                                     session     required      pam_unix.so
  session     optional      pam_ldap.so                                   | session     optional      pam_sss.so  
  ```
  grep -v -e "#" -e "^$"  /etc/nsswitch.conf
  ```text
  nslcd:                                         sssd:
  passwd:     files sss ldap                  |  passwd:     files sss
  shadow:     files sss ldap                  |  shadow:     files sss
  group:      files sss ldap                  |  group:      files sss
  hosts:      files dns                          hosts:      files dns
  bootparams: nisplus [NOTFOUND=return] files    bootparams: nisplus [NOTFOUND=return] files
  ethers:     files                              ethers:     files
  netmasks:   files                              netmasks:   files
  networks:   files                              networks:   files
  protocols:  files                              protocols:  files
  rpc:        files                              rpc:        files
  services:   files sss                          services:   files sss
  netgroup:   files sss ldap                  |  netgroup:   files sss
  publickey:  nisplus                            publickey:  nisplus
  automount:  files ldap                      |  automount:  files sss
  aliases:    files nisplus                      aliases:    files nisplus
  ```

## Chap 7 : lapipa
1
2
3
scp root@server1.example.com:/etc/yum.repos.d/labipa.repo /etc/yum.repos.d/

### Chap 8 Networking
In https://www.safaribooksonline.com/library/view/learning-path-red/9780134664040/RCSA_01_09_08.html favorites mentions netstat —> ip -s link & ss -tulpen, see https://dougvitale.wordpress.com/2011/12/21/deprecated-linux-networking-commands-and-their-replacements/
Appendix A “10. tar xvf /tmp/etchome.tgz /etc/passwd” should be “tar xvf /tmp/etchome.tgz etc/passwd"
Appendix B: Table 2.4 remove
“at (i) or after (a) the current cursor position.”
 “The ! forces …are doing”
“, which allows you … the selection”
“Use d to cut, or y to copy the selection.”
Appendix B: Why is "Table 3.6 Overview of tar Options” missing?
Appendix B & Appendix C: Table 2.4 add
in visual mode use to cut
in visual mode use to copy
Appendix C: Table 2.4 add
^ or 0

### Chap 18 Managing and Understanding the Boot Procedure
- p413 _Understanding Wants_: **default** .wants are in **/usr/lib**/systemd/system/*.wants, system-specific (created with `systemctl enable`) wants are in **/etc**/systemd/system/*.wants **
- p147 from [Understanding systemd units and unit files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files#unit-section-directives)
  - `[Unit]`
    - **Wants=**: This directive is similar to Requires=, but less strict. Systemd _will attempt_ to start any units listed here when this unit is activated. If these units are not found or fail to start, the current unit will **continue** to function. This is the recommended way to configure most dependency relationships. Again, this implies a parallel activation unless modified by other directives.
    - **Requires=**: This directive lists any units upon which this unit _essentially_ depends. If the current unit is activated, the units listed here _must_ successfully activate as well, else this unit will **fail**. These units are started _in parallel_ with the current unit by default.
  - `[Install]`
    - **WantedBy=:**
    <br />The WantedBy= directive is the **most common way** to specify how a unit should be enabled. This directive allows you to specify a dependency relationship in a similar way to the Wants= directive does in the [Unit] section. The difference is that this directive is included in the ancillary unit allowing the primary unit listed **to remain relatively clean**. When a unit with this directive is enabled, a _directory_ will be created within /etc/systemd/system named after the specified unit with `.wants` appended to the end. Within this, a symbolic link to the current unit will be created, creating the dependency. For instance, if the current unit has WantedBy=multi-user.target, a directory called multi-user.target.wants will be created within /etc/systemd/system (if not already available) and a symbolic link to the current unit will be placed within. Disabling this unit removes the link and removes the dependency relationship.
    - **RequiredBy=**: This directive is very similar to the WantedBy= directive, but instead specifies a required dependency that will cause the activation to **fail** if not met. When enabled, a unit with this directive will create a directory ending with **.requires**.
- from _man 7 systemd.unit_:
  - Along with a unit file foo.service, the directory foo.service**.wants/** may exist. All unit files symlinked from such a directory are **implicitly** added as dependencies of _type Wants=_ to the unit. This is useful to hook units into the start-up of other units, **without having to modify their unit files.** For details about the semantics of Wants=, see below. The preferred way to create symlinks in the .wants/ directory of a unit file is with the enable command of the systemctl(1) tool which reads information from the [Install] section of unit files (see below). A similar functionality exists for Requires= type dependencies as well, the directory suffix is .requires/ in this case.
  - **Wants=**
  <br />A weaker version of _Requires=_. Units listed in this option will be started if the configuring unit is. However, if the listed units fail to start or cannot be added to the transaction this has no impact on the validity of the transaction as a whole. This is the
  recommended way to hook start-up of one unit to the start-up of another unit.
  <br />Note that dependencies of this type _may also be configured_ outside of the unit configuration file **by adding symlinks** to a .wants/ directory accompanying the unit file.
- p417 `cat /usr/lib/systemd/system/iptables.service` file doesn't exist on Server2 (non-GUI)
- p418 on Server**2** `systemctl start iptables` generates message `Failed to issue method call: Unit iptables.service failed to load: No such file or directory.` as iptables is not installed on non-GUI. Therefor `systemctl enable iptables`also failes on Server2. <br /> Notice that `systemctl mask iptables` **does** generate the link to /dev/null although the services is not installed on Server2.
- p149 from _man systeemctl_:
  <br />`systemctl isolate`
  <br />Start the unit specified on the command line and its dependencies and **stop all others**.
  <br />This is similar to changing the runlevel in a traditional init system. The isolate command will **immediately stop** processes that are not enabled in the new unit, possibly including the graphical environment or terminal you are currently using.
  <br /> Note that this is allowed **only** on units where **AllowIsolate=** is enabled.

  ## Chapter 16: Basic Kernel Management
- p383 demonstrate how to create options in an /etc/modprobe.d/\*.conf file with options that will persist after reboot.  
Notice that these commands disconnect the network, run `modprobe -r` from the console, put the commands in a bash-script (and hope tcp will not notice the disconnection) or reboot.  
In a second tty (eg ctrl-leftalt-F2 or `chvt 2`) run `udevadm monitor` and see the udev events.
    ```bash
  modinfo e1000 | grep -e parm: -e description: # what params do we have?
  ethtool eth0 | grep -A2 "Advertised link modes" # what is the advertised Speed?
  echo "options e1000 Speed=100" > /etc/modprobe.d/e1000.conf
  modprobe -r e1000
  modprobe e1000
  ethtool eth0 | grep -A2 "Advertised link modes"
  rm -f /etc/modprobe.d/e1000.conf
  modprobe -r e1000
  modprobe e1000
  ethtool eth0 | grep -A2 "Advertised link modes"
    ```
    See `man 5 modprobe.d` for details on the .conf file.

### Chapter 25: Configuring External Authentication and Authorization
- p563 Centos 7.2 `man authconfig` states "The **authconfig-tui** is _deprecated_. No new configuration settings will be supported by its text user interface. Use **system-config-authentication GUI** application or the command line options instead." dated 22 July 2011
- p563 for the exercises to work, create a few accounts first

```bash
dig labipa.example.com
yum groups install Directory\ Client -y
yum install -y pam_krb5 krb5-workstation
yum -y install nss-pam-ldapd # sander doesn't mention this package
mkdir -p /etc/openldap/cacerts
scp labipa.example.com:/etc/ipa/ca.crt /etc/openldap/cacerts
authconfig-tui
 ┌────────────────┤ Authentication Configuration ├─────────────────┐
│                                                                 │
│  User Information        Authentication                         │
│  [*] Cache Information   [*] Use MD5 Passwords                  │
│  [*] Use LDAP            [*] Use Shadow Passwords               │
│  [ ] Use NIS             [*] Use LDAP Authentication            │
│  [ ] Use IPAv2           [ ] Use Kerberos                       │
│  [ ] Use Winbind         [ ] Use Fingerprint reader             │
│                          [ ] Use Winbind Authentication         │
│                          [*] Local authorization is sufficient  │
│                                                                 │
│            ┌────────┐                      ┌──────┐             │
│            │ Cancel │                      │ Next │             │
│            └────────┘                      └──────┘             │
│                                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘



systemctl list-unit-files | grep -e sssd -e nslcd
nslcd -V
grep -v -e "#" -e "^$" /etc/nslcd.conf
authconfig --test | grep SSSD # SSSD ... *enabled*
grep -e pam_sss -e -pam_ldap /etc/pam.d/system-auth
grep -v -e "#" -e "^$" -e "\[" /etc/sssd/sssd.conf
systemctl restart sssd
systemctl status -l sssd
kinit admin
su - ldapuser1
authconfig-tui
yum search pam_ldap
yum search nss-pam-ldapd
yum -y install nss-pam-ldapd
authconfig-tui
kinit admin
kinit ldapuser1


  ```

### Chapter 34: Configuring DNS
- p750 Exercise 34.1
  ```bash
  yum clean all # server2 has issues with yum
  yum -y install unbound vim bind-utils # bindutils for dig
  yum list installed | grep unbound
  ss -tulpen | grep :53             # is unbound bound to ip-adresses?
  systemctl start unbound; systemctl enable unbound
  vim /etc/unbound/unbound.conf     # see p750
  unbound-checkconf
  systemctl restart unbound
  firewall-cmd --permanent --add-service=dns; firewall-cmd --reload
  firewall-cmd --list-all
  ss -tulpen | grep :53             # is unbound bound to ip-adresses?
  dig DNSKEY example.com. @192.168.4.220  # no ad-bit
  dig DNSKEY rhatcert.com. @192.168.4.220 | grep -E " ad|$"   
  ```
- p751 The output of `dig +dnssec DNSKEY rhatcert.com` is fake,
  - _either_ the domain is signed and returns the `ad` "Authenticated Data" flag,
  - _or_ the domain is not signed, doesn't return the ad-bit and can't return DNSKEYs.  
Try it yourself and verify that the rhatcert.com domain now is signed and the AD-flag is shown.
- p752 TIP part2: using `unbound-control-setup` is far from travial. Unless practised, don't try on the exam.
- p752 you get the trust anchors  
- p752
