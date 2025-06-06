# 在c上运行编译app

## 基本环境配置

### 安装clangd

https://github.com/clangd/clangd/releases/tag/19.1.2

![install_clangd.png](install_clangd.png)

下载好后放入容器中，进行解压

修改/data/ci_build_code-main/app_rk1126/.vscode/settings.json中的内容。
修改clangd这部分的地址为自己安装的地址（黄色标注处）

![install_clangd1.png](install_clangd1.png)

然后重新加载一下

ctrl+shift+p，然后输入clangd。选择Restart language server

### vscode快捷键

ctrl+shift+p，然后输入reload。可以实现重新加载窗口。

右键点击文件夹，选择"在集成终端打开"，终端就可以定位到该文件夹。

然后再输入命令code .就可以以该文件夹重新打开一个窗口

    code .

