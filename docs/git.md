

### Git 无法检测到文件名大小写的更改

解决方法一：通过 `git config`

```
$ git config core.ignorecase false
```

解决方法二：`git mv`

```
git mv myfolder tmp
git mv tmp MyFolder
```