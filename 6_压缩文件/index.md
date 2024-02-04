# Go语言学习之压缩文件



# Go中如何压缩文件

## zip zlib gzip 的区别

总的来说，ZIP适用于多文件和目录的归档，ZLIB适用于轻量级的数据压缩，而GZIP适用于单个文件的压缩。

如果需要跨平台支持并涉及多文件归档，ZIP可能是更好的选择；如果只是需要对单个文件或数据进行压缩，ZLIB或GZIP可能更适合。

### ZIP

设计目标：ZIP是一种通用的压缩和归档格式，旨在提供对多个文件和目录的支持。

压缩算法：使用DEFLATE算法进行数据压缩。

使用场景：适用于需要在不同操作系统之间进行文件传输或存档的场景，因为ZIP格式是跨平台支持的。

### ZLIB

设计目标：ZLIB是一个用于数据压缩和解压缩的库，它并不直接提供归档功能，而是专注于提供一种通用的压缩算法。

压缩算法：使用DEFLATE算法，与ZIP相同。

使用场景：适用于需要在程序中对单个文件或数据进行轻量级压缩和解压缩的场景，而不涉及归档。

### GZIP

设计目标：GZIP是一种常用的压缩和归档格式，通常用于对单个文件进行压缩。

压缩算法：也使用DEFLATE算法，但与ZIP格式略有不同的头部和尾部格式。

使用场景：适用于需要在文件系统中存储或传输单个文件的场景，因为GZIP格式在UNIX/Linux环境中广泛支持


## 在Go中如何应用

### ZIP（archive/zip）

archive/zip包主要用于创建和解析ZIP格式的归档文件。使用archive/zip包的Create函数创建一个zip.Writer，然后添加文件并进行压缩。

缺点：ZIP格式是通用的归档格式，但相对于一些其他格式，它可能在某些情况下产生较大的文件。

```go
import (
    "archive/zip"
    "os"
)

func main() {
    outputFile, _ := os.Create("output.zip")
    defer outputFile.Close()

    zipWriter := zip.NewWriter(outputFile)
    defer zipWriter.Close()

    // 添加文件到ZIP文件中
    fileToZip, _ := os.Open("file.txt")
    defer fileToZip.Close()

    fileInfo, _ := fileToZip.Stat()
    zipHeader, _ := zip.FileInfoHeader(fileInfo)
    fileInZip, _ := zipWriter.CreateHeader(zipHeader)

    // 将文件内容拷贝到ZIP文件中
    io.Copy(fileInZip, fileToZip)
}
```


### ZLIB（compress/zlib）：

compress/zlib包提供了对ZLIB格式的压缩和解压缩支持，适用于需要在程序中进行轻量级压缩和解压缩操作的场景。

使用compress/zlib包的NewWriter函数创建一个zlib.Writer，然后将数据写入以进行压缩。

缺点：ZLIB格式通常不用于直接创建归档文件，而是用于对单个文件或数据进行压缩。

```go
import (
    "compress/zlib"
    "os"
)

func main() {
    outputFile, _ := os.Create("output.zlib")
    defer outputFile.Close()

    zlibWriter := zlib.NewWriter(outputFile)
    defer zlibWriter.Close()

    // 将数据写入以进行压缩
    dataToCompress := []byte("Hello, world!")
    zlibWriter.Write(dataToCompress)
}
```

### GZIP（compress/gzip）：

compress/gzip包提供了对GZIP格式的压缩和解压缩支持，适用于需要在文件系统中存储或传输数据的场景，因为GZIP格式是广泛支持的。

使用compress/gzip包的NewWriter函数创建一个gzip.Writer，然后将数据写入以进行压缩。

缺点：GZIP格式相对于一些其他格式可能产生较大的文件，但通常比ZIP格式更有效。

```go
import (
    "compress/gzip"
    "os"
)

func main() {
    outputFile, _ := os.Create("output.gz")
    defer outputFile.Close()

    gzipWriter := gzip.NewWriter(outputFile)
    defer gzipWriter.Close()

    // 将数据写入以进行压缩
    dataToCompress := []byte("Hello, world!")
    gzipWriter.Write(dataToCompress)
}
```

