# zf-hadoop.spec

```bash
BuildArch:     noarch
Name:          zf-hadoop
Version:       3.1.1
Release:       1
License:       Zhongfu.net
Group:         Applications/Server
Summary:       ZF Hadoop server

URL:           http://www.zhongfu.net


%description
ZF Hadoop server

%install
mkdir -p ${RPM_BUILD_ROOT}/opt/zfbdp/hadoop
mkdir -p ${RPM_BUILD_ROOT}/etc/profile.d
mkdir -p ${RPM_BUILD_ROOT}/lib/systemd/system

cp -r ${RPM_BUILD_DIR}/hadoop/* ${RPM_BUILD_ROOT}/opt/zfbdp/hadoop/

mv ${RPM_BUILD_ROOT}/opt/zfbdp/hadoop/zf-hadoop.sh ${RPM_BUILD_ROOT}/etc/profile.d/
mv ${RPM_BUILD_ROOT}/opt/zfbdp/hadoop/zf-*.service ${RPM_BUILD_ROOT}/lib/systemd/system/
chmod 644 ${RPM_BUILD_ROOT}/etc/profile.d/zf-hadoop.sh
chmod 644 ${RPM_BUILD_ROOT}/lib/systemd/system/zf-*.service
chmod 755 ${RPM_BUILD_ROOT}/opt/zfbdp/hadoop/setuphadoop.sh
chmod 755 ${RPM_BUILD_ROOT}/opt/zfbdp/hadoop/inithadoop.sh

%files
%dir %attr(0755, root, root) "/opt/zfbdp/hadoop"
%defattr(-, root, root, -)
   /opt/zfbdp/hadoop/*
   /etc/profile.d/zf-hadoop.sh
   /lib/systemd/system/zf-*.service

%pre
/usr/bin/getent group hadoop >/dev/null || /usr/sbin/groupadd -r hadoop
/usr/bin/getent passwd hdfs >/dev/null || /usr/sbin/useradd -r -g hadoop -d /var/lib/hadoop -s /bin/bash -c "hdfs System User" hdfs 2>/dev/null || :
/usr/bin/getent passwd yarn >/dev/null || /usr/sbin/useradd -r -g hadoop -d /var/lib/hadoop -s /bin/bash -c "yarn System User" yarn 2>/dev/null || :
/usr/bin/getent passwd mapred >/dev/null || /usr/sbin/useradd -r -g hadoop -d /var/lib/hadoop -s /bin/bash -c "mapred System User" mapred 2>/dev/null || :

if [ ! -e "/var/lib/hadoop" ]; then
  /usr/bin/install -d -o hdfs -g hadoop -m 0775  /var/lib/hadoop
fi

if [ ! -e "/var/log/hadoop" ]; then
  /usr/bin/install -d -o hdfs -g hadoop -m 0775  /var/log/hadoop
fi

%preun
systemctl stop zf-hadoop

%post
systemctl daemon-reload

%changelog
```

