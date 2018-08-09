# AZURE BLOB使用教程
- - - -
**Windows Azure** 是微软提供的云计算服务，旨在通过微软管理的全球化的数据中心完成搭建、测试、部署和管理应用和服务。Azure提供包括存储、计算和数据管理等服务，其中Azure存储又包括：

* Azure Blob：适用于文本和二进制数据的可大规模缩放的对象存储。
* Azure 文件：适用于云或本地部署的托管文件共享。
* Azure 队列：用于在应用程序组件之间进行可靠的消息传送的消息传送存储。
* Azure 表：一种 NoSQL 存储，适合用作结构化数据的无模式存储。

本教程将要介绍Azure Blob，即对文本和二进制数据的进行存储的主要方式。因为具有良好的权限管理和API接口，Azure Blob在项目生产过程中被广泛使用，本篇通过从项目经理，外包方和技术人员三个角色的角度去讲解如何通过Azure Blob协作完成数据的传输和流转。

## 基本概念
Blob 存储公开了三种资源：存储帐户、帐户中的容器，以及容器中的 blob。 以下图示显示了这些资源之间的关系：
![](AZURE%20BLOB%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/blob1.png)

### 账户
对 Azure 存储中所有数据对象的访问都是通过存储帐户进行的。存储账户可以通过向运维申请获得，详情请参考 `申请Azure Blob资源`。
账户可以通过授权的方式允许其他角色访问数据（[授权访问 Azure 存储 ](https://docs.microsoft.com/zh-cn/azure/storage/common/storage-auth?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)），包括以下几种方式：
* 用于 blob 和队列的 **Azure Active Directory (Azure AD)** 集成；
* 用于 blob、文件、队列和表的**共享密钥授权**；
* 用于 blob、文件、队列和表的**共享访问签名**（SAS，Shared Access Signatures）；
* 用于容器和 blob 的匿名公共读取访问。

在接下来的章节中，我们会主要介绍**共享访问签名(SAS URL)**的方式，作为提供给其他角色的数据访问办法。

### 容器
容器对一组 blob 进行组织，类似于文件系统中的文件夹。 所有 Blob 都驻留在容器中。 一个存储帐户可以包含无限数量的容器，一个容器可以存储无限数量的 Blob。 但是请注意，容器名称必须小写。

### Blob
Blob是数据存储的最小单元，任何图片、音频、文本等文件都可以理解为Azure上的blob资源。需要注意的是，文件夹不能成为blob，无法单独保存在容器中，但是可以作为一个或多个文件的容器被保存。


## 使用教程
本节从管理人员，数据提供方和技术支持三方协同工作的场景，描述不同角色通过使用Azure Blob相互配合完成数据传输的最佳实践。

### 管理人员（Manager）
一般来说，管理人员具有最高的管理权限，控制各方对数据的访问及数据的生命周期。因此，管理人员主要负责资源的申请和关闭，数据访问权限的开通、配置及撤销，以及了解数据的回收进度和保障数据安全。
具体来说，包含以下职责：
* 向**运维**申请Azure Blob资源，审批通过后，运维会提供资源及相应账户；
* 利用账户登陆，创建容器并向**数据提供方**提供_SAS URI_。关于容器创建和权限分配上，我们提供以下两种实践策略：
	* 根据**数据提供方**的创建容器，并向其提供对应的_SAS URI_，数据提供方具有读、写和列出的权限。这种方式可以对多个数据提供方进行隔离，避免信息泄露等安全问题，但与此同时，较难将多个数据提供方提供的同批次的数据进行管理，只能通过容器中创建文件夹的方式对批次进行区分；
	* 根据**批次**创建容器，并向所有人提供每批次的_SAS URI_，数据提供方只具有写的权限。这种方式对批次具有较好地控制，但数据提供方对自己提交的数据无法管理，且不同批次对应的_SAS URI_也需要区分。

### 数据提供方（Data Provider）
**数据提供方**使用_SAS URI_登录，并通过_Microsoft Azure Storage Explorer_上传数据。（详细使用教程请参考：《使用SAS URI登录并传输数据》）

### 技术支持（Technical Support）
由于**Windows Azure**提供了很好的API特性，所以对于数据进行批量处理时，我们建议使用其提供的SDK完成数据的传输。[Windows Azure官方](https://docs.microsoft.com/zh-cn/azure/storage/blobs/storage-quickstart-blobs-python)提供了针对不同语言的使用教程，包括Java，Python，Ruby，Node.js等，我们在这里不再一一说明，仅以Python上传为例：

```python
from azure.storage.blob import BlockBlobService

# 使用账户 account_name/account_key 登录
# endpoint_suffix指示了blob存储的服务器所在区域，一般中国为‘core.chinacloudapi.cn’
# 详细情况请咨询运维
blob_service = BlockBlobService(account_name, account_key， endpoint_suffix='core.chinacloudapi.cn')

# 创建容器：mycontainername，并设置访问权限为公共blob
blob_service.create_container(
    'mycontainername',
    public_access=PublicAccess.Blob
)

# 创建blob：上传/path/to/mylocalfile 到容器mycontainername，并命名为myblobname
blob_service.create_blob_from_path(
    'mycontainername',
    'myblobname',
    '/path/to/mylocalfile'
)

# 打印该blob的url
print(blob_service.make_blob_url('mycontainername', 'myblobname'))
```

[Azure 快速入门 - 使用 Python 在对象存储中创建 blob](https://docs.microsoft.com/zh-cn/azure/storage/blobs/storage-quickstart-blobs-python)提供了更多示例代码及说明，[Azure Storage SDK for Python](http://azure-storage.readthedocs.io/) 提供了完整的API说明，包括下载，拷贝，修改容器权限等。

## 流程&操作指南
### 申请Azure Blob资源
为规范使用公司的各种云资源，和提高云资源利用率，减少资源浪费。公司将对云资源进行集中化管理，优化云资源的审批流程。

针对云资源的申请、开通、关闭、维护等工作，由创新业务部统一管理，负责人：范万强。

#### 申请流程：
1. 申请部门填写《云资源申请单》（如附件）；
2. 写清使用起止时间、用途、资源大小、云资源类别、使用责任人等；
3. 申请单由部门总监审批签字后，需经产品线总裁审批签字；
4. 纸质《云资源申请单》交由创新业务部进行备案，并创建和交付资源。

#### 申请须知：
1. 申请部门需要根据业务情况进行评估，确定虚拟机类型，虚拟机系统及版本，需要安装的服务及版本，存储空间类型，磁盘大小、磁盘个数、虚拟机个数、使用周期；
2. 新项目建议不申请开通虚拟机，在公司内网进行充分测试，正式上线时再开通虚拟机；
3. 使用期限最长为1年，资源到期前一个星期由创新业务部发邮件通知申请人和部门总监，到期后如需继续使用，需要重新申请进行延期；
4. 资源到期而没有延期的，创新业务部将关闭资源，一周后仍没有办理延期将删除并释放资源；
5. 由于资源命名的需要，项目名称后需要备注项目的英文单词或英文字母缩写；
6. 申请人更换或离职，需要提前说明，进行变更，否则申请人将自动变更为部门总监；
7. 每个月5号，创新业务部会将上一个月云资源消耗及详细资费情况发送给各部门总监；
8. 云资源提前结束使用的，请及时通知创新业务部进行删除释放资源；
9. 创新业务部负责云资源后期运维工作；
10. 创新业务部会对云资源进行定期巡检，对有效期内的云资源长期处于空闲状态、或者长期利用率低的，创新业务部有权对资源进行整合、降级或者释放。

### 创建容器


### 权限分配




## 参考资料
* [Microsoft Azure - Wikipedia](https://en.wikipedia.org/wiki/Microsoft_Azure)
* [Azure 存储简介 - Azure 中的云存储](https://docs.microsoft.com/zh-cn/azure/storage/common/storage-introduction)
* [Azure 快速入门 - 使用 Python 在对象存储中创建 blob](https://docs.microsoft.com/zh-cn/azure/storage/blobs/storage-quickstart-blobs-python)
* [容器，blob和Metadata命名参考](https://docs.microsoft.com/zh-cn/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata)
* [用于 Python 的 Azure 存储库](https://docs.microsoft.com/zh-cn/python/api/overview/azure/storage?view=azure-python)

## 附件
* Azure Blob操作手册

* 云资源申请单2018<a href='%E4%BA%91%E8%B5%84%E6%BA%90%E7%94%B3%E8%AF%B7%E5%8D%952018.xlsx'>%E4%BA%91%E8%B5%84%E6%BA%90%E7%94%B3%E8%AF%B7%E5%8D%952018.xlsx</a>

#技术服务中心

