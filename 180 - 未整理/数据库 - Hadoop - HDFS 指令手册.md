HDFS 作为分布式管理系统，其的客户端能执行的指令分为三大类：上传，下载，文件管理

文件管理主要包含：文件的更名，移动，删除，文件和文件夹的判断，读取文件内容，获取文件大小，修改文件权限列表等

下列的指令用法和 linux 基本相同

```
bin/hadoop fs --help  

[-appendToFile <localsrc> ... <dst>]  
[-cat [-ignoreCrc] <src> ...]  
[-chgrp [-R] GROUP PATH...]  
[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]  
[-chown [-R] [OWNER][:[GROUP]] PATH...]  
[-copyFromLocal [-f] [-p] <localsrc> ... <dst>]  
[-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]  
[-count [-q] <path> ...]  
[-cp [-f] [-p] <src> ... <dst>]  
[-df [-h] [<path> ...]]  
[-du [-s] [-h] <path> ...]  
[-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]  
[-getmerge [-nl] <src> <localdst>]  
[-help [cmd ...]]  
[-ls [-d] [-h] [-R] [<path> ...]]  
[-mkdir [-p] <path> ...]  
[-moveFromLocal <localsrc> ... <dst>]  
[-moveToLocal <src> <localdst>]  
[-mv <src> ... <dst>]  
[-put [-f] [-p] <localsrc> ... <dst>]  
[-rm [-f] [-r|-R] [-skipTrash] <src> ...]  
[-rmdir [--ignore-fail-on-non-empty] <dir> ...]  
<acl_spec> <path>]]  
[-setrep [-R] [-w] <rep> <path> ...]
[-stat [format] <path> ...]  
[-tail [-f] <file>]  
[-test -[defsz] <path>]  
[-text [-ignoreCrc] <src> ...]
```

