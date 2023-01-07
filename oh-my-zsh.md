# Oh-My-Zsh

## 插件

### 插件使用方式：

- 将插件安装到zsh的指定路径：`${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/`  下面一般通过 git clone 克隆到指定路径即可，少数需要特殊下载依赖等

- 然后在 `.zshrc` 里配置一下即可：在 `plugins=()`的括号里新加上插件名，通过空格来间隔不同插件名

- 最后再重载一下环境 `source ~/.zshrc` 即可生效新插件了

### 常用插件

| 插件                    | 描述                                           | 下载命令                                                     |
| ----------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| git                     | 略                                             | 自带                                                         |
| z                       | 通过z+path智能跳转到路径                       | 自带，需要配置                                               |
| zsh-autosuggestions     | 智能提示历史输入命令，用方向键右键来选中       | git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions |
| zsh-syntax-highlighting | 提醒输入的命令是否正确，红色错误🙅‍♂️，绿色正确🙆‍♂️ | git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting |
|                         |                                                |                                                              |



