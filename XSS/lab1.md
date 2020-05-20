# 实验记录

1. 写一个简单网页

   ```html
   <html>
   <body>
   <form method="post">
           <input type="text">
           <button>提交</button>
         </form>
   </body>
   </html>
   ```

2. 在vscode中，安装一个php插件，然后编写一个简单的php脚本，调试运行这个脚本。F5 vscode会自动选择脚本运行的方式 ，把用户的表单输入写入到html文件。在通过浏览器访问这个文件html文件，这就是一个最简单的xss运行环境了。

3. 简单的攻击

   ```html
   <html>
   <head lang="en">
       <meta charset="UTF-8">
       <title>XSS</title>
   </head>
   <body>
       <form action="2.php" method="post">
           <input type="text" name="name" />
           <input type="submit" value="submit">
       </form>
   </body>
   </html>
    
   
   ```

```php
<?php
    $name=$_POST["name"]; 
	echo $name;
?>
```

在输入框中提交数据： alert('hack') ，即可得到结果