# NewDemo

2.	框架介绍
2.1.	核心项目简介
1.	Rundong.Infrastructure
基础层项目，提供容器依赖注入，AOP拦截，日志记录，后台任务，缓存管理，上下文，本地化（多语言）及其它工具类。
2.	Rundong.Framework
核心框架项目，提供领域实体基类，领域事件，单元仓储，仓储上下文，单元服务基类，应用服务基类，DTO基类等。
3.	Rundong.Web.Framework
前端Web框架项目，提供控制器基类，视图基类，Model配置基类，Web上下文实现，Ext.net控件扩展。
2.2.	应用项目简介
以系统设置应用(Rundong.Sys)为例
1.	Rundong.Sys.Domain
引用依赖：无
领域层：定义领域实体，事件，服务，单元仓储接口等
2.	Rundong.Sys.Repositories
引用依赖：Rundong.Sys.Domain
仓储层：实现领域层单元仓储。
3.	Rundong.Sys.Application
引用依赖：Rundong.Sys.ServiceContracts，Rundong.Sys.Domain
服务应用层：实现服务契约。
4.	Rundong.Sys.ServiceContracts
引用依赖：无
服务契约层：定义服务契约及DTO（数据传输对象）
5.	Rundong.Sys.Web
引用依赖：Rundong.Sys.ServiceContracts
Web UI层：界面显示及交互实现。


3.	简要开发步骤及核心类说明
3.1.	领域实体及仓储接口定义
要实现某个功能，首先要做的就是根据需求在Domain项目中建立领域实体和仓储接口。
1.	实体基类介绍
1)	BaseEntity实体抽象基类
命名空间: Rundong
程序集：Rundong.Framework
实现接口: IEntity
		   所有实体均从此类继承或派生。
2)	BaseEntityLog 带日志字段的实体基类
命名空间: Rundong
程序集：Rundong.Framework
继承：BaseEntity
实现接口: IEntityLog
带有创建人员Id，创建时间，最后修改人员Id，最后修改时间四个属性
3)	BaseAggregateRoot 聚合根抽象基类
命名空间: Rundong
程序集：Rundong.Framework
继承：BaseEntityLog
实现接口: IAggregateRoot
4)	BaseAggregateRootLog带Log导航的聚合根抽象基类
命名空间: Rundong.Sys.Domain
程序集：Rundong.Sys.Domain
继承：BaseAggregateRoot
实现接口: ILogNavigationEntity
带有创建人员，最后修改人员导航属性的基类
5)	WorkflowBill 工作流表单基类
命名空间: Rundong.Workflow.Domain
程序集：Rundong.Workflow.Domain
继承：BaseAggregateRootLog
实现接口: IWorkflowBill,IUserOrganizationAccess
所有工作流审核对象(表单或档案)的基类

2.	实体定义
根据需求文档，确定实体类型：聚合根（如：表单）还是实体（如：表单明细），实体属性（字段）及与其它的实体的关系（一对一，一对多，多对多），然后从上面实体基类中选择合适基类的继承。
P.S.  导航属性一定定义为virtual的，否则EF无法实现延迟加载功能。

3.	仓储基类接口介绍
a)	IRepository <TEntity>  泛型单元仓储
命名空间: Rundong.Repositories
程序集：Rundong.Framework
泛型约束: where TEntity : class, IEntity

b)	IStatusEntityRepository <TEntity>状态实体仓储接口
命名空间: Rundong.Repositories
程序集：Rundong.Framework
继承: IRepository <TEntity>
泛型约束: where TEntity : class, IStatusEntity

4.	仓储接口定义
大部分聚合根及实体，如果IRepository <TEntity>等泛型仓储能满足要求，可以不定义单独的仓储接口，但如果是工作流实体(从WorkflowBill继承)则需定义单独的仓储接口（从IRepository <TEntity>等泛型继承），详情见仓储接口实现。

3.2.	仓储接口实现及实体Map
Domain项目中的仓储接口实现及实体与数据库如何映射均在Repositories项目中完成。
1.	基类仓储介绍

a)	EFRepository<TEntity>  泛型单元仓储
命名空间: Rundong.Repositories
程序集：Rundong.Framework
实现接口: IRepository<TEntity>
泛型约束: where TEntity : class, IEntity

P.S. 所有工作流类型实体仓储均需从此类继承，不能使用EFRepository <TEntity>的泛型实现。




2.	仓储上下文
a)	IRepositoryContext
命名空间: Rundong.Repositories
程序集：Rundong.Framework
继承: IUnitOfWork，IDisposable
可以注册实体状态(新增，修改，删除)，及事务提交。
b)	IEFRepositoryContext 
命名空间: Rundong.Repositories
程序集：Rundong.Framework
继承: IRepositoryContext
EntityFramework 仓储上下文接口，可执行原始SQL及存储过程。
3.		实体Map基类
a)	BaseMap<TEntity>
命名空间: Rundong.Repositories
程序集：Rundong.Framework
继承: EntityTypeConfiguration<TEntity>
    泛型约束: where TEntity :  class,IEntityLog
b)	BaseMapLog<TEntity>
命名空间: Rundong.Sys.Repositories.Mapping
程序集：Rundong.Sys.Repositories
继承: BaseMap <TEntity>
    泛型约束: where TEntity :  class, ILogNavigationEntity

3.3.	服务契约接口及DTO定义
服务契约是UI与应用服务层间沟通的接口，DTO是UI与应用服务层的数据传输对象。服务契约接口及DTO都定义在ServiceContracts项目中。
1.	契约接口基类：IapplicationServiceContract，所有契约接口均从此继承
2.	DTO基类：
a)	BaseDto
命名空间: Rundong.Infrastructure
程序集：Rundong.Infrastructure	
b)	BaseDtoLog
命名空间: Rundong.Infrastructure
程序集：Rundong.Infrastructure
3.4.	应用服务实现及DTO与实体映射
应用服务层（Application）是具体业务事务逻辑的实现。
1.	应用服务基类
a)	BaseApplicationService
命名空间: Rundong.Services
程序集：Rundong.Framework
应用服务抽象基类,提供仓储上下文，工作上下文，及本地化服务接口
b)	BaseUnitService<TEntity, TDto>
命名空间: Rundong.Services
程序集：Rundong.Framework
继承: BaseApplicationService
泛型约束: where TEntity : class, IentityLog where TDto : BaseDto
单元服务基类，提供基本的增，删，改，查操作。
c)	ApplicationService<TAggregateRoot, TDto>
命名空间: Rundong.Services
程序集：Rundong.Framework
继承: BaseUnitService<TEntity, TDto>
泛型约束: where TAggregateRoot : class, IAggregateRoot, IentityLog where TDto : BaseDto
        聚合根应用服务泛型抽象基类,提供带权限控制的查询，修改，删除及删除前的引用检测。
P.S. 所有应用服务需重写FuncCode属性。
2.	DTO和实体映射基类
a)	DtoAndEntityBaseMap<TDto, TEntity>
命名空间: Rundong
程序集：Rundong.Framework
泛型约束: where TDto : class, IDto where TEntity : class, IEntity

b)	DtoAndEntityLogMap<TDto, TEntity>
命名空间: Rundong.Sys.Application
程序集：Rundong.Sys.Application
泛型约束: where TTargetEntity : class, IlogNavigationEntity 
where TTargetDto : class, IDtoLog

3.5.	控制器及视图实现
1.	控制器基类
a)	BaseController
命名空间：Rundong.Web.Framework
控制器基类，提供工作上下文及本地化服务。
b)	BaseAppController
命名空间：Rundong.Web.Framework
继承：BaseController

	需登录才能访问的控制器，提供权限控制，非特殊页面都从此控制器继承。

2.	控制器Action命名规则
新增方法:以Create命名开头，修改方法以Edit开头

3.	视图基类
所有视图基类的命名空间都是: Rundong.Web.Framework
a)	BaseViewPage
视图基类，提供上下文，本地化，查询方案，自定列待功能，是所有视图的基类。
b)	BaseListViewPage
列表查询页面视图基类。
c)	ListViewPage
列表维护界面视图（基类）
d)	FormInTabViewPage
可编辑表单Form(Tab页编辑)视图基类
4.	Model配置基类
命名空间：Rundong.Web.Framework
a)	BaseModelConfig<TModel>
Model配置抽象基类，主要提供本地化服务功能
b)	BaseDtoConfig<TModel>
日志DTO Model 配置基类，主要配置日志DTO(创建时间，创建人员，修改人员，修改时间)这个字段的本地化显示
c)	WorkflowBillDtoConfig<TModel>
工作流表单Dto配置基类
4.1.	核心JS类简要说明
所有公共JS库存放在Scripts\Rundong目录下，下面列出一些主要及常用类，其它类请参见目录下源文件。
1.	Rundong.core.BaseApp
所有页面App的基类，提供组件查找，容器加载，Ajax访问，表单提交等基础功能。
2.	Rundong.core.BaseToolbarApp
带工具栏（toolbar）的基类APP，主要提供了工具栏按钮点击事件处理。
3.	Rundong.core.BaseListpageApp
列表查询界面App（grid+toolbar)，主要提供Grid查询，绑定万用查询面板，及列表操作（启用/停用，删除，审核/反审核）等处理
4.	Rundong.core.ListpageApp
列表维护(新增，修改)界面App，主要提供开窗或打开新Tab页做新增，修改待操作
5.	Rundong.core.FormInTab
在Tab中新增，编辑页面的App,主要提供新增，修改数据验证及保存及页面跳转，删除，审核等功能。
6.	Rundong.core.BasePopups
所有弹窗的基类，提供弹窗加载，初始化待基本功能。
7.	Rundong.core.EditformPopups
编辑弹窗基类，提供数据验证及保存等功能
5.	工作流应用简要说明
1.	工作流表单类型配置说明。
在定义工作流表单实体及对应的DTO后，需在表Wf_WorkflowSysBillType中手工添加表单类型数据。如下图：

 
BillTypeCode：	指定为表单对应的程序功能代码
DtoAssemblyQualifiedName：DTO对应Type全名
EntityAssemblyQualifiedName：工作流表单实体对应的Type全名


2.	工作流表单基类重写说明
重写GetBillCode()方法，返回工作流表单的单据编码字段。
重写SysBillTypeCode属性，返回该表单对应的程序的功能代码。
 

3.	工作流事件对象说明
命令空间：Rundong.Workflow.Domain.Events
a)	WorkflowBillStartAuditEvent<T>
表单开始审核事件
b)	WorkflowBillAuditedEvent<T>
表单审核事件
c)	WorkflowBillUnAuditedEvent<T>
表单反审核事件
4.	编译及发包
1.	预编译命令：
打开命令提示符,将下面命令中的中文字符替换为实际的目录及文件后执行即可。

%windir%\Microsoft.NET\Framework64\v4.0.30319\aspnet_compiler.exe -v / -p 待编译目录 -f 编译后目录 -fixednames

"%programfiles(x86)%\Microsoft SDKs\Windows\v7.0A\Bin\NETFX 4.0 Tools\Aspnet_merge.exe" 编译后目录 -o 编译后视图文件名.dll

2.	关闭IIS进程回收
选中待更改的应用程序池，点击右边的“高级设置”,弹出如下窗口：
将”固定时间间隔”和”闲置超时”更改为0.

