### Nodejs | Npm

- 卸载当前目录所有node模块
````
for package in `ls node_modules`; do npm uninstall $package; done;
````