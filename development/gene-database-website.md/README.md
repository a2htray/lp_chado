基于 Docker 服务，制作开发、测试、正式环境镜像，保证环境的一致性

需要的镜像包括:

* 数据库 PostgreSQL
* Web 服务器 Nginx + PHP 运行环境
* PHP Composer(单独的工具镜像)
* Node + NPM 镜像，用于打包前端构建


```json
"repositories": {
    "a2htray/gdb-chado": {
        "type": "path",
        "url": "gdb-vendor/a2htray/gdb-chado",
        "options": {
            "symlink": true
        }
    },
    "a2htray/gdb-basic": {
        "type": "path",
        "url": "gdb-vendor/a2htray/gdb-basic",
        "options": {
            "symlink": true
        }
    }
}
```