# How_to_set_oracle18c_to_systemd

this project explains oracle database 18c service set to systemd.you need to set oracle database 18c service to systemd except oracle linux.
# script

set oracle database 18c starting and stopping to systemd.
```shell
# reference from [systemd launch rc-local](https://wiki.archlinux.org/index.php/User:Herodotus/Rc-Local-Systemd)
cat << END >> /etc/systemd/system/oracle-xe-18c.service
[Unit]
Description=Oracle Database Service
After=network.target

[Service]
Type=forking
RemainAfterExit=yes
TimeoutStartSec=10min
TimeoutStopSec=10min
Restart=no
# User=oracle
# Group=dba
User=root
Group=root
ExecStart=/usr/local/bin/oracle_startup
ExecStop=/usr/local/bin/oracle_shutdown

[Install]
WantedBy=multi-user.target

END
```

create script for starting up oracle database 18c.
```shell
cat << END >> /usr/local/bin/oracle_startup
#!/usr/bin/bash
if getent passwd oracle > /dev/null; then

su - oracle << EOF
# mount,open and start oracle instant.
echo "startup;" | /opt/oracle/product/18c/dbhomeXE/bin/sqlplus / as sysdba
/opt/oracle/product/18c/dbhomeXE/bin/lsnrctl start
EOF
fi

END

# set executable permission.
chmod 755 /usr/local/bin/oracle_startup
```

create script for stopping oracle database.
```shell
cat << END >> /usr/local/bin/oracle_shutdown
#!/usr/bin/bash
if getent passwd oracle > /dev/null; then

su - oracle << EOF
# shutdown. tranzaction is rollbacking.
echo "shutdown immediate;" | /opt/oracle/product/18c/dbhomeXE/bin/sqlplus / as sysdba
/opt/oracle/product/18c/dbhomeXE/bin/lsnrctl stop
EOF
fi

END

# set executable permission.
chmod 755 /usr/local/bin/oracle_shutdown
```

reload systemd settings.
```shell
systemctl daemon-reload
```

Fedora only need this script.
```shell
# /usr/lib/systemd/systemd-sysv-install is not installed in fedora. reference from [fedora systemd](https://www.it-swarm-ja.tech/ja/fedora/systemdsysvinstall%E3%81%8C%E3%81%AA%E3%81%84%E3%81%9F%E3%82%81%E3%80%81fedora%E3%81%AE%E8%B5%B7%E5%8B%95%E6%99%82%E3%81%ABgrafana%E3%82%92%E6%9C%89%E5%8A%B9%E3%81%AB%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%9B%E3%82%93/962285807/)
dnf install -y chkconfig
```

set enable oracle-xe-18c to systemd.
```shell
systemctl enable oracle-xe-18c
```
