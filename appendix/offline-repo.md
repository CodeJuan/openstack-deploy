Offline Repo
=================

在实际应用中，经常会使用rpm包安装系统。由于网络的原因，经常会导致下载时间特别长。因此，制作离线源变得非常迫切。在本文中，将会介绍如何制作用于CentOS 6.5的离线源。

此处制作的离线源主要用于安装OpenStack Icehouse版本使用，其他版本仅供参考。
# 重要提示

建议至(https://github.com/JiYou/openstack-deploy/appendix/offline-repo)中寻找本文相关脚本。

#方案1
##基本思路
在利用yum安装各种包时，会在本地形成缓存，安装完成之后，利用这些缓存文件，便可成功地制作离线源。

##Step 1 保留缓存 
方案1所采用的方法较为简单，只需要将yum.conf中的keepcache设置为1：

	keepcache=1

此项一定需要修改，否则默认yum会将安装之后的缓存目录清空。完成之后，添加RDO源：

	yum install -y https://rdo.fedorapeople.org/rdo-release.rpm

##Step 2 安装包
首先建立安装包索引：

	yum makecache

安装各种相关包，如果要安装OpenStack，那么需要安装如下包：

	yum install -y memcached mod_wsgi mongodb \
		mongodb-server mysql MySQL-python \
		mariadb-galera-server \
		openstack-ceilometer-alarm \
		openstack-ceilometer-api \
		openstack-ceilometer-central \
		openstack-ceilometer-collector \
		openstack-ceilometer-compute \
		openstack-ceilometer-notification \
		openstack-cinder openstack-dashboard \
		openstack-glance openstack-heat-api \
		openstack-heat-api-cfn \
		openstack-heat-engine openstack-keystone \
		openstack-neutron openstack-neutron-ml2 \
		openstack-neutron-openvswitch \
		opentack-nova openstack-nova-api \
		openstack-nova-cert openstack-nova-compute \
		openstack-nova-conductor openstack-nova-console \
		openstack-nova-network openstack-nova-novncproxy \
		openstack-nova-scheduler openstack-selinux \
		openstack-swift-account openstack-swift-container \
		openstack-swift-object openstack-swift-proxy \
		openstack-trove openstack-utils \
		python-ceilometerclient python-glanceclient \
		python-keystoneclient python-memcached \
		python-neutronclient python-novaclient \
		python-pecan python-pip python-swiftclient \
		python-troveclient qpid-cpp-server \
		scsi-target-utils xfsprogs xinetd rabbitmq-server

**注意**安装时，请务必确保所有的这些包都能正常安装成功。如果缺少包，请至网络上查找相应的rpm，并设置相应的源。

##Step 3 复制缓存文件
安装完成之后：

	yum install -y createrepo
	cd /var/cache/yum
	mkdir -p /tmp/openstack-repo
	find . -name "*.rpm" | xargs -i cp -rf {} /tmp/openstack-repo
	cd /tmp/openstack-repo
	createrepo .


如果看到如下输出：

	Saving Primary metadata
	Saving file lists metadata
	Saving other metadata
	Generating sqlite DBs
	Sqlite DBs complete

则说明离线源已经制作成功。

## Step 4使用离线源
将做好离线源的/tmp/openstack-repo保存好，使用时，将此目录存放至目标节点的/opt/目录下，在/etc/yum.repos.d目录下添加文件openstack-repo.repo:

	[openstack-repo]
	name=openstack-repo
	baseurl=file:///opt/openstack-repo/
	gpgcheck=0
	enabled=1
	proxy=_none_

**注意**openstack-repo.repo文件开始并没有空格或者Tab。

接下来，就可以使用做好的离线源了。

	yum makecache
	yum install openstack-nova-compute

## 优缺点
这种方法优点如下：

	- 只需要下载需要的包及其依赖包。
	- 制作的离线源通常比较小。

但也存在着如下缺点：

	- 由于网络原因，yum下载时间，有时会特别长。
	- 当缺少某个包时，依然比较麻烦。
	- 添加包时，需要重新制作离线源。

## 补充

如果在使用方案一时，出现某些包无法下载时，请添加online-icehouse.repo至/etc/yum.repos.d目录：

	[theforeman-plugin-source]
	name=theforeman-plugins
	baseurl=http://yum.theforeman.org/plugins/1.5/el6/source
	gpgcheck=0
	enabled=1

	[theforman-plugins-x86_64]
	name=theforman-plugins-x86_64
	baseurl=http://yum.theforeman.org/plugins/1.5/el6/x86_64
	gpgcheck=0
	enabled=1

	[theforeman-release-source]
	name=theforeman-release-source
	baseurl=http://yum.theforeman.org/releases/1.5/el6/source
	gpgcheck=0
	enabled=1

	[theforeman-release-x86_64]
	name=theforeman-release-x86_64
	baseurl=http://yum.theforeman.org/releases/1.5/el6/x86_64
	gpgcheck=0
	enabled=1

	[puppetlabs-products-x86_64]
	name=puppetlabs-products-x86_64
	baseurl=http://yum.puppetlabs.com/el/6/products/x86_64
	gpgcheck=0
	enabled=1

	[puppetlabs-dependencies]
	name=puppetlabs-dependencies
	baseurl=http://yum.puppetlabs.com/el/6/dependencies/x86_64
	gpgcheck=0
	enabled=1

	[puppetlabs-devel]
	name=puppetlabs-devel
	baseurl=http://yum.puppetlabs.com/el/6/devel/x86_64
	gpgcheck=0
	enabled=1

	[puppetlabs-products-srpms]
	name=puppetlabs-products-srpms
	baseurl=http://yum.puppetlabs.com/el/6/products/SRPMS
	gpgcheck=0
	enabled=1

	[puppetlabs-dependencies-srpms]
	name=puppetlabs-dependencies-srpms
	baseurl=http://yum.puppetlabs.com/el/6/dependencies/SRPMS
	gpgcheck=0
	enabled=1

	[puppetlabs-devel-srpms]
	name=puppetlabs-devel-srpms
	baseurl=http://yum.puppetlabs.com/el/6/devel/SRPMS
	gpgcheck=0
	enabled=1

	[openstack-icehouse]
	name=openstack-icehouse
	baseurl=http://repos.fedorapeople.org/repos/openstack/openstack-icehouse/epel-6/
	gpgcheck=0
	enabled=1

	[ceph]
	name=ceph
	baseurl=http://ceph.com/rpm/el6/x86_64
	gpgcheck=0
	enabled=1

	[ceph-extras]
	name=ceph-extras
	baseurl=http://ceph.com/packages/ceph-extras/rpm/centos6/x86_64
	gpgcheck=0
	enabled=1

	[epel]
	name=epel
	baseurl=http://mirror.steadfast.net/epel/6/x86_64/
	gpgcheck=0
	enabled=1

	[rpmfind]
	name=rpmfind
	baseurl=http://rpmfind.net/linux/centos/6.5/os/x86_64/
	gpgcheck=0
	enabled=1

	[nac-net]
	name=nac-net
	baseurl=http://centos.mirror.nac.net/6.5/os/x86_64/
	gpgcheck=0
	enabled=1

	[cs.vt]
	name=cs-vt
	baseurl=http://mirror.cs.vt.edu/pub/CentOS/6.5/updates/x86_64/
	gpgcheck=0
	enabled=1

	[download-fedoraproject]
	name=download-fedoraporject
	baseurl=http://download.fedoraproject.org/pub/epel/6/x86_64
	gpgcheck=0
	enabled=1

	[maridaDB]
	name=maridaDB
	baseurl=http://yum.mariadb.org/5.5.36/centos6-amd64/
	gpgcheck=0
	enabled=1

然后，运行命令如下：

	yum makecache

# 方案2
##思路
与方案1相比，方案2显得更加简单粗暴，直接将所有可能用到的源全部下载至本地，然后存放至http服务器。供所有的节点使用。

下载方法，首先根据链接，下载所有rpm包的列表：

	wget http://repos.fedorapeople.org/repos/openstack/openstack-icehouse/epel-6/epel/ -O index.html

然后，根据index.html，解析出所有的rpm包：

	rpm_list=`cat index.html  | \
        grep rpm | \
        awk -F "href" '{print $2}' | \
        awk -F ">" '{print $1}' | \
        awk -F "=" '{print $2}' | \
        sed "s/\"//" | sed "s/\"//"`
	echo $rpm_list

再接着，下载这些rpm包：

	for n in $rpm_list; do
		wget -c $n
	done

下载包完成之后，进入到rpm包所在目录，运行：

	createrepo .

便可创建离线源。下面是具体步骤。

## Step 1下载rpm包

为了保证本文脚本能正确运行，请务必至/media目录下运行。空间消耗大概是30G，请务必保证有足够大的空间以供使用。

下载包脚本download.sh代码如下：

	#!/bin/bash
	function _get_rpm() {
		cat index.html  | \
        	grep rpm | \
        	awk -F "href" '{print $2}' | \
        	awk -F ">" '{print $1}' | \
        	awk -F "=" '{print $2}' | \
        	sed "s/\"//" | sed "s/\"//" | \
        	grep -v nexuiz-data | \
			grep -v ceph-debuginfo # 这个两种包太大，用不着，跳过
	}

	urls="baseurl=http://yum.theforeman.org/plugins/1.5/el6/source
	baseurl=http://yum.theforeman.org/plugins/1.5/el6/x86_64
	baseurl=http://yum.theforeman.org/releases/1.5/el6/source
	baseurl=http://yum.theforeman.org/releases/1.5/el6/x86_64
	baseurl=http://yum.puppetlabs.com/el/6/products/x86_64
	baseurl=http://yum.puppetlabs.com/el/6/dependencies/x86_64
	baseurl=http://yum.puppetlabs.com/el/6/devel/x86_64
	baseurl=http://yum.puppetlabs.com/el/6/products/SRPMS
	baseurl=http://yum.puppetlabs.com/el/6/dependencies/SRPMS
	baseurl=http://yum.puppetlabs.com/el/6/devel/SRPMS
	baseurl=http://repos.fedorapeople.org/repos/openstack/openstack-icehouse/epel-6/
	baseurl=http://ceph.com/rpm/el6/x86_64/
	baseurl=http://ceph.com/packages/ceph-extras/rpm/centos6/x86_64/
	baseurl=http://download.fedoraproject.org/pub/epel/6/x86_64
	baseurl=http://mirror.steadfast.net/epel/6/x86_64/
	baseurl=https://repos.fedorapeople.org/repos/openstack/openstack-icehouse/epel-6/epel/
	baseurl=http://rpmfind.net/linux/centos/6.5/os/x86_64/Packages/
	baseurl=http://centos.mirror.nac.net/6.5/os/x86_64/Packages/
	baseurl=http://mirror.cs.vt.edu/pub/CentOS/6.5/updates/x86_64/Packages/
	baseurl=http://yum.mariadb.org/5.5.36/centos6-amd64/rpms
	"

	for n in $urls; do
	    url=`echo $n | awk -F "=" '{print $2}'`
	    todir=`echo $url|awk -F"//" '{print $2}'`

	    mkdir -p $todir
    	old_dir=`pwd`
	    cd $todir
	    wget $url -O index.html
	    for prpm in `_get_rpm `; do
        	if [[ ! -e $prpm ]]; then
	            nohup wget $url/$prpm >/dev/null 2>&1 &
        	fi
    	done
    	rm -rf index.html
    	cd $old_dir
	done

	while [[ `ps aux| grep wget | wc -l` -gt 1 ]]; do
	    sleep 1
	    ps aux| grep wget | wc -l
	done


运行脚本：

	./download

将会开始下载所有的rpm包。下载成功之后，会在当前目录形成如下结构：

 	$ ls -l
	drwxr-xr-x 3 root root  4096 Oct 24 04:02 cd1
	drwxrwxr-x 3 1003 1003  4096 Oct 23 22:50 centos.mirror.nac.net
	drwxrwxr-x 4 1003 1003  4096 Oct 23 05:26 ceph.com
	rwxrwxr-x 1 1003 1003  1900 Oct 24 03:51 download
	drwxrwxr-x 3 1003 1003  4096 Oct 23 12:01 download.fedoraproject.org
	drwxrwxr-x 3 1003 1003  4096 Oct 23 23:06 mirror.cs.vt.edu
	drwxrwxr-x 3 1003 1003  4096 Oct 23 05:52 mirror.steadfast.net
	drwxrwxr-x 3 1003 1003  4096 Oct 23 05:16 repos.fedorapeople.org
	drwxrwxr-x 3 1003 1003  4096 Oct 23 12:19 rpmfind.net
	drwxrwxr-x 3 1003 1003  4096 Oct 23 05:16 yum.puppetlabs.com
	drwxrwxr-x 4 1003 1003  4096 Oct 23 05:16 yum.theforeman.org

**注意**在下载时，会不断地输出数字，比如345，324等等，这表明还有这么多个包还未下载成功，还处在于下载状态。请不要退出程序。

## Step 2 构建离线源

编写build脚本如下：


	#!/bin/bash

	old_dir=`pwd`

	for n in `cat download | grep baseurl= | awk -F "//" '{print $2}'`; do
    	cd $old_dir
    	cd $n
    	nohup createrepo .
    	cd $old_dir
	done
	echo "build success"


然后，执行build命令：

	./build >build-log 2>&1 &

### 出错处理

在build过程中，可以通过:

	tail -f build-log

查看进展，如果看build success，则表明构建成功。但是容易出现如下错误：

	Worker 0: error: rpmts_HdrFromFdno: headerRead failed: hdr blob(12128): BAD, read returned 2683
	Worker 0: Error: Could not open local rpm file: /media/mirror.cs.vt.edu/pub/CentOS/6.5/updates/x86_64/Packages/./mod_nss-1.0.8-19.el6_5.x86_64.rpm: RPM Error opening Package
	Worker 0: error: rpmts_HdrFromFdno: headerRead failed: hdr blob(40712): BAD, read returned 2682
	Worker 0: Error: Could not open local rpm file: /media/mirror.cs.vt.edu/pub/CentOS/6.5/updates/x86_64/Packages/./nss-devel-3.16.1-4.el6_5.x86_64.rpm: RPM Error opening Package

在这种情况下，需要使用re-download脚本下载这些出错的rpm包：

	#!/bin/bash

	for n in `cat /tmp/build-log  | grep rpm | awk -F "rpm file:" '{print $2}' | awk '{print $1}' | awk -F ":" '{print $1}' | grep rpm | grep -v "^$"`; do
    	url=`echo $n | awk -F "media/" '{print $2}'`
    	echo $url
    	wget -c http://$url -O $n
	done
	echo "download over"

下载成功之后，请运行re-build脚本：

	#!/bin/bash

	old_dir=`pwd`

	function _get_dir() {
	    for n in `cat build-log  | \
              grep rpm | \
              awk -F "rpm file:" '{print $2}' | \
              awk '{print $1}' | \
              awk -F ":" '{print $1}' | \
              grep rpm | grep -v "^$" | \
              sort -u`; do
    		echo ${n%%./*}
		done
	}

	for n in `_get_dir | sort -u`; do
	    cd $n
	    creatrerepo .
	    cd $old_dir
	done
	echo "re-build success"

## Step 3 存放至http服务器

将些离线源做好之后，存放至http服务。Ubuntu可以直接存放于/var/www/目录下，而CentOS需要存放于/var/www/html/目录下，并且CentOS需要开启如下规则：

	iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
	chcon -R -h -t httpd_sys_content_t /var/www/html/
	chmod -R a+r /var/www/html/
	setsebool -P httpd_can_network_connect 1 &

在本文中，存放于10.239.82.94服务器的：/var/www/html/icehouse目录下，形成目录结构如下，仅供参考：

**注意**为了使用方便，本文也将CentOS-6.5-DVD1.iso中的所有的rpm包拷贝至cd1/Packages/目录，构建成离线源以供使用。

	icehouse/
	├── cd1
	│   └── Packages
	│       ├── repodata
	│       ├── TRANS.TBL
	├── centos.mirror.nac.net
	│   └── 6.5
	│       └── os
	│           └── x86_64
	│               └── Packages
	│                   ├── repodata
	├── ceph.com
	│   ├── packages
	│   │   └── ceph-extras
	│   │           └── centos6
	│   │               └── x86_64
	│   │                   ├── index.html?C
	│   │                   ├── repodata
	│       └── el6
	│           └── x86_64
	│               ├── index.html?C
	│               ├── repodata
	├── download
	├── download.fedoraproject.org
	│   └── pub
	│       └── epel
	│           └── 6
	│               └── x86_64
	│                   ├── repodata
	├── error
	├── mirror.cs.vt.edu
	│   └── pub
	│       └── CentOS
	│           └── 6.5
	│               └── updates
	│                   └── x86_64
	│                       └── Packages
	│                           ├── repodata
	├── mirror.steadfast.net
	│   └── epel
	│       └── 6
	│           └── x86_64
	│               ├── repodata
	├── re-build
	├── re-download
	├── repos.fedorapeople.org
	│   └── repos
	│       └── openstack
	│           └── openstack-icehouse
	│               └── epel-6
	│                   ├── epel
	│                   │   └── repodata
	│                   ├── repodata
	│   └── linux
	│       └── centos
	│           └── 6.5
	│               └── os
	│                   └── x86_64
	│                       └── Packages
	│                           ├── repodata
	├── yum.puppetlabs.com
	│   └── el
	│       └── 6
	│           ├── dependencies
	│           │   ├── SRPMS
	│           │   │   ├── repodata
	│           │   └── x86_64
	│           │       ├── repodata
	│           ├── devel
	│           │   ├── SRPMS
	│           │   │   └── repodata
	│           │   └── x86_64
	│           │       └── repodata
	│           └── products
	│               ├── SRPMS
	│               │   └── repodata
	│               └── x86_64
	│                   └── repodata
	└── yum.theforeman.org
	    ├── plugins
	    │   └── 1.5
	    │       └── el6
	    │           ├── source
	    │           │   ├── repodata
    	│           └── x86_64
    	│               ├── repodata
    	└── releases
	        └── 1.5
            	└── el6
	                ├── source
                	│   ├── repodata
                	└── x86_64
	                    ├── repodata

使用时，添加如下repo文件：

    [cd1]
    name=cd1
    baseurl=http://10.239.82.94/icehouse/cd1/Packages
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [theforeman-plugin-source]
    name=theforeman-plugins
    baseurl=http://10.239.82.94/icehouse/yum.theforeman.org/plugins/1.5/el6/source
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [theforman-plugins-x86_64]
    name=theforman-plugins-x86_64
    baseurl=http://10.239.82.94/icehouse/yum.theforeman.org/plugins/1.5/el6/x86_64
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [theforeman-release-source]
    name=theforeman-release-source
    baseurl=http://10.239.82.94/icehouse/yum.theforeman.org/releases/1.5/el6/source
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [theforeman-release-x86_64]
    name=theforeman-release-x86_64
    baseurl=http://10.239.82.94/icehouse/yum.theforeman.org/releases/1.5/el6/x86_64
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [puppetlabs-products-x86_64]
    name=puppetlabs-products-x86_64
    baseurl=http://10.239.82.94/icehouse/yum.puppetlabs.com/el/6/products/x86_64
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [puppetlabs-dependencies]
    name=puppetlabs-dependencies
    baseurl=http://10.239.82.94/icehouse/yum.puppetlabs.com/el/6/dependencies/x86_64
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [puppetlabs-devel]
    name=puppetlabs-devel
    baseurl=http://10.239.82.94/icehouse/yum.puppetlabs.com/el/6/devel/x86_64
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [puppetlabs-products-srpms]
    name=puppetlabs-products-srpms
    baseurl=http://10.239.82.94/icehouse/yum.puppetlabs.com/el/6/products/SRPMS
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [puppetlabs-dependencies-srpms]
    name=puppetlabs-dependencies-srpms
    baseurl=http://10.239.82.94/icehouse/yum.puppetlabs.com/el/6/dependencies/SRPMS
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [puppetlabs-devel-srpms]
    name=puppetlabs-devel-srpms
    baseurl=http://10.239.82.94/icehouse/yum.puppetlabs.com/el/6/devel/SRPMS
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [openstack-icehouse]
    name=openstack-icehouse
    baseurl=http://10.239.82.94/icehouse/repos.fedorapeople.org/repos/openstack/openstack-icehouse/epel-6/
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [ceph]
    name=ceph
    baseurl=http://10.239.82.94/icehouse/ceph.com/rpm/el6/x86_64/
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [ceph-extras]
    name=ceph-extras
    baseurl=http://10.239.82.94/icehouse/ceph.com/packages/ceph-extras/rpm/centos6/x86_64/
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [epel]
    name=epel
    baseurl=http://10.239.82.94/icehouse/mirror.steadfast.net/epel/6/x86_64/
    gpgcheck=0
    enabled=0
    proxy=_none_
    
    [rpmfind]
    name=rpmfind
    baseurl=http://10.239.82.94/icehouse/rpmfind.net/linux/centos/6.5/os/x86_64/Packages/
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    
    [nac-net]
    name=nac-net
    baseurl=http://10.239.82.94/icehouse/centos.mirror.nac.net/6.5/os/x86_64/Packages/
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [cs.vt]
    name=cs-vt
    baseurl=http://10.239.82.94/icehouse/mirror.cs.vt.edu/pub/CentOS/6.5/updates/x86_64/Packages/
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [download-fedoraproject]
    name=download-fedoraporject
    baseurl=http://10.239.82.94/icehouse/download.fedoraproject.org/pub/epel/6/x86_64
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [repos-fedorapeople]
    name=repos-fedorapeople
    baseurl=http://10.239.82.94/icehouse/repos.fedorapeople.org/repos/openstack/openstack-icehouse/epel-6/epel/
    gpgcheck=0
    enabled=1
    proxy=_none_
    
    [maridaDB]
    name=maridaDB
    baseurl=http://yum.mariadb.org/5.5.36/centos6-amd64/rpms
    gpgcheck=0
    enabled=1
    proxy=_none_

接下来，只需要yum makecache就可以使用离线源了。
