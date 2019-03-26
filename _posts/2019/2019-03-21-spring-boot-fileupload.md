---
layout: post
title: springboot(六)：Spring boot实现基于web文件上传下载功能
category: springboot
tags: [springboot]
---



## 说明

​	最近部门的实施人员需要一个基于web的传输大文件工具，于是寻找了一下资料，集成springboot书写了一个文件上传的小工具。

​	web中上传大文件不像桌面软件那么容易，在web端上传文件常用的做法就是表单上传，但是当文件较大时，如果网络速度比较慢，那么就只能等着文件上传完成，期间不能做刷新页面的操作，并且如果出现网络波动，那么就会上传失败，前面上传的操作就白费了，需要重新上传。

​	该项目是基于百度开源的一款前端上传控件WebUploader来实现的，WebUploader是由Baidu WebFE(FEX)团队开发的一个简单的以HTML5为主，FLASH为辅的现代文件上传组件。

​	对于大文件的上传，我们需要实现哪些必须要的功能呢？

- **断点续传** 文件上传最主要的功能，在断网或者暂停的情况下， 能够接着上次传输的进度继续上传
- **分块上传** 通过前端将文件分块，后台将分块的文件组装起来成为新的文件，这是实现断点续传的基础
- **文件秒传** 如果服务端已经存在该文件，那么再次上传此文件的时候，就能实现秒传到服务端
- **基于WebUploader实现的功能** 这些功能是通过[WebUploader](http://fex.baidu.com/webuploader) 来实现的，使用方法很简单
  - **多线程上传** 实现多个线程同时上传不同块的文件
  - **文件进度显示** 显示文件上传的进度情况
  - **多文件同时上传** 可以选择多个文件同时上传


## 项目搭建

### 创建一个简单的springboot项目

在pom文件中引入文件上传的基础依赖包

~~~xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- 公共工具包 -->
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.5</version>
		</dependency>
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.4</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.45</version>
		</dependency>

		<dependency>
			<groupId>io.netty</groupId>
			<artifactId>netty-all</artifactId>
			<version>4.1.6.Final</version>
		</dependency>
~~~



## 文件上传

### 分块上传

​	上传和下载过程中的断点续传，暂停都需要用到分块，分块是整个项目中的基础，前端的分块采用的是百度的开源框架webuploader来实现，它将分块等很多基本功能封装起来，使用起来非常方便。

​	后端取得上传的分块文件后，再将这些文件合并成原来的文件，将这些文件按照原本的位置写入到文件中即可，这里将使用到java中的**RandomAccessFile**，在java发展的过程中，**RandomAccessFile** 中的绝大部分功能都被内存映射文件**memory-mapped files** 取代，这里书写了两种方法来实现分块文件的还原。



### webuploader 实现分块详解

​ js实现分片

~~~javascript
// 实例化对象
var uploader = WebUploader.create({
    //dnd: '#dndArea',
    //paste: '#uploader',
    // 指定选择文件的按钮容器，不指定则不创建按钮。
    pick: {
        id:'#getFile', // 指定选择文件的按钮容器，不指定则不创建按钮。注意 这里虽然写的是 id, 但是不是只支持 id, 还支持 class, 或者 dom 节点
        label: '选择文件', // 请采用 innerHTML 代替
        multiple: true   // 是否开起同时选择多个文件能力
    },
    auto: false , // 设置为 true 后，不需要手动调用上传，有文件选择即开始上传
    prepareNextFile: true, //否允许在文件传输时提前把下一个文件准备好。 某些文件的准备工作比较耗时，比如图片压缩，md5序列化。 如果能提前在当前文件传输期处理，可以节省总体耗时。
    chunked: true, // 是否要分片处理大文件上传
    chunkSize: chunkSize,  // 如果要分片，分多大一片？ 默认大小为5M
    chunkRetry: 4, //如果某个分片由于网络问题出错，允许自动重传多少次
    chunkRetryDelay: 1000, // 开启重试后，设置重试延时时间, 单位毫秒
    threads: 3, // 上传并发数。允许同时最大上传进程数
    formData: {
        uid: 0,
        md5: '',
        chunkSize: chunkSize
    },
    server: 'file/fileUpload',
    swf: 'js/Uploader.swf',
    fileNumLimit: 1000, //验证文件总数量, 超出则不允许加入队列
    fileSizeLimit: 1024 * 1024 * 1024 * 1024, // 10G,验证文件总大小是否超出限制, 超出则不允许加入队列
    fileSingleSizeLimit: 1024 * 1024 * 1024 * 10,// 验证单个文件大小是否超出限制, 超出则不允许加入队列
    duplicate: false, //去重， 根据文件名字、文件大小和最后修改时间来生成hash Key
});
~~~

这里对几个重要的参数做一下说明：

> chunked: 表示是否分片处理大文件，默认为false，只有开启了这个参数才会将文件分片传输到服务端
>
> chunkSize: 表示每个分片的大小，此大小需要和后端接收分片的大小保持一直，这样才能保证文件不被破环
>
> threads: 表示允许同时上传的进程数，根据电脑的性能，可以适当调整此值来提高文件上传的速度
>
> fileNumLimit: 一次能够选择的文件中数量，如果选择的文件数量超过了这个限制，则不能加入到上传队列中，不配置此参数，则不会限制
>
> fileSizeLimit: 队列中所有文件总大小限制，这个根据实际情况来指定总文件大小限制，如果不配置，则不会限制
>
> fileSingleSizeLimit: 验证单个文件的大小限制，根据实际情况来制定此大小，如果不配置，则不会限制



上传事件说明

| `dndAccept`        | `items`{DataTransferItemList}DataTransferItem | 阻止此事件可以拒绝某些类型的文件拖入进来。目前只有 chrome 提供这样的 API，且只能通过 mime-type 验证。 |
| ------------------ | ---------------------------------------- | ---------------------------------------- |
| `beforeFileQueued` | `file` {File}File对象                      | 当文件被加入队列之前触发，此事件的handler返回值为`false`，则此文件不会被添加进入队列。 |
| `fileQueued`       | `file` {File}File对象                      | 当文件被加入队列以后触发。                            |
| `filesQueued`      | `files` {File}数组，内容为原始File(lib/File）对象。  | 当一批文件添加进队列以后触发。                          |
| `fileDequeued`     | `file` {File}File对象                      | 当文件被移除队列后触发。                             |
| `reset`            |                                          | 当 uploader 被重置的时候触发。                     |
| `startUpload`      |                                          | 当开始上传流程时触发。                              |
| `stopUpload`       |                                          | 当开始上传流程暂停时触发。                            |
| `uploadFinished`   |                                          | 当所有文件上传结束时触发。                            |
| `uploadStart`      | `file` {File}File对象                      | 某个文件开始上传前触发，一个文件只会触发一次。                  |
| `uploadBeforeSend` | `object` {Object}`data` {Object}默认的上传参数，可以扩展此对象来控制上传参数。`headers` {Object}可以扩展此对象来控制上传头部。 | 当某个文件的分块在发送前触发，主要用来询问是否要添加附带参数，大文件在开起分片上传的前提下此事件可能会触发多次。 |
| `uploadAccept`     | `object` {Object}`ret` {Object}服务端的返回数据，json格式，如果服务端不是json格式，从ret._raw中取数据，自行解析。 | 当某个文件上传到服务端响应后，会派送此事件来询问服务端响应是否有效。如果此事件handler返回值为`false`, 则此文件将派送`server`类型的`uploadError`事件。 |
| `uploadProgress`   | `file` {File}File对象`percentage` {Number}上传进度 | 上传过程中触发，携带上传进度。                          |
| `uploadError`      | `file` {File}File对象`reason` {String}出错的code | 当文件上传出错时触发。                              |
| `uploadSuccess`    | `file` {File}File对象`response` {Object}服务端返回的数据 | 当文件上传成功时触发。                              |
| `uploadComplete`   | `file` {File} [可选]File对象                 | 不管成功或者失败，文件上传完成时触发。                      |
| `error`            | `type` {String}错误类型。                     | 当validate不通过时，会以派送错误事件的形式通知调用者。通过`upload.on('error', handler)`可以捕获到此类错误，目前有以下错误会在特定的情况下派送错来。`Q_EXCEED_NUM_LIMIT` 在设置了`fileNumLimit`且尝试给`uploader`添加的文件数量超出这个值时派送。`Q_EXCEED_SIZE_LIMIT` 在设置了`Q_EXCEED_SIZE_LIMIT`且尝试给`uploader`添加的文件总大小超出这个值时派送。`Q_TYPE_DENIED` 当文件类型不满足时触发。。 |



### Java后端实现分片合成

方法一：

~~~java
public void uploadFileRandomAccessFile(MultipartFileParam param) throws IOException {
        RandomAccessFile accessFile = null;

        String fileName;
        File tmpFile;
        try {
            fileName = param.getName();
            String tmpFileName = fileName + "_tmp";
            File dir = new File(rootPath, DateUtils.format(new Date()));
            tmpFile = new File(dir, tmpFileName);
            if (!dir.exists()) {
                dir.mkdirs();
            }
            long offset = chunkSize * param.getChunk();
            accessFile = new RandomAccessFile(tmpFile, "rw");
            // 定义分片的偏移量
            accessFile.seek(offset);
            // 将分片数据写入到文件中
            accessFile.write(param.getFile().getBytes());
        } finally {
            if (accessFile != null) {
                accessFile.close();
            }
        }
        // 检测是否已经传输完成，并且设置传输进度
        Boolean isOk = checkAndSetProgress(param);
        if(isOk) {
            boolean result = renameFile(tmpFile, fileName);
            log.info("重命名结果：{}", result);
            log.info("文件{}上传完成", fileName);
        }

    }
~~~



方法二：

```java
public void uploadFileByMappedByteBuffer(MultipartFileParam param) throws IOException {
    FileChannel fileChannel = null;

    String fileName;
    File tmpFile;
    try {
        fileName = param.getName();
        String tmpFileName = fileName + "_tmp";
        File dir = new File(rootPath, DateUtils.format(new Date()));
        tmpFile = new File(dir, tmpFileName);
        if (!dir.exists()) {
            dir.mkdirs();
        }

        long offset = chunkSize * param.getChunk();
        RandomAccessFile accessFile = new RandomAccessFile(tmpFile, "rw");
        fileChannel = accessFile.getChannel();
        byte[] fileData = param.getFile().getBytes();
        MappedByteBuffer mappedByteBuffer = fileChannel.map(FileChannel.MapMode.READ_WRITE, offset, fileData.length);
        mappedByteBuffer.put(fileData);
        // 释放
        FileMD5Util.freedMappedByteBuffer(mappedByteBuffer);
        fileChannel.close();
    } finally {
        if (fileChannel != null && fileChannel.isOpen()) {
            fileChannel.close();
        }
    }
    // 检测是否已经传输完成，并且设置传输进度
    Boolean isOk = checkAndSetProgress(param);
    if(isOk) {
        boolean result = renameFile(tmpFile, fileName);
        log.info("重命名结果：{}", result);
        log.info("文件{}上传完成", fileName);
    }

}
```



> 方法二相对于方法一传输的速度更快一些，推荐使用方法二



### 秒传功能

​	使用过百度云的都知道，上传一个文件时，如果服务端已经存在当前文件，那么我们可以将文件秒传到我们的网盘中，其中的原理就是计算文件的md5值，然后和服务端已经存在的数据做比对，如果已经存在，那么文件就秒传到服务器上了，本项目中采用了同样的方法，通过计算文件的md5值，然后调用服务端的接口校验md5值对应的文件是否存在，如果已经存在，则直接显示上传成功，前端文件的md5值计算也采用了**webuploader**自带的方法，此方法还提供了实时进度的生成，具体使用为：

~~~javascript
 uploader.md5File( file )
        // 及时显示进度
        .progress(function(percentage) {
            console.log('Percentage:', percentage);
        })
        // 完成
        .then(function(val) {
            console.log('md5 result:', val);
        });
~~~



### 断点续传功能

​	目前我们使用的网盘，都有断点续传的功能，就是在文件由于人为因素或者网络因素导致文件上传到一半的时候中断了，在恢复后，再次上传文件时，接着上次上传的进度继续上传，而不是从头开始。

​	断点续传的实现是基于分块来实现的，服务端会把文件每一块上传的状态记录下来，出现断点续传的时候，会到服务端去验证，获取到还没有上传到服务端的分块编号，前端根据返回的编号重新上传未上传的部分。

前端代码的实现为：

~~~javascript
 $.post("file/checkFileMd5", {uid : file.uid, md5 : file.md5},
                function (data) {
                    console.log(data.result);
                    var status = data.result.code;
                    task.resolve();
                    if (status == 101) {
                        // 文件不存在，则进行正常流程
                    } else if(status == 100) {
                        // 文件已经存在，则直接跳过
                        uploader.skipFile(file);
                        file.pass = true;
                    } else if (status == 102) {
                        // 已经上传一部分，继续上传
                        file.missChunks = data.data;
                    }
            });
~~~



后端校验的代码实现：

~~~java
@RequestMapping(value = "/checkFileMd5", method = RequestMethod.POST)
    public ResultVo checkFileMd5(String md5) throws IOException {
        Object o = CacheUtils.cache.hget(Constants.FILE_UPLOAD_STATUS, md5);
        // 文件没有上传
        if (o == null) {
            return new ResultVo(Result.NO_HAVE);
        }
         // 文件已经上传
        boolean fileStatus = Boolean.parseBoolean(o.toString());
        if(fileStatus) {
            return new ResultVo(Result.IS_HAVE);
        }
        // 文件上传了一部分
        else {
            String value = (String)CacheUtils.cache.get(Constants.FILE_MD5_KEY + md5);
            // 在redis中没有存储进度文件的地址，则重新上传
            if (StringUtils.isEmpty(value)) {
                return new ResultVo(Result.NO_HAVE);
            }
            File confFile = new File(value);
            if(!confFile.exists()) {
                return new ResultVo(Result.NO_HAVE);
            }
            byte[] bytes = FileUtils.readFileToByteArray(confFile);
            List<String> missChunkList = new LinkedList<>();
            for (int i = 0; i < bytes.length; i++) {
                if (Byte.MAX_VALUE != bytes[i]) {
                    missChunkList.add(i + "");
                }
            }
            return new ResultVo(Result.ING_HAVE, missChunkList);
        }
    }
~~~

### 缓存的实现

服务端记录文件的上传状态是通过缓存的形式实现的，目前服务端实现了两种缓存的方法

方法一：基于map实现的简单缓存方式

~~~java
@Component
public class MapCache<T> implements ICache{

    /**
     * 默认存储1024个缓存
     */
    private static final int DEFAULT_CACHES = 1024;

    private static final int MAX_CACHE = 1024 * 20;

    /**
     * 缓存容器
     */
    private Map<String, CacheObject> cachePool;

    public MapCache() {
        this(DEFAULT_CACHES);
    }

    public MapCache(int cacheCount) {
        cachePool = new ConcurrentHashMap<>(cacheCount);
    }

    /**
     * 读取一个缓存
     *
     * @param key
     * @param key 缓存key
     * @return
     */
    @Override
    public T get(String key) {
        CacheObject cacheObject = cachePool.get(key);
        if (null != cacheObject) {
            long cur = System.currentTimeMillis() / 1000;
            //未过期直接返回
            if (cacheObject.getExpired() <= 0 || cacheObject.getExpired() > cur) {
                Object result = cacheObject.getValue();
                return (T) result;
            }
            //已过期直接删除
            if (cur > cacheObject.getExpired()) {
                cachePool.remove(key);
            }
        }
        return null;
    }

    /**
     * 读取一个hash类型缓存
     *
     * @param key   缓存key
     * @param field 缓存field
     * @return
     */
    @Override
    public T hget(String key, String field) {
        key = key + ":" + field;
        return this.get(key);
    }

    /**
     * 设置一个缓存
     *
     * @param key   缓存key
     * @param value 缓存value
     */
    @Override
    public void set(String key, Object value) {
        this.set(key, value, -1);
    }

    /**
     * 设置一个缓存并带过期时间
     *
     * @param key     缓存key
     * @param value   缓存value
     * @param expired 过期时间，单位为秒
     */
    @Override
    public void set(String key, Object value, long expired) {
        expired = expired > 0 ? System.currentTimeMillis() / 1000 + expired : expired;
        //cachePool大于MAX_CACHE时，强制清空缓存池，这个操作有些粗暴会导致误删问题，后期考虑用redis替代MapCache优化
        if (cachePool.size() > MAX_CACHE) {
            cachePool.clear();
        }
        CacheObject cacheObject = new CacheObject(key, value, expired);
        cachePool.put(key, cacheObject);
    }

    /**
     * 设置一个hash缓存
     *
     * @param key   缓存key
     * @param field 缓存field
     * @param value 缓存value
     */
    @Override
    public void hset(String key, String field, Object value) {
        this.hset(key, field, value, -1);
    }

    /**
     * 设置一个hash缓存并带过期时间
     *
     * @param key     缓存key
     * @param field   缓存field
     * @param value   缓存value
     * @param expired 过期时间，单位为秒
     */
    @Override
    public void hset(String key, String field, Object value, long expired) {
        key = key + ":" + field;
        expired = expired > 0 ? System.currentTimeMillis() / 1000 + expired : expired;
        CacheObject cacheObject = new CacheObject(key, value, expired);
        cachePool.put(key, cacheObject);
    }

    /**
     * 根据key删除缓存
     *
     * @param key 缓存key
     */
    @Override
    public void del(String key) {
        cachePool.remove(key);
    }

    /**
     * 缓存是否存在
     * @param key
     * @return
     */
    @Override
    public boolean hasKey(String key) {
        return cachePool.containsKey(key);
    }

    /**
     * hash缓存是否存在
     * @param key
     * @param field
     * @return
     */
    @Override
    public boolean hHasKey(String key, String field) {
        key = key + ":" + field;
        return cachePool.containsKey(key);
    }

    /**
     * 根据key和field删除缓存
     *
     * @param key   缓存key
     * @param field 缓存field
     */
    @Override
    public void hdel(String key, String field) {
        key = key + ":" + field;
        this.del(key);
    }

    /**
     * 清空缓存
     */
    @Override
    public void clean() {
        cachePool.clear();
    }

    static class CacheObject {
        private String key;
        private Object value;
        private long expired;

        public CacheObject(String key, Object value, long expired) {
            this.key = key;
            this.value = value;
            this.expired = expired;
        }

        public String getKey() {
            return key;
        }

        public Object getValue() {
            return value;
        }

        public long getExpired() {
            return expired;
        }
    }
}
~~~



方法二：基于redis实现的kv缓存形式

~~~java
@Component
public class RedisCache implements ICache{

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 读取一个缓存
     * @param key 缓存key
     * @return
     */
    @Override
    public String get(String key) {
        return stringRedisTemplate.opsForValue().get(key);
    }

    /**
     * 读取一个hash类型缓存
     *
     * @param key   缓存key
     * @param field 缓存field
     * @return
     */
    @Override
    public Object hget(String key, String field) {
        return stringRedisTemplate.opsForHash().get(key, field);
    }

    /**
     * 设置一个缓存
     *
     * @param key   缓存key
     * @param value 缓存value
     */
    @Override
    public void set(String key, Object value) {
        stringRedisTemplate.opsForValue().set(key, JSONObject.toJSONString(value));
    }

    /**
     * 设置一个缓存并带过期时间
     *
     * @param key     缓存key
     * @param value   缓存value
     * @param expired 过期时间，单位为秒
     */
    @Override
    public void set(String key, Object value, long expired) {
        stringRedisTemplate.opsForValue().set(key, JSONObject.toJSONString(value), expired);
    }

    /**
     * 设置一个hash缓存
     *
     * @param key   缓存key
     * @param field 缓存field
     * @param value 缓存value
     */
    @Override
    public void hset(String key, String field, Object value) {
        stringRedisTemplate.opsForHash().put(key, field, value);
    }

    /**
     * 设置一个hash缓存并带过期时间
     *
     * @param key     缓存key
     * @param field   缓存field
     * @param value   缓存value
     * @param expired 过期时间，单位为秒
     */
    @Override
    public void hset(String key, String field, Object value, long expired) {
        stringRedisTemplate.opsForHash().put(key, field, value);
        stringRedisTemplate.expire(key, expired, TimeUnit.SECONDS);
    }

    /**
     * 根据key删除缓存
     *
     * @param key 缓存key
     */
    @Override
    public void del(String key) {
        stringRedisTemplate.delete(key);
    }

    /**
     * 是否已存在key
     * @param key
     */
    @Override
    public boolean hasKey(String key) {
        return stringRedisTemplate.hasKey(key);
    }

    /**
     * 是否已经存在hash key
     * @param key
     * @param field
     * @return
     */
    @Override
    public boolean hHasKey(String key, String field) {
        return stringRedisTemplate.opsForHash().hasKey(key, field);
    }

    /**
     * 根据key和field删除缓存
     *
     * @param key   缓存key
     * @param field 缓存field
     */
    @Override
    public void hdel(String key, String field) {
        stringRedisTemplate.opsForHash().delete(key, field);
    }

    /**
     * 清空缓存
     */
    @Override
    public void clean() {
        stringRedisTemplate.delete(Constants.FILE_MD5_KEY);
        stringRedisTemplate.delete(Constants.FILE_UPLOAD_STATUS);
    }

}
~~~



### 文件上传示例

目前实现的功能支持文件秒传，断点续传，上传暂停等功能。![springboot-06](https://ldmyown.github.io/assets/images/2019/springboot/springboot-06.png)



## 文件下载

​	文件下载采用的是netty服务器的方式实现的，通过浏览器访问可以查看服务端的文件列表，可以下载目标文件到本地。

### 创建netty服务

~~~java
		EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new FileServerInitializer(webroot));
            ChannelFuture future = bootstrap.bind(port).sync();
           logger.info("服务器已启动>>网址:" + "0.0.0.0:" + port);
            future.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
~~~



### 设置服务的参数

~~~java
// HttpServerCodec：将请求和应答消息解码为HTTP消息
        // socketChannel.pipeline().addLast("http-codec",new HttpServerCodec());
        socketChannel.pipeline().addLast("http-decoder", new HttpRequestDecoder());
        socketChannel.pipeline().addLast("http-encoder", new HttpResponseEncoder());

        // HttpObjectAggregator：将HTTP消息的多个部分合成一条完整的HTTP消息
        socketChannel.pipeline().addLast("http-aggregator", new HttpObjectAggregator(65536));
        // ChunkedWriteHandler：向客户端发送HTML5文件
        socketChannel.pipeline().addLast("http-chunked", new ChunkedWriteHandler());
        // 进行设置心跳检测
        socketChannel.pipeline().addLast(new IdleStateHandler(60, 30, 60 * 30, TimeUnit.SECONDS));
        // 配置通道处理  来进行业务处理
        socketChannel.pipeline().addLast("fileServerHandler", new HttpFileServerHandler(url));
~~~



### 处理文件

这里只贴下载文件的方法，其他的方法实现请看示例中的源码

~~~java
  private void solveFile(ChannelHandlerContext ctx, HttpRequest msg, File file) {
        try {
            RandomAccessFile randomAccessFile = new RandomAccessFile(file, "r");
            Long fileLength = randomAccessFile.length();
            HttpResponse httpResponse = new DefaultHttpResponse(HTTP_1_1, OK);
            setContentLength(httpResponse, fileLength);
            setContentTypeHeader(httpResponse, file);

            if (isKeepAlive(msg)) {
                httpResponse.headers().set(CONNECTION, KEEP_ALIVE);
            }
            ctx.writeAndFlush(httpResponse);
            ChannelFuture sendFileFuture = ctx.write(
                    new ChunkedFile(randomAccessFile, 0, fileLength, 8192), ctx.newProgressivePromise());

            sendFileFuture.addListener(new ChannelProgressiveFutureListener() {
                @Override
                public void operationProgressed(ChannelProgressiveFuture future, long progress, long total) {
                    if (total < 0) {
                        System.err.println("progress:" + progress);
                    } else {
                        System.err.println("progress:" + progress + "/" + total);
                    }
                }
                @Override
                public void operationComplete(ChannelProgressiveFuture future) {
                    System.err.println("complete");
                }
            });

            ChannelFuture lastChannelFuture = ctx.writeAndFlush(LastHttpContent.EMPTY_LAST_CONTENT);
            if (!isKeepAlive(msg)) {
                lastChannelFuture.addListener(ChannelFutureListener.CLOSE);

            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            sendError(ctx, HttpResponseStatus.NOT_FOUND);
            return;
        } catch (IOException e) {
            e.printStackTrace();
            sendError(ctx, HttpResponseStatus.NOT_FOUND);
        }
    }
~~~



### 下载实例

点击对应的文件即可将文件下载到本地。 ![springboot-07](https://ldmyown.github.io/assets/images/2019/springboot/springboot-07.png)

## ## 总结

此项目为一个简易的文件上传和下载的服务，实现了文件的分块上传与下载，满足了基于web的大文件的上传操作，后续会慢慢加入更多的功能。

 


详细的代码可以查看

**[示例代码-github](**https://github.com/ldmyown/springboot-learning**)**

-------------

**作者：telAngel**  
**出处：[https://ldmyown.github.io](https://ldmyown.github.io)**      
**版权所有，欢迎保留原文链接进行转载：)**