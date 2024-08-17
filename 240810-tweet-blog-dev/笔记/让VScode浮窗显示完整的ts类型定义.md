https://segmentfault.com/a/1190000043495782

1、找到VScode的安装目录
```
VSCode-install-path/resources/app/extensions/node_modules/typescript/lib/tsserver.js

"C:\Program Files\Microsoft VS Code\resources\app\extensions\node_modules\typescript\lib\tsserver.js"
```

2、打开tsserver.js（提前备份）
```
搜索 "defaultMaximumTruncationLength"  
找到 "var defaultMaximumTruncationLength = 160;" 这一行  
(在VScode 1.85中, 上面这句出现在15087行)
```

3、修改参数  
```
var defaultMaximumTruncationLength = 600; 保存
```

重启，成功

感觉600还是不太够，改为1600