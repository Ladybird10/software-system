# 实验过程

1. 安装 django 

```shell
python -m pip install Django
```

* 如果pip安装速度很慢，大家可以修改pip使用的镜像源，改为国内的源，速度就会很快

然后执行 

```
django-admin startproject mysite 
cd  mysite
python manage.py startapp polls
python manage.py runserver
```

四个命令，构建了一个基于Django的基本框架的web应用程序。

然后访问 http://127.0.0.1:8000/ ,可以看到结果

<img src="pic\1.png" style="zoom:80%;" />

在命令行里，可以看到服务器的打印输出，表示服务器收到了 request。看到的页面就是框架自动返回给用户的response。说明，request和response，请求相应的处理过程已经搭建起来了。

<img src="pic\2.png" style="zoom:80%;" />

2. 数据库建表

* 现在我们使用的数据库分两种，一种叫关系型数据库，一种叫非关系型数据库。其中教务系统这种信息管理类的软件，一般是使用关系型数据库。关系型数据库的基本结构是表。那如何体现“关系”呢？关系其实是指表与表之间的关系。首先就是设计数据库表结构。

* Django最方便的一点，就是把建表这种基本操作，变成了python中的类的定义。所定义的这些类，直接继承models.

  Model 程序员只需要写好这个models.py文件。所有的建表的操作框架就可以完成。
  
  ```python
  from django.db import models #models.py
  
  # Create your models here.
  
  from django.contrib.auth.models import AbstractUser
  
  class Course(models.Model):
      name = models.CharField(verbose_name='课程名',  max_length=100)
      number = models.IntegerField(verbose_name='编号', default=0)
      summary = models.CharField(verbose_name='摘要', max_length=500, null=True)
  
  
  class Student(models.Model):
      class_name = models.CharField(verbose_name="班级", max_length=100, blank=True,null=True)
      name=models.CharField(verbose_name="姓名",max_length=100, blank=True, null=True)
      number=models.IntegerField(verbose_name="学号",default=0)
      phone_number = models.CharField(verbose_name='手机号', max_length=11,null=True)
  
  class Score(models.Model):
      course = models.ForeignKey(Course, verbose_name='课程', on_delete=models.CASCADE, related_name='students')
      student = models.ForeignKey(Student, verbose_name='学生', on_delete=models.CASCADE, related_name='my_courses')
      score = models.FloatField(verbose_name='成绩',  null=True)
  ```
  
  

* 再建一个app ，比如叫 edu_admin。然后用vscode打开工程

* 然后在 edu_admin的models.py文件就是上面的models.py文件.

  我们需要把这个表结构，真实的写入到数据库中，也就是create table的过程，也就是create table的过程。

  打开 mysite的settings.py 在  INSTALLED_APPS 这里增加一个 edu_admin 表示 edu_admin 这个是我们这个site的一个app。之前startapp命令只是创建了app，必须要把app写入到这里，这个app才会被纳入到站点功能中。

  ```shell
  python manage.py startapp edu_admin
  code . 
  python .\manage.py makemigrations
  ```

<img src="pic\4.png" style="zoom:80%;" />

<img src="pic\3.png" style="zoom:80%;" />

执行以上步骤，会得到结果如下，还会出现一个 db.sqlite3

<img src="pic\5.png" style="zoom:80%;" />

<img src="pic\6.png" style="zoom:80%;" />

* 加了app以后，执行makemigrations和migrate。makemigrations成功的标志是在app的目录下有migrations目录，也就是我们这里的edu_admin

<img src="pic\7.png" style="zoom:80%;" />

* 为了验证Django真的建立了表，下载一个sqlite的客户端软件，来看一下它的表结构。

> https://www.sqlite.org/download.html
>
> Windows，下载sqlite-tools-win32-x86-3310100.zip
>
> Linux的同学直接 apt install sqlite3

把这个exe加入在PATH环境变量，或者放在db.sqlite同一个目录。然后执行

```shell
sqlite3.exe db.sqlite #进入到sqlite的命令行以后 执行下一命令
.table
```

然后可以看到所有的表，其中这三个表是我们在models中定义的。其他表是Django自己要用的。

然后执行sql语句，插入一条记录。insert和select可以成功，说明表是好的。

<img src="pic\8.png" style="zoom:80%;" />