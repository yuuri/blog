





### 1.删除子模块文件夹

```bash
$ git rm --cached GWToolkit
$ rm -rf GWToolkit
```

```
echo -e "backend\ndbinit\nfrontend\ndocs\nreleases\ngreatdas" |  xargs -t -I {} git rm --cached {}
echo -e "backend\ndbinit\nfrontend\ndocs\nreleases\ngreatdas" |  xargs -t -I {}  rm -rf {} 
```



### 2. 删除 `.gitmodules` 文件中相关子模块的信息，类似于：

```
[submodule "GWToolkit"]
        path = GWToolkit
        url = https://github.com/iphysresearch/GWToolkit.git

```

```
rm -rf .gitmodules
```



### 3.删除 `.git/config` 中相关子模块信息，类似于：

```fallback
[submodule "GWToolkit"]
        url = https://github.com/iphysresearch/GWToolkit.git
        active = true
```



### 4.删除 `.git` 文件夹中的相关子模块文件

```bash
$ rm -rf .git/modules/GWToolkit
```