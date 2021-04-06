# uni-app-d3
在Uni-app中引用d3插件绘制图形

[博客地址](https://blog.csdn.net/M_Eve/article/details/106745067)

## 安装教程
1. git clone https://github.com/Eveveen/uni-app-d3.git
2. 进入 uni-app-d3 目录下，执行命令 ``` npm install ```
3. 打开HBuilderX导入项目
4. 运行到手机或浏览器查看效果图
![预览](https://github.com/Eveveen/uni-app-d3/blob/master/static/view.png)

## 已实现功能
1. 根据 path 绘制图形
2. 放大、缩小、自适应屏幕
2. 点击获取选中块信息，并高亮

## 使用平台
目前仅在 浏览器 和 android 测试过

## 文件介绍
1. 主要代码在 [/pages/index/index.vue](https://github.com/Eveveen/uni-app-d3/blob/master/pages/index/index.vue) 文件中
2. [pages/index/data.js](https://github.com/Eveveen/uni-app-d3/blob/master/pages/index/data.js) 存放引用的数据