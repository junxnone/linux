-----

| Title     | Tools Remote tmux plugins                            |
| --------- | ---------------------------------------------------- |
| Created @ | `2019-08-27T06:23:23Z`                               |
| Updated @ | `2024-04-06T04:37:37Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/112) |

-----

# Tmux Plugins

| Plugins        | Description |
| -------------- | ----------- |
| tpm            | 管理          |
| tmux-sensible  |             |
| tmux-resurrect | 保存恢复现场      |
| tmux-continuum | 保存恢复现场      |

## tpm

    git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

  - set `.tmux.conf`

<!-- end list -->

    set -g @plugin 'tmux-plugins/tpm'
    set -g @plugin 'tmux-plugins/tmux-sensible'
    
    run -b '~/.tmux/plugins/tpm/tpm'

    tmux source ~/.tmux.conf

> 之后新安装的plugins 会放在 `~/.tmux/plugins/` 下

## Restore plugins

### tmux-resurrect

  - Set `.tmux.conf`

<!-- end list -->

    set -g @plugin 'tmux-plugins/tmux-resurrect'

  - Install : <kbd>ctrl</kbd> + <kbd>b</kbd> -\> <kbd>shift</kbd> +
    <kbd>i</kbd>
  - 保存现场 : <kbd>ctrl</kbd> + <kbd>b</kbd> -\> <kbd>ctrl</kbd> +
    <kbd>s</kbd>
  - 恢复现场 : <kbd>ctrl</kbd> + <kbd>b</kbd> -\> <kbd>ctrl</kbd> +
    <kbd>r</kbd>

### tmux-continuum

  - Set `.tmux.conf`

<!-- end list -->

    set -g @plugin 'tmux-plugins/tmux-resurrect'
    set -g @plugin 'tmux-plugins/tmux-continuum'

> 依赖于 tmux-resurrect

  - Install : <kbd>ctrl</kbd> + <kbd>b</kbd> -\> <kbd>shift</kbd> +
    <kbd>i</kbd>
  - 选择要恢复的status

<!-- end list -->

    cd ~/.tmux/resurrect/
    rm last
    ln -s tmux_xxx.txt last

> tmux\_xxx.txt 为保存的状态

  - 恢复现场 : <kbd>ctrl</kbd> + <kbd>b</kbd> -\> <kbd>ctrl</kbd> +
    <kbd>r</kbd>

## Reference

  - [Tmux Plugin Manager](https://github.com/tmux-plugins/tpm)
  - [Tmux Resurrect](https://github.com/tmux-plugins/tmux-resurrect)
  - [tmux-continuum](https://github.com/tmux-plugins/tmux-continuum)
