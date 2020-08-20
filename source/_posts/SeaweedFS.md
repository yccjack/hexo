---
title: SeaweedFS 
categories: 工具
tags: [SeaweedFS,分布式文件系统] 
description: seaweedfs是一个非常优秀的由 golang 开发的分布式存储开源项目。它是用来存储文件的系统，并且与使用的语言无关，使得文件储存在云端变得非常方便。

---


## SeaweedFS简介

SeaweedFS是基于go语言开发的高可用文件存储系统，主要特征

> - 存储上亿的文件(最终受限制于硬盘大小)
> - 速度快，内存占用小

<br/>

上手使用比fastDFS要简单很多，自带Rest API。

SeaWeeDFS作为对象存储库来有效地处理小文件。不是管理中央主机中的所有文件元数据，中央主机只管理文件卷，它允许这些卷服务器管理文件和它们的元数据。
这减轻了来自中央主机的并发压力，并将文件元数据扩展到卷服务器，允许更快的文件访问（仅一个磁盘读取操作）。

<br/>

每个文件的元数据只有40字节的磁盘存储开销。

<br/>

## 概述

seaweedfs是一个非常优秀的由 golang 开发的分布式存储开源项目。它是用来存储文件的系统，并且与使用的语言无关，使得文件储存在云端变得非常方便。

在逻辑上Seaweedfs的几个概念：

> - master 存储映射关系，文件和fid的映射关系 weed master
>- Node 系统抽象的结点，抽象为datacenter、rack、datanode
> - datacenter 数据中心，包含多个rack，类似一个机房
>- rack ：属于一个datacenter，类似机房中的一个机架
> - datanode ： 存储节点，存储多个volume，类似机架中的一个机器 weed volume
>- volume ：逻辑卷，存储needle
> - needle： 逻辑卷中的object，对应存储的文件
>- collection：文件集，默认所有文件都属于""文件集。如果想给某些文件单独分类，可以在申请id的时候指定相同的文件集
> - filer ：指向一个或多个master的file服务器，多个使用逗号隔开。
>

weed volume会创建一个 datanode ，可以指定所属的 datacenter rack和master ，会根据配置存储文件，默认一开始没有volume，当开始存储文件的时候才会创建一个volume，当这一个volume大小超过了volumeSizeLimitMB 就会新增一个volume，当volume个数超过了max则该datanode就不能新增数据了。那就需要在通过weed volume命令新增一个datanode。





## 搭建服务

### 安装go环境

> - 查看系统位数 getconf LONG_BIT
> - 下载源码包
> - 选择对应版本

```
cd /usr/local
# 下载
wget https://golangtc.com/static/go/1.9.2/go1.9.2.linux-amd64.tar.gz
# 将其传到其他两台机器
# 解压
tar -zxf go1.9.2.linux-amd64.tar.gz
# 配置
vim /etc/profile
#加入
export GOPATH=/opt/go
export GOROOT=/usr/local/go
export GOOS=linux
export GOBIN=$GOROOT/bin
export GOTOOLS=$GOROOT/pkg/tool/
export PATH=$PATH:$GOBIN:$GOTOOLS

# 使配置文件生效
source /etc/profile

# 查看
go version
```

### 安装seaweedfs

(1)下载 https://github.com/chrislusf/seaweedfs/releases/选择对应的版本

(2)解压 tar -zxf linux_amd64.tar.gz

(3)./weed -h 查看帮助创建运行需要的目录   



```xml
/../data            
 /../ vol/vol[1-3]    
 /../logs
```



(4)配置运行master(如单机删除defaultReplication)

```xml
./weed master -mdir=/../data -port=9333 -defaultReplication="001" -ip="172.16.20.71" &>> /../logs/master.log &

```

(5) 配置运行volume

具体参数查看帮助
/usr/local/weed volume -h

```xml
./weed volume -port=9331 -dir=vol/vol1/ -max=100 -mserver="192.168.6.224:9333" -ip="192.168.6.224" &>vol/vol1/vol1.log &
```



## 单机基准测试(Benchmarks)

````xml-dtd
# prepare directories
mkdir vol/vol1 vol/vol2 vol/vol3
# start 3 servers
./weed server -dir=./vol/vol1 -master.port=9333 -volume.port=8083 &
./weed volume -dir=./vol/vol2 -port=8084 &
./weed volume -dir=./vol/vol3 -port=8085 &
./weed benchmark -master=localhost:9333
````

可以使用命令查看基准测试帮助信息：

```xml-dtd
./weed benchmark -h
```

默认情况下weed的基准测试使用100万1KB file

机器配置：

```xml-dtd
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                8
On-line CPU(s) list:   0-7
Thread(s) per core:    1
Core(s) per socket:    1
CPU socket(s):         8
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 45
Stepping:              7
CPU MHz:               2400.000
BogoMIPS:              4800.00
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              10240K
NUMA node0 CPU(s):     0-7
```

基准测试结果：

```xml-dtd
Write 1 million 1KB file:
Concurrency Level:      16
Time taken for tests:   131.658 seconds
Complete requests:      1048576
Failed requests:        0
Total transferred:      1106762855 bytes
Requests per second:    7964.42 [#/sec]
Transfer rate:          8209.35 [Kbytes/sec]

Connection Times (ms)
              min      avg        max      std
Total:        0.4      1.9       338.9      2.8

Percentage of the requests served within a certain time (ms)
   50%      1.6 ms
   66%      1.9 ms
   75%      2.2 ms
   80%      2.4 ms
   90%      3.2 ms
   95%      4.2 ms
   98%      5.2 ms
   99%      6.1 ms
  100%    338.9 ms
  
 
Randomly read 1 million files:

Concurrency Level:      16
Time taken for tests:   39.724 seconds
Complete requests:      1048576
Failed requests:        0
Total transferred:      1106738848 bytes
Requests per second:    26396.35 [#/sec]
Transfer rate:          27207.53 [Kbytes/sec]

Connection Times (ms)
              min      avg        max      std
Total:        0.1      0.5       64.5      0.7

Percentage of the requests served within a certain time (ms)
   50%      0.3 ms
   66%      0.4 ms
   75%      0.5 ms
   80%      0.6 ms
   90%      0.9 ms
   95%      1.3 ms
   98%      2.3 ms
   99%      3.1 ms
  100%     64.5 ms
```

删除测试数据:

```xml-dtd
curl "http://192.168.6.226:9333/col/delete?collection=benchmark&pretty=y"
```



## 启动服务的方式
> 方式1
>
> weed scaffold -config=filer -output=. 然后修改里面leveldb的目录
>
> weed server -dir=./vtmp -master.port=9333 -master.dir=./mtmp -volume.max=5 -volume.port=9991 -filer -filer.port=8888 -master.volumeSizeLimitMB=10
>
> -whiteList
>
> -filer.dir 目录来存储元数据，默认为指定-dir的“filer”子目录
>
> -master.volumeSizeLimitMB 默认最大30000000 （30G）
>
> -master.dir用于存储元数据的数据目录，默认为与指定的-dir相同
>
> 

> 方式2
>
> weed master -port=9333 -mdir=./mtmp
>
> weed volume -port=9991 -dir=./vtmp -max=100 -mserver=localhost:9333
>
> weed scaffold -config=filer -output=.
>
> weed filer -port=8888 -master=localhost:9333




## volume的备份机制 Replication
默认000 不备份

defaultReplication

000 不备份， 只有一份数据

001 在相同的rackj里备份一份数据

010 在相同数据中心内不同的rack间备份一份数据

100 在不同的数据中心备份一份数据

200 在两个不同的数据中心各复制2次

110 在不同的rack备份一份数据， 在不同的数据中心备份一次

如果数据备份类型是 xyz形式

各自的意义

x 在别的数据中心备份的份数

y 不相同数据中心不同的racks备份的份数

z 在别的服务器相同的rack的备份份数



## filer的使用
首先，运行**weed scaffold -config=filer**生成filer.toml文件.

最简单的filer.toml可以是:

```xml-dtd
[leveldb]
enabled = true
dir = "."					# directory to store level db files
```
**启动filer功能**



```xml-dtd
# assuming you already started weed master and weed volume
weed filer

# Or assuming you have nothing started yet,
# this command starts master server, volume server, and filer in one shot. 
# It's strictly the same as starting them separately
weed server -filer=true
```

**增加/删除/查看文件**



```xml-dtd
# POST a file and read it back
curl -F "filename=@README.md" "http://localhost:8888/path/to/sources/"
curl "http://localhost:8888/path/to/sources/README.md"

# POST a file with a new name and read it back
curl -F "filename=@Makefile" "http://localhost:8888/path/to/sources/new_name"
curl "http://localhost:8888/path/to/sources/new_name"

# list sub folders and files
visit "http://localhost:8888/path/to/sources/"

# if lots of files under this folder, here is a way to efficiently paginate through all of them
visit "http://localhost:8888/path/to/sources/?lastFileName=abc.txt&limit=50"
```



Filer有一个连接到Master的持久客户端，以获取所有卷的位置更新。没有网络往返来查找卷ID位置。



对于文件读取：



1. 文件管理器从Filer Store查找元数据，可以是Cassandra / Mysql / Postgres / Redis / LevelDB / etcd。

   

2. Filer从卷服务器读取并传递给读取请求。

   

   

   

   ![](/images/storage/FilerRead.png)





对于文件写入：

1. 客户端流文件到Filer

   

2. Filer将数据上传到Weed Volume Servers，并将大文件分成块。

   

3. Filer将元数据和块信息写入Filer存储区。

   

## Filer Store

**复杂度**

> 对于一个文件检索，（file_parent_directory，fileName）=>元数据查找将是用于LSM树或Btree实现的O（logN），其中N是现有条目的数量，或者对于Redis是O（1）。
>
> 对于特定目录下的文件列表，列表只是对LSM树或Btree的简单扫描，或对于Redis的O（1）。
>
> 对于添加一个文件，如果不存在，将以递归方式创建父目录。然后将创建文件条目。
>
> 对于文件重命名，它只是O（1）操作，删除旧元数据并插入新元数据，而不更改卷服务器上的实际文件内容。
>
> 对于目录重命名，它将是O（N）操作，其中N是要重命名目录下的文件和文件夹的数量。这是因为他们每个人都需要调整元数据。但是，卷服务器上的实际文件内容仍然没有变化。



**用例**

客户可以通过HTTP评估一个“weed filer”，列出目录下的文件，通过HTTP POST创建文件，直接通过HTTP POST读取文件。

虽然一个“weed filer”只能位于一台机器上，但您可以在多台机器上启动多个“weed filer”，每个“weed filer”实例在其自己的集合中运行，具有自己的命名空间，但共享相同的SeaweedFS存储。

**Filer线性可扩展**

Filer被设计为线性可伸缩的，并且仅受底层元数据存储的限制。



**Filer工作负载**

Filer有两个用例。

当filer直接用于上传和下载文件时，以及与“weed s3”一起使用时，文件管理器还需要在读取和写入期间处理文件内容以及元数据。所以添加多个文件服务器是个好主意。



当filer与“weed mount”一起使用时，filer仅提供文件元数据检索。实际文件内容直接在“weed mount”和“weed volume”服务器之间读写。所以文件管理器没有那么多。

## 文件管理器命令和操作

**复制到Filer**
weed filer.copy 可以将一个或一个文件或目录列表复制到文件管理器。

```xml-dtd
// copy all go files under current directory to filer's /github/ folder.
// The directory structure is copied also.
> weed filer.copy -include *.go . http://localhost:8888/github/
...
Copy ./unmaintained/change_replication/change_replication.go => http://localhost:8888/github/./unmaintained/change_replication/change_replication.go
Copy ./unmaintained/fix_dat/fix_dat.go => http://localhost:8888/github/./unmaintained/fix_dat/fix_dat.go
Copy ./unmaintained/see_idx/see_idx.go => http://localhost:8888/github/./unmaintained/see_idx/see_idx.go
Copy ./weed/command/backup.go => http://localhost:8888/github/./weed/command/backup.go
Copy ./weed/command/benchmark.go => http://localhost:8888/github/./weed/command/benchmark.go
Copy ./weed/command/command.go => http://localhost:8888/github/./weed/command/command.go
Copy ./weed/command/compact.go => http://localhost:8888/github/./weed/command/compact.go
```



## 大文件处理


为了实现高并发性，SeaweedFS尝试将整个文件读写并写入内存。但这对大文件不起作用。


以下是在“weed upload”命令中实现的。对于第三方客户，这是规范.


为了支持大文件，SeaweedFS支持以下两种文件：

> 块文件。每个块文件实际上只是SeaweedFS的普通文件。
> 大块文件。一个简单的json文件，包含所有块的列表。
> 

这段代码显示了json文件结构：
```js
package operation

import (
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"io/ioutil"
	"net/http"
	"sort"

	"google.golang.org/grpc"

	"sync"

	"github.com/chrislusf/seaweedfs/weed/glog"
	"github.com/chrislusf/seaweedfs/weed/util"
)

var (
	// when the remote server does not allow range requests (Accept-Ranges was not set)
	ErrRangeRequestsNotSupported = errors.New("Range requests are not supported by the remote server")
	// ErrInvalidRange is returned by Read when trying to read past the end of the file
	ErrInvalidRange = errors.New("Invalid range")
)

type ChunkInfo struct {
	Fid    string `json:"fid"`
	Offset int64  `json:"offset"`
	Size   int64  `json:"size"`
}

type ChunkList []*ChunkInfo

type ChunkManifest struct {
	Name   string    `json:"name,omitempty"`
	Mime   string    `json:"mime,omitempty"`
	Size   int64     `json:"size,omitempty"`
	Chunks ChunkList `json:"chunks,omitempty"`
}

// seekable chunked file reader
type ChunkedFileReader struct {
	Manifest *ChunkManifest
	Master   string
	pos      int64
	pr       *io.PipeReader
	pw       *io.PipeWriter
	mutex    sync.Mutex
}

func (s ChunkList) Len() int           { return len(s) }
func (s ChunkList) Less(i, j int) bool { return s[i].Offset < s[j].Offset }
func (s ChunkList) Swap(i, j int)      { s[i], s[j] = s[j], s[i] }

func LoadChunkManifest(buffer []byte, isGzipped bool) (*ChunkManifest, error) {
	if isGzipped {
		var err error
		if buffer, err = util.UnGzipData(buffer); err != nil {
			return nil, err
		}
	}
	cm := ChunkManifest{}
	if e := json.Unmarshal(buffer, &cm); e != nil {
		return nil, e
	}
	sort.Sort(cm.Chunks)
	return &cm, nil
}

func (cm *ChunkManifest) Marshal() ([]byte, error) {
	return json.Marshal(cm)
}

func (cm *ChunkManifest) DeleteChunks(master string, grpcDialOption grpc.DialOption) error {
	var fileIds []string
	for _, ci := range cm.Chunks {
		fileIds = append(fileIds, ci.Fid)
	}
	results, err := DeleteFiles(master, grpcDialOption, fileIds)
	if err != nil {
		glog.V(0).Infof("delete %+v: %v", fileIds, err)
		return fmt.Errorf("chunk delete: %v", err)
	}
	for _, result := range results {
		if result.Error != "" {
			glog.V(0).Infof("delete file %+v: %v", result.FileId, result.Error)
			return fmt.Errorf("chunk delete %v: %v", result.FileId, result.Error)
		}
	}

	return nil
}

func readChunkNeedle(fileUrl string, w io.Writer, offset int64) (written int64, e error) {
	req, err := http.NewRequest("GET", fileUrl, nil)
	if err != nil {
		return written, err
	}
	if offset > 0 {
		req.Header.Set("Range", fmt.Sprintf("bytes=%d-", offset))
	}

	resp, err := util.Do(req)
	if err != nil {
		return written, err
	}
	defer func() {
		io.Copy(ioutil.Discard, resp.Body)
		resp.Body.Close()
	}()

	switch resp.StatusCode {
	case http.StatusRequestedRangeNotSatisfiable:
		return written, ErrInvalidRange
	case http.StatusOK:
		if offset > 0 {
			return written, ErrRangeRequestsNotSupported
		}
	case http.StatusPartialContent:
		break
	default:
		return written, fmt.Errorf("Read chunk needle error: [%d] %s", resp.StatusCode, fileUrl)

	}
	return io.Copy(w, resp.Body)
}

func (cf *ChunkedFileReader) Seek(offset int64, whence int) (int64, error) {
	var err error
	switch whence {
	case 0:
	case 1:
		offset += cf.pos
	case 2:
		offset = cf.Manifest.Size - offset
	}
	if offset > cf.Manifest.Size {
		err = ErrInvalidRange
	}
	if cf.pos != offset {
		cf.Close()
	}
	cf.pos = offset
	return cf.pos, err
}

func (cf *ChunkedFileReader) WriteTo(w io.Writer) (n int64, err error) {
	cm := cf.Manifest
	chunkIndex := -1
	chunkStartOffset := int64(0)
	for i, ci := range cm.Chunks {
		if cf.pos >= ci.Offset && cf.pos < ci.Offset+ci.Size {
			chunkIndex = i
			chunkStartOffset = cf.pos - ci.Offset
			break
		}
	}
	if chunkIndex < 0 {
		return n, ErrInvalidRange
	}
	for ; chunkIndex < cm.Chunks.Len(); chunkIndex++ {
		ci := cm.Chunks[chunkIndex]
		// if we need read date from local volume server first?
		fileUrl, lookupError := LookupFileId(cf.Master, ci.Fid)
		if lookupError != nil {
			return n, lookupError
		}
		if wn, e := readChunkNeedle(fileUrl, w, chunkStartOffset); e != nil {
			return n, e
		} else {
			n += wn
			cf.pos += wn
		}

		chunkStartOffset = 0
	}
	return n, nil
}

func (cf *ChunkedFileReader) ReadAt(p []byte, off int64) (n int, err error) {
	cf.Seek(off, 0)
	return cf.Read(p)
}

func (cf *ChunkedFileReader) Read(p []byte) (int, error) {
	return cf.getPipeReader().Read(p)
}

func (cf *ChunkedFileReader) Close() (e error) {
	cf.mutex.Lock()
	defer cf.mutex.Unlock()
	return cf.closePipe()
}

func (cf *ChunkedFileReader) closePipe() (e error) {
	if cf.pr != nil {
		if err := cf.pr.Close(); err != nil {
			e = err
		}
	}
	cf.pr = nil
	if cf.pw != nil {
		if err := cf.pw.Close(); err != nil {
			e = err
		}
	}
	cf.pw = nil
	return e
}

func (cf *ChunkedFileReader) getPipeReader() io.Reader {
	cf.mutex.Lock()
	defer cf.mutex.Unlock()
	if cf.pr != nil && cf.pw != nil {
		return cf.pr
	}
	cf.closePipe()
	cf.pr, cf.pw = io.Pipe()
	go func(pw *io.PipeWriter) {
		_, e := cf.WriteTo(pw)
		pw.CloseWithError(e)
	}(cf.pw)
	return cf.pr
}
```

```js
type ChunkInfo struct {
	Fid    string `json:"fid"`
	Offset int64  `json:"offset"`
	Size   int64  `json:"size"`
}

type ChunkList []*ChunkInfo

type ChunkManifest struct {
	Name   string    `json:"name,omitempty"`
	Mime   string    `json:"mime,omitempty"`
	Size   int64     `json:"size,omitempty"`
	Chunks ChunkList `json:"chunks,omitempty"`
}
```
在读取Chunk Manifest文件时，SeaweedFS将根据ChunkInfo列表查找并发送数据文件。

**创建新的大文件**
SeaweedFS将努力委托给客户方。步骤是：

**将大文件拆分成块**

>像往常一样上传每个文件块，使用mime类型“application / octet-stream”。将相关信息保存到ChunkInfo结构中。每个块可以分布到不同的卷上，可能提供更快的并行访问。
>使用mime类型“application / json”上传清单文件，并添加url参数“cm = true”。存储清单文件的FileId是大文件的入口点。

**更新大文件**
通常我们只是附加大文件。更新特定的文件块几乎是一样的。
附加大文件的步骤：
> 像往常一样上传新文件块，使用mime类型“application / octet-stream”。将相关信息保存到ChunkInfo结构中。
> 使用mime类型“application / json”更新更新的清单文件，并添加url参数“cm = true”。
> 


## 优化
以下是优化SeaweedFS的策略或最佳方法。

### 使用LevelDB

启动卷服务器时，可以指定索引类型。默认情况下它使用内存。启动卷服务器时这很快，但启动时间可能很长，以便将文件索引加载到内存中。

`weed volume -index=leveldb`可以改为leveldb。启动卷服务器要快得多，但访问文件时要慢一点。与网络速度相比，在大多数情况下额外的成本并不是那么多。

### 预分配卷文件磁盘空间

在某些Linux文件系统中，例如XFS，ext4，Btrfs等，SeaweedFS可以选择为卷文件分配磁盘空间。这可以确保文件数据位于连续的块上，这可以在文件很大时提高性能，并且可以覆盖多个扩展区。

要启用磁盘空间预迁移，请在具有支持文件系统的Linux OS上使用这些选项启动主站。

```xml-dtd
-volumePreallocate
    	Preallocate disk space for volumes.
  -volumeSizeLimitMB uint
    	Master stops directing writes to oversized volumes. (default 30000)
```



### 增加并发写入

默认情况下，SeaweedFS会自动增大卷。例如，对于非复制卷，将同时分配7个可写卷。

如果要将写入分发到更多卷，可以通过此URL指示SeaweedFS master。

```xml-dtd
curl http：// localhost：9333 / vol / grow ？count = 12 ＆ replication = 001
```

这将为12个卷分配001复制。由于001复制意味着相同数据的2个副本，因此实际上将消耗24个物理卷。

### 增加并发读取

与上面相同，更多卷将增加读并发性。

另外，增加复制也会有所帮助。将相同的数据存储在多个服务器上肯定会增加读取并发性。

### 添加更多硬盘

更多硬盘将为您提供更好的写入/读取吞吐量。

### 增加用户打开文件限制

 SeaweedFS通常只打开一些实际的磁盘文件。但是网络文件请求可能超过默认限制，通常默认为1024.对于生产，您需要root权限才能将限制增加到更高的限制，例如“ulimit -n 10240”。



### 内存消耗

对于卷服务器，内存消耗与文件数密切相关。例如，如果每个文件只有20KB，则一个32G卷可以轻松拥有150万个文件。为了将150万个元数据条目存储在内存中，目前SeaweedFS消耗36MB内存，每个内存大约24字节。因此，如果您分配64个卷（2TB），则需要2个卷~~3GB内存。但是，如果平均文件大小较大，例如200KB，则只有200~~需要300MB内存。

SeaweedFS还具有leveldb，boltdb和btree模式支持，可以进一步降低内存消耗。

要使用它，“weed server -volume.index = [memory | leveldb | boltdb | btree]”或“weed volume -index = [memory | leveldb | boltdb | btree]”。您可以随时尽可能在4种模式之间切换。如果leveldb或boltdb的文件已过期或缺失，将根据需要重新生成它们。

boltdb的编写速度相当慢，大约需要6分钟来重建1553934文件的索引。Boltdb从磁盘加载1,553,934 x 16 = 24,862,944bytes，并在6分钟内生成大小为134,217,728字节的boltdb。为了进行比较，leveldb在8秒内重新创建了大到27,188,148字节的索引。

要测试内存消耗，请创建leveldb或boltdb索引。基准集合中共有7卷，每卷约有1553K个文件。服务器重新启动，然后我启动基准测试工具来读取大量文件。对于leveldb，服务器内存从142,884KB开始，并保持在179,340KB。对于boltdb，服务器内存从73,756KB开始，并保持在144,564KB。对于内存，服务器内存从368,152KB开始，并保持在448,032KB。

为了测试写入速度，我使用带有默认参数的基准测试工具。对于boltdb，写入大约是4.1MB / s，4.1K文件/ s对于leveldb，写入大约是10.4MB / s，10.4K文件/ s对于内存来说，它更快一点，没有统计差异。但我使用SSD，而os缓冲区缓存也会影响数字。所以你的结果可能会有所不同。

在v0.75中添加了Btree模式，以优化无序自定义文件密钥的内存。对于SeaweedFS主服务器分配的普通文件密钥，Btree模式可能会花费更多内存，但通常比自定义文件密钥更有效。请测试你的病例。

注意：BoltDB的限制是32位系统上的最大db大小为256MB。



### 使用您自己的密钥插入

文件ID生成实际上非常简单，您可以使用自己的方式生成文件密钥。

文件密钥有3个部分：

> - volume id：具有可用空间的卷
>
>   
>
> - 针头ID：单调增加且唯一的数字
>
>   
>
> - 文件cookie：随机数，您可以以任何方式自定义它

您可以直接要求主服务器分配文件密钥，并将针ID部分替换为您自己的唯一ID，例如用户ID。

您还可以从服务器状态获取每个卷的可用空间。

```xml-dtd
curl “ http：// localhost：9333 / dir / status？pretty = y ”
```

一旦确定了无空间空间，就可以使用自己的文件ID。只需要确保文件密钥格式兼容。

指定的文件cookie也可以自定义。

自定义针ID和/或文件cookie是可接受的行为。“严格单调增加”不是必需的，但是为了保持存储器数据结构的有效性，期望保持文件id以“大多数”增加的顺序。

### 上传大文件
如果文件很大且网络很慢，服务器将花费时间来读取文件。请增加卷服务器的“-readTimeout = 3”限制设置。如果上传时间超过限制，则会切断连接。

**使用“自动拆分/合并”上载大型文件**
如果文件很大，最好以这种方式上传：
```xml-dtd
weed upload -maxMB = 64 the_file_name
```
这会将文件拆分为每个64MB的数据块，并单独上传。所有数据块的文件ID都保存到另一个元块中。返回元块的文件ID。
下载文件时，只需
```xml-dtd
weed download the_meta_chunk_file_id
```
元块具有文件ID列表，每行上有每个文件ID。因此，如果要并行处理它们，可以下载元块并直接处理每个数据块。

### 集合作为简单名称空间
分配文件ID时，
```xml-dtd
curl http://master:9333/dir/assign?collection=pictures
curl http://master:9333/dir/assign?collection=documents
```
如果尚未创建“图片”集合和“文档”集合，也会生成它们。每个集合都有其专用卷，并且它们不会共享相同的卷。
实际上，实际数据文件具有集合名称作为前缀，例如“pictures_1.dat”，“documents_3.dat”。



## Java操作SeaweedFS

首先导入pom依赖

```xml
     <dependency>
            <groupId>net.anumbrella.seaweedfs</groupId>
            <artifactId>seaweedfs-java-client</artifactId>
            <version>0.0.2.RELEASE</version>
        </dependency>
```

编写SeaweedFS配置类SeaweedFSConfig

```java
@Configuration
public class SeaweedFSConfig {

    @Value("${seaweedfs.host}")
    private String host;
    @Value("${seaweedfs.port}")
    private int port;

    @Bean
    public FileTemplate fileTemplate() {
        FileSource fileSource = new FileSource();
        // SeaweedFS master服务ip地址
        fileSource.setHost(host);
        // SeaweedFS master服务端口
        fileSource.setPort(port);
        try {
            // 启动服务
            fileSource.startup();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return new FileTemplate(fileSource.getConnection());
    }
}
```

我的application.properties配置文件如下:

```xml
seaweedfs.host=192.168.6.224 #多个ip地址用逗号隔开
seaweedfs.port=9333
```

上传文件的方法如下,用的是Spring的JUnit测试:

```java
@Autowired
private FileTemplate template;

@Test
public void testSeaweedFS() throws IOException {
        // 上传可以指定文件名
        FileHandleStatus handleStatus = template.saveFileByStream("file.type", new FileInputStream(new File("filePath")));
        // 获取文件ID,可通过这个ID获取到文件
        String fileId = handleStatus.getFileId();
        StreamResponse fileStream = template.getFileStream(fileId);
        InputStream inputStream = fileStream.getInputStream();
        // 获取流之后流拷贝输出到本地
        IOUtils.copy(inputStream,new FileOutputStream(new File("outPath")));
    }
```


