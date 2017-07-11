---
layout:     post
title:      "php CodeIgniter框架"
subtitle:   "使用注意点"
date:       2017-5-23 22:00:00
author:     "Tang"
header-img: "img/php-codeIgniter.jpg"
catalog:    true
tags:
    - php
    - CodeIgniter
---

### CodeIgniter框架

> 这个框架具有mvc的思想，而且方法的url映射的方式很想java spring mvc。

具体查看可以[CodeIgniter文档](http://codeigniter.org.cn/user_guide)

---

#### base_url和site_url

如何在页面中使用配置文件中设置的静态文件路径和访问的url，先找到CodeIgniter根目录下 *application->config->config.php*

![](../../../../img/php-codeIgniter/1.png)

里面的base_url是自带的，这是在引入静态文件时用的，我们还可以再加入一个`site_url`配置，用来在提交表单或者ajax异步提交数据时url的根路径。

```php
$config['base_url'] = 'http://localhost/bikeShare';
$config['site_url'] = 'http://localhost/bikeShare/index.php';
```

---

当然如果我们要在页面上使用 **site_url**或 **base_url**，我们需要在使用这两个的地方的控制器里先通过 `$this->load->helper(array('url'))` 引入。如下：

```php
// controllers/Login.php
class Login extends CI_Controller {
    public function login(){
        $this->load->helper(array('url'));
        $this->load->view('login/login');
    }
}
```

登录页面

```html
<!-- login/login.php -->
<form id="form" method="post" 
        action="<?=site_url('login/getLogin') ?>">
    <input name="username" id="username" type="text">
    <input type="password" id="passwd" name="passwd">
    <button type="submit" id="submit_btn" name="action">Submit</button>
</form>
```

处理登录的界面

```php
// controllers/Login.php
public function getLogin(){
    $this->load->helper('url');
    $this->load->model('userModel');
    $username = $this->input->post('username');
    $passwd = $this->input->post('passwd');
    if($this->userModel->checkLogin($username,$passwd)===1){
        $this->load->view('home/index');
    }else{
        $this->load->view('login/login',array('error'=>'用户名或密码错误'));
        // redirect('/', 'refresh');
    }
}
```

#### 获取表单传值

获取表单项的值可以通过如下的方法，根据 提交的类型是 *post* 或是 *get* 来进行选择。

```php
$this->input->post('name');
$this->input->get('name');
```

#### 使用model类里的方法

对数据库的操作都放在 **models**文件夹下，假如需要调用 **UserModel.php**里的方法时，可以通过

```php
$this->load->model('userModel');
$this->userModel->getUser();
```

---

#### 给页面传入数据

显示 **views**文件夹下的页面，

```html
$this->load->view('user/list');
```

若是想在页面里面传入数据，类似java的Model，可以在`view()`方法里加入第二个参数,如下，这样就可以将关联数组data传入页面。

```php
$data = array('username'=>'小唐','studio'=>'betahouse');
$this->load->view('user/list',$data);
```

#### 生成json字符串 `json_encode`

> 由于json只接受utf-8编码的字符，所以json_encode()的参数必须是utf-8编码，否则会得到空字符或者null。当中文使用GB2312编码，或者外文使用ISO-8859-1编码的时候，这一点要特别注意。

若是需要返回json的字符串，则需要用 `json_encode()`方法，该方法可以将数组或对象转成json字符串。

```php
// 直接向前端输出json字符串就行
echo json_encode(array('status'=>1,'num'=>$num));
```

> 由于javascript不支持关联数组，所以json_encode()只将索引数组（indexed array）转为数组格式，而将关联数组（associative array）转为对象格式。

1.若是传入索引数组，则 `json_encode()`会将索引数组转成数组格式。

```php
$arr = Array('one', 'two', 'three');　　 
echo json_encode($arr);　
// 输出： ["one","two","three"]
```

2.若是传入的是关联数组，则 `json_encode()`会将关联数组转成对象格式。

```php
$arr = Array('1'=>'one', '2'=>'two', '3'=>'three'); 
echo json_encode($arr);
// 输出： {"1":"one","2":"two","3":"three"}
```

#### 解析json字符串 `json_decode()`

```php
$json = json_decode('{"info": "nice"}';)
```

> 通常情况下，json_decode()总是返回一个PHP对象，而不是数组。

如果想要强制生成PHP关联数组，json_decode()需要加一个参数true：

```php
$json = {"1":"one","2":"two","3":"three"};
json_decode($json,true);
```



