---
title: tomcat8登陆用户配置
date: 2019-11-18 11:51:34
description: ""
category: Learn
tags:
    - tomcat8
---

<!-- more -->

### tomcat-users.xml

 在**Tomcat**根目录下找到 `conf/tomcat-users.xml`文件，在`<tomcat-users></tomcat-user>`标签中添加如下内容 

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>

<user username="admin" password="123456" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
```



### context.xml

还需要修改**Tomcat** 根目录下的 `webapps/manager/META_INF/context.xml`文件

将其中的

```xml
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
```

注释掉就行了。 因为默认tomcat不可以通过外部ip访问管理界面。一定要启动Tomcat，不然等构建等时候会报拒绝连接 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<Context antiResourceLocking="false" privileged="true" >
 <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
 -->
<Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```









