---
layout: post
title: 'Spring Boot 编码指导'
date: 2019-04-07
author: lipeng
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: SpringBoot
---

## 包规约

- 项目基本包：com.company.{项目英文名（较长时适当简化）}.{模块名（可选）}
- config：配置类
- startup：与服务启动相关的类
- client：提供客户端实现的相关类
- common：公共类，定义常量类，组件
- entity：数据库相关的实体类
- model：数据模型类（DTO、VO）
- controller：控制层
- service：服务层
- dao：数据库访问层

## 各层方法命名规约

Service/DAO/Mapper 层方法命名规约如下：

- 获取单个对象的方法用 get 作为前缀
- 获取多个对象的方法用 list 作为前缀
- 获取统计值的方法用 count 作为前缀
- 插入方法用 save / insert 作为前缀
- 删除方法用 delete / remove 作为前缀
- 修改方法用 update 作为前缀

## 异常规约

- 运行时异常：通过参数检查等方式避免抛出运行时异常，日志记录。
- 检查异常：检查异常需要捕获和处理，日志记录。
