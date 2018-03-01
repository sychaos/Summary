## ContentProvider TODO
    进行跨进程通信，实现进程间得数据交互和共享。
    通过 Context 中 getContentResolver() 获得实例，通过 Uri 匹配进行数据的增
    删改查。ContentProvider 使用表的形式来组织数据，无论数据的来源是什么，ContentProvider 都会认为是一种表，然后把数据组织成表格

    当一个应用程序需要把自己的数据暴露给其他程序使用时，该就用程序就可通过提供ContentProvider来实现；其他应用程序就可通过ContentResolver来操作ContentProvider暴露的数据。
    一旦某个应用程序通过ContentProvider暴露了自己的数据操作接口，那么不管该应用程序是否启动，其他应用程序都可以通过该接口来操作该应用程序的内部数据，包括增加数据、删除数据、修改数据、查询数据等。

## ContentProvider是如何实现数据共享
    ContentProvider以某种Uri的形式对外提供数据，允许其他应用访问或修改数据；其他应用程序使用ContentResolver根据Uri去访问操作指定数据。 步骤：
    1. 定义自己的ContentProvider类，该类需要继承Android提供的ContentProvider基类。
    2. 在AndroidManifest.xml文件中注册个ContentProvider，注册ContenProvider时需要为它绑定一个URL。
    例： android:authorities=”com.myit.providers.MyProvider” /> 说明：authorities就相当于为该ContentProvider指定URL。
    注册后，其他应用程序就可以通过该Uri来访问MyProvider所暴露的数据了。
    3. 接下来，使用ContentResolver操作数据，Context提供了如下方法来获取ContentResolver对象。
    一般来说，ContentProvider是单例模式，当多个应用程序通过ContentResolver来操作 ContentProvider提供的数据时，
    ContentResolver调用的数据操作将会委托给同一个ContentProvider处理。 使用ContentResolver操作数据只需两步：
        1、调用Activity的ContentResolver获取ContentResolver对象。
        2、根据需要调用ContentResolver的insert()、delete()、update()和query（）方法操作数据即可

## ContentProvider和sql的区别
    ContentProvider的主要还是用于数据共享，其可以对SQLite，SharePreferences，File等进行数据操作用来共享数据。
    而sql的可以理解为数据库的一门语言，可以使用它完成CRUD等一系列的操作