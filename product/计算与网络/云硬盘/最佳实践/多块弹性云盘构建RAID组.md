## RAID 简介
RAID 可以将多个磁盘组合起来构成一个磁盘阵列组，提高数据的读写性能和可靠性。操作系统只会将磁盘阵列组当作一个硬盘来使用。目前RAID有多种等级，根据选择版本不同，磁盘阵列组相较于一块容量相当的大硬盘有增强数据集成度、增强容错功能、增加处理量或容量等优势。
>!
>- 请及时对即将到期的弹性云硬盘进行 [续费](https://cloud.tencent.com/document/product/362/6739) 操作，以免由于弹性云硬盘到期被系统强制隔离对 RAID 产生影响。
>- 建议创建 RAID 1、RAID 01、RAID 10 时，使用相同大小的分区，减少磁盘空间的浪费。

 RAID 0、RAID 1、RAID 01 和 RAID 10之间的相同点与差异点如下表所示：
<table>
     <tr>
         <th>RAID 级别</th>  
         <th>简介</th>  
		 <th>优/缺点</th>
		 <th>使用场景（建议）</th>
     </tr>
	 <tr>
         <td>RAID 0</td>
				 <td>存储模式：数据分段存放在不同的磁盘中。<br /></br>虚拟盘大小：阵列中所有盘容量之和。</td>
		 <td>
		 <ul><li><b>优点</b>：读写可以并行。</br>理论上读写速率可达单个磁盘的 N 倍（N 为组成 RAID0 的磁盘个数），但实际上受限于文件大小、文件系统大小等多种因素。</li>
		 <li><b>缺点</b>：没有数据冗余，即便只有单个磁盘损坏，在最严重的情况下也有可能导致所有数据的丢失。</li></ul></td>
		 <td>对 I/O 性能要求很高，并且已通过其他方式对数据进行备份处理或者不需要进行数据备份的场景</td>
     </tr> 
	 <tr>
         <td>RAID 1</td>
         <td>存储模式：数据被镜像存储在多个磁盘中。<br /></br> 虚拟盘大小：阵列中容量最小的盘的容量。</td>
		 <td>
		 <ul><li><b>优点</b>：<ul>
		 <li>读取速度快。</li>
		 <li>数据可靠性较高，单个磁盘的损坏不会导致数据的不可修复。</li>
		 </ul>
		 <li><b>缺点</b>：<ul>
		 <li>磁盘利用率最低。</li>
		 <li>写入速度受限于单个磁盘的写入速度。</li>
		 </ul></ul>
		 </td>
		 <td>对读性能要求较高，并且需要对写入的数据进行备份处理的场景。</td>
     </tr>
	 <tr>
         <td>RAID 01</td>
         <td>先用多个盘构建成 RAID 0，再用多个 RAID 0 构建成 RAID 1。</td>
		 <td><ul><li><b>优点</b>：同时具备 RAID 0 和 RAID 1 的优点。</li>
		 <li><b>缺点</b>：<ul><li>成本相对较高，至少需要使用四块盘。</li><li>单磁盘的损坏会导致同组的磁盘都不可用。</li></td>
		 <td   rowspan="2">推荐使用 RAID 10。</td>
     </tr>
	 <tr>
         <td>RAID 10</td>
         <td>先用多个盘构建成 RAID 1，再用多个 RAID 1 构建成 RAID 0。</td>
		 <td><ul><li><b>优点</b>：同时具备 RAID 0 和 RAID 1 的优点。</li>
		 <li><b>缺点</b>：成本相对较高，至少需要使用四块盘。</li></td>
     </tr>
</table>



## 构建 RAID
>?本文以在 CentOS 云服务器中使用四块弹性云硬盘构建 RAID 0 为例。不同操作系统、不同 RAID 级别的操作可能不同，本文仅供参考，具体操作步骤和差异请参考对应操作系统的产品文档或 RAID 相关文档。

Linux 内核提供用于管理 RAID 设备的 md 模块，可以直接使用 mdadm 工具来调用 md 模块创建 RAID 0。
![](//mccdn.qcloud.com/static/img/9f42e96976ee6f3655090a4208f461c5/image.png)


1. 执行以下命令，安装 mdadm。
```
mdadm -y
```
![](//mccdn.qcloud.com/static/img/59896b0ee3f20cd0f20f2f3633e56a1f/image.png)
2. 执行以下命令，使用 mdadm 创建 RAID 0。
```
mdadm --create /dev/md0 --level=0 --raid-devices=4 /dev/vd[cdef]1
```
![](https://main.qcloudimg.com/raw/7c2d92dc0cf8225d56c6696127e1a6ce.png)
3. 执行以下命令，使用 mkfs 创建文件系统（以使用 EXT3 文件系统为例）。
```
mkfs.ext3 /dev/md0
```
![](//mccdn.qcloud.com/static/img/e92608f31d914556a585e3190a009a64/image.png)
4. 依次执行以下命令，挂载文件系统。
```
mount /dev/md0 md0/
tree md0
```
![](//mccdn.qcloud.com/static/img/a4c36941609c64a3753648622392de65/image.png)
5. 执行以下命令，查看文件系统详情，并记录其 UUID（以`3c2adec2:14cf1fa7:999c29c5:7d739349`为例）。
```
mdadm --detail --scan
```
![](//mccdn.qcloud.com/static/img/e42b1f74126420929cd3b3668cca3f21/image.png)
6. 执行以下命令，编辑 mdadm 配置文件。
```
vi /etc/mdadm.conf
```
7. 在编辑模式下，输入相关内容。
对于弹性云硬盘，建议输入以下配置：
```
DEVICE /dev/disk/by-id/virtio-弹性云盘1ID-part1 
DEVICE /dev/disk/by-id/virtio-弹性云盘2ID-part1 
DEVICE /dev/disk/by-id/virtio-弹性云盘3ID-part1 
DEVICE /dev/disk/by-id/virtio-弹性云盘4ID-part1 
ARRAY 逻辑设备路径 metadata= UUID=
```
本文以逻辑设备路径是 /dev/md0，metadata 是1.2为例，则输入：
```
DEVICE /dev/disk/by-id/virtio-弹性云盘1ID-part1 
DEVICE /dev/disk/by-id/virtio-弹性云盘2ID-part1 
DEVICE /dev/disk/by-id/virtio-弹性云盘3ID-part1 
DEVICE /dev/disk/by-id/virtio-弹性云盘4ID-part1 
ARRAY /dev/md0 metadata=1.2 UUID=3c2adec2:14cf1fa7:999c29c5:7d739349
```
8. 按 Esc，输入`:wq`保存并退出。
