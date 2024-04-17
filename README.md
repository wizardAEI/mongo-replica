## 使用镜像

1. **创建MongoDB的keyFile**:

   虽然项目中有 keyFile ，但是最好还是手动重新创建一个 keyFile：
    
   MongoDB要求keyFile的权限必须为600。你可以在你的项目目录中创建一个名为`mongo-keyfile`的文件，并生成一个密钥。在Linux或macOS上，你可以使用以下命令：

   ```bash
   openssl rand -base64 756 > mongo-keyfile
   chmod 600 mongo-keyfile
   ```

   这将生成一个足够安全的密钥，并设置正确的权限。

3. **启动服务**:

   确保你已经安装了Docker和Docker Compose。然后，在包含`docker-compose.yml`文件的目录中，运行以下命令来启动MongoDB容器：

   ```bash
   docker-compose up -d
   ```

   `-d`标志表示在后台运行。

4. **初始化副本集**:

   MongoDB启动后，你需要初始化副本集。你可以通过执行以下命令进入MongoDB容器的shell：

   ```bash
   docker exec -it mongo bash
   ```

   然后，在容器内部，连接到MongoDB实例并初始化副本集：

   ```bash
   mongo -u root -p example --authenticationDatabase admin
   ```

   进入MongoDB shell后，运行以下命令初始化副本集：

   ```javascript
   rs.initiate({
     _id: "rs0",
     members: [
       { _id: 0, host: "localhost:27017" }
     ]
   });
   ```

   现在，你的MongoDB单节点副本集应该已经配置好并在运行了。记得根据实际情况调整用户名、密码和密钥内容。

## 添加数据库和管理员

要在MongoDB中创建一个数据库并为这个数据库添加管理员，你可以按照以下步骤操作。这些步骤假设你已经按照前面的指示启动了MongoDB容器，并且已经初始化了副本集。

1. **连接到MongoDB实例**:

   首先，你需要连接到MongoDB实例。如果你已经在容器内部，可以直接使用`mongo`命令行工具。如果你是从宿主机连接，确保你使用正确的认证参数。以下命令示例假设你使用的是默认的root用户和密码：

   ```bash
   docker exec -it mongo mongo -u root -p example --authenticationDatabase admin
   ```

2. **创建新数据库**:

   MongoDB中，数据库会在你第一次向其中写入数据时自动创建。但是，你可以通过切换到一个尚不存在的数据库上下文来“创建”一个数据库。例如，如果你想创建一个名为`myNewDatabase`的数据库，可以执行：

   ```javascript
   use myNewDatabase
   ```

   这条命令实际上并没有立即创建数据库，但它会将后续操作指向`myNewDatabase`。

3. **为数据库添加管理员**:

   你可以在特定的数据库上创建用户，并赋予他们管理该数据库的权限。以下命令创建了一个名为`dbAdminUser`的用户，密码为`dbAdminPass`，并赋予了管理`myNewDatabase`数据库的权限：

   ```javascript
   db.createUser({
     user: "dbAdminUser",
     pwd: "dbAdminPass",
     roles: [
       { role: "dbAdmin", db: "myNewDatabase" },
       { role: "readWrite", db: "myNewDatabase" }
     ]
   });
   ```

   这里，`dbAdmin`角色允许用户执行管理任务，如创建索引和查看统计信息，而`readWrite`角色允许用户读写数据库的数据。

4. **验证**:

   要验证新用户是否能够连接并对数据库进行操作，你可以尝试以新用户的身份连接到数据库。首先，退出当前的MongoDB shell，然后使用新用户的凭据重新连接：

   ```bash
   docker exec -it mongo mongo -u dbAdminUser -p dbAdminPass --authenticationDatabase myNewDatabase
   ```

   现在，你应该能够执行对`myNewDatabase`数据库的读写操作了。

请注意，根据你的具体需求，你可能需要调整用户名、密码和角色。始终确保使用强密码并根据实际情况调整权限。

## tips

有时候使用后端 ORM 或 mongodb 框架连接数据库会出现 timeout 的情况，这时候可以尝试加上 directConnection 选项来连接：

```bash
MONGODB_URL=mongodb://root:example@x.x.x.x:27017/database?directConnection=true
```
