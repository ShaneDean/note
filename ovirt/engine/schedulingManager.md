

# 调度策略

管网文档定义：调度策略是一组规则，它定义了虚拟机在应用这个调度的集群中的主机间的分配逻辑。

调度策略主要由下面三类对象组成。
- filter:用来过滤不符合条件的host
- weighting:用来增加符合条件的host的权重
- load balancing:上两者的集合，代表一个执行策略。


以上的三类对象都由policyUnitImpl对象的子类来完成。
filter 的都是xxxUnit
weighting 的都是xxxWightPolicyUnit
load balancing的都是 xxxbalancePolicyUnit


PolicyUnitImpl是一个abstract类，它有很多具体的实现子类。
其中包括CpuPiningPolicyUnit,CompatibilityVersionFilterPolicyUnit等

在InternalPolicyUnits内部定义internal的内可用的策略，放在enabledUnits中。
并且ovirt定义了一个接口注解， SchedulingUnit ，用来标志policyUnit

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface SchedulingUnit {
    String guid(); 
    
    //每个unit名称，可以在手册中的1.5找到描述的功能说明
    String name(); 
    PolicyUnitType type() default PolicyUnitType.FILTER;
    
    //每一个策略单元的具体功能可以通过这个属性来快速了解
    String description() default "";
    PolicyUnitParameter[] parameters() default {};
}
```

在InternalPolicyUnits.instantiate中检测了每一个policyUnit是否在InternalPolicyUnits.enabledUnits中定义、是否被SchedulingUnit注解标志出来



调度策略对应的对象是**ClusterPolicy**，负责保存主机选择和负载均衡的BE。


```
private Guid id;
private String name;
private String description;

//true表示是保留的内置策略
private boolean locked;
//true表示没有可用的特定policy，使用默认
private boolean defaultPolicy;
//可用的filters的id列表
private ArrayList<Guid> filters;
 //上面filter的对应顺序，可接受的value包括  first(-1) | last(1) | no position(0)
private Map<Guid, Integer> filterPositionMap;
//定义了每个权重的分数
private ArrayList<Pair<Guid, Integer>> functions;
//一个loadbalance
private Guid balance;
//提供的参数集
private Map<String, String> parameterMap;
```










## SchedulingManager
继承自BackendService，主要作用于系统的集群调度策略。

### 成员分析

```
@Inject
private AuditLogDirector auditLogDirector;
@Inject
private ResourceManager resourceManager;//todo
@Inject
private MigrationHandler migrationHandler;//todo
@Inject
private ExternalSchedulerDiscovery exSchedulerDiscovery;//用来发现外部的policy
@Inject
private DbFacade dbFacade;//用来返回vds\cluster\policyunit\clusterpolicy等dao来查询数据库
@Inject
private NetworkDeviceHelper networkDeviceHelper;//todo
@Inject
private HostDeviceManager hostDeviceManager;//todo

private PendingResourceManager pendingResourceManager;

/**
 * [policy id, policy] map
 */
private final ConcurrentHashMap<Guid, ClusterPolicy> policyMap;
/**
 * [policy unit id, policy unit] map
 */
private volatile ConcurrentHashMap<Guid, PolicyUnitImpl> policyUnits;

private final Object policyUnitsLock = new Object();

private final ConcurrentHashMap<Guid, Semaphore> clusterLockMap = new ConcurrentHashMap<>();

private final VdsFreeMemoryChecker noWaitingMemoryChecker = new VdsFreeMemoryChecker(new NonWaitingDelayer());

private final Map<Guid, Boolean> clusterId2isHaReservationSafe = new HashMap<>();

```




### 初始化分析


```
    @PostConstruct
    public void init() {
        log.info("Initializing Scheduling manager");
        //负责获取各类变化的资源情况，操作对象是所有继承PendingResource类的资源类，比如CPU、内存等
        initializePendingResourceManager();
        //加载内部外部policyUnit;内部在enabledUnits里面定义;外部在db里面查询;
        loadPolicyUnits();
        //加载内部外部ClusterPolicies;内部由createBuilder构建;外部在db里面查询;
        loadClusterPolicies();
        //加载外部的scheduler,默认关闭;如果发现就重新加载policyUnit
        loadExternalScheduler();
        //启动loadBalance,默认开启;使用quartz定时器调用performLoadBalancing();
        enableLoadBalancer();
        //启用HA检查，默认开启;使用quartz定时器调用performHaResevationCheck();
        enableHaReservationCheck();
        log.info("Initialized Scheduling manager");
    }
```






## PendingResource
用作所有trackable pending resource一个基础类模板
实现一个静态方法给PendingResourceManager来调用，完成收集每台host和VM的数据信息。

子类包括
- PendingCpuCores   
核心数量
- PendingMemory     
host内存大小，单位MB
- PendingOvercommitMemory
- PendingVm 

### PendingResourceManager


```
//Tracking service for all pending resources
public class PendingResourceManager{
    ...
    // All internal structures have to be thread-safe for concurrent access
    private final Map<Guid, Set<PendingResource>> resourcesByHost = new ConcurrentHashMap<>();
    private final Map<Guid, Set<PendingResource>> resourcesByVm = new ConcurrentHashMap<>();
    private final Map<PendingResource, PendingResource> pendingResources = new ConcurrentHashMap<>();
    
    private final ResourceManager resourceManager;
    
    ...
    
    public void addPending(PendingResource resource) { ... }
    
    public void clearHost(VDS host) { ... }
    
    public void clearVm(VmStatic vm) { ... } 
    
    public <T extends PendingResource> Iterable<T> pendingResources(Class<T> type) { ... }
    
    public <T extends PendingResource> Iterable<T> pendingHostResources(Guid host, Class<T> type) { ... }
    
    public <T extends PendingResource> Iterable<T> pendingVmResources(Guid vm, Class<T> type) { ... } 
    
    public void notifyHostManagers(Guid hostId) { ... }
    
    public <T extends PendingResource> T getExactPendingResource(T template) { ... }
}

```



### InternalPolicyUnits



### InternalClusterPolicies


通过createBuilder来构建了5个预定义的集群策略。代码逻辑如下：

```
createBuilder("xxxx-a7ac-4d5f-xxxx")            //传入uuid
    .name("NAME")                               //传入集群策略名称
    .isDefault()                                //制定当前策略是默认策略
    .setBalancer(XXXBalancePolicyUnit.class)    //设置balancer

    .addFilters(XXXPolicyUnit.class)            //设置filter
    ...

    .addFunction(1, xxxWeightPolicyUnit.class)  //设置weight和权重值
    ...
    
    .set(PolicyUnitParameter.XXX , 'VALUE')     //设置调度策略的参数和值
    
    .register();                                //注册集群调度策略到InternalClusterPolicies里面内置的clusterPolicies中
        
```

通过这种方式定义的集群调度策略有5个：
- Evenly_Distributed
- InClusterUpgrade
- None
- Power_Saving
- VM_Evenly_Distributed



## 涉及的entity

- AffinityGroup ：
    -  保留一些VMs和属性。
    -  每个VM可以关联几个AffinityGroup
    -  每个VM由调度逻辑取决于它所属的AffinityGroup的规则
- ClusterPolicy :
    -   保存主机选择和load balancing的逻辑
    -   内部clusterPolicy是由ovirt内置存在的,用户可以自己添加外部的ClusterPolicy,由is_locked的字段来标识这个区别
-  PolicyUnit
    -  filter,weight,balance三类方法
-  PerHostMessages
    -       Map<Guid, List<String>> message; //保存各个host的消息list
-   OptimizationType
    -   定义集群优化的三种方案：none,optimize_for_speed,allow_overbooking
    -   //TODO 三者区别

