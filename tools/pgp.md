# 上传公钥到PGP服务器

1，下载安装pgp2 https://sourceforge.net/p/gpgosx/docu/Download/

2，创建公钥

gpg2 --gen-key

输入姓名、邮箱、Passphase（自定义密钥密码）

3，导出公钥文件

```shell
gpg2 --export -a youpublickey > youpublickey.key
```

youpublickey 为你的公钥

4，上传公钥到服务器

打开链接https://keys.openpgp.org/

选择刚导出的公钥上传

ps 直接命令行上传公钥总是出现 该问题的错误，还是直接文件上传靠谱

问题：https://www.oschina.net/question/2408030_2317890



5，确认身份

上传后会收到邮件确认作者身份，确认后即可在该网站查询到自己的公钥信息，可用于maven中央库发布啥的



PS maven库发布jar文件会去这些keyserver验证

```shell
Event: Failed: Signature Validation
Thursday, July 30, 2020 12:38:18 (GMT+0800)
typeId	signature-staging
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://keyserver.ubuntu.com:11371/">http://keyserver.ubuntu.com:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://keys.openpgp.org:11371/">http://keys.openpgp.org:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://pool.sks-keyservers.net:11371/">http://pool.sks-keyservers.net:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://keyserver.ubuntu.com:11371/">http://keyserver.ubuntu.com:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://keys.openpgp.org:11371/">http://keys.openpgp.org:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://pool.sks-keyservers.net:11371/">http://pool.sks-keyservers.net:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://keyserver.ubuntu.com:11371/">http://keyserver.ubuntu.com:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://keys.openpgp.org:11371/">http://keys.openpgp.org:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://pool.sks-keyservers.net:11371/">http://pool.sks-keyservers.net:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://keyserver.ubuntu.com:11371/">http://keyserver.ubuntu.com:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://keys.openpgp.org:11371/">http://keys.openpgp.org:11371/</a>. Upload your public key and try the operation again.
failureMessage	No public key: Key with id: (xxxid) was not able to be located on <a href="http://pool.sks-keyservers.net:11371/">http://pool.sks-keyservers.net:11371/</a>. Upload your public key and try the operation again.
```

