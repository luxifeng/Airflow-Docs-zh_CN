# 安全连接

默认情况下，Airflow会在元数据库中以纯文本格式保存连接密码。强烈推荐在安装过程中使用`crypto`包。注意，`crypto`包要求你的操作系统已经安装`libffi-dev`。

如果起初未安装`crypto`包，那么你的`airflow.cfg`文件中Fernet key一项会是空的。

按照以下步骤，你仍然可以为连接密码加密：

1. 安装crypto包 `pip install apache-airflow[crypto]`；
2. 使用下面的代码生成fernet\_key。`fernet_key`必须是Base64编码的32字节密钥；



   ```bash
   from cryptography.fernet import Fernet
   fernet_key= Fernet.generate_key()
   print(fernet_key.decode()) # your fernet_key, keep it in secured place!
   ```

3. 用Step 2中的fernet\_key替换`airflow.cfg`文件中的fernet\_key值。另外，你可以保存fernet\_key到系统环境变量。这样你就不需要修改`airflow.cfg`，因为Airflow会优先使用环境变量：



   ```bash
   # Note the double underscores
   EXPORT AIRFLOW__CORE__FERNET_KEY = your_fernet_key
   ```

4. 重启Airflow web服务器；
5. 对于已存在的连接（在安装`airflow[crypto]`和创建Fernet key之前已定义的连接），你需要在连接管理界面中打开每个连接，重新输入密码，并保存。

