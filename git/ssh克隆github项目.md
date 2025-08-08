# github SSH Clone

> [生成新的 SSH 密钥并添加到 ssh-agent ](https://docs.github.com/cn/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

## mac 生成新 SSH 密钥

### 1. 查看当前是否已经存在ssh key:
`cd ~./ssh`

### 2. 生成新的SSH密钥

复制下面的命令(替换为您的 GitHub 电子邮件地址)
```
$ ssh-keygen -t ed25519 -C "your_email@example.com"
> Generating public/private ed25519 key pair.
> Enter file in which to save the key (/Users/chentao/.ssh/id_ed25519): 
```

注：如果您使用的是不支持 Ed25519 算法的旧系统，请使用以下命令：
```
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

在提示时输入安全密码，我在这里不输入任何密码，直接回车
```
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```
## 将 SSH 密钥添加到 ssh-agent

### 在后台启动 ssh 代理
```
$ eval "$(ssh-agent -s)"
> Agent pid 79273
```

### 如果您使用的是 macOS Sierra 10.12.2 或更高版本，则需要修改 ~/.ssh/config 文件以自动将密钥加载到 ssh-agent 中并在密钥链中存储密码
- 首先，检查您的 ~/.ssh/config 文件是否在默认位置。
```
$ open ~/.ssh/config
> The file /Users/you/.ssh/config does not exist.
```

- 如果文件不存在，则创建文件
```
$ touch ~/.ssh/config
```

- 打开您的 ~/.ssh/config 文件，然后修改文件以包含以下行。 如果您的 SSH 密钥文件与示例代码具有不同的名称或路径，请修改文件名或路径以匹配您当前的设置。
```
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```


> 注意：
>
> 如果您选择不向密钥添加密码，应该省略 UseKeychain 行。
>
> 如果您看到 Bad configuration option: usekeychain 错误，
> 请在配置的 Host * 部分另外添加一行。
> ```
> Host *
>  IgnoreUnknown UseKeychain
> ```

### 将 SSH 私钥添加到 ssh-agent 并将密码存储在密钥链中。 如果您创建了不同名称的密钥，或者您要添加不同名称的现有密钥，请将命令中的 id_ed25519 替换为您的私钥文件的名称。
```
$ ssh-add -K ~/.ssh/id_ed25519
```

## SSH密钥应用于github 
> https://docs.github.com/cn/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account

### 将SSH公钥复制到剪切板
```
$ pbcopy < ~/.ssh/id_ed25519.pub
# Copies the contents of the id_ed25519.pub file to your clipboard
```

### 转到 https://github.com/settings/keys
将密钥粘贴到 "Key"（密钥）字段。


