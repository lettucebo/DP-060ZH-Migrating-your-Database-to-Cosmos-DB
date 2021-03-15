
# 实验室 2：将 MongDB 工作负载迁移到 Cosmos DB
<!-- TOC -->

- [实验室 2：将 MongDB 工作负载迁移到 Cosmos DB](#lab-2-migrate-mongdb-workloads-to-cosmos-db)
    - [练习 1：设置](#exercise-1-setup)
        - [任务 1：创建资源组和虚拟网络](#task-1-create-a-resource-group-and-virtual-network)
        - [任务 2：创建一个 MongoDB 数据库服务器](#task-2-create-a-mongodb-database-server)
        - [任务 3：配置 MongoDB 数据库](#task-3-configure-the-mongodb-database)
    - [练习 2：填充和查询 MongoDB 数据库](#exercise-2-populate-and-query-the-mongodb-database)
        - [任务 1：生成并运行应用以填充 MongoDB 数据库](#task-1-build-and-run-an-app-to-populate-the-mongodb-database)
        - [任务 2：生成并运行其他应用以查询 MongoDB 数据库](#task-2-build-and-run-another-app-to-query-the-mongodb-database)
    - [练习 3：将 MongoDB 数据库迁移到 Cosmos DB](#exercise-3-migrate-the-mongodb-database-to-cosmos-db)
        - [任务 1：创建 Cosmos 帐户和数据库](#task-1-create-a-cosmos-account-and-database)
        - [任务 2：创建数据库迁移服务](#task-2-create-the-database-migration-service)
        - [任务 3：创建并运行新的迁移项目](#task-3-create-and-run-a-new-migration-project)
        - [任务 4：验证迁移是否成功](#task-4-verify-that-migration-was-successful)
    - [练习 4：重新配置并运行现有应用程序以使用 Cosmos DB](#exercise-4-reconfigure-and-run-existing-applications-to-use-cosmos-db)
    - [练习 5：知识总结](#exercise-5-clean-up)

<!-- /TOC -->
在本实验室中，你将使用现有 MongoDB 数据库并将其迁移到 Cosmos DB。你将使用 Azure 数据库迁移服务。你还将了解如何将使用 MongoDB 数据库的现有应用程序重新配置为连接到 Cosmos DB 数据库。

本实验室以从一系列 IoT 设备中捕获温度数据的示例系统为基础。温度和时间戳一起记录在 MongoDB 数据库中。每个设备都有唯一的 ID。你将运行一个模拟这些设备并将数据存储在数据库中的 MongoDB 应用程序。你还将使用支持用户查询每个设备的统计信息的第二个应用程序。将数据库从 MongoDB 迁移到 Cosmos DB 之后，你将这两个应用程序配置为连接到 Cosmos DB，并验证其是否仍正常运行。

本实验室使用 Azure Cloud Shell 和 Azure 门户运行。

## 练习 1：设置

在第一个练习中，你将创建 MongoDB 数据库，以保存从温度设备捕获的数据。

### 任务 1：创建资源组和虚拟网络

1. 在 Internet 浏览器中，导航到 https://portal.azure.com 并登录。
2. 在 Azure 门户中，单击 **“资源组”**，再单击 **“+添加”**。
3. 在**创建资源组页面**，输入以下详细信息，然后单击 **“查看 + 创建”**：

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | mongodbrg |
    | 区域 | 选择离你最近的位置 |

4. 单击 **“创建”**，然后等待创建资源组。
5. 在 Azure 门户的侧边栏菜单中，单击 **“+ 创建资源”**。
6. 在 **“新建”** 页面的 **“搜索市场”** 框中，键入 **“虚拟网络”**，然后按 Enter。
7. 在 **“虚拟网络”** 页面上，单击 **“创建”**。
8. 在 **“创建虚拟网络”** 页面，输入以下详细信息，然后单击 **“下一步：IP 地址**：

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | mongodbrg |
    | 名称 | databasevnet |
    | 区域 | 选择为资源组指定的相同位置 |
    
9. 在 **“IP 地址”** 页面，将 **“IPv4 地址空间”** 设置为 **“10.0.0.0/24”**，然后单击 **“+ 添加子网”**。

10. 在 **“添加子网”** 窗格中，将 **“子网名称”** 设置为 **“默认”**，将 **“子网地址范围”** 设置为 **“10.0.0.0/28”**，然后单击 **“添加”**。

11. 在 **“IP 地址”** 页面上，选择 **“默认”** 子网，然后单击 **“下一步：安全性”**。

12. 在 **“安全性”** 页面上，验证 **“DDoS 保护”** 是否设置为 **“基本”**，**“防火墙”** 是否设置为 **“禁用”**。单击 **“查看 创建”**。

13. 在 **“创建虚拟网络”** 页面上，单击 **“创建”**。请先等待虚拟网络创建，然后再继续操作。

### 任务 2：创建一个 MongoDB 数据库服务器

1. 在 Azure 门户的侧边栏菜单中，单击 **“+ 创建资源”**。
2. 在 **“搜索市场”** 框中，键入 **“MongoDB 4.4 for LINUX CentOS 7.8”**，然后按 Enter。
3. 在 **“市场”** 页面上，单击 **“MongoDB 4.4 for LINUX CentOS 7.8 (Tidal Media Inc)”**。
4. 在 **“MongoDB 4.4 for LINUX CentOS 7.8”** 页面上，单击 **“创建”**。
5. 在 **“创建虚拟机”** 页面上，输入以下详细信息，然后单击 **“下一步：磁盘 \>”**。

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | mongodbrg |
    | 虚拟机名称 | mongodbserver | 
    | 区域 | 选择为资源组指定的相同位置 |
    | 可用性选项 | 无需基础结构冗余 |
    | 映像 | Ubuntu 中的 MongoDB Community 4.0 |
    | Azure Spot 实例 | 否 |
    | 大小 | 标准 A1_v2 |
    | 身份验证类型 | 密码 |
    | 用户名 | azureuser |
    | 密码 | Pa55w.rd |
    | 确认密码 | Pa55w.rd |

6. 在 **“磁盘”** 页面上，接受默认设置，然后单击 **“下一步：网络 \>”**。
7. 在 **“网络”** 页面上，输入以下详细信息，然后单击 **“下一步：管理 \>”**。

    | 属性  | 值  |
    |---|---|
    | 虚拟网络 | databasevnet |
    | 子网 | 默认 (10.0.0.0/28) |
    | 公共 IP | （新）mongodbserver-ip |
    | NIC 网络安全组 | 高级 |
    ! 配置网络安全组 | （新）mongodbserver-nsg |
    | 加速网络 | 关闭 |
    | 负载均衡 | 否 |

8. 在 **“管理”** 页面上，接受默认设置，然后单击 **“查看 + 创建”**。
9. 在验证页面上，单击 **“创建”**。
10. 请先等待虚拟机部署，然后再继续操作。
11. 在 Azure 门户的侧边栏菜单中，单击 **“所有资源”**。
12. 在 **“所有资源”** 页面上，单击 **“mongodbserver-nsg”**。
13. 在 **“mongodbserver-nsg”** 页面的 **“设置”** 下，单击 **“入站安全规则”**。
14. 在 **“mongodbserver-nsg - 入站安全规则”** 页面上，单击 **“+ 添加”**。
15. 在 **“添加入站安全规则”** 窗格中，输入以下详细信息，然后单击 **“添加”**：

    | 属性  | 值  |
    |---|---|
    | 源 | 任何 |
    | 源端口范围 | * |
    | 目标 | 任何 |
    | 目标端口范围 | 27017 |
    | 协议 | 任何 |
    | 操作 | 允许 |
    | 优先级 | 1030 |
    | 名称 | Mongodb-port |
    | 描述 | 客户端用于连接到 MongoDB 的端口 |

### 任务 3：配置 MongoDB 数据库

默认情况下，Mongo DB 实例配置为无需进行身份验证即可运行。在本任务中，你将启用身份验证并创建必需的用户帐户来执行迁移。你还将添加一个帐户，供测试应用程序查询数据库。

1. 在 Azure 门户的侧边栏菜单中，单击 **“所有资源”**。
2. 在 **“所有资源”** 页面上，单击 **“mongodbserver-ip”**。
3. 在 **“mongodbserver-ip”** 页面上，记下 **“IP 地址”**。
4. 在 Azure 门户顶部的工具栏中，单击 **“Cloud Shell”**。
5. 如果出现 **“你没有装载存储”** 消息框，请单击 **“创建存储”**。
6. Cloud Shell 启动时，在 Cloud Shell 窗口上方的下拉列表中，选择 **“Bash”**。
7. 在 Cloud Shell 中，输入以下命令以连接 mongodbserver 虚拟机。请将 *“\<ip address\>”* 替换为 **“mongodbserver-ip”** IP 地址的值：

    ```bash
    ssh azureuser@<ip address>
    ```

8. 在提示符下，键入 **“是”** 以继续连接。
9. 输入密码 **“Pa55w.rd”**
10. 停止 MongoDB 服务：

    ```bash
    sudo service mongod stop
    ```

11. 以 **mongodb** 用户身份启动 bash shell：

    ```bash
    sudo -u mongodb bash
    ```

12. 以 **mongodb** 用户身份在本地重启 MongoDB 服务。

    ```bash
    mongod --dbpath /data/mongo &
    ```

    服务重启时，控制台上会显示大量消息。按 Enter 显示 bash 命令提示符。

13. 运行以下命令以连接到 MongoDB 服务：

    ```bash
    mongo
    ```

14. 在 **“>”** 提示符下，运行以下命令。这些命令会创建名为 **administartor** 的新用户，该用户可管理和监视数据库服务器：

    ```mongosh
    use admin
    db.createUser(
        {
            user: "administrator",
            pwd: "Pa55w.rd",
            roles: [
                { role: "userAdminAnyDatabase", db: "admin" },
                { role: "clusterMonitor", db:"admin" },
                "readWriteAnyDatabase"
            ]
        }
    )
    ```

15. 运行以下命令，为名为 **DeviceData** 的数据库创建另一个名为 **deviceadmin** 的用户。运行 `db.shutdownserver();` 命令后，你将收到一些错误，这些错误可以忽略：

    ```mongosh
    use DeviceData;
    db.createUser(
        {
            user: "deviceadmin",
            pwd: "Pa55w.rd",
            roles: [ { role: "readWrite", db: "DeviceData" } ]
        }
    );
    use admin;
    db.shutdownServer();
    exit;
    ```

16. 在 bash 提示符处，关闭以 **mongodb** 用户身份运行的 bash shell：

    ```bash
    exit
    ```

17. 运行以下命令，重启 mongodb 服务。验证服务是否重启且没有出现任何错误消息，并且是否正在侦听端口 27017：

    ```bash
    sudo service mongod start
    ```

18. 运行以下命令，以验证你现在是否能够以 deviceadmin 用户身份登录 mongodb：

    ```bash
    mongo -u "deviceadmin" -p "Pa55w.rd" --authenticationDatabase DeviceData
    ```

19. 在 **“>”** 提示符下，运行以下命令以退出 mongo shell：

    ```mongosh
    exit;
    ```

20. 在 bash 提示符下，运行以下命令以断开与 MongoDB 服务器的连接并返回到 Cloud Shell：

    ```bash
    exit
    ```

## 练习 2：填充和查询 MongoDB 数据库

你现在已经创建了 MongoDB 服务器和数据库。下一步是演示可以填充和查询此数据库中的数据的示例应用程序。

### 任务 1：生成并运行应用以填充 MongoDB 数据库

0. 在 Azure Cloud Shell 中，运行以下命令以安装环境：

    ```bash
    sudo yum install git
    
    sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
    sudo yum install dotnet-sdk-2.1
    ```

1. 在 Azure Cloud Shell 中，运行以下命令以下载此研讨会的示例代码：

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-060ZH-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

2. 移至 **“migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceCapture”** 文件夹：

    ```bash
    cd DP-060ZH-Migrating-your-Database-to-Cosmos-DB/MongoDeviceDataCapture/MongoDeviceDataCapture
    ```

3. 使用 **“代码”** 编辑器来检查 **“TemperatureDevice.cs”** 文件：

    ```bash
    nano TemperatureDevice.cs
    ```

    此文件中的代码包含一个名为 **“TemperatureDevice”** 的类，它模拟温度设备捕获数据并将数据保存在 MongoDB 数据库中。该类使用适用于 .NET Framework 的 MongoDB 库。 **“TemperatureDevice”** 构造函数使用存储在应用程序配置文件中的设置连接到数据库。 **“RecordTemperatures”** 方法生成读数并将其写入数据库。

4. 关闭代码编辑器，然后打开 **“ThermometerReading.cs”** 文件：

   ```bash
   nano ThermometerReading.cs
   ```

    此文件显示了应用程序存储在数据库中的文档的结构。每个文档都包含以下字段：

    - 对象 ID。它是 MongoDB 生成的“_id”字段，用于唯一标识每个文档。
    - 设备 ID。每个设备都有一个带有前缀“Device”的数字。
    - 设备记录的温度
    - 记录温度时的日期和时间。
  
5. 关闭代码编辑器，然后打开 **“App.config”** 文件：

    ```bash
    nano App.config
    ```

    此文件包含用于连接到 MongoDB 数据库的设置。请将 **“地址”** 键的值设置为先前记录的 MongoDB 服务器的 IP 地址，然后保存文件并关闭编辑器。

6. 若要重新构建应用程序，请运行以下命令：

    ```bash
    dotnet build
    ````

7. 运行应用程序：

    ```bash
    dotnet run
    ```

    应用程序模拟了 100 个同时运行的设备。让应用程序运行几分钟，然后按 Enter 停止运行。

### 任务 2：生成并运行其他应用以查询 MongoDB 数据库

1. 移至 **“migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery”** 文件夹：

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

    此文件夹包含另一个应用程序，可使用该应用程序分析由每台设备捕获的数据。

2. 使用 **“代码”** 编辑器来检查 **“Program.cs”** 文件：

    ```bash
    code Program.cs
    ```

    应用程序（使用文件底部的 **ConnectToDatabase** 方法）连接到数据库，然后提示用户输入设备编号。应用程序使用适用于 .NET Framework 的 MongoDB 库来创建并运行聚合管道，该管道可为指定设备计算以下统计信息：

    - 记录的读数数目。
    - 记录的平均温度。
    - 最低读数。
    - 最高读数。
    - 最新读数。

3. 关闭代码编辑器，然后打开 **“App.config”** 文件：

    ```bash
    code App.config
    ```

    与之前一样，请将 **“地址”** 键的值设置为先前记录的 MongoDB 服务器的 IP 地址，然后保存文件并关闭编辑器。

4. 生成并运行应用程序：

    ```bash
    dotnet build
    dotnet run
    ```

5. 在 **“输入设备编号”** 提示符下，请输入一个介于 0 和 99 之间的值。应用程序将查询数据库、计算统计信息并显示结果。请按 **“Q”** 以退出应用程序。

## 练习 3：将 MongoDB 数据库迁移到 Cosmos DB

下一步是获取 MongoDB 数据库并将其转移到 Cosmos DB。

### 任务 1：创建 Cosmos 帐户和数据库

1. 返回 Azure 门户。
2. 在侧边栏菜单中，单击 **“+ 创建资源”**。
3. 在 **“新建”** 页面的 **“搜索市场”** 框中，键入 **“Azure Cosmos DB”**，然后按 Enter。
4. 在 **“Azure Cosmos DB”** 页面上，单击 **“创建”**。
5. 在 **“创建 Azure Cosmos DB 帐户”** 页面上，输入以下设置，然后单击 **“查看 + 创建”**：

    | 属性  | 值  |
    |---|---|
    | 订阅 | 选择你的订阅 |
    | 资源组 | mongodbrg |
    | 帐户名 | mongodb*nnn*，其中 *“nnn”* 表示你选择的随机编号 |
    | API | 适用于 MongoDB API 的 Azure Cosmos DB |
    | 位置 | 指定用于 MongoDB 服务器和虚拟网络的相同位置 |
    | 异地冗余 | 禁用 |
    | 多区域写入 | 禁用 |

6. 在“验证”页面上，单击 **“创建”**，然后等待部署 Cosmos DB 帐户。
7. 在 Azure 门户的侧边栏菜单中，单击 **“所有资源”**，然后单击新建的 Cosmos DB 帐户（**mongodb*nnn***）。
8. 在 **“mongodb*nnn”*** 页面上，单击 **“数据资源管理器”**。
9. 在 **“数据资源管理器”** 窗格中，单击 **“新建集合”**。
10. 在 **“添加集合”** 窗格中，指定以下设置，然后单击 **“确定”**：

    | 属性  | 值  |
    |---|---|
    | 数据库 ID | 单击 **“新建”**，然后键入 **“DeviceData”** |
    | 预配数据库吞吐量 | 已选择 |
    | 吞吐量 | 1000 |
    | 集合 ID | 温度 |
    | 存储容量 | 无限制 |
    | 分片键 | deviceID |
    | 我的分片键超过 100 个字节 | 取消选中 |

### 任务 2：创建数据库迁移服务

1. 在 Azure 门户的侧边栏菜单中，单击 **“所有服务”**。
2. 在 **“所有服务”** 搜索框中，键入 **“Subscriptions”**，然后按 **Enter**。
3. 在 **“订阅”** 页面上，单击你的订阅。
4. 在订阅页面上的 **“设置”** 下，单击 **“资源提供程序”**。
5. 在 **“按名称筛选”** 框中，键入 **“DataMigration”**，然后单击 **“Microsoft.DataMigration”**。
6. 请单击 **“注册表”**，然后等待 **“状态”** 更改为 **“已注册”**。可能需要单击 **“刷新”** 才能查看状态是否更改。
7. 在 Azure 门户的侧边栏菜单中，单击 **“+ 创建资源”**。
8. 在 **“新建”** 页面的 **“搜索市场”** 框中，键入 **“Azure 数据库迁移服务”**，然后按 Enter。
9. 在 **“Azure 数据库迁移服务”** 页面上，单击 **“创建”**。
10. 在 **“创建迁移服务”** 页面上，输入以下设置，然后单击 **“下一步：网络**：

    | 属性  | 值  |
    |---|---|
    | 订阅 | 选择你的订阅 |
    | 资源组 | mongodbrg |
    | 服务名称 | MongoDBMigration |
    | 位置 | 选择你以前使用的同一位置 |
    | 服务模式 | Azure |
    | 定价层 | 标准：1 个 vCore |


11. 在 **“网络”** 页面上，选择 **“databasevnet/default”**，**“查看 + 创建”**
12. 单击 **“创建”**，等待服务部署完毕，然后再继续操作。此操作将花费几分钟时间。

### 任务 3：创建并运行新的迁移项目

1. 在 Azure 门户的侧边栏菜单中，单击 **“资源组”**。
2. 在 **“资源组”** 窗口中，单击 **“mongodbrg”**。
3. 在 **“mongodbrg”** 窗口中，单击 **“MongoDBMigration”**。
4. 在 **“MongoDBMigration”** 页面上，单击 **“+ 新建迁移项目”**。
5. 在 **“新建迁移项目”** 页面上，输入以下设置，然后单击 **“创建并运行活动”**：

    | 属性  | 值  |
    |---|---|
    | 项目名 | MigrateTemperatureData |
    | 源服务器类型 | MongoDB |
    | 目标服务器类型 | Cosmos DB (MongoDB API) |
    | 选择活动类型 | 离线数据迁移 |

6. 启动 **“迁移向导”** 时，在 **“源详细信息”** 页面上，输入以下详细信息，然后单击 **“保存”**：

    | 属性  | 值  |
    |---|---|
    | 模式 | 标准模式 |
    | 源服务器名称 | 指定之前记录的 **“mongodbserver-ip”** IP 地址的值 |
    | 服务器端口 | 27017 |
    | 用户名 | administrator |
    | 密码 | Pa55w.rd |
    | 需要 SSL | 留空 |
    | 我的服务器已启用 TLS 1.2 | select |

7. 在 **“迁移详细信息”** 页面上，输入以下详细信息，然后单击 **“保存”**：

    | 属性  | 值  |
    |---|---|
    | 模式 | 选择 Cosmos DB 目标 |
    | 订阅 | 选择你的订阅 |
    | 选择 Comos DB 名称 | mongodb*nnn* |
    | 连接字符串 | 接受为 Cosmos DB 帐户生成的连接字符串 |

8. 在 **“映射到目标数据库”** 页面上，输入以下详细信息，然后单击 **“保存”**：

    | 属性  | 值  |
    |---|---|
    | 源数据库 | DeviceData |
    | 目标数据库 | DeviceData |
    | 吞吐量（RU/秒） | 1000 |
    | 清理集合 | 清除此框 |

9. 在 **“集合设置”** 页面上，单击 DeviceData 数据库旁边的下拉箭头，输入以下详细信息，然后单击 **“保存”**：

    | 属性  | 值  |
    |---|---|
    | 名称 | 温度 |
    | 目标集合 | 温度 |
    | 吞吐量（RU/秒） | 1000 |
    | 分片键 | deviceID |
    | 唯一 | 留空 |

10. 在 **“迁移摘要”** 页面上的 **“活动名称”** 字段，输入 **“mongodb-migration”**，选择 **“在迁移期间提升 RU”**，然后单击 **“运行迁移”**。
11. 在 **“mongodb-migration”** 页面上，每隔 30 秒单击一次 **“刷新”**，直到迁移完成。注意已处理的文档数。

### 任务 4：验证迁移是否成功

1. 在 Azure 门户的侧边栏菜单中，单击 **“所有资源”**。
2. 在 **“所有资源”** 页面上，单击 **“mongodb*nnn”***。
3. 在 **“mongodb*nnn”*** 页面上，单击 **“数据资源管理器”**。
4. 在 **“数据资源管理器”** 窗格中，依次展开 **“DeviceData”** 数据库、**“温度”** 集合，然后单击 **“文档”**。
5. 在 **“文档”** 窗格中，滚动浏览文档列表。你应看到每个文档的文档 ID (**_id**) 和共享密钥 (**/deviceID**)。
6. 单击任意文档。你应看到系统显示的该文档的详细信息。常见文档如下所示：

    ```JSON
    {
	    "_id" : ObjectId("5ce8104bf56e8a04a2d0929a"),
	    "deviceID" : "Device 83",
	    "temperature" : 19.65268837271849,
	    "time" : 636943091952553500
    }
    ```

7. 在 **“文档资源管理器”** 窗格的工具栏中，单击 **“新建 Shell”**。
8. 在 **“Shell 1”** 窗格中，在 **“\>”** 提示符下，输入以下命令，然后按 Enter：

    ```mongosh
    db.Temperatures.count()
    ```

    此命令显示“温度”集合中的文档数量。文档数量应与“迁移向导”报告的数量一致。

9. 输入以下命令，然后按 Enter：

    ```mongosh
    db.Temperatures.find({deviceID: "Device 99"})
    ```

    此命令提取并显示设备 99 的相关文档。

## 练习 4：重新配置并运行现有应用程序以使用 Cosmos DB

最后一步是重新配置现有 MongoDB 应用程序以连接到 Cosmos DB，并验证它们是否照常\运行。此过程要求修改应用程序连接数据库的方式，但是应用程序的逻辑应保持不变。

1. 在 **“mongodb*nnn”*** 窗格中的 **“设置”** 下，单击 **“连接字符串”**。
2. 在 **“mongodb*nnn* 连接字符串”** 页面上，记下以下设置：

    - 主机
    - 用户名
    - 主密码
  
3. 返回 Cloud Shell 窗口（如果会话超时，请重新连接），然后移至 **“migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery”** 文件夹：

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

4. 在代码编辑器中打开 App.config 文件：

    ```bash
    code App.config
    ```

5. 在文件的 **“MongoDB 设置”** 部分中，注释掉现有设置。
6. 取消注释 **“Cosmos DB Mongo API 设置”** 部分中的设置，并按如下所示设置这些设置的值：

    | 设置  | 值  |
    |---|---|
    | 地址 | **“mongodb*nnn* 连接字符串”** 页面中的**主机** |
    | 用户名 | **“mongodb*nnn* 连接字符串”** 页面中的 **Username** |
    | 密码 | **“mongodb*nnn* 连接字符串”** 页面中的**主密码** |

    完成的文件应如下所示：

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="Database" value="DeviceData" />
            <add key="Collection" value="Temperatures" />

            <!-- Settings for MongoDB -->
            <!--add key="Address" value="nn.nn.nn.nn" />
            <add key="Port" value="27017" />
            <add key="Username" value="deviceadmin" />
            <add key="Password" value="Pa55w.rd" /-->
            <!-- End of settings for MongoDB -->

            <!-- Settings for CosmosDB Mongo API -->
            <add key="Address" value="mongodbnnn.documents.azure.com"/>
            <add key="Port" value="10255"/>
            <add key="Username" value="mongodbnnn"/>
            <add key="Password" value="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=="/>
            <!-- End of settings for CosmosDB Mongo API -->
        </appSettings>
    </configuration>
    ```

7. 保存文件，然后关闭代码编辑器。

8. 使用代码编辑器打开 Program.cs 文件：

    ```bash
    code Program.cs
    ```

9. 向下滚动到 **ConnectToDatabase** 方法。
10. 注释掉设置用于连接 MongoDB 的凭据的行，并取消注释指定用于连接 Cosmos DB 的凭据的语句。代码应如下所示：

    ```C#
    // 连接 MongoDB 数据库
    MongoClient client = new MongoClient(new MongoClientSettings
    {
        Server = new MongoServerAddress(address, port),
        ServerSelectionTimeout = TimeSpan.FromSeconds(10),

        //
        // MongoDB 的凭据设置
        //

        // 凭据 = MongoCredential.CreateCredential(database, azureLogin.UserName, azureLogin.SecurePassword),

        //
        // CosmosDB Mongo API 的凭据设置
        //

        UseSsl = true,
        SslSettings = new SslSettings
        {
            EnabledSslProtocols = SslProtocols.Tls12
        },
        Credential = new MongoCredential("SCRAM-SHA-1", new MongoInternalIdentity(database, azureLogin.UserName), new PasswordEvidence(azureLogin.SecurePassword))

        // Mongo API 设置结束 
    });

    ```

    这些更改是必需的，因为原始 MongoDB 数据库没有使用 SSL 连接。Cosmos DB 始终使用 SSL。

11. 保存文件，然后关闭代码编辑器。
12. 重新生成并运行应用程序：

    ```bash
    dotnet build
    dotnet run
    ```

13. 在 **“输入设备编号”** 提示符下，请输入一个介于 0 和 99 之间的设备编号。应用程序应照常运行，只是这次使用的是 Cosmos DB 数据库中保存的数据。
14. 使用其他设备编号测试应用程序。输入 **“Q”** 完成操作。

你已成功将 MongoDB 数据库迁移到 Cosmos DB，并将现有 MongoDB 应用程序重新配置为连接到 Cosmos DB 数据库。

## 练习 5：知识总结

1. 返回 Azure 门户。
2. 在侧边栏菜单中，单击 **“资源组”**。
3. 在 **“资源组”** 窗口中，单击 **“mongodbrg”**。
4. 单击 **“删除资源组”**。
5. 在 **“确定要删除“mongodbrg”吗”** 页面的 **“输入资源组名称”** 框中，输入 **“mongodbrg”**，然后单击 **“删除”**。

---
© 2020 Microsoft Corporation。保留所有权利。

本文档中的文本在[知识共享署名 3.0 许可](https://creativecommons.org/licenses/by/3.0/legalcode)下提供，并且可能受到附加条款约束。本文档中包含的所有其他内容（包括但不限于商标、徽标、图片等）均**不**包含在知识共享许可授权范围内。本文档不提供针对任何 Microsoft 产品中任何知识产权的任何合法权利。你可以出于内部参考目的复制和使用本文档。

本文档“按原样”提供。本文档中表达的信息和观点（包括 URL 和其他 Internet 网站参考）如有更改，恕不另行通知。你需要自行承担使用风险。有些示例仅用于说明，而且是虚构的。不能将其视为有意关联或推断。Microsoft 对此处提供的信息不做任何明示或暗示的保证。
