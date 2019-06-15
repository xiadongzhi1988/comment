# atom参考文档
https://flight-manual.atom.io/hacking-atom/sections/hacking-on-atom-core/#platform-windows

# atom插件
```
简体中文支持 simplified-chinese-menu
markdown 语法支持 Markdown-Preview
代码增强 language-markdown
图片粘贴 markdown-image-paste
表格编辑 markdown-table-editor
Markdown-Preview-Plus
markdown 同步预览支持(有问题)  markdown-scroll-sync
apm install markdown-scroll-sync
```

# 本地创建仓库



# github创建一个空项目


# 本地创建测试文件


# 推送本地仓库导GitHub仓库
```bash
Administrator@DESKTOP-8VQOLAF MINGW64 ~/Desktop
$ cd /d/test/

Administrator@DESKTOP-8VQOLAF MINGW64 /d/test (master)
$ ls
test.md

Administrator@DESKTOP-8VQOLAF MINGW64 /d/test (master)
$ git remote add origin https://github.com/xiadongzhi1988/test.git

Administrator@DESKTOP-8VQOLAF MINGW64 /d/test (master)
$ git push -u origin master
Fatal: HttpRequestException encountered.
Username for 'https://github.com': xiadongzhi1988
Counting objects: 3, done.
Writing objects: 100% (3/3), 202 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/xiadongzhi1988/test.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```
