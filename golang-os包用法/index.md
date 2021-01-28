# Golang os包用法


# os 常用导出函数

- func Hostname() (name string, err error) // Hostname 返回内核提供的主机名
- func Hostname() (name string, err error) // Hostname 返回内核提供的主机名
- func Environ() []string // Environ 返回表示环境变量的格式为"key=value"的字符串的切片拷贝
- func Getenv(key string) string // Getenv 检索并返回名为 key 的环境变量的值
- func Getpid() int // Getpid 返回调用者所在进程的进程 ID
- func Exit(code int) // Exit 让当前程序以给出的状态码 code 退出。一般来说，状态码 0 表示成功，非 0 表示出错。程序会立刻终止，defer 的函数不会被执行
- func Stat(name string) (fi FileInfo, err error) // 获取文件信息
- func Getwd() (dir string, err error) // Getwd 返回一个对应当前工作目录的根路径
- func Mkdir(name string, perm FileMode) error // 使用指定的权限和名称创建一个目录
- func MkdirAll(path string, perm FileMode) error // 使用指定的权限和名称创建一个目录，包括任何必要的上级目录，并返回 nil，否则返回错误
- func Remove(name string) error // 删除 name 指定的文件或目录
- func TempDir() string // 返回一个用于保管临时文件的默认目录
- var Args []string Args 保管了命令行参数，第一个是程序名。

