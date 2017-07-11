---
layout:     post
title:      "ionic side menu"
subtitle:   "ionic2 beta@37 侧滑导航制作"
date:       2016-08-30 11:00:00
author:     "Tang"
header-img: "img/ionicsidemenu.jpg"
catalog:    true
tags:
    - ionic2
    - side menu
---

## 侧滑效果图

![img](../../../../img/ionic-side-menu/ionic_side_menu2.png)

## 目录结构

我们需要如图所示的目录结构，其中我们需要的`app.html`、`app.ts`、`app.scss`需要放在`app`文件夹的下，

![img](../../../../img/ionic-side-menu/ionic_side_menu3.jpg)

## app.html

`app.html`展示的是侧栏菜单里的内容，可以这么写：

```html
<!-- app.html
// in same directory as app.js -->

<!-- this menu section should hold what content and destine to where
// at what point in time?
// That is determined by the last line in this -->
<ion-menu [content]="content">

<!-- The side menu title -->
  <ion-toolbar>
    <ion-title>Menu</ion-title>
  </ion-toolbar>

  <ion-content>
    <ion-list>
      <!-- remember the `this.pages` object we created in `app.js`?
      // Iterate through, then when clicked, run the associated function
      // passing in the #p item. -->

      <!-- // changed #p to let p instead -->
      <button ion-item *ngFor="let p of pages" >
        {{p.title}}
      </button>
    </ion-list>
  </ion-content>

</ion-menu>
<!-- What's my root? remember the this.rootPage?-->
<ion-nav id="nav" [root]="rootPage" #content swipe-back-enabled="false"></ion-nav>
```

## app.ts

`app.ts`文件写的是一些事件，可以这么写

```typescript
import {Component, ViewChild} from '@angular/core';
import {Platform, ionicBootstrap, MenuController, NavController} from 'ionic-angular';
import {StatusBar} from 'ionic-native';
import {TabsPage} from './pages/tabs/tabs';
import {HomePage} from './pages/home/home';

@Component({
  templateUrl: 'build/app.html',
  providers: [NavController]
})
export class MyApp {
  @ViewChild('nav') nav : NavController;
  private rootPage: any;
  private pages: any[];

  constructor(private platform: Platform, private menu: MenuController) {
    this.menu = menu;
    this.pages = [
        { title: 'Home', component: HomePage },
    ];
    this.rootPage = TabsPage;

    platform.ready().then(() => {
      // Okay, so the platform is ready and our plugins are available.
      // Here you can do any higher level native things you might need.
      StatusBar.styleDefault();
    });
  }

  openPage(page) {
    this.menu.close()
    // Using this.nav.setRoot() causes
    // Tabs to not show!
    this.nav.push(page.component);
  };
}

ionicBootstrap(MyApp);
```

## home.html

在每个需要侧滑按钮的tab显示的页面，顶部加上一个按钮，` <button type="button" name="button" menuToggle> </button>`，`menuToggle`就是添加侧滑的事件。

```html
<ion-header>
  <ion-navbar>
    <button type="button" name="button" menuToggle>
      <ion-icon name="menu"></ion-icon>
    </button>

    <ion-title>Home</ion-title>
  </ion-navbar>
</ion-header>

<ion-content padding class="home">
  <h2>Welcome to Ionic!</h2>
  <p>
    This starter project comes with simple tabs-based layout for apps
    that are going to primarily use a Tabbed UI.
  </p>
  <p>
    Take a look at the <code>app/</code> directory to add or change tabs,
    update any existing page or create new pages.
  </p>
</ion-content>
```

## 参考

> 本文的侧滑菜单参照自[Ionic 2 – Side Menu and Tabs](https://blog.khophi.co/ionic-2-side-menu-tabs/)



