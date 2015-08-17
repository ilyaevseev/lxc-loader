# Conventions

  * No bridge, ProxyARP on host.
  * No DHCP, static IP in container.
  * Its recommended to use the same suffix for vethXXX, MAC and IP.

# Install LXC and required packages

**Ubuntu 12.04 (Precise Pangolin):**

```
apt-get --no-install-recommends install lxc debootstrap cgroup-lite arping mercurial
service lxc stop
sed -i.orig -e 's,USE_LXC_BRIDGE="true",USE_LXC_BRIDGE="false",' /etc/default/lxc
service lxc start
```

**Ubuntu 14.04 (Trusty Tahr):**

```
apt-get --no-install-recommends install lxc debootstrap cgroup-lite arping mercurial lxc-templates
service lxc stop
sed -i.orig -e 's,USE_LXC_BRIDGE="true",USE_LXC_BRIDGE="false",' /etc/default/lxc-net
service lxc start
```

**Debian 7 (Wheezy):**

```
apt-get --no-install-recommends install lxc debootstrap arping mercurial
mkdir /cgroup
echo "cgroup  /cgroup  cgroup  defaults  0  0" >> /etc/fstab
mount /cgroup
wget -qO - http://freedomboxblog.nl/wp-content/uploads/lxc-debian-wheezy.gz \
| zcat - > /usr/share/lxc/templates/lxc-debian-wheezy
```

See also https://github.com/simonvanderveldt/lxc-debian-wheezy-template

**RHEL6:**

```
yum install --enablerepo=epel lxc lxc-templates libcgroup debootstrap febootstrap arping mercurial
chkconfig cgconfig on
service cgconfig start
```

# Install loader

```
cd /etc &&
hg clone https://bitbucket.org/evseev/lxc-loader
```

# On boot

Put to **/etc/rc.local**:
```
/etc/lxc-loader/lxc-onboot [names of containers...] &
```

# Create container

  * **Ubuntu:** `lxc-create -n $NAME -t ubuntu -- -a i386 -r trusty`
  * **Debian:** `lxc-create -n $NAME -t debian-wheezy` (..select i386 in dialog window)

### Edit /var/lib/lxc/$NAME/config

```
lxc.network.type = veth
lxc.network.flags = up
lxc.network.veth.pair = vethXXX
lxc.network.name = eth0
lxc.network.hwaddr = 00:16:3e:xx:xx:xx
```

### Edit /var/lib/lxc/$NAME/rootfs/etc/network/interfaces

```
iface eth0 inet static
        address x.x.x.x
        gateway x.x.x.1
        netmask 255.255.255.0
        dns-nameservers x.x.x.2 x.x.x.3
```

# Start container

  * `/etc/lxc-loader/lxc-start1 $NAME`
  * `lxc-console -n $NAME`
    * Ubuntu default login = ubuntu, password = ubuntu
    * Debian default login = root, password = root
    * disconnect = Ctrl+a, q
