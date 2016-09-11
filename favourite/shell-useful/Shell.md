### Nodejs | Npm

- 卸载当前目录所有node模块
````
for package in `ls node_modules`; do npm uninstall $package; done;
````

- 仅安装第一级开发依赖(**2015.08.28有效**)
````
npm install --dev--save
````
注意的是,如果使用npm install dev,会把子模块的开发依赖都装上。