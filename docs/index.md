# 服务使用说明

## MyTopling 数据库实例创建流程

### 1. 开通 [Topling SAAS 弹性计算服务](https://market.aliyun.com/products/56024006/cmgj00064106.html)
<div align="center" >
<img src="img/api-image.PNG" height="200"/>
</div>
<br />
本服务是后续资源创建的基础，如果未开通，则无法继续。

### 2. 创建 [Topling 数据库运行环境](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceId=service-cb1b7a70ed544bbcaa75) 

<div align="center" >
<img src="img/topling-db-env.bmp" height="300"/>
</div>
<br />

为数据库实例准备VPC,每个地域必须创建一次且仅能创建一次。

如果未开通 [Topling SAAS 弹性计算服务](https://market.aliyun.com/products/56024006/cmgj00064106.html) 则 [Topling 数据库运行环境](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceId=service-cb1b7a70ed544bbcaa75)
会创建失败。请删除后重建


### 3. 创建 [MyTopling 数据库实例](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceId=service-7e82cdf7c86f4d2f906e)

MyTopling 实例为用户实际使用的数据库。如图，用户按自身需求填写表单内容，填写完成后即可创建数据库。

用户可以使用[阿里云DMS](https://dms.aliyun.com/) 管理创建的数据库，使用用户名 `mytopling_dms` 密码 `MyToplingDmsPw` 连接新创建的数据库即可。
此用户仅供 DMS 服务连接，在不修改数据库白名单的前提下，除阿里云官方服务外，其他客户端无法使用此账号连接数据库。



### 4. 管理与连接数据库

MyTopling默认创建的安全组没有放行3306端口，但同安全组内可以访问任意端口。

因此，用户可以在 MyTopling 相同安全组内创建实例以连接数据库，或自行放行 10.0.0.0/8 等网段的3306端口。

注意，切勿放行 MyTopling 3306 端口到 0.0.0.0/0 。

如果现有其他 VPC 连接 MyTopling 数据库，可以使用[云企业网](https://cen.console.aliyun.com/cen/list) 或 [对等连接](https://vpc.console.aliyun.com/vpcpeer) 打通网段.

操作方法参见相关文档。


## 产品说明

ToplingDB SAAS系列数据库依赖于由以下三部分组成:
* [Topling SAAS 弹性计算服务](https://market.aliyun.com/products/56024006/cmgj00064106.html) 执行代理运算。在数据库运行的过程中，Topling SAAS 弹性计算服务会按照使用运算量收取费用。

* [Topling 数据库运行环境](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceId=service-cb1b7a70ed544bbcaa75) 
为后续创建 MyTopling 等数据库创建运行环境。包含VPC、vSwitch以及连接到 Topling SAAS 弹性计算服务的对等连接。ToplingDB 系列数据库必须部署在对应地域(Region)的运行环境下。

* [MyTopling 数据库](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceId=service-7e82cdf7c86f4d2f906e)
用户实际使用的数据库。除了数据库本身使用的ECS资源会由阿里云代扣之外，我方会收取数据库调用`Topling SAAS 弹性计算服务`的费用(数据库自动调用，无需用户干涉), 费用按照服务使用量计算，计算规则：
以 ToplingDB 的写放大估计，代理运算量约为总写入量的 10 倍, `Topling SAAS 弹性计算服务` 原价约为每 GiB 0.5 元。

`Topling SAAS 弹性计算服务` 每小时消耗 Unit 数量计算公式:
$ Unit = \sqrt {每小时压缩前数据量(MiB) \times 每小时压缩后数据量(MiB)}$

每个 Unit 收费 0.0005 元

注: 对于一个长时间持续 6MiB/s 写入的数据库, 每小时产生的代理运算量约为 170G, 原价约合 85 元。


## 常见问题

* 实例创建失败
 - 弹性计算服务被关闭
    如果您开通 `Topling SAAS 弹性计算服务` 后关闭了服务将无法继续创建 ToplingDB 相关实例
 - 未开通DMS服务
    如果实例创建完成，但此用户尚未使用过阿里云的 DMS 服务，则可能会导致创建失败。然而此时实例数据库已创建成功，不影响DMS管理数据库之外的功能。
    注意，此时虽然计算巢资源栈创建失败，但 ECS 实例已经创建完成，因此阿里云会收取实例费用。
* 忘记用户名密码
  可以前往DMS控制台，使用用户名 `mytopling_dms` 密码 `MyToplingDmsPw`连接数据库重置。
  注意，不要修改自动创建的用户，如`sync`，`mytopling_dms`等用户。
* 资源栈删除失败
  - 数据库资源栈删除失败
  查看对应安全组(名字以 ros_VpcSecurityGroup_stack 开头)下是否存在其他ECS等资源，如果存在，移出此安全组后重新删除资源栈
  - Topling 数据库运行环境删除失败
    查看 VPC 下是否有未释放的实例以及未删除的 ECS、NAS 挂载点等资源



