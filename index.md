---
title: 联机托管说明
permalink: index.html
layout: home
---

# 内容目录

下面列出了每个实验室练习和演示的超链接。

## 实验室 \(Entra ID\)

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs_EntraID'" %}
| 模块 | 实验室 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

## 实验室 \(AD DS\)

可在[此处下载](https://github.com/MicrosoftLearning/AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop/archive/master.zip)所需的实验室文件

{% assign labs = site.pages | where_exp:"page", "page.url contains '_ADDS'" %}
| 模块 | 实验室 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

## 实验室 \(Entra DS\)

可在[此处下载](https://github.com/MicrosoftLearning/AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop/archive/master.zip)所需的实验室文件

{% assign labs = site.pages | where_exp:"page", "page.url contains '_AADDS'" %}
| 模块 | 实验室 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

<!--
## Demos

{% assign demos = site.pages | where_exp:"page", "page.url contains '/Instructions/Demos'" %}
| Module | Demo |
| --- | --- | 
{% for activity in demos  %}| {{ activity.demo.module }} | [{{ activity.demo.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
-->
