

# 安装hexo工具
```bash
$ sudo install nodejs npm
$ cd /root
$ npm install -g hexo-cli
$ hexo init blog 
$ cd blog
```

### 安装主题
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
### 应用主题 & 下载插件
npm install hexo-renderer-pug hexo-renderer-stylus --save

# 安装pm2，实现部署
npm install -g pm2

### 编写启动脚本 hexo.js
``` javascript
const { exec } = require('child_process')
exec('hexo server -p 指定端口', (error, stdout, stderr) => {
    if (error) {
        console.log('exec error: ${error}')
        return
    }
    console.log('stdout: ${stdout}');
    console.log('stderr: ${stderr}');
})
```
### 启动服务
`pm2 start hexo.js`