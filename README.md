## 文档目录结构
```
├── docs
│   ├── .vuepress  //存放核心内容的文件夹
│   │   ├── components  //存放你需要添加的vue组件
│   │   ├── public  //存放静态文件，如图片等
│   │   ├── styles  //存放需要定制的样式
│   │   │   └── palette.styl  //配置页面主题颜色的文件
│   │   └── config.js   //设定顶部导航栏、侧边导航栏等项目配置的核心文件
│   ├── pages   //存放markdown文件，用于设置其他页面内容
│   ├── README.md   //首页展示用的markdown文件
├── deploy.sh     //之后用于编写上传、发布脚本的文件
└── package.json  //之前创建的Node.js项目描述文件
```