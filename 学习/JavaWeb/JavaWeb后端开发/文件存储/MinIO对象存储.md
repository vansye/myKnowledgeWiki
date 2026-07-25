# MinIO 对象存储

## 目录

- [什么是 MinIO](#什么是-minio)
- [MinIO vs 阿里云 OSS](#minio-vs-阿里云-oss)
- [为什么选择 MinIO](#为什么选择-minio)
- [MinIO 的核心概念](#minio-的核心概念)
- [MinIO 安装与部署](#minio-安装与部署)
- [MinIO 控制台使用](#minio-控制台使用)
- [MinIO 的基本使用](#minio-的基本使用)
- [文件上传流程](#文件上传流程)
- [在 SpringBoot 中集成 MinIO](#在-springboot-中集成-minio)
- [MinIO 的访问方式](#minio-的访问方式)
- [MinIO 的高级特性](#minio-的高级特性)
- [MinIO 的最佳实践](#minio-的最佳实践)
- [常见问题与解决方案](#常见问题与解决方案)
- [小结](#小结)

---

## 什么是 MinIO

**MinIO** 是一个**开源的对象存储服务器**，兼容 Amazon S3 API，可以部署在自己的服务器上。

它和阿里云 OSS 做的是同一件事——存储图片、视频、文件等非结构化数据，但 MinIO 是**自建**的，数据完全掌握在自己手中。

### 生活中的类比

| 概念 | 类比 |
|------|------|
| **阿里云 OSS** | 租一个公共仓库（数据在阿里云的服务器上，按量付费） |
| **MinIO** | 自己建一个仓库（数据在自己的服务器上，一次性投入） |

两者功能类似，区别在于**数据存在哪里、谁来管理**。

---

## MinIO vs 阿里云 OSS

| 维度 | MinIO | 阿里云 OSS |
|------|-------|-----------|
| **部署方式** | 自建，部署在自己的服务器 | 云服务，阿里云托管 |
| **数据存储** | 自己的服务器/硬盘 | 阿里云的服务器 |
| **费用** | 服务器成本（一次性） | 按存储量 + 流量付费（持续性） |
| **数据安全** | 数据完全自控 | 数据在阿里云，依赖云厂商安全 |
| **S3 兼容** | 原生兼容 S3 API | 兼容 S3 API |
| **适用场景** | 内网、私有化部署、数据敏感 | 公网应用、不想运维存储 |
| **运维成本** | 需要自己维护 | 阿里云维护，零运维 |
| **扩展性** | 受服务器限制 | 几乎无限 |

### 选择建议

| 场景 | 推荐 |
|------|------|
| 公司内部系统、内网应用 | MinIO |
| 数据敏感、不能上云 | MinIO |
| 不想花钱租云存储 | MinIO |
| 公网面向用户的应用 | 阿里云 OSS |
| 不想自己运维 | 阿里云 OSS |
| 需要全球 CDN 加速 | 阿里云 OSS |

---

## 为什么选择 MinIO

### MinIO 的优势

| 优势 | 说明 |
|------|------|
| **开源免费** | Apache 2.0 协议，可以免费使用 |
| **S3 兼容** | 兼容 Amazon S3 API，代码可以无缝切换 |
| **部署简单** | 一个二进制文件就能跑起来 |
| **高性能** | 号称世界上最快的对象存储服务器 |
| **数据自控** | 数据存在自己的服务器上，安全可控 |
| **轻量级** | 资源占用小，普通服务器就能跑 |
| **支持分布式** | 可以集群部署，实现高可用 |

### MinIO 的劣势

| 劣势 | 说明 |
|------|------|
| **需要自己运维** | 服务器挂了要自己处理 |
| **没有 CDN** | 需要自己配置 CDN 加速 |
| **扩展受限** | 受服务器硬盘和带宽限制 |
| **没有现成的管理控制台** | 虽然有 Web UI，但功能不如云厂商丰富 |

---

## MinIO 的核心概念

| 概念 | 说明 | 对应阿里云 OSS |
|------|------|---------------|
| **Bucket（桶）** | 存储文件的容器 | Bucket（存储空间） |
| **Object（对象）** | 存储的文件 | Object（对象） |
| **Object Name** | 文件的唯一标识（路径 + 文件名） | Key |
| **Endpoint** | MinIO 服务的访问地址 | Endpoint |
| **Access Key** | 访问凭证的用户名 | AccessKey ID |
| **Secret Key** | 访问凭证的密码 | AccessKey Secret |
| **Region** | 区域（MinIO 中通常不需要） | Region（地域） |

### 关系图

```
MinIO 服务器（http://localhost:9000）
    │
    ├── Bucket: images（图片桶）
    │       │
    │       ├── avatar/001.jpg（Object）
    │       ├── product/001.jpg（Object）
    │       ── banner/home.jpg（Object）
    │
    ├── Bucket: files（文件桶）
    │       │
    │       ── docs/report.pdf（Object）
    │
    └── Bucket: videos（视频桶）
            │
            └── tutorial/01.mp4（Object）
```

---

## MinIO 安装与部署

### 方式一：Docker 部署（推荐）

```bash
# 拉取镜像
docker pull minio/minio

# 运行容器
docker run -d \
  --name minio \
  -p 9000:9000 \
  -p 9001:9001 \
  -e "MINIO_ROOT_USER=minioadmin" \
  -e "MINIO_ROOT_PASSWORD=minioadmin123" \
  -v /data/minio:/data \
  minio/minio server /data --console-address ":9001"
```

**端口说明：**

| 端口 | 用途 |
|------|------|
| 9000 | API 端口（程序访问用） |
| 9001 | 控制台端口（浏览器管理用） |

### 方式二：直接运行

```bash
# 下载 MinIO（Linux）
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio

# 设置账号密码并启动
export MINIO_ROOT_USER=minioadmin
export MINIO_ROOT_PASSWORD=minioadmin123
./minio server /data --console-address ":9001"
```

### 方式三：Windows 运行

```powershell
# 下载 minio.exe
# 设置环境变量
$env:MINIO_ROOT_USER = "minioadmin"
$env:MINIO_ROOT_PASSWORD = "minioadmin123"

# 启动
.\minio.exe server C:\data --console-address ":9001"
```

### 启动后访问

- **API 地址：** `http://localhost:9000`
- **控制台地址：** `http://localhost:9001`
- **默认账号：** `minioadmin`
- **默认密码：** `minioadmin123`

---

## MinIO 控制台使用

### 登录控制台

浏览器访问 `http://localhost:9001`，输入账号密码登录。

### 创建 Bucket

1. 点击左侧 **Buckets**
2. 点击 **Create Bucket**
3. 输入 Bucket 名称（如 `images`）
4. 点击 **Create Bucket**

### 设置访问策略

Bucket 创建后默认是私有的，需要设置访问策略才能公开访问：

1. 进入 Bucket → **Access Policy**
2. 选择策略类型：

| 策略 | 说明 |
|------|------|
| **Private** | 私有，需要认证才能访问 |
| **Public** | 公开，任何人可以读写 |
| **Custom** | 自定义策略 |

3. 选择 **Public** 或配置自定义策略

### 上传文件

1. 进入 Bucket
2. 点击 **Upload** → **Upload file**
3. 选择文件上传
4. 上传后可以获取文件的访问 URL

### 管理 Access Key

1. 点击左侧 **Access Keys**
2. 点击 **Create Access Key**
3. 生成新的 Access Key 和 Secret Key
4. 用于程序访问 MinIO

---

## MinIO 的基本使用

### 1. 引入依赖

```xml
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.5.7</version>
</dependency>
```

### 2. 创建 MinIO 客户端

```java
// 配置信息
String endpoint = "http://localhost:9000";
String accessKey = "minioadmin";
String secretKey = "minioadmin123";

// 创建 MinIO 客户端
MinioClient minioClient = MinioClient.builder()
        .endpoint(endpoint)
        .credentials(accessKey, secretKey)
        .build();
```

### 3. 创建 Bucket

```java
// 检查 Bucket 是否存在，不存在则创建
boolean exists = minioClient.bucketExists(
    BucketExistsArgs.builder().bucket("images").build()
);
if (!exists) {
    minioClient.makeBucket(
        MakeBucketArgs.builder().bucket("images").build()
    );
}
```

### 4. 上传文件

```java
// 上传文件到 MinIO
public String uploadFile(MultipartFile file) throws Exception {
    // 生成唯一文件名
    String originalFilename = file.getOriginalFilename();
    String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
    String fileName = UUID.randomUUID().toString() + extension;

    // 构建 Object Name（文件路径）
    String objectName = "images/" + fileName;

    // 上传
    minioClient.putObject(
        PutObjectArgs.builder()
            .bucket("images")
            .object(objectName)
            .stream(file.getInputStream(), file.getSize(), -1)
            .contentType(file.getContentType())
            .build()
    );

    // 返回文件的访问 URL
    return endpoint + "/images/" + objectName;
}
```

### 5. 下载文件

```java
// 下载文件
public InputStream downloadFile(String bucketName, String objectName) throws Exception {
    return minioClient.getObject(
        GetObjectArgs.builder()
            .bucket(bucketName)
            .object(objectName)
            .build()
    );
}
```

### 6. 删除文件

```java
// 删除文件
public void deleteFile(String bucketName, String objectName) throws Exception {
    minioClient.removeObject(
        RemoveObjectArgs.builder()
            .bucket(bucketName)
            .object(objectName)
            .build()
    );
}
```

### 7. 获取文件信息

```java
// 获取文件信息
public StatObjectResponse getFileInfo(String bucketName, String objectName) throws Exception {
    return minioClient.statObject(
        StatObjectArgs.builder()
            .bucket(bucketName)
            .object(objectName)
            .build()
    );
}
```

---

## 文件上传流程

### 完整流程

```
前端页面                    Spring Boot 后端                    MinIO 服务器
   │                              │                              │
   │  1. 选择文件                  │                              │
   │  2. 提交表单                  │                              │
   ├─────────────────────────────►│                              │
   │  POST /upload                 │                              │
   │  (multipart/form-data)        │                              │
   │                              │  3. 接收文件                   │
   │                              │  4. 生成唯一文件名             │
   │                              │  5. 调用 MinIO SDK 上传        │
   │                              ├─────────────────────────────►│
   │                              │  PUT /images/xxx.jpg          │
   │                              │                              │
   │                              │  6. 上传成功，返回 URL          │
   │                              │─────────────────────────────┤
   │                              │                              │
   │  7. 返回文件 URL              │                              │
   │◄─────────────────────────────┤                              │
   │  {"url": "http://..."}        │                              │
   │                              │                              │
   │  8. 用 URL 展示图片            │                              │
   │  <img src="http://...">       │                              │
```

### 要点

- 文件和阿里云 OSS 的上传流程完全一样
- 只是把 OSS SDK 换成了 MinIO SDK
- 因为 MinIO 兼容 S3 API，代码结构几乎相同

---

## 在 SpringBoot 中集成 MinIO

### 1. 配置文件

```yaml
# application.yml
minio:
  endpoint: http://localhost:9000
  access-key: minioadmin
  secret-key: minioadmin123
  bucket-name: images
```

### 2. 配置类

```java
@Configuration
public class MinioConfig {

    @Value("${minio.endpoint}")
    private String endpoint;

    @Value("${minio.access-key}")
    private String accessKey;

    @Value("${minio.secret-key}")
    private String secretKey;

    @Bean
    public MinioClient minioClient() {
        return MinioClient.builder()
                .endpoint(endpoint)
                .credentials(accessKey, secretKey)
                .build();
    }
}
```

### 3. Service 层

```java
@Service
public class MinioService {

    @Autowired
    private MinioClient minioClient;

    @Value("${minio.bucket-name}")
    private String bucketName;

    @Value("${minio.endpoint}")
    private String endpoint;

    /**
     * 上传文件
     */
    public String upload(MultipartFile file) throws Exception {
        // 确保 Bucket 存在
        ensureBucketExists();

        // 生成唯一文件名
        String originalFilename = file.getOriginalFilename();
        String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
        String fileName = UUID.randomUUID().toString() + extension;

        // 按日期分目录
        String datePath = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));
        String objectName = datePath + "/" + fileName;

        // 上传
        minioClient.putObject(
            PutObjectArgs.builder()
                .bucket(bucketName)
                .object(objectName)
                .stream(file.getInputStream(), file.getSize(), -1)
                .contentType(file.getContentType())
                .build()
        );

        // 返回访问 URL
        return endpoint + "/" + bucketName + "/" + objectName;
    }

    /**
     * 删除文件
     */
    public void delete(String objectName) throws Exception {
        minioClient.removeObject(
            RemoveObjectArgs.builder()
                .bucket(bucketName)
                .object(objectName)
                .build()
        );
    }

    /**
     * 确保 Bucket 存在
     */
    private void ensureBucketExists() throws Exception {
        boolean exists = minioClient.bucketExists(
            BucketExistsArgs.builder().bucket(bucketName).build()
        );
        if (!exists) {
            minioClient.makeBucket(
                MakeBucketArgs.builder().bucket(bucketName).build()
            );
        }
    }
}
```

### 4. Controller 层

```java
@RestController
@RequestMapping("/upload")
public class UploadController {

    @Autowired
    private MinioService minioService;

    @PostMapping
    public Result upload(@RequestParam("file") MultipartFile file) throws Exception {
        String url = minioService.upload(file);
        return Result.success(url);
    }

    @DeleteMapping
    public Result delete(@RequestParam("objectName") String objectName) throws Exception {
        minioService.delete(objectName);
        return Result.success();
    }
}
```

---

## MinIO 的访问方式

### 1. 公开访问（Public Bucket）

- Bucket 设置为 Public
- 任何人可以通过 URL 直接访问文件
- 适合：公开的图片、文档

```
http://localhost:9000/images/2026/07/22/abc123.jpg
```

### 2. 预签名 URL（私有文件）

- Bucket 设置为 Private
- 通过预签名 URL 临时授权访问
- 适合：需要权限控制的文件

```java
// 生成预签名 URL（有效期 1 小时）
String url = minioClient.getPresignedObjectUrl(
    GetPresignedObjectUrlArgs.builder()
        .bucket(bucketName)
        .object(objectName)
        .expiry(60 * 60)  // 秒
        .build()
);
// 返回类似：http://localhost:9000/images/xxx.jpg?X-Amz-Algorithm=...&X-Amz-Expires=3600
```

### 3. 通过 Nginx 反向代理（生产环境）

```nginx
server {
    listen 80;
    server_name minio.example.com;

    location / {
        proxy_pass http://localhost:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        client_max_body_size 100m;  # 限制上传文件大小
    }
}
```

---

## MinIO 的高级特性

### 1. 分布式部署

MinIO 支持多节点集群部署，实现高可用和水平扩展。

```bash
# 4 节点分布式部署示例
minio server http://node{1...4}/data{1...4}
```

**要点：**

- 至少 4 个节点（推荐）
- 数据自动分片存储
- 任意节点故障不影响服务

### 2. 生命周期管理

可以设置文件的生命周期规则，自动删除过期文件。

```java
// 设置 Bucket 的生命周期规则
LifecycleConfiguration config = new LifecycleConfiguration();
config.setRules(Arrays.asList(
    new LifecycleRule(
        Status.ENABLED,
        null,  // 过滤条件
        new Expiration((Date) null, 30, null),  // 30 天后过期
        null, null, null, null, "rule1"
    )
));

minioClient.setBucketLifecycle(
    SetBucketLifecycleArgs.builder()
        .bucket(bucketName)
        .config(config)
        .build()
);
```

### 3. 事件通知

可以配置 Bucket 事件通知，当文件上传/删除时触发通知。

支持的通知目标：

| 目标 | 说明 |
|------|------|
| **Webhook** | 发送 HTTP 请求到指定 URL |
| **MySQL** | 将事件记录到 MySQL |
| **Redis** | 将事件发布到 Redis |
| **Kafka** | 将事件发送到 Kafka |

### 4. 分片上传（大文件）

对于大文件，可以使用分片上传，将文件分成多个部分并行上传。

```java
// 分片上传（MinIO SDK 自动处理）
minioClient.uploadObject(
    UploadObjectArgs.builder()
        .bucket(bucketName)
        .object("large-file.zip")
        .filename("/path/to/large-file.zip")
        .partSize(10 * 1024 * 1024)  // 每片 10MB
        .build()
);
```

---

## MinIO 的最佳实践

### 1. 文件名唯一化

```java
// 用 UUID + 原始扩展名生成唯一文件名
String fileName = UUID.randomUUID().toString() + extension;
```

**原因：** 避免不同用户上传同名文件导致覆盖。

### 2. 按日期分目录

```java
String datePath = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));
String objectName = datePath + "/" + fileName;
```

**好处：** 文件按日期组织，便于管理和清理。

### 3. 限制文件大小和类型

```java
@PostMapping("/upload")
public Result upload(@RequestParam("file") MultipartFile file) {
    // 限制文件大小（5MB）
    if (file.getSize() > 5 * 1024 * 1024) {
        return Result.error("文件大小不能超过 5MB");
    }

    // 限制文件类型
    String contentType = file.getContentType();
    if (!Arrays.asList("image/jpeg", "image/png", "image/gif").contains(contentType)) {
        return Result.error("只支持 JPG、PNG、GIF 格式");
    }

    // 上传...
}
```

### 4. 凭证安全

- **不要**把 Access Key 和 Secret Key 硬编码在代码中
- **不要**把凭证提交到 Git
- 用环境变量或配置中心管理

```yaml
# 用环境变量
minio:
  access-key: ${MINIO_ACCESS_KEY}
  secret-key: ${MINIO_SECRET_KEY}
```

### 5. 生产环境配置

| 配置项 | 建议 |
|--------|------|
| **修改默认密码** | 不要用 `minioadmin/minioadmin123` |
| **使用 HTTPS** | 生产环境一定要用 HTTPS |
| **配置 Nginx 反向代理** | 隐藏 MinIO 的真实端口 |
| **设置合理的 Bucket 策略** | 公开桶只放公开文件 |
| **定期备份数据** | MinIO 数据要定期备份 |
| **监控磁盘空间** | 磁盘满了会导致上传失败 |

---

## 常见问题与解决方案

### 问题 1：`Connection refused` 连接被拒绝

**原因：**

- MinIO 服务没有启动
- 端口配置错误
- 防火墙阻止了连接

**解决方案：**

```bash
# 检查 MinIO 是否在运行
docker ps | grep minio

# 检查端口是否被监听
netstat -ano | findstr 9000

# 检查防火墙
# Windows：关闭防火墙或添加规则
# Linux：firewall-cmd --add-port=9000/tcp --permanent
```

### 问题 2：`AccessDenied` 权限不足

**原因：**

- Access Key 或 Secret Key 错误
- Bucket 策略不允许访问

**解决方案：**

- 检查凭证是否正确
- 检查 Bucket 的访问策略设置

### 问题 3：上传后访问 404

**原因：**

- Bucket 是私有的，不能直接通过 URL 访问
- Object Name 不对

**解决方案：**

- 如果是公开文件，将 Bucket 设置为 Public
- 如果是私有文件，用预签名 URL 访问
- 检查上传时返回的 Object Name 是否正确

### 问题 4：跨域问题（CORS）

**原因：**

前端直接上传文件到 MinIO 时，浏览器的跨域限制。

**解决方案：**

在 MinIO 中配置 CORS 规则：

```java
// 通过 mc 命令行工具配置
mc cors set myminio/images cors.json

// cors.json 内容
[
    {
        "AllowedOrigins": ["*"],
        "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
        "AllowedHeaders": ["*"],
        "MaxAgeSeconds": 3600
    }
]
```

### 问题 5：大文件上传超时

**原因：**

- Nginx 的 `client_max_body_size` 限制
- MinIO 的超时设置

**解决方案：**

```nginx
# Nginx 配置
client_max_body_size 100m;  # 允许最大 100MB
proxy_read_timeout 300s;    # 读取超时 5 分钟
```

---

## 小结

### 核心要点

| 要点 | 说明 |
|------|------|
| **MinIO** | 开源的对象存储服务器，兼容 S3 API |
| **vs 阿里云 OSS** | MinIO 自建（数据自控），OSS 云托管（零运维） |
| **部署方式** | Docker 部署最简单，一个命令就能跑 |
| **核心概念** | Bucket（桶）、Object（对象）、Access Key（凭证） |
| **SDK 使用** | `MinioClient` 创建客户端，`putObject` 上传，`getObject` 下载 |
| **访问方式** | 公开访问（Public Bucket）、预签名 URL（私有文件） |
| **高级特性** | 分布式部署、生命周期管理、事件通知、分片上传 |

### MinIO vs 阿里云 OSS 速查

| 场景 | MinIO | 阿里云 OSS |
|------|-------|-----------|
| 内网/私有化 | ✅ 推荐 | ❌ |
| 数据敏感 | ✅ 推荐 |  |
| 公网应用 |  | ✅ 推荐 |
| 不想运维 | ❌ | ✅ 推荐 |
| 成本敏感 | ✅ 推荐 |  |
| 需要 CDN | ❌ | ✅ 推荐 |

### 口诀

**MinIO 开源自建库，S3 兼容好切换；  
Docker 一键就部署，Bucket Object 存文件；  
公开访问设 Public，私有文件预签名；  
内网应用数据控，MinIO 是首选。**

### 最容易混淆的三个点

- **MinIO vs 阿里云 OSS**：MinIO 是自建（数据在自己服务器），OSS 是云托管（数据在阿里云）
- **9000 端口 vs 9001 端口**：9000 是 API 端口（程序访问），9001 是控制台端口（浏览器管理）
- **Public Bucket vs 预签名 URL**：Public 是永久公开，预签名 URL 是临时授权访问

---

MinIO 的本质是：**在自己的服务器上搭建一个兼容 S3 API 的对象存储服务，实现数据自控、成本可控的文件存储方案。**

掌握 MinIO，你就能在内网环境或数据敏感场景中实现高效的文件存储和管理！
