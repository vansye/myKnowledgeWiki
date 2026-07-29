# 阿里云 OSS 对象存储

## 目录

- [什么是 OSS](#什么是-oss)
- [为什么需要 OSS](#为什么需要-oss)
- [OSS 的核心概念](#oss-的核心概念)
- [OSS 的基本使用](#oss-的基本使用)
- [文件上传流程](#文件上传流程)
- [在 SpringBoot 中集成 OSS](#在-springboot-中集成-oss)
- [OSS 的访问方式](#oss-的访问方式)
- [OSS 的最佳实践](#oss-的最佳实践)
- [常见问题与解决方案](#常见问题与解决方案)
- [小结](#小结)

---

## 什么是 OSS

**OSS（Object Storage Service，对象存储服务）** 是阿里云提供的一种海量、安全、低成本的云存储服务。自托管方案见 MinIO对象存储。

它用来存储**非结构化数据**，如图片、视频、音频、文档等文件。

### 生活中的类比

把 OSS 想象成**云端的无限硬盘**：

- 传统方式：文件存在应用服务器的硬盘上
  - 硬盘空间有限
  - 服务器宕机文件就没了
  - 用户访问慢（受服务器带宽限制）
- OSS 方式：文件存在阿里云的存储集群上
  - 空间几乎无限
  - 数据多副本备份，不会丢失
  - 全球 CDN 加速，访问快

---

## 为什么需要 OSS

### 文件存在本地的问题

| 问题 | 说明 |
|------|------|
| **磁盘空间有限** | 服务器硬盘满了就无法继续上传 |
| **单点故障** | 服务器宕机，文件就访问不了 |
| **带宽瓶颈** | 大量用户同时下载文件，服务器带宽不够 |
| **扩展困难** | 多台服务器之间文件不共享 |
| **备份麻烦** | 需要自己实现文件备份机制 |

### OSS 的好处

| 好处 | 说明 |
|------|------|
| **海量存储** | 空间几乎无限，按需使用 |
| **高可用** | 数据多副本存储，99.99% 可用性 |
| **CDN 加速** | 全球节点加速，访问速度快 |
| **成本低** | 按使用量付费，比自建存储便宜 |
| **易于扩展** | 多台服务器共享同一个 OSS |

---

## OSS 的核心概念

| 概念 | 说明 |
|------|------|
| **Bucket（存储空间）** | 存储文件的容器，类似文件夹 |
| **Object（对象）** | 存储的文件，每个 Object 有唯一的 Key（文件名） |
| **Endpoint（访问域名）** | OSS 服务的访问地址 |
| **AccessKey** | 访问 OSS 的密钥（ID + Secret） |
| **Region（地域）** | OSS 服务所在的物理位置（如 cn-hangzhou） |

### 关系图

```
阿里云账号
    │
    ├── Bucket: my-bucket（存储空间）
    │       │
    │       ├── images/avatar/001.jpg（Object）
    │       ├── images/product/001.jpg（Object）
    │       ── docs/report.pdf（Object）
    │
    └── Bucket: another-bucket
            │
            └── ...
```

---

## OSS 的基本使用

### 1. 引入依赖

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.17.1</version>
</dependency>
```

### 2. 创建 OSS 客户端

```java
// 配置信息
String endpoint = "https://oss-cn-hangzhou.aliyuncs.com";
String accessKeyId = "your-access-key-id";
String accessKeySecret = "your-access-key-secret";
String bucketName = "your-bucket-name";

// 创建 OSS 客户端
OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
```

### 3. 上传文件

```java
// 上传文件到 OSS
public String uploadFile(MultipartFile file) throws IOException {
    // 生成唯一的文件名（避免重名覆盖）
    String originalFilename = file.getOriginalFilename();
    String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
    String fileName = UUID.randomUUID().toString() + extension;

    // 构建 Object 的 Key（文件路径）
    String key = "images/" + fileName;

    // 上传
    ossClient.putObject(bucketName, key, file.getInputStream());

    // 返回文件的访问 URL
    return "https://" + bucketName + "." + endpoint + "/" + key;
}
```

### 4. 下载文件

```java
// 下载文件
public void downloadFile(String key, String localPath) {
    ossClient.getObject(new GetObjectRequest(bucketName, key), new File(localPath));
}
```

### 5. 删除文件

```java
// 删除文件
public void deleteFile(String key) {
    ossClient.deleteObject(bucketName, key);
}
```

---

## 文件上传流程

### 完整流程

```
前端页面                    Spring Boot 后端                    阿里云 OSS
   │                              │                              │
   │  1. 选择文件                  │                              │
   │  2. 提交表单                  │                              │
   ├─────────────────────────────►│                              │
   │  POST /upload                 │                              │
   │  (multipart/form-data)        │                              │
   │                              │  3. 接收文件                   │
   │                              │  4. 生成唯一文件名             │
   │                              │  5. 调用 OSS SDK 上传          │
   │                              ├─────────────────────────────►│
   │                              │  PUT /images/xxx.jpg          │
   │                              │                              │
   │                              │  6. 上传成功，返回 URL          │
   │                              │◄─────────────────────────────┤
   │                              │                              │
   │  7. 返回文件 URL              │                              │
   │◄─────────────────────────────┤                              │
   │  {"url": "https://..."}       │                              │
   │                              │                              │
   │  8. 用 URL 展示图片            │                              │
   │  <img src="https://...">      │                              │
```

### 要点

- 文件不经过后端服务器存储，直接传到 OSS
- 后端只负责接收文件、调用 OSS SDK、返回 URL
- 前端用返回的 URL 展示或下载文件

---

## 在 SpringBoot 中集成 OSS

### 1. 配置文件

```yaml
# application.yml
aliyun:
  oss:
    endpoint: https://oss-cn-hangzhou.aliyuncs.com
    access-key-id: your-access-key-id
    access-key-secret: your-access-key-secret
    bucket-name: your-bucket-name
```

### 2. 配置类

```java
@Configuration
public class OssConfig {

    @Value("${aliyun.oss.endpoint}")
    private String endpoint;

    @Value("${aliyun.oss.access-key-id}")
    private String accessKeyId;

    @Value("${aliyun.oss.access-key-secret}")
    private String accessKeySecret;

    @Value("${aliyun.oss.bucket-name}")
    private String bucketName;

    @Bean
    public OSS ossClient() {
        return new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
    }
}
```

### 3. Service 层

```java
@Service
public class OssService {

    @Autowired
    private OSS ossClient;

    @Value("${aliyun.oss.bucket-name}")
    private String bucketName;

    @Value("${aliyun.oss.endpoint}")
    private String endpoint;

    /**
     * 上传文件
     */
    public String upload(MultipartFile file) throws IOException {
        // 生成唯一文件名
        String originalFilename = file.getOriginalFilename();
        String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
        String fileName = UUID.randomUUID().toString() + extension;

        // 按日期分目录
        String datePath = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));
        String key = "images/" + datePath + "/" + fileName;

        // 上传
        ossClient.putObject(bucketName, key, file.getInputStream());

        // 返回访问 URL
        return "https://" + bucketName + "." + endpoint.replace("https://", "") + "/" + key;
    }

    /**
     * 删除文件
     */
    public void delete(String fileUrl) {
        // 从 URL 中提取 Key
        String key = fileUrl.substring(fileUrl.indexOf(".com/") + 4);
        ossClient.deleteObject(bucketName, key);
    }
}
```

### 4. Controller 层

```java
@RestController
@RequestMapping("/upload")
public class UploadController {

    @Autowired
    private OssService ossService;

    @PostMapping
    public Result upload(@RequestParam("file") MultipartFile file) throws IOException {
        String url = ossService.upload(file);
        return Result.success(url);
    }
}
```

---

## OSS 的访问方式

### 1. 公共读（简单场景）

- Bucket 设置为公共读
- 任何人可以通过 URL 直接访问文件
- 适合：公开的图片、文档

```
https://your-bucket.oss-cn-hangzhou.aliyuncs.com/images/avatar.jpg
```

### 2. 签名 URL（私有文件）

- Bucket 设置为私有
- 通过签名 URL 临时授权访问
- 适合：需要权限控制的文件

```java
// 生成签名 URL（有效期 1 小时）
Date expiration = new Date(System.currentTimeMillis() + 3600 * 1000);
GeneratePresignedUrlRequest request = new GeneratePresignedUrlRequest(bucketName, key);
request.setExpiration(expiration);
String signedUrl = ossClient.generatePresignedUrl(request);
```

### 3. CDN 加速（大规模访问）

- 配置 CDN 域名指向 OSS
- 用户通过 CDN 域名访问，速度更快

```
https://cdn.your-domain.com/images/avatar.jpg
```

---

## OSS 的最佳实践

### 1. 文件名唯一化

```java
// 用 UUID + 原始扩展名生成唯一文件名
String fileName = UUID.randomUUID().toString() + extension;
```

**原因：** 避免不同用户上传同名文件导致覆盖。

### 2. 按日期分目录

```java
String datePath = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));
String key = "images/" + datePath + "/" + fileName;
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

### 4. AccessKey 安全

- **不要**把 AccessKey 硬编码在代码中
- **不要**把 AccessKey 提交到 Git
- 用环境变量或配置中心管理

```yaml
# 用环境变量
aliyun:
  oss:
    access-key-id: ${ALIYUN_ACCESS_KEY_ID}
    access-key-secret: ${ALIYUN_ACCESS_KEY_SECRET}
```

### 5. 上传后删除本地临时文件

```java
// 如果先保存到本地再上传，上传后要删除本地文件
File tempFile = File.createTempFile("upload", ".tmp");
file.transferTo(tempFile);
try {
    ossClient.putObject(bucketName, key, tempFile);
} finally {
    tempFile.delete();  // 删除临时文件
}
```

---

## 常见问题与解决方案

### 问题 1：`AccessDenied` 权限不足

**原因：**

- AccessKey 没有 OSS 访问权限
- Bucket 权限设置不对

**解决方案：**

- 检查 AccessKey 是否有 OSS 相关权限
- 检查 Bucket 的读写权限设置

### 问题 2：`ConnectionTimeout` 连接超时

**原因：**

- Endpoint 配置错误
- 网络问题

**解决方案：**

- 检查 Endpoint 是否正确（注意地域）
- 检查服务器是否能访问阿里云

### 问题 3：上传后访问 404

**原因：**

- Bucket 是私有的，不能直接通过 URL 访问
- 文件路径（Key）不对

**解决方案：**

- 如果是公开文件，将 Bucket 设置为公共读
- 如果是私有文件，用签名 URL 访问
- 检查上传时返回的 Key 是否正确

### 问题 4：跨域问题（CORS）

**原因：**

前端直接上传文件到 OSS 时，浏览器的跨域限制。

**解决方案：**

在 OSS Bucket 中配置 CORS 规则：

- 允许的来源：`*` 或指定域名
- 允许的方法：`GET`、`POST`、`PUT`
- 允许的头：`*`

---

## 小结

### 核心要点

| 要点 | 说明 |
|------|------|
| **OSS** | 阿里云对象存储服务，存储图片、视频等文件 |
| **Bucket** | 存储空间，类似文件夹 |
| **Object** | 存储的文件，有唯一的 Key |
| **上传流程** | 前端 → 后端 → OSS SDK → OSS → 返回 URL |
| **文件名唯一** | 用 UUID 生成唯一文件名，避免覆盖 |
| **按日期分目录** | 便于文件管理和清理 |
| **AccessKey 安全** | 不要硬编码，用环境变量管理 |

### 口诀

**OSS 存储云上面，海量空间成本低；  
Bucket 容器 Object 文件，Endpoint 来访问；  
上传生成唯一名，日期分目录好管理；  
AccessKey 要保密，环境变量最安全。**

### 最容易混淆的两个点

- **公共读 vs 私有**：公共读可以直接用 URL 访问，私有需要签名 URL
- **OSS vs 本地存储**：OSS 是云存储（空间无限、高可用），本地存储受服务器限制

---

OSS 的本质是：**把文件存储从应用服务器转移到云端，解决存储空间、可用性、访问速度等问题，让开发者专注于业务逻辑。**

掌握 OSS，你就能在项目中实现高效的文件存储和管理！
