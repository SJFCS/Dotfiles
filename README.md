## 前言

我们经常需要对软件和系统进行各种个性化配置，这些配置文件通常被称为"dotfiles"，因为它们的文件名以点（.）开头，例如.bashrc、.vimrc等。这些文件通常遵循 `XDG 约束规范` 默认被存放在 `$HOME/.config` 下

:::info
<details>
<summary>什么是 XDG 约束规范</summary>

参考 [arch wiki XDG_Base_Directory](https://wiki.archlinux.org/title/XDG_Base_Directory)

[XDG Base Directory Specification](https://sspai.com/link?target=https%3A%2F%2Fspecifications.freedesktop.org%2Fbasedir-spec%2Fbasedir-spec-latest.html) 约束了不同类型的文件的存放位置和命名规则，主要包括配置文件，缓存文件，数据文件等。它的提出能极大地改善上述配置文件太凌乱的情况。

XDG 主要定义了三个很必须的路径，它很好地告诉了你，哪些是配置文件应该保留，哪些是缓存当空间不够时可以删除，软件依赖了哪些数据文件。

| 环境变量        | 说明                     | 默认值             |
| --------------- | ------------------------ | ------------------ |
| XDG_CONFIG_HOME | 软件的配置文件存放位置   | $HOME/.config      |
| XDG_CACHE_HOME  | 软件的缓存应该存放的位置 | $HOME/.cache       |
| XDG_DATA_HOME   | 软件的依赖数据存放的位置 | $HOME/.local/share |

</details>
:::

传统的做法是在特定目录中保留原始配置文件，并在其原本对应的位置创建软链接。然而，这种方法存在一些缺点，例如需要手动创建目录结构，手动建立版本管理，而且无法轻松地处理隐私数据的加密和不同平台下配置文件的差异。

为了解决这些问题，最终我决定转而使用 Chezmoi 来管理我的 dotfiles。

> ps: home-manager 学习成本较高,其他的管理器参考这个文档： [dotfiles 管理器对比](https://www.chezmoi.io/comparison-table/)，



## chezmoi 初始化
安装地址 https://www.chezmoi.io/install/
- 无远程仓库
  
    `chezmoi init` 会在 `~/.local/share/chezmoi` 创建一个本地git仓库。  
    `chezmoi cd` 可以直接进入到该目录  

    后续可以为其添加远程仓库  
    ```bash
    git remote add origin git@github.com:{username}/{remote-repository}.git
    git checkout {branch}
    ```

- 有远程仓库
  
    当然如果你已经有远程仓库则初始化时候可以直接指定仓库地址  
    `chezmoi init git@github.com:{username}/{remote-repository}.git`  

## chezmoi 添加文件

```bash
chezmoi add {file} # 添加文件
```
通过git进行管理
```bash
chezmoi cd
git add {file}
git commit -m 'add {file}'
git push origin main
# 或 chezmoi 后面直接接 git 命令
chezmoi git add {file}
chezmoi git commit -m 'add {file}'
chezmoi git push origin main"
```

## chezmoi 修改文件
修改chezmoi目录的文件。 比如 chezmoi/dot_vimrc" 
```bash
chezmoi edit {file} 
```
应用修改
```bash
chezmoi apply 
```
等价于
```bash
chezmoi edit ~/.bashrc --apply
```

## chezmoi 更新文件
```bash
$ chezmoi managed # 列出所管理的内容路径
$ chezmoi source pull -- --rebase && chezmoi diff 
```

## 从远程仓库拉取文件

远程仓库有更新了，需要拉取到本地。

```bash
chezmoi update

等价于

chezmoi git pull origin main
chezmoi apply
```


## 配置迁移
如何将配置搬到其他机器：

```bash
chezmoi init git@github.com:{username}/{remote-repository}.git
chezmoi apply
or
chezmoi init --apply git@github.com:{username}/{remote-repository}.git

如果仓库命名方式就是dotfiles，那么可以：
chezmoi init --apply username 
```

## 模版
chezmoi 支持[模版](https://www.chezmoi.io/reference/templates/)，可以为不同的 host 生成不同的配置文件。

模板文件以 .tmpl 结尾，或者放在 .chezmoitemplates 目录。

使用 `chezmoi data` 可参考模板可用变量

以 .gitconfig 为例：

```toml
[user]
    name = example
    email = example@example.com
```
为了在不同的设备电脑上使用不同的 git 账户这一需求，要将其中的用户信息数据与配置文件进行绑定，而原文件将作为 .tmpl 为后缀的模板文件保存。

```
# ~/.local/share/chezmoi/dot_gitconfig.tmpl
[user]
    name = "{{ .name }}"
    email = "{{ .email }}"
```


chezmoi 提供了自动生成模板的功能，但有时需要手动进一步修改
```
$ chezmoi add --autotemplate ~/.gitconfig
```

对模板进行测试：`chezmoi execute-template "{{ .chezmoi.hostname }}"`，其中`"{{ .chezmoi.hostname }}"`代表的是模板内容。

也可以这样写：`chezmoi execute-template < dot_zshrc.tmpl`。



<!-- ## tips
-   config file，用于chezmoi自身的配置文件，默认为`~/.config/chezmoi/chezmoi.toml`。
-   使用`chezmoi unmanaged`可以列举所有不归属chezmoi管理的文件。
-   可以通过source state中的`.chezmoiignore`来显示阻止某些文件添加到chezmoi管理。
-   可以通过`chezmoi doctor`来检查当前主机上chezmoi的配置状况


```
# ~/test.tmpl
{{- $email := promptString "email" -}}
[data]
    email = "{{ $email }}"
```
通过 promptString 这个函数解析等下从命令行中传入的参数，并传入到配置文件中。
```
$ chezmoi execute-template --init --promptString email=example@example.com < ~/test.tmpl

[data]
    email = "example@example.com"
``` -->

<!-- ## 管理私有数据 -->
<!-- https://axionl.me/p/%E5%BD%92%E6%A1%A3-%E7%94%A8-chezmoi-%E7%AE%A1%E7%90%86%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6/ -->