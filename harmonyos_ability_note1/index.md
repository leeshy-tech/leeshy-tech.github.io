# 鸿蒙开发笔记——Ability（上）

# Ability

{{< admonition type=quote title="HarmonyOS文档" open=true >}}
Ability是应用所具备能力的抽象，也是应用程序的重要组成部分。一个应用可以具备多种能力（即可以包含多个Ability），HarmonyOS支持应用以Ability为单位进行部署。
{{< /admonition >}}

Ability相当于功能集合，把实现某个具体功能的逻辑归类到一起。分类如下：    

```
└─ Ability                     
    ├─ FA（Feature Ability）   
        └─ Page Ability：页面，可以与用户交互。     
    └─ PA（Particle Ability）
        ├─ Service Ability：后台任务。      
        └─ Data Ability：统一数据访问抽象。     
```
## 注册Ability
要使用Ability必须要在config.json里进行配置，格式如下:   
```json
{
    "module": {
        "abilities": [
            {
                "type": "page"
            }
        ]
    }
}
```
配置项：[abilities--应用配置文件](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/basic-config-file-elements-0000000000034463#ZH-CN_TOPIC_0000001064016070__table593616515457)
> 并且，要想删除一个Ability，需要将配置信息同步删除，否则编译会失败。

注意：如果通过以下方式新建Ability，则系统会自动注册Ability，不需要手动更改config.json：

![image-20220129161614782](/image/image-20220129161614782.png)

如果通过新建Java类来新建Ability，则需要手动注册Ability。

## Page Ability
> 一个Page由一个或多个AbilitySlice构成，AbilitySlice是指应用的单个页面及其控制逻辑的总和。  

Page是Slice的集合，Slice = 页面 + 控制逻辑

### Ability生存周期
如图所示：  ![生存周期](/image/生存周期.png) 

- onStart()
  当系统首次创建Page实例时触发。开发者必须重写该方法，并在此配置默认显示的Slice。
  
    ```Java
    @Override
    public void onStart(Intent intent) {
        super.onStart(intent);
        super.setMainRoute(FooSlice.class.getName());
    }
    ```
  
- onActive()
  Page进入ACTIVE状态时调用，Page保持在此状态除非：应用进入后台或导航到其他页面，从而进入INACTIVE状态。
  
- onInactive()
  Page进入INACTIVE状态时调用。
  
- onBackground()
  Page由前台进入后台时调用，其调用之前一定会调用onInactive()。
  
- onForeground()
  Page由后台进入前台时调用，其调用之后一定会调用onActive()。
  
- onStop()
  系统销毁Page时调用。

### AbilitySlice生命周期

> AbilitySlice和Page具有相同的生命周期状态和同名的回调，当Page生命周期发生变化时，它的AbilitySlice也会发生相同的生命周期变化。此外，AbilitySlice还具有独立于Page的生命周期变化，这发生在同一Page中的AbilitySlice之间导航时，此时Page的生命周期状态不会改变。

开发者必须重写AbilitySlice的onStart()回调，并在此方法中通过setUIContent()方法设置页面，如下所示：
```Java
    @Override
    protected void onStart(Intent intent) {
        super.onStart(intent);

        setUIContent(ResourceTable.Layout_main_layout);
    }
```
> setUIContext( )提供了两个重载：	
>
> setUIContext( int )：通过布局文件的ID，加载layout目录下的xml文件完成页面渲染。
>
> setUIContext(  ComponentContainer )：加载使用Java创建的组件完成页面渲染。

### Slice间导航

详见另一篇博客[https://leeshy-tech.github.io/harmonyos_pagetopage/](https://leeshy-tech.github.io/harmonyos_pagetopage/)

## 结束语
### 参考文献
[HarmonyOS文档](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/ability-ability-overview-0000000000029852)
