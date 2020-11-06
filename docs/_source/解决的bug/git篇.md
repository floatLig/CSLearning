**出现的问题：**

Windows下使用git bash进行 clone 的时候，无法进行clone，出现Permission denied (publickey).这个提示

**解决：**

在搜索引擎搜索Permission denied (publickey)，很容易能够搜索[官方的解答](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/error-permission-denied-publickey)，照着官方的教程来，可以解决问题。

官方提供了几个排查的方法，挺有学习意义的：

1. 检查是否能连到服务器
2. 检查进行连接的用户名对不对
3. 需要启动 ssh-agent
4. 确保已经生成了公钥私钥，并将公钥放到了GitHub上
5. 通过ssh -vT 查看详细的调试信息