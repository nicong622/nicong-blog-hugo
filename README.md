# nicong-blog-hugo
my personal blog built with hugo.

## 如何更新
1. 在本地编写好文章之后，把修改 push 到 master 分支；
2. 之后会自动触发 Github Action 进行构建；
3. 腾讯的 web 应用托管在检测到 master 分支有变动之后就会自动触发网站的部署。

所以博客的整个更新部署流程中，需要手动完成的只有把本地代码 push 到 Github 的 master 分支，剩余的步骤都是自动完成。
