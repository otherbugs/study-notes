# Vue学习笔记

## 一、Vue 项目化 

### 1.1 快速构建项目

#### 1.1.1 Vue CLI 简介

Vue CLI 是一个脚手架，构建项目的的框架；

可以快速搭建一个带有热重载、lint 代码语法检测及构件生产版本等功能的单页面应用；

#### 1.1.2 Vue CLI 构建项目

1. 在要放置项目的目录打开 terminal 输入： vue init webpack project-name;
2. 安装项目依赖：cnpm install;



#### 1.1.3 项目目录介绍

| 名称          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| build         | 开发和生产版本的构建脚本;                                    |
| config        | 开发和生产版本的部分构建配置;                                |
| dist          | 由 npm run build 生成；项目的生产版本；项目完成后，交付该文件夹即可； |
| src           | 关键资源目录和主要工作空间；                                 |
| static        | 静态资源（如使用 JS 赋值图片的 src 时，图片资源应该放此下面）； |
| .babeIrc      |                                                              |
| .eslintignore |                                                              |
| .eslintrc.js  |                                                              |
| .gitignore    |                                                              |
| index.html    |                                                              |

