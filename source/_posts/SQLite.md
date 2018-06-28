---
title: SQLite
date: 2016-11-21 17:55:15
tags: [sql,SQLite,数据库,stetho]
---
Android上的自带数据库SQLite操作，虽然`GreenDao`和`ORMLite`都挺厉害的，封装的也都挺好的。不过本篇文章不是介绍这两个开源库，简单介绍了下Android原生数据库的操作

### 数据库基本操作

##### 1.首先创建数据库类 SQLiteOpenHelper
<font size="3" color="green">就直接上代码吧</font>

<!--more-->

``` java
public class PermissionHelper extends SQLiteOpenHelper {
    /**
     * 数据库名
     */
    public static final String DB_NAME = "privacymanager.db";
    /**
    数据库版本
    */
    public static final int DB_VERSION = 1;
    /**
    * 表名
    */
    public static final String TABLE_PERMISSION = "Permission";
    public static final String permissionName = "permissionName";
    public static final String permissionCName = "permissionCName";
    /**
     * 应用权限表
     */
    public static final String TABLE_APP_PERMISSION = "AppPermission";
    public static final String id = "id";
    public static final String appUid = "uid";
    public static final String permission = "permission";
 
    /**
     * 创建表sql语句
     */
    private static final String SQL_CREATE_TABLE_PERMISSION = "CREATE TABLE " + TABLE_PERMISSION + "(" + //
            id + " INTEGER PRIMARY KEY AUTOINCREMENT, " +//
            permissionCName + " VARCHAR, " +//
            permissionName + " VARCHAR)";
    private static final String SQL_CREATE_TABLE_APP_PERMISSION = "CREATE TABLE " + TABLE_APP_PERMISSION + "(" + //
            id + " INTEGER PRIMARY KEY AUTOINCREMENT, " +//
            appUid + " VARCHAR, " +//
            permission + " VARCHAR)";
  
 
    public PermissionHelper(Context context) {
        /**
         * 必须有构造方法，参数分别是 上下文、数据库名字、游标工厂（一般null为默认）、数据库版本（数据库的版本必须大于0，否则报错）
         */
        super(context, DB_NAME, null, DB_VERSION);
    }
 
    @Override
    public void onCreate(SQLiteDatabase db) {
        /**
         * 主动开启事务执行sql， 没有开启事务，系统会默认每执行一句sql语句就会开启事务
         * 在第一次打开数据库的时候才会走
         * 在清除数据之后再次运行-->打开数据库，这个方法会走
         * 没有清除数据，不会走这个方法
         * 数据库升级的时候这个方法不会走
         *
         */
        db.beginTransaction();
        try {
            db.execSQL(SQL_CREATE_TABLE_PERMISSION);
            db.execSQL(SQL_CREATE_TABLE_APP_PERMISSION);
            db.setTransactionSuccessful();
        } finally {
            db.endTransaction();
        }
 
 
    }
 
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        /**
         * 数据库更新操作
         */
//        if (newVersion != oldVersion) {
//            db.beginTransaction();
//            try {
//                db.execSQL(SQL_DELETE_TABLE_APP_PERMISSION);
//                db.execSQL(SQL_DELETE_TABLE_PERMISSION);
//                db.execSQL(SQL_CREATE_TABLE_APP_PERMISSION);
//                db.execSQL(SQL_CREATE_TABLE_PERMISSION);
//                db.setTransactionSuccessful();
//            } finally {
//                db.endTransaction();
//            }
//        }
    }
 
    @Override
    public void onDowngrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        /**
         * 降级操作
         */
        onUpgrade(db, oldVersion, newVersion);
    }
}
```
##### 2.一般可以创建Manager去管理数据库的操作（设计比较简单 ，这里没有对数据库加锁操作）
<font size="3" color="green">还是用代码说明吧</font>

``` java
public class PermissionDBManager {
    private Context mContext;
    private PermissionHelper mPermissionHelper;
    public PermissionDBManager(Context context) {
        this.mContext = context;
        mPermissionHelper = new PermissionHelper(context);
    }
    /**
     * 数据库插入方法
     * @param uid
     * @param permission
     */
    public void add(String uid, String permission) {
        SQLiteDatabase db = null;
        try {
            db = mPermissionHelper.getWritableDatabase();
            ContentValues values = new ContentValues();
            values.put(PermissionHelper.appUid, uid);
            values.put(PermissionHelper.permission, permission);
            db.insert(PermissionHelper.TABLE_APP_PERMISSION, null, values);
        } catch (Exception e) {
            // TODO: handle exception
        } finally {
            if (db != null) {
                db.close();
            }
        }
    }
    /**
     * 根据uid 查询所有的权限
     *
     * @param uid
     * @return
     */
    public List<String> searchPermission(String uid) {
        List<String> list = new ArrayList<>();
        SQLiteDatabase db = null;
        Cursor cursor = null;
        try {
            db = mPermissionHelper.getReadableDatabase();
            cursor = db.query(PermissionHelper.TABLE_APP_PERMISSION, null, PermissionHelper.appUid + "=?", new String[]{uid}, null, null, null);
            while (cursor.moveToNext()) {
                String permission = cursor.getString(cursor.getColumnIndex(PermissionHelper.permission));
                list.add(permission);
            }
        } catch (Exception e) {
            // TODO: handle exception
        } finally {
            cursor.close();
            db.close();
        }
        return list;
    }
    /**
     * 根据权限查询所有uid
     *
     * @param permission
     * @return
     */
    public List<String> searchUid(String permission) {
        List<String> list = new ArrayList<>();
        SQLiteDatabase db = null;
        Cursor cursor = null;
        try {
            db = mPermissionHelper.getReadableDatabase();
            cursor = db.query(PermissionHelper.TABLE_APP_PERMISSION, null, PermissionHelper.permission + "=?", new String[]{permission}, null, null, null);
            while (cursor.moveToNext()) {
                String uid = cursor.getString(cursor.getColumnIndex(PermissionHelper.appUid));
                list.add(uid);
            }
        } catch (Exception e) {
            // TODO: handle exception
        } finally {
            cursor.close();
            db.close();
        }
        return list;
    }
    public void delete(String permission, String uid) {
        SQLiteDatabase db = null;
        try {
            db = mPermissionHelper.getWritableDatabase();
            String whereClause = "uid=? and permission =?";
            db.delete(PermissionHelper.TABLE_APP_PERMISSION, whereClause, new String[]{uid, permission});
        } catch (Exception e) {
            // TODO: handle exception
        } finally {
            if (db != null) {
                db.close();
            }
        }
    }
    /**
     * 批量插入方法
     * @param map
     */
    public void addCPermission(Map<String, String> map) {
        SQLiteDatabase db = null;
        try {
            db = mPermissionHelper.getWritableDatabase();
            String sql = "insert into" + PermissionHelper.TABLE_PERMISSION + "(permissionName,permissionCName) values(?,?)";
            SQLiteStatement stat = db.compileStatement(sql);
            db.beginTransaction();
            for (Map.Entry<String, String> mEntry : map.entrySet()) {
                //循环所要插入的数据
                stat.bindString(1, mEntry.getKey());
                stat.bindString(2, mEntry.getValue());
                stat.executeInsert();
            }
            db.setTransactionSuccessful();
            db.endTransaction();
        } catch (Exception e) {
            // TODO: handle exception
        } finally {
            if (db != null) {
                db.close();
            }
        }
    }
 }
```
 以上，即可一个简单的数据库操作，写的比较简单
### 数据库更新策略

我们知道在SQLiteOpenHelper的构造方法:
``` java
super(Context context, String name, SQLiteDatabase.CursorFactory factory, int version)
```
<font color="blue">中最后一个参数表示数据库的版本号.当新的版本号大于当前的version时会调用方法onUpgrade。</font>
``` java
@Override
onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)
```
所以我们的重点是在该方法中实现***SQLite数据库版本升级***的管理,当我们项目刚开始的时候第一版SQLiteOpenHelper是这样写的:
``` java
/**
 * SQLite数据库版本升级的管理实现
 */
public class DataBaseOpenHelper extends SQLiteOpenHelper {
/**
 * 数据库名
 */
  private final static String DATABASE_NAME="test.db";
  private static DataBaseOpenHelper mDataBaseOpenHelper;
  //创建person表
  public static final String CREATE_PERSON=
  "create table person(personid integer primary key autoincrement,name varchar(20),phone VARCHAR(12))";
  
  public DataBaseOpenHelper(Context context,String name,CursorFactory factory,int version) {
    super(context, name, factory, version);
  }
  //注意:
  //将DataBaseOpenHelper写成单例的.
  //否则当在一个for循环中频繁调用openHelper.getWritableDatabase()时
  //会报错,提示数据库没有执行关闭操作
  static synchronized DataBaseOpenHelper getDBInstance(Context context) {
    if (mDataBaseOpenHelper == null) {
      mDataBaseOpenHelper = new DataBaseOpenHelper(context,DATABASE_NAME,null,1);
    }
    return mDataBaseOpenHelper;
  } 
  @Override
  public void onCreate(SQLiteDatabase db) {
    db.execSQL(CREATE_PERSON);
  }
  @Override
  public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
  }
} 
```


在几天之后根据项目需求，<font color="red">需要添加一张student表，于是DataBaseOpenHelper就出现了第二版:</font>

``` java
public class DataBaseOpenHelper extends SQLiteOpenHelper {
//数据库名
  private final static String DATABASE_NAME="test.db";
  private static DataBaseOpenHelper mDataBaseOpenHelper;
  //创建person表
  public static final String CREATE_PERSON=
  "create table person(personid integer primary key autoincrement,name varchar(20),phone VARCHAR(12))";
  //创建student表
  public static final String CREATE_STUDENT=
  "create table student(studentid integer primary key autoincrement,name varchar(20),phone VARCHAR(12))";

  public DataBaseOpenHelper(Context context,String name,CursorFactory factory,int version) {
    super(context, name, factory, version);
  }
  //注意:
  //将DataBaseOpenHelper写成单例的.
  //否则当在一个for循环中频繁调用openHelper.getWritableDatabase()时
  //会报错,提示数据库没有执行关闭操作
  static synchronized DataBaseOpenHelper getDBInstance(Context context) {
    if (mDataBaseOpenHelper == null) {
     /*
      * TODO: 改动1 更改数据版本
      */
      mDataBaseOpenHelper = new DataBaseOpenHelper(context,DATABASE_NAME,null,2);
    }
    return mDataBaseOpenHelper;
  } 
  @Override
  public void onCreate(SQLiteDatabase db) {
    db.execSQL(CREATE_PERSON);
    /*
     * TODO:改动2   老版本  直接创建2张表
     */
    db.execSQL(CREATE_STUDENT);
  }
  @Override
  public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
    //改动3 根据版本执行不同的sql
    switch (oldVersion) {
    case 1: //版本1 直接创建新表
    db.execSQL(CREATE_STUDENT);
    default:
    }
  }
}
```
较版本一在版本二中有**三处修改**的地方:
- **版本号变成了2:**
- **在onCreate()方法中添加了代码db.execSQL(CREATE_STUDENT);创建student表**

<font color=blue>因为有的用户根本就没有第一版本的APP，直接从市场下载了第二版本的App。所以当然会执行onCreate()而不会执行onUpgrade()</font>

- **在onUpgrade()做了处理：当oldVersion为1时调用db.execSQL(CREATE_STUDENT);创建student表**




<font color=blue>因为有的用户手机上本来就有第一版本的APP，所以在App升级到第二版本时会执行onUpgrade()，不会执行onCreate()</font>

通过这样的处理使得不同的情况下使用第二版APP时都会生成student表。
<font color=red>又过了一个月，根据项目变更，需要给person表添加一个字段genderid，于是DataBaseOpenHelper就出现了第三版:</font>

``` java
public class DataBaseOpenHelper extends SQLiteOpenHelper {
    //数据库名
  private final static String DATABASE_NAME="test.db";
  private static DataBaseOpenHelper mDataBaseOpenHelper;
  //改动1  更改sql语句
  public static final String CREATE_PERSON=
  "create table person(personid integer primary key autoincrement,name varchar(20),phone VARCHAR(12)),genderid integer)";
  //新增sql  用于增加列  
  //详细sql请见 http://blog.leanote.com/post/cornflower1991@163.com/%E4%B8%AA%E4%BA%BA%E4%B8%8D%E5%B8%B8%E7%94%A8%E7%9A%84-sql%E8%AF%AD%E5%8F%A5
  public static final String ALTER_PERSON="alter table person add column genderid integer";
  
  
  public static final String CREATE_STUDENT=
  "create table student(studentid integer primary key autoincrement,name varchar(20),phone VARCHAR(12))";
  
  public DataBaseOpenHelper(Context context,String name,CursorFactory factory,int version) {
    super(context, name, factory, version);
  }
  //注意:
  //将DataBaseOpenHelper写成单例的.
  //否则当在一个for循环中频繁调用openHelper.getWritableDatabase()时
  //会报错,提示数据库没有执行关闭操作
  static synchronized DataBaseOpenHelper getDBInstance(Context context) {
    if (mDataBaseOpenHelper == null) {
      //改动2 版本更改
      mDataBaseOpenHelper = new DataBaseOpenHelper(context,DATABASE_NAME,null,3);
    }
    return mDataBaseOpenHelper;
  } 
  @Override
  public void onCreate(SQLiteDatabase db) {
    db.execSQL(CREATE_PERSON);
    db.execSQL(CREATE_STUDENT);
  }
  @Override
  public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
    switch (oldVersion) {
    case 1:
    db.execSQL(CREATE_STUDENT);
    //改动3
    case 2:
    db.execSQL(ALTER_PERSON);
    default:
    }
  }
}
```

较版本二在版本三中有**三处修改**的地方：

- **改变了CREATE_PERSON语句，在改语句中增加了一个字段genderid**

和前面的描述类似，有的用户第一次安装该APP时就直接下载了第三版执行Oncreate()。

- **修改版本号为3 应对了用户从第一版本或者第二版本升级到第三版本的情况(见下分析)**

- **在onUpgrade()方法中)做了处理：当oldVersion为2时调用 db.execSQL(ALTER_PERSON);修改person表，增加genderid字段**

应对了用户从第二版本升级到第三版本的情况(见下分析)

> **注意**一个问题：为什么这里的switch语句在每个case中**没有break**？？这是为了保证跨版本升级的时候**每次数据库的升级都会执行到**。
> - 比如从**第二版升级到第三版本**，那么case 2会被执行。
> - 比如从**第一版直接升级到第三版本**，那么case 1肯定会被调用，由于没有break所以会传透switch语句又执行case 2语句继续升级，从而保证了数据的所有版本中的升级都会被执行到。


 <font color=red> 以上就是数据库版本更新时注意的地方。</font>

### 查看真机数据库

Android手机root之后可以使用 `su root` 命令查看授权，然后进入data/data目录下。sqlite3命令打不开，可以直接执行`cp`命令复制到本地，进行查看。当然使用第三方软件如`RE管理器`等。
这里想介绍的是`Facebook`的调试插件[stetho][1]，使用该插件你可以在`Chrome Developer Tools`查看App的布局，网络请求，sqlite，preference等。个人网络请求没怎么用过，不太会用。看数据库和sp还是很爽的，主要是不需要手机`root`。
集成步骤：
- 1.引入依赖包
>   compile 'com.facebook.stetho:stetho:1.4.1' 

- 2.初始化一下
```java
 public class MyApplication extends Application {
   public void onCreate() {
     super.onCreate();
     Stetho.initializeWithDefaults(this);
   }
 }
```
- 3.运行App, 打开Chrome输入chrome://inspect/#devices（别忘了用数据线把手机和电脑连起来哦）
![@运行效果界面 | center](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/114633563.png)
如上图，chrome会检测到我们的app，点击`inspect`进入查看页面
![@查看App 数据界面 | center](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/114924961.png)
数据库和sp文件都在Resources标签下，即可查看，还有网络等其他功能就不展示了，更多[高级用法][2]。





 [1]: https://github.com/facebook/stetho
 [2]: http://facebook.github.io/stetho/