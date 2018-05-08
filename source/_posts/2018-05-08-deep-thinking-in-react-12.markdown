---
layout: post
title: "React的思考（十二）- 组织state和reducer（4）"
date: 2018-05-08 23:45:26 +0800
comments: true
categories:
---

## 不同的页面也可能有相同领域，不同状态的数据，也算是一个Domain吧？

## 没有BFF的情况下

基于Redux的应用程序中最常见的state结构是一个简单的JavaScript对象，它最外层的每个key中拥有特定域的数据。

然而，BFF可以是来自不同Domain的数据库合并后返回到前端的。

## 暴露的服务和接口

思考后端的操作逻辑，微服务的场景，按照业务的Restful API，Redux的这种设计模式导致这种CRUD和Service的冲突

## Normalize State

## BFF层的数据结构

## 一个action要操作多个reducer中的state

## redux的state和页面的关系

## 开发过程中state的演进过程
