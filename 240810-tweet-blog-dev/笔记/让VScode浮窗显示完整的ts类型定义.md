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


### 另一种方法
对于新版本vs，上面的不管用了，这里的管用
https://github.com/microsoft/vscode/issues/64566#issuecomment-2199018004

1. 使用 Ctrl+P，转到 `node_modules/typescript/lib/tsserver.js`
2. 查找行 `var defaultMaximumTruncationLength = 160`
3. 将其更改为 `1000` 或您选择的价值。
4. 打开 Ctrl+Shift+P 菜单，选择 TypeScript：选择 TypeScript 版本...
5. 确保将其设置为“使用工作区版本”
6. 完全重新启动 VSCode - 重新加载工作区是不够的。
7. 验证...缩写已从类型提示中消失。


