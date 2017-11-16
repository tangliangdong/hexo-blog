---
title: ionic3 http请求
date: 2017-09-17 12:09:49
category: Learn
toc: true
tags:
    - ionic3
    - http
---

在ionic中不管是登录还是请求列表信息，都会需要使用http请求。

<!--more-->

 - 想要使用Http请求，就需要先**App.module.ts**中注册HttpModule

```ts
// 导入Angular 的 HttpModule
import { HttpModule} from '@angular/http';

@NgModule({
  imports: [
    HttpModule
  ],
  declarations: [ MyApp ],
  bootstrap:    [ IonicApp ]
})
export class AppModule {}
```

 - 注册好之后，就可以使用了，先把home.html页面表单写好

```html
<ion-header>
  <ion-navbar>
    <ion-title>Home</ion-title>
  </ion-navbar>
</ion-header>

<ion-content padding>
  <h2>Welcome to Ionic!</h2>
  <ion-list>
    <ion-item>
      <ion-label fixed>用户名：</ion-label>
      <ion-input type="text" [(ngModel)]="User.username"></ion-input>
    </ion-item>
    
    <ion-item>
      <ion-label fixed>密码：</ion-label>
      <ion-input type="password" [(ngModel)]="User.passwd"></ion-input>
    </ion-item>
    
    <ion-item>
      <button ion-button color="red" (click)="submit($event)">提交</button>
    </ion-item>
  </ion-list>
</ion-content>
```

##### 四种数据绑定的形式

```html
<!--插值表达式    显示组件的hero.name属性的值-->
<li>{{hero.name}}</li>

<!--属性绑定    把父组件selectedHero的值传到子组件的hero属性中-->
<hero-detail [hero]="selectedHero"></hero-detail>

<!--事件绑定    用户点击英雄的名字时调用组件的selectHero方法-->
<li (click)="selectHero(hero)"></li>

<!--双向绑定    数据属性值通过属性绑定从组件流到输入框。用户的修改通过事件绑定流回组件，把属性值设置为最新的值-->
<input [(ngModel)]="hero.name">
```

------
 
 - 再写home.ts文件，提交表单数据

```ts
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { Http,Response,Jsonp } from '@angular/http';

@Component({
  selector: 'page-home',
  templateUrl: 'home.html',
})

export class HomePage {
  // 申明成员变量
  User = {
    username: '',
    passwd: '',
  }
    
  // 依赖注入
  constructor(public navCtrl: NavController,private http: Http,public jsonp: Jsonp) {
    
  }

  submit(event){
    // 使用jsonp请求，是解决跨域问题的一种办法
    this.http.request('http://localhost:9090/app/login?account='+this.User.username+'&passwd='+this.User.passwd)
      .subscribe((res: Response) => {
        this.listData = res.json();
      });
  }
}
```

 * this.http.request()
 * this.http.get()
 * this.http.post()

------
##### 但这样会造成如下跨域的问题

![](cross-domain.png)

可以先用Jsonp来解决，

同样需要先在App.module.ts中注册JsonpModule的模块

```
import { HttpModule,JsonpModule} from '@angular/http';
import { HomePage } from '../pages/home/home';

@NgModule({
  declarations: [
    MyApp,
    HomePage,
  ],
  imports: [
    BrowserModule,
    HttpModule,
    JsonpModule,
    IonicModule.forRoot(MyApp)
  ],
  bootstrap: [IonicApp],
})
```

修改home.ts中的代码

```ts
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { Http,Jsonp } from '@angular/http';

@Component({
  selector: 'page-home',
  templateUrl: 'home.html',
})

export class HomePage {
  User = {
    username: '',
    passwd: '',
  }

  constructor(public navCtrl: NavController,private http: Http,public jsonp: Jsonp) {

  }

  submit(event){
    this.jsonp.request('http://localhost:9090/app/login?account='+this.User.username+'&passwd='+this.User.passwd+'&jsonp=callback&callback=JSONP_CALLBACK', { method: 'Get' })
      .subscribe(
        (data) => {
            console.log(data);
            // data.json() 将获取的json字符串转换成对象
            console.log(data.json());
        },
        (error) => {
            console.log(error);
        }
      );
  }
}
```

### Promise写法

> 如果想在网络错误的情况下也提供解决的方式

```js
this.http.post('http://localhost:9090/app/login?'+
    'account='+this.User.username +
    '&passwd='+this.User.passwd)
  .toPromise()
  .then(res => { this.listData = res.json(); })
  .catch(err => { console.error(err) }); // 错误的时候返回的
```


