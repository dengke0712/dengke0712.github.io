# 文件上传与下载


# MINIO

## 前端上传OR后端上传

### 前端上传：
优点：

1. 用户体验： 前端上传能够提供更好的用户体验，用户可以直接在浏览器中选择文件并监控上传进度。
2. 减轻服务器负担： 文件可以直接从用户浏览器上传到对象存储服务，减轻了后端服务器的负担。
3. 断点续传： 前端可以更容易地实现断点续传功能，因为文件直接从用户浏览器上传，中间环节的网络中断也不会影响上传。

缺点：
1. 安全性： 前端上传需要特别注意安全性，因为用户可以直接操作上传的请求，ak sk暴露。
2. 限制： 浏览器对文件大小和类型有一些限制，大文件可能会受到浏览器的限制。

### 安全性解决办法

问题：敏感信息（例如 Access Key 和 Secret Key）可能会暴露给用户。

解决：
使用临时凭证： 不要在前端直接使用永久的 Access Key 和 Secret Key。相反，可以通过后端生成临时凭证，前端使用这些临时凭证进行文件上传。
这可以通过使用 AWS 的临时凭证机制（如 STS）或 MinIO 中的 Policy 来实现。
定期轮换凭证可以减少凭证被滥用的风险。


服务端生成临时凭证

```go
func main(){
    minioClient, err := minio.New(endpoint, &minio.Options{
    Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
    Secure: useSSL,
    })

    // 设置过期时间为1小时
	expiry := time.Now().Add(1 * time.Hour)

	// 生成临时凭证
	policy := minio.NewPostPolicy()
	policy.SetBucket(bucketName)
	policy.SetKey(objectName)
	policy.SetExpires(expiry)

	// 生成临时凭证的URL
	url, formData, err := minioClient.PresignedPostPolicy(policy)
	if err != nil {
		log.Fatalln(err)
	}

	fmt.Printf("Presigned URL: %s\n", url)
	fmt.Printf("Form data: %+v\n", formData)
}

```


### 后端上传：
优点：
1. 安全性： 后端上传可以更好地控制和验证文件上传，确保数据的安全性。
2. 文件处理： 后端可以处理大文件的存储、备份、转码等需求。

缺点：
1. 用户体验： 用户体验可能较差，因为文件上传的过程可能需要等待，用户无法直接监控上传进度。
2. 服务器负担： 后端需要处理文件上传的请求，可能对服务器造成一定的负担，特别是在高并发情况下。
3. 复杂性： 后端上传可能需要更复杂的技术和代码，包括文件合并、断点续传的实现等。


## 上传文件到MinIO

- 如果主要使用 MinIO，并且希望与 MinIO 的特定功能交互，可以选择 minio-go/v7。它专注于 MinIO 的特性和用法，可以更方便地使用 MinIO 特有的功能和选项，例如分片上传。
- aws-sdk-go 是 Amazon S3 官方 SDK，支持与 Amazon S3 以及其他 AWS 服务的交互。如果应用可能涉及到多个 AWS 服务，或者在一个混合云环境中操作，可能更倾向于使用 aws-sdk-go。

### Minio-go/v7包
使用 minio-go/v7 包进行 MinIO 的上传,使用 minio.New 初始化 MinIO 客户端，然后使用 PutObject 方法将本地文件上传到指定的 MinIO 存储桶（Bucket）和路径（ObjectName）。

```go
import (
    "github.com/minio/minio-go/v7"
    "github.com/minio/minio-go/v7/pkg/credentials"
)

func main() {
	endpoint := "your-minio-endpoint"
	accessKeyID := "your-access-key"
	secretAccessKey := "your-secret-key"
	useSSL := false

	minioClient, err := minio.New(endpoint, &minio.Options{
		Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
		Secure: useSSL,
	})
	if err != nil {
		log.Fatalln(err)
	}

	bucketName := "your-bucket-name"
	objectName := "path/to/destination/file"
	filePath := "path/to/local/file.txt"

	file, err := os.Open(filePath)
	if err != nil {
		log.Fatalln(err)
	}
	defer file.Close()

	// 使用 PutObject 上传文件
	info, err := minioClient.PutObject(context.TODO(), bucketName, objectName, file, -1, minio.PutObjectOptions{})
	if err != nil {
		log.Fatalln(err)
	}

	fmt.Printf("Successfully uploaded %s of size %d\n", objectName, info.Size)
}

```

如果想分片上传，可以用以下方法代替PutObject 上传文件

```go
func MultipartUpload(){
    uploadID, err := minioClient.NewMultipartUpload(context.TODO(), bucketName, objectName, minio.PutObjectOptions{})
    if err != nil {
        log.Fatalln(err)
    }

    // 分片大小
    partSize := int64(5 * 1024 * 1024) // 5MB

    // 初始化缓冲区
    buffer := make([]byte, partSize)

    // 上传分片
    partNumber := 1
    for {
        n, err := file.Read(buffer)
        if err != nil && n <= 0 {
            break
        }

        _, err = minioClient.PutObjectPart(context.TODO(), bucketName, objectName, uploadID, minio.PartNumber(partNumber), minio.PutObjectPartOptions{
            Reader:      file,
            ContentType: "application/octet-stream",
        })
        if err != nil {
            log.Fatalln(err)
        }

        partNumber++
    }

    // 完成 Multipart Upload
    _, err = minioClient.CompleteMultipartUpload(context.TODO(), bucketName, objectName, uploadID, minio.CompleteMultipartUploadOptions{})
    if err != nil {
        log.Fatalln(err)
    }

    fmt.Printf("Successfully uploaded %s\n", objectName)
}

```

### aws-sdk-go包

```go
import (
    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/credentials"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/s3"
)

func main() {
	accessKey := "your-access-key"
	secretKey := "your-secret-key"
	region := "your-region"
	bucketName := "your-bucket-name"
	filePath := "path/to/your/file.txt" // 本地文件路径
	objectKey := "your-s3-object-key"   // S3 对象键（文件名）

	sess, err := session.NewSession(&aws.Config{
		Region:      aws.String(region),
		Credentials: credentials.NewStaticCredentials(accessKey, secretKey, ""),
	})
	if err != nil {
		log.Fatal(err)
	}

	svc := s3.New(sess)

	file, err := os.Open(filePath)
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	// 配置上传参数
	params := &s3.PutObjectInput{
		Bucket: aws.String(bucketName),
		Key:    aws.String(objectKey),
		Body:   file,
	}

	// 执行上传操作
	resp, err := svc.PutObject(params)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Successfully uploaded file to %s/%s\n", bucketName, objectKey)
	fmt.Printf("ETag: %s\n", *resp.ETag)
}
```



## 从MinIO下载文件

### 后端下载

使用 FGetObject 方法从 MinIO 存储桶中下载文件到本地.

```go
import (
    "github.com/minio/minio-go/v7"
    "github.com/minio/minio-go/v7/pkg/credentials"
)

func main() {
	endpoint := "your-minio-endpoint"
	accessKeyID := "your-access-key"
	secretAccessKey := "your-secret-key"
	useSSL := false

	minioClient, err := minio.New(endpoint, &minio.Options{
		Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
		Secure: useSSL,
	})
	if err != nil {
		log.Fatalln(err)
	}

	bucketName := "your-bucket-name"
	objectName := "path/to/source/file"
	filePath := "path/to/local/destination/file.txt"

	// 使用 GetObject 下载文件
	err = minioClient.FGetObject(context.TODO(), bucketName, objectName, filePath, minio.GetObjectOptions{})
	if err != nil {
		log.Fatalln(err)
	}

	fmt.Printf("Successfully downloaded %s to %s\n", objectName, filePath)
}

```

### 下载文件时返回Presigned URL(预签名URL)

让前端直接对接 MinIO 进行文件下载，可以使用 MinIO 提供的临时授权（Presigned URL）的机制。

Presigned URL 允许你在前端生成一个包含访问权限和有效期限的 URL，通过这个 URL 可以直接下载文件，而不需要将访问密钥（access key）和秘密密钥（secret key）暴露给前端。

#### minio-go/v7包
```go
import (
	"github.com/minio/minio-go/v7"
	"github.com/minio/minio-go/v7/pkg/credentials"
	"github.com/minio/minio-go/v7/pkg/policy"
)

func main() {
	endpoint := "your-minio-endpoint"
	accessKeyID := "your-access-key"
	secretAccessKey := "your-secret-key"
	useSSL := false

	minioClient, err := minio.New(endpoint, &minio.Options{
		Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
		Secure: useSSL,
	})
	if err != nil {
		log.Fatalln(err)
	}

	bucketName := "your-bucket-name"
	objectName := "path/to/uploaded/file"

	expiry := time.Now().Add(1 * time.Hour)

	// 生成临时凭证
	policy := minio.NewPostPolicy()
	policy.SetBucket(bucketName)
	policy.SetKey(objectName)
	policy.SetExpires(expiry)

	// 可以设置特定的上传条件，例如文件大小、文件类型等
	policy.SetContentLengthRange(1, 1024) // 限制文件大小为1B到1KB之间

	// 生成临时凭证的URL
	url, formData, err := minioClient.PresignedPostPolicy(policy)
	if err != nil {
		log.Fatalln(err)
	}

	fmt.Printf("Presigned URL: %s\n", url)
	fmt.Printf("Form data: %+v\n", formData)
}

```

如果不需要设置特定的上传条件，例如文件大小、文件类型等，可以直接使用PresignedGetObject()
```go
// 生成预签名 URL
presignedURL, err := minioClient.PresignedGetObject("your-bucket-name", "path/to/source/file", expiry, nil)
if err != nil {
	log.Fatalln(err)
}
```

#### aws-sdk-go包

```go
import (
	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/s3"
)

func main() {
	sess, err := session.NewSession(&aws.Config{
		Region: aws.String("your-region"),
	})
	if err != nil {
		log.Fatal(err)
	}

	s3Client := s3.New(sess)

	bucket := "your-s3-bucket"
	objectKey := "path/to/your/file.txt"

	expiration := time.Now().Add(1 * time.Hour)

	// 生成预签名 URL
	req, _ := s3Client.GetObjectRequest(&s3.GetObjectInput{
		Bucket: aws.String(bucket),
		Key:    aws.String(objectKey), // 桶里的文件路径+文件名
	})
	url, err := req.Presign(expiration)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Presigned URL: %s\n", url)
}
```

# SFTP

SFTP (SSH File Transfer Protocol) 是一种通过安全加密的 SSH 连接进行文件传输的协议。它提供了对文件的安全访问和传输，通过在 SSH 连接上建立一个加密通道，保护了数据的机密性和完整性。

SFTP 既可以在交互式用户界面中使用，也可以在命令行中使用，使用户能够执行文件传输和文件管理操作，就像使用传统的 FTP 协议一样。然而，与 FTP 不同，SFTP 在传输过程中使用 SSH 协议进行加密，因此更加安全。

## SFTP VS FTP

SFTP 具有以下优势：

1. 安全性： SFTP 使用 SSH 连接，通过加密通道传输数据，提供更高级别的安全性，防止数据在传输过程中被窃听或篡改。

2. 端到端加密： SFTP 提供端到端的加密，包括身份验证和数据传输，确保数据的完整性和机密性。

3. 单一连接： SFTP 使用单一的连接进行文件传输和管理，而不像 FTP 那样需要多个连接（控制连接和数据连接）。

4. 支持用户身份验证： SFTP 通常通过用户名和密码进行身份验证，也支持其他身份验证方法，如公钥、证书等。

## 建立SFTP连接

```go
import (
	"golang.org/x/crypto/ssh"
	"golang.org/x/crypto/ssh/agent"
)

func main() {
	sftpServer := "your-sftp-server"
	sftpPort := 22

	// 用户名和密码（可以使用其他身份验证方法，如密钥）
	username := "your-username"
	password := "your-password"

	sshConfig := &ssh.ClientConfig{
		User: username,
		Auth: []ssh.AuthMethod{
			ssh.Password(password),
		},
		HostKeyCallback: ssh.InsecureIgnoreHostKey(), // 不验证服务器的 HostKey
	}

	// 建立 SSH 连接
	conn, err := ssh.Dial("tcp", fmt.Sprintf("%s:%d", sftpServer, sftpPort), sshConfig)
	if err != nil {
		log.Fatalf("Failed to dial: %v", err)
	}
	defer conn.Close()

	// 建立 SFTP 客户端
	client, err := sftp.NewClient(conn)
	if err != nil {
		log.Fatalf("Failed to create SFTP client: %v", err)
	}
	defer client.Close()
}
```

为了安全起见，应该使用密钥而不是密码进行身份验证，并且在生产环境中，应该验证服务器的 HostKey。


## 上传和下载文件
```go
// 上传文件
func uploadFile(client *sftp.Client, localPath, remotePath string) {
	localFile, err := os.Open(localPath)
	if err != nil {
		log.Fatalf("Failed to open local file: %v", err)
	}
	defer localFile.Close()

	remoteFile, err := client.Create(remotePath)
	if err != nil {
		log.Fatalf("Failed to create remote file: %v", err)
	}
	defer remoteFile.Close()

	_, err = localFile.WriteTo(remoteFile)
	if err != nil {
		log.Fatalf("Failed to upload file: %v", err)
	}
	fmt.Printf("File '%s' uploaded to '%s'\n", localPath, remotePath)
}

// 下载文件
func downloadFile(client *sftp.Client, remotePath, localPath string) {
	remoteFile, err := client.Open(remotePath)
	if err != nil {
		log.Fatalf("Failed to open remote file: %v", err)
	}
	defer remoteFile.Close()

	localFile, err := os.Create(localPath)
	if err != nil {
		log.Fatalf("Failed to create local file: %v", err)
	}
	defer localFile.Close()

	_, err = remoteFile.WriteTo(localFile)
	if err != nil {
		log.Fatalf("Failed to download file: %v", err)
	}
	fmt.Printf("File '%s' downloaded to '%s'\n", remotePath, localPath)
}
```

