
# 敏感信息获取

存在敏感信息的文件、安装包、备份文件

- github
    - 源代码以及配置上传到github
    - 云平台aksk

- 用户目录下的敏感文件
    - .bash_history
    - .zsh_history
    - .profile
    - .bashrc
    - .gitconfig
    - .viminfo
    - passwd

- 应用的配置文件
    - /etc/apache2/apache2.conf
    - /etc/nginx/nginx.conf

- 应用的日志文件
    - /var/log/apache2/access.log
    - /var/log/nginx/access.log

- 站点目录下的敏感文件
    - .svn/entries
    - .git/HEAD
    - WEB-INF/web.xml
    - .htaccess
    - web.config
    - .db
    - .properties
    - .json
    - .conf
    - Index off

- 特殊的备份文件
    - .swp
    - .swo
    - .bak
    - index.php~
    - ...

- Python的Cache
    - ``__pycache__\__init__.cpython-35.pyc``