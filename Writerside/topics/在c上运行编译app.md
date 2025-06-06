# 在c上运行编译app

## 基本环境配置

### 安装clangd

保存ci_build_code-main/build.sh的代码中的一个地址，在45行。

    GXX_COMPILER="/tmp/arm_toolchain/bin/arm-none-linux-gnueabihf-g++"

用vscode打开ci_build_code-main/app_rk1126文件夹，在该文件夹下进行操作。

修改/data/ci_build_code-main/app_rk1126/.vscode/settings.json中的内容。第29行。

    "--query-driver=/tmp/arm_toolchain/bin/arm-none-linux-gnueabihf-g++"

下载clangd安装包链接：https://github.com/clangd/clangd/releases/tag/19.1.2

![install_clangd.png](install_clangd.png)

下载好后放入容器中，进行解压

修改/data/ci_build_code-main/app_rk1126/.vscode/settings.json中的内容。
修改clangd这部分的地址为自己安装的地址（黄色标注处）。第11行。

    "clangd.path": "/data/ci_build_code-main/app_rk1126/clangd_19.1.2/bin/clangd",

![install_clangd1.png](install_clangd1.png)

然后重新加载一下。

ctrl+shift+p，然后输入clangd。选择Restart language server

vscode上安装扩展cmake

最后编译ci_build_code-main文件夹，点击"生成"

![install_clangd3.png](install_clangd3.png)

安装clangd，编译成功后，将会对代码进行分析，给出代码提示，类似如下。

![install_clangd4.png](install_clangd4.png)


### vscode快捷键

ctrl+shift+p，然后输入reload。可以实现重新加载窗口。

右键点击文件夹，选择"在集成终端打开"，终端就可以定位到该文件夹。

然后再输入命令code .就可以以该文件夹重新打开一个窗口

    code .

