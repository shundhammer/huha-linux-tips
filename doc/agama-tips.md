# HuHa's Tips for Agama Development


## Live Media

(formerly known as inst-sys)


### Enable root login

Add boot parameter:

```
live.password=mypassword
```


## Boot Options

- https://agama-project.github.io/docs/user/boot_options


## Getting an XTerm

Hit `Ctrl`+`Alt`+`T`


## Setting the Keyboard layout

```
localectl set-x11-keymap de

localectl set-x11-keymap us
```

Check:

```
cat /etc/vconsole.conf
```


## CLI (Command Line Interface)

https://agama-project.github.io/docs/user/cli



## Reference

- https://agama-project.github.io/docs

