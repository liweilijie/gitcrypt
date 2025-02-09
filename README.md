# git crypt

encrypt a part file in git repo.

## 安装

```bash
# install git-crypt in mac
brew install git-crypt
```

安装完后，我们可以在 `git repo` 里执行 `git crypt init` 来完成 **git crypt** 的初始化，它会：

- 生成一个默认的加密 **key** 文件：`.git/git-crypt/keys/default`
- 修改 **.git/config**，在其中加上filter和diff相关的配置


## 配置需要加密的文件

我们可以通过`.gitattributes`文件来配置需要被加密的文件，它有些类似`.gitignore`文件，可以放在`root`目录或者对应的子目录里。文件匹配的规则也是和gitignore 文件类似。

例如我在仓库 root 目录里添加了如下的`.gitattributes`文件：

```
# files will be auto encrypted when commit
link/.gauth filter=git-crypt diff=git-crypt
link/.ssh/** filter=git-crypt diff=git-crypt

# avoid accidentally encrypt files
# NOTES: need to be at last to have highest priority
.gitignore !filter !diff
.gitattributes !filter !diff
```

这个配置的含义是：

- 加密文件 link/.gauth 以及 link/.ssh/ 目录下的所有文件
- 避免加密.gitignore 和 .gitattributes 文件

也可以在子目录里添加配置，例如我添加了文件`link/.config/.gitattributes`：

```
hub filter=git-crypt diff=git-crypt 
```

在配置完成后我们可以通过如下的命令检查加密配置会对哪些文件生效：

```bash
git crypt status -e
```

## 如何解密

解密需要有对应的密钥，这需要我们先提前在安全的地方保存好密钥。可以通过如下的命令导出密钥：

```bash
git-crypt export-key /path/to/key
```

git-crypt 也支持 gpg 的方式管理密钥，具体使用方式可以查看帮助文档`git help crypt`。

当把仓库新 clone 下来之后，我们直接通过 git crypt unlock /path/to/key 即可解密之前加密过的文件。

也可以通过如下的命令手工解密文件：

```bash
cat encrypted_file | git-crypt smudge --key-file exported.key > decrypted_file
```

## 去除加密的文件

当不小心把某个文件设成了加密，或者觉得某个文件没必要再进行加密时，我们可以通过如下的步骤去除对它的加密配置：

- 在`.gitattributes`中去除这个文件对应的加密配置，提交并`push`
- 在另外的地方`clone`仓库，复制明文的文件替换当前的密文版本，确认无误后，提交并`push`
- 在原来的仓库执行`git pull`操作即可把文件从加密变成非加密版本

## 始终用一个加密文件 

我比较懒，喜欢总是用一个**private key**来管理所有的项目。所以在生成`git crypt init`之后，将default 替换成之前的文件即可。

```bash
# 先导出来以便日后用作默认的
git crypt export-key /Users/liwei/.config/gitcrypt/default 
# replace new key
cp /Users/liwei/.config/gitcrypt/default .git/git-crypt/keys/default
```

## 总结
git-crypt 借助 git 的 attributes 属性来实现对特定文件的自动加解密。目前在去除对文件的加密时会显的有些繁琐，后面可以调研下 gitattribute 的工作机制，看看有没有更简单的方法。
