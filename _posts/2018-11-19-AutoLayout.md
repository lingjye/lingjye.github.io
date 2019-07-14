---
layout: post
title: "谈谈iOS自动布局AutoLayout"
subtitle: "UIStackView, VFL, Masonry分析对比"
author: "lingjye"
header-style: text
tags:
  - iOS
---

# 简介

AutoLayout 是一种自动布局技术, 是一种来帮助开发者进行页面布局的技术. 由iOS 6开始引入, 相比于之前的autoresizing, 不仅可以设置当前控件与父控件之间的关系, 还可以设置兄弟控件之间的关系.

使用Autolayout需要设置autoresizing属性为NO, 他们两者只能用其中一个.

核心概念:

	1. 参照: 将某个UI控件作为参照标示, 进行确定该控件的位置;
	2. 约束: 为控件的布局进行加入限定, 实现在iOS设备上都能按照限定的格式、位置进行显示.

本文[Demo](https://github.com/lingjye/iOS-Learning/tree/master/AutoLayoutLearning){:target="_blank"}