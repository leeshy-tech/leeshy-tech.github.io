# 鸿蒙开发实践——电话簿


# 电话簿

## 项目简介

### 项目结构

```
├─ entry.src.main
	├─ com.example.address_book
        ├─slice
            ├─ UserAddSlice
            ├─ UserListSlice
            └─ MainAbilitySlice
        ├─ DataBaseAbility	
        ├─ MainAbility
        └─ MyApplication
	└─ resources.base.layout
        ├─ ability_main.xml
        ├─ user_add.xml
        └─ user_list.xml
```

### 效果

![20220208_162723](/image/address_book/20220208_162723.gif)

## 数据库结构

- 数据库UserStore：本地、关系型
  - 表users
    - 属性userId：int、主键、自增
    - 属性userName：text、不为空
    - 属性userTel：text、唯一
    - 属性userAddr：text

## MainAbilitySlice

核心代码：

**MainAbilitySlice的onStart方法：**

```java
Button btn1 = (Button) findComponentById(ResourceTable.Id_btn1);
btn1.setClickedListener(listener->present(new UserAddSlice(),new Intent()));

Button btn2 = (Button) findComponentById(ResourceTable.Id_btn2);
btn2.setClickedListener(listener->present(new UserListSlice(),new Intent()));
```

执行简单的页面跳转。

## DataBaseAbility

核心代码：

**DataBaseAbility类：**

```java
private RdbStore rdbStore;
private StoreConfig config = StoreConfig.newDefaultConfig("UserStore.db");
```

RdbStore对象：表示与数据库的连接，通过此对象可以完成对数据表中数据的CRUD操作
StoreConfig对象：关联数据⽂件配置(数据库)

```java
private RdbOpenCallback callback = new RdbOpenCallback() {
    @Override
    public void onCreate(RdbStore rdbStore) {
        //使⽤rdbStore对象执⾏SQL创建数据表
        rdbStore.executeSql("create table if not exists users(" +
                "userId integer primary key autoincrement," +
                "userName text not null," +
                "userTel text not null unique," +
                "userAddr text)");
    }
};
```

RdbOpenCallback.onCreate()：数据库创建时被回调，初始化，创建数据表users（当其不存在时）。

**DataBaseAbility类的onStart方法：**

```Java
@Override
public void onStart(Intent intent) {
    super.onStart(intent);
    HiLog.info(LABEL_LOG, "DataBaseAbility onStart");

    //初始化与数据库的连接
    DatabaseHelper helper = new DatabaseHelper(this);
    rdbStore = helper.getRdbStore(config,1,callback);
}
```

**重写insert方法：**

```java
public int insert(Uri uri, ValuesBucket value) {
    int i = -1;
    String path = uri.getLastPath();
    if("users".equalsIgnoreCase(path)){
        i = (int)rdbStore.insert("users",value);
    }
    return i;
}
```

**重写query、delete、update方法：**

```Java
public ResultSet query(Uri uri, String[] columns, DataAbilityPredicates predicates) {
    RdbPredicates rdbPredicates = DataAbilityUtils.createRdbPredicates(predicates, "users");
    ResultSet resultSet = rdbStore.query(rdbPredicates, columns);
    return resultSet;
}
public int delete(Uri uri, DataAbilityPredicates predicates) {
    RdbPredicates rdbPredicates = DataAbilityUtils.createRdbPredicates(predicates, "users");
    int i = rdbStore.delete(rdbPredicates);
    return i;
}

public int update(Uri uri, ValuesBucket value, DataAbilityPredicates predicates) {
    RdbPredicates rdbPredicates = DataAbilityUtils.createRdbPredicates(predicates, "users");
    int i = rdbStore.update(value, rdbPredicates);
    return i;
}
```

## UserAddSlice

核心代码：

```Java
public class UserAddSlice extends AbilitySlice {
    private DataAbilityHelper dataAbilityHelper;
    @Override
    public void onStart(Intent intent) {
        super.onStart(intent);
        super.setUIContent(ResourceTable.Layout_user_add);
        dataAbilityHelper = DataAbilityHelper.creator(this);
        //获取组件对象
        Button btn_add = (Button) findComponentById(ResourceTable.Id_btn_add);
        TextField tf1 = (TextField) findComponentById(ResourceTable.Id_textField1);
        TextField tf2 = (TextField) findComponentById(ResourceTable.Id_textField2);
        TextField tf3 = (TextField) findComponentById(ResourceTable.Id_textField3);
        //绑定事件监听器
        btn_add.setClickedListener(component -> {
            String userName = tf1.getText();
            String userTel = tf2.getText();
            String userAddr = tf3.getText();
            //构造VB
            ValuesBucket valuesBucket = new ValuesBucket();
            valuesBucket.putString("userName",userName);
            valuesBucket.putString("userTel",userTel);
            valuesBucket.putString("userAddr",userAddr);
            //插入数据
            try{
                Uri uri = Uri.parse("dataability:///com.example.address_book.DataBaseAbility/users");
                int i = dataAbilityHelper.insert(uri,valuesBucket);
                System.out.println("------------>>>>>>"+i);
            }catch (DataAbilityRemoteException e){
                e.printStackTrace();
            }
        });
    }
}
```

- 数据库的相关操作要依靠dataAbilityHelper。

- 从TextField组件中获取输入的信息。

- 设置监听器，将获得的信息插入数据表。

- 插入数据通过ValuesBucket储存，指数据表的一行。


## UserListSlice

  核心代码：

```Java
public class UserListSlice extends AbilitySlice {
    private DataAbilityHelper dataAbilityHelper;

    @Override
    protected void onStart(Intent intent) {
        super.onStart(intent);
        this.setUIContent(ResourceTable.Layout_user_list);

        Text text = (Text) findComponentById(ResourceTable.Id_infoText);
        text.setText("");
        dataAbilityHelper = DataAbilityHelper.creator(this);
        //查询所有联系⼈信息
        Uri uri = Uri.parse("dataability:///com.example.address_book.DataBaseAbility/users");
        String[] colums = {"userId","userName","userTel","userAddr"};
        DataAbilityPredicates dataAbilityPredicates = new DataAbilityPredicates();
        try {
            ResultSet rs = dataAbilityHelper.query(uri,colums,dataAbilityPredicates);
            //从rs中获取查询结果
            int rowCount = rs.getRowCount();
            if(rowCount>0){
                rs.goToFirstRow();
                do{
                    int userId = rs.getInt( 0);
                    String userName = rs.getString(1);
                    String userTel = rs.getString(2);
                    String userAddr = rs.getString(3);
                    String info = " ["+userId+","+userName+","+userTel+","+userAddr+"]";
                    text.setText( text.getText()+info );
                }while(rs.goToNextRow());
            }
        } catch (DataAbilityRemoteException e) {
            e.printStackTrace();
        }
    }
}
```

- 通过调用查询接口来查询。
- ResultSet为查询结果集，类似于指针，指向结果的第一行。

## 结束语

### 项目源码

[https://github.com/leeshy-tech/HarmonyOS_example/tree/main/address_book](https://github.com/leeshy-tech/HarmonyOS_example/tree/main/address_book)

### 参考文献

[HarmonyOS文档——关系型数据库](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/database-relational-overview-0000000000030046)	

[HarmonyOS 2.0应用开发实战教程丨锋迷商城项目](https://www.bilibili.com/video/BV1DM4y1G7MN)

