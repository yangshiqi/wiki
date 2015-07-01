### 初级程序员自检手册


#### 在函数中声明了一些不用的变量；

```
function sayHello($name)
{
    $time = date('Y-m-d H:i:s');
    $weather = 'what a sunshinny day!';

    echo "Hello $name!";
}
```

症状：可能编程的时候买的股票刚升职、刚交了女朋友，心情格外的好，随便写上点什么？可能之前做需求变更时，代码逻辑没有删干净？可能在练习打字。

解毒：提交代码库前，自己做review，看能够让自己满意么？如果你真检查了，就是要保留上述代码，那么，恭喜你，做程序员太委屈了你，其实你应该去做产品经理！


#### 只注释不清理代码

![例子1](https://github.com/yangshiqi/wiki/blob/master/imgs/wrongcode/ex1.png)

症状：废了好大的劲，最终没有调用。猜测代码逻辑是一开始有用的，但是后续需求调整，从动态请求变成一张静态图片了，做需求的人没有认真看代码逻辑，直接修改了最终展示图片的位置，而没有管其他的逻辑，导致这种空跑逻辑的存在。

解毒：修改别人代码的时候是需要谨小慎微的，但是也不能容忍垃圾代码的存在，拿不准的可以找人结对或者review。没有代码洁癖的程序不是好产品经理！

#### 代码中有死循环

![例子1](https://github.com/yangshiqi/wiki/blob/master/imgs/wrongcode/ex2.png)

症状：我中有你，你中有我。相互调用前没有沟通清楚？还是就是一个人写的？死循环调用，直至到达系统设置的最大执行时间或者内存溢出，进程挂掉。

解毒：在加入版本库前走读代码，自检一下，看人脑能跑通么？多问问自己如果出错了怎么办？总不能只考虑正常逻辑吧。


#### 反复调用同一函数

```
$p = new Person();
$p->name = 'Tom';
$p->sex = 'male';
$p->address = 'balizhuang #1 1-1-101, chaoyang district, beijing, china';
$p->save();

//确保存入db
sleep(3);
$p->save();

```

症状：数据来的不容易，一定要保障数据的可靠性，要存入db。担心系统忙，万一当时系统真的很忙呢，还是多保存一遍比较稳妥，心理踏实。

解毒：额，这个么，建议你写代码时候桌子上摆个关公再点上三炷香，似乎更靠谱。尝试用系统执行的角度去思考代码的运行状况，或者step by step的进行debug调试，能够帮助我们思考，识别出无用的代码。如果第一次save()报错，那就让他报错好了


#### 捕获异常但是不处理或者改变异常信息

```
try
{
	$p = new Person();
	$p->name = 'Tom';
	$p->sex = 'male';
	$p->address = 'balizhuang #1 1-1-101, chaoyang district, beijing, china';
	$p->save();
}
catch (Exception $e)
{
	//再次尝试下看db能保存么？
	if (false == $p->save())
	{
		throw new MyException('保存db失败了');
	}	
}
```

症状：这是上个问题的升级版，也许是想加强对错误的处理，起码再努力看看？实在不成再抛出一个我的异常。捕获异常进行错误重试未尝不可，但是首先要注意是否只捕获了你关心的异常？而不是一揽子全收了，再次失败后，再抛出了一个新的异常，那底层带上来的上下文信息可就被吞并了，上层更无法判断是什么问题了。

解毒：只捕获业务关心的异常，不要吞噬上下文信息。


#### 同步依赖外部系统

```
function getUser($userName)
{
    $user = getUserByDB($userName);
    
    //取用户相关微信号
    $api = new WeixinAPI();
    $user->code = $api->callWeixinHttpApi('http://weixin.qq.com/xxxx');
    return $user;
}
```

症状：在本系统调用过程中，同步调用了外部系统（微信）的远程接口，如果外部服务挂了，接口返回不正常时，本系统也无法正常使用。

解毒：依赖外部服务的功能分成：强依赖，比如这个功能是调用对方提供的接口来打电话，如果对方不可用，本系统即不可用；和弱依赖，比如只是在页面右侧广告位显示下微信的二维码，如果对方不可用，不要影响我方其它业务。

要加入错误捕获或者放入消息队列进行异步处理，不要让其阻断本系统的其它功能。如：

```
function getUser($userName)
{
    $user = getUserByDB($userName);

    try
    {
        $api = new WeixinAPI();
        $user->code = $api->callWeixinHttpApi('http://weixin.qq.com/xxxx');
    }
    catch (WeixinException $e)
    {
        //add logs here
    }
    return $user;
}
```
一旦微信不可用，WeixinAPI抛出异常，会导致整个系统也不可用。主动捕获可能发生的异常，不要让其阻碍本系统的正常使用。


#### 命名过于笼统或者随意


症状：HTMLHelper，UserManager，XString... 具体可以参考: [xstring.php](http://svn.haodf.net/svn/libs/trunk/framework/util/xstring.php)。在各种helper，manager类中，往往混杂了大量说不清楚，或者职责模棱两可的地方，于是我们发明了粘合剂类，用于填充那些谁也不管的功能。

解毒：粘合剂不是一定不能有，但是要在设计时就考虑清楚其职责，而不是笼统的起一个xxxHelper来放。其他人可不知道你这个到底是干嘛的，也不敢用。久而久之就成了垃圾箱。

进阶：深层次的问题是在面向对象开发过程中的职责不清，到底什么逻辑放到service中，什么逻辑放到实体或者知识类中？可以参考：[message系统服务](http://svn.haodf.net/svn/services/message/trunk/src/service/smssvc.php).

推荐阅读：

* [![agile](http://img3.douban.com/mpic/s1671095.jpg)](http://book.douban.com/subject/1140457/)

* [![clean](http://img3.douban.com/mpic/s4103991.jpg)](http://book.douban.com/subject/4199741/) 

书的图片可以点击哦。







