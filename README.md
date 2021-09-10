# nicong-blog-hugo
my personal blog built with hugo.

## 如何更新
1. 在本地编写好 post 之后，把修改 push 到 main 分支；
2. 之后会自动触发 Github Action 进行构建；
3. 构建产物会自动提交到 master 分支；
4. 腾讯的 web 应用托管在检测到 master 分支有变动之后就会自动触发网站的部署。

所以博客的整个更新部署流程中，需要手动完成的只有把本地代码 push 到 Github 的 main 分支，剩余的步骤都是自动完成。

### 为什么要先 push 到 main ？

为什么不直接把本地构建好的产物推送到 master 分支，而是要推送到 main 然后出发 GitHub Action 的构建？

因为如果直接推送到 master 分支，不管修改了哪个文件，都会触发腾讯 webify 的构建。但如果我只是修改了 readme ，就没必要触发腾讯 webify 的构建了。
推送到 main 分支之后，会检查某几个特定的目录下的文件是否有修改，有修改才会进行构建，然后把构建产物推送到 master 分支，继而触发腾讯 webify 的构建，最终更新网站。
