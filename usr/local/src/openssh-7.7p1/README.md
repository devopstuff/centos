# OpenSSH

## Problems

> securing and hardening with recommended configuration need a recent openssh version 6.7+  
> rhel 6 and centos 6 run older openssh version 5.3+

* https://wiki.mozilla.org/Security/Guidelines/OpenSSH
* https://stribika.github.io/2015/01/04/secure-secure-shell.html


## Workaround

need to compile from the source

* http://mirrors.ircam.fr/pub/OpenBSD/OpenSSH/portable/


## Compile and install from source

```bash
pushd /usr/local/src
openssh_version=7.7p1
curl -O http://mirrors.ircam.fr/pub/OpenBSD/OpenSSH/portable/openssh-$openssh_version.tar.gz
tar xfz openssh-$openssh_version.tar.gz openssh-$openssh_version/contrib/redhat/openssh.spec
cd openssh-$openssh_version
mkdir -p {SOURCES,SPECS}
cp contrib/redhat/openssh.spec SPECS/
cp ../openssh-$openssh_version.tar.gz SOURCES/
sed -i -e "s/%define no_gnome_askpass 0/%define no_gnome_askpass 1/g" SPECS/openssh.spec
sed -i -e "s/%define no_x11_askpass 0/%define no_x11_askpass 1/g" SPECS/openssh.spec
sed -i -e "s/BuildPreReq/BuildRequires/g" SPECS/openssh.spec
sed -i -e "/%attr.*slogin/d" SPECS/openssh.spec
yum -y install rpm-build gcc make wget openssl-devel krb5-devel pam-devel libX11-devel xmkmf libXt-devel
rpmbuild --define "_topdir $(pwd)" -v -bb --clean  SPECS/openssh.spec
yum -y erase openssh\*
rpm -Uvh --force RPMS/$(uname -m)/openssh-$openssh_version-?.$(uname -m).rpm
rpm -Uvh --force RPMS/$(uname -m)/openssh-clients-$openssh_version-?.$(uname -m).rpm
rpm -Uvh --force RPMS/$(uname -m)/openssh-server-$openssh_version-?.$(uname -m).rpm
sed -i -e "/restorecon.*ssh_host_key.pub/d" /etc/init.d/sshd
unset openssh_version
popd
#nohup /etc/init.d/sshd restart &>/dev/null
```

### Before

```bash
$ yum list installed openssh\* | grep openssh
openssh.x86_64                        5.3p1-118.1.el6_8                 @updates
openssh-clients.x86_64                5.3p1-118.1.el6_8                 @updates
openssh-server.x86_64                 5.3p1-118.1.el6_8                 @updates

$ rpm -qa openssh\*
openssh-5.3p1-118.1.el6_8.x86_64
openssh-server-5.3p1-118.1.el6_8.x86_64
openssh-clients-5.3p1-118.1.el6_8.x86_64

$ ssh -V
OpenSSH_5.3p1, OpenSSL 1.0.1e-fips 11 Feb 2013

$ ssh -Q cipher-auth
ssh: illegal option -- Q
```

### After

```bash
$ yum list installed openssh\* | grep openssh
openssh.x86_64                           7.7p1-1.el6                   installed
openssh-clients.x86_64                   7.7p1-1.el6                   installed
openssh-server.x86_64                    7.7p1-1.el6                   installed

$ rpm -qa openssh\*
openssh-7.7p1-1.el6.x86_64
openssh-server-7.7p1-1.el6.x86_64
openssh-clients-7.7p1-1.el6.x86_64

$ ssh -V
OpenSSH_7.7p1, OpenSSL 1.0.1e-fips 11 Feb 2013

$ ssh -Q cipher-auth
aes128-gcm@openssh.com
aes256-gcm@openssh.com
chacha20-poly1305@openssh.com
```

## Credits

* @devopstuff

## License

* nothing to report
