# flutter_orm
简介
===

一个以注解方式实现的ORM数据库解决方案，主要基于sqflite、source_gen、mustache4dart和build_runner。
# 涉及思路
由于source_gen 提供用于Dart自动生成源代码的实用程序，用于编写使用和生成Dart代码的Builder 框架。通过继承Generator或者GeneratorForAnnotation来实现一个基于注解的生成器。然后通过注解把entity里需要生成数据库属性的字段名称和类型标识。

build_runner包提供了一种编译、启动服务、和测试 web 应用的方式。使用build_runner代替弃用的pub build和pub serve命令。我们使用build_runner去使用flutter sdk进行构建。flutter pub run build_runner build 。然后就会生成dao文件，这样就完成了基本的dao文件生成过程。

原理就是source_gen通过拦截Annotation，解析其上下文element然后通过builder即可动态生成代码。mustache4dart根据注解拿到的参数进行模板填充。

# 安装
pubspec.yaml 添加
```Dart
 dependencies:
   flutter_orm:
   git: https://github.com/pxx11111/flutter_orm.git
 ```
使用
===
1、创建你的实体类并以*_entity.dart作为文件名,编译后生成的数据库操作文件将以*.entity.dao.dart命名 例:
```Dart
你的实体类文件:               class_entity.dart
编译后生成的数据库操作类文件:   class_entity.dao.dart
```


2、使用 **@Entity** 注解你的 **实体类** ,并且 **nameInDb**属性来定义其在数据库中的表名。
   **propertyList** 的类型为List,通过他来定义表中的字段 
   **注意**表中字段名,必须与实体类中的属性名一一对应

例:
```Dart
@Entity(nameInDb:'class',propertyList:[Property(name:'name',type:PropertyType.STRING)])
class ClassEntity{
   String name;
}
```

3、实体类必须拥有主键,并在 **@Property** 注解中通过 **isPrimary=true** 来声明这个属性为主键
```Dart
import 'package:flutter_orm/entity.dart';

@Entity(nameInDb:'class',
    propertyList:[
      Property(name:'id',type:PropertyType.INT,isPrimary:true),
      Property(name:'name',type:PropertyType.STRING),
      Property(name:'checked',type:PropertyType.INT),
      Property(name:'class_id',type:PropertyType.INT),
      Property(name:'teacher_name',type:PropertyType.STRING),
    ])
class ClassEntity{
  String name;
  int id;
  int checked;
  int class_id;
  String teacher_name;
}
}
```

4、pubspec.yaml里dev_dependencies:增加如下代码：
    
 >  process_run: '>=0.10.0'

 写工具shell.dart文件：
```Dart
 import 'package:process_run/shell.dart';
  Future<void> main() async {
   final Shell shell = Shell();
   await shell.run('''
   flutter packages pub run build_runner build
 ''');
}
```
         
直接运行工具类shell.dart即可自动生成dao文件

5、编译后生成的数据库操作文件中包含当前表的创建、增删改查等方法,在项目中使用需要先进行数据库的初始化
```Dart
///导入数据库管理类
import 'package:flutter_orm/db_manager.dart';

///传入数据库版本、数据库路径以及数据库名称来初始化数据库,DBManager为单例,每次创建拿到的都是同一个
DBManager dBManager = DBManager();
await dBManager.initByPath(1,"dbPath","dbName");
///你也可以使用默认路径来初始化数据库 默认的路径为 getDatabasesPath()
await dBManager.init(1,"dbName");
```


6、在项目中调用生成的数据库操作文件的 **init()** 方法来创建表, **init()** 方法中会做相应的判断不会重复创建表格
```Dart
  await ClassEntityDao.init();
```


7、然后就可以在项目中方便的进行数据库的增删改查操作了,所有的数据库操作方法都是静态
```Dart

  Future initDb() async {
    DBManager dBManager = DBManager();
    await dBManager.init(1, "dbName");
    await ClassEntityDao.init();
    List<ClassEntity> list = List();
    for (int i = 0; i < 100; i++) {
      ClassEntity classEntity = ClassEntity();
      classEntity.id = i;
      classEntity.checked = i + 1;
      classEntity.class_id = i + 2;
      classEntity.name = "class$i";
      classEntity.teacher_name="teacher$i";
      list.add(classEntity);
    }
    await ClassEntityDao.insertList(list);
    list=await ClassEntityDao.queryAll();
    list.forEach((f)=>print(f.id));
  }
}
```

8、你也可以通过构造查询器来查询数据
```Dart
  list = await ClassEntityDao.queryBuild()
      .where(ClassEntityDao.ID.equal("1"))
      .list();
  print("length${list.length}");
  list.forEach((f) => print("id${f.id}"));
```





