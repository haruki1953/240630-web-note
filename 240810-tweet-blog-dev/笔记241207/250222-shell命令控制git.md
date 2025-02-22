### 模板
```sh
cd /e/Project/tweet-blog

# 定义目录数组
directories=(
  "Tweblog"
  "tweblog-electron-hono"
  "tweblog-electron-vue3"
  "tweet-blog-hono"
  "tweet-blog-public-vue3"
  "tweet-blog-vue3"
)

# 遍历数组并执行操作
for dir in "${directories[@]}"; do
  cd "$dir"
  # 执行操作开始
  
  # 执行操作结束
  cd ..
done
```

### 打标签并推送
```sh
cd /e/Project/tweet-blog

# 定义目录数组
directories=(
  "Tweblog"
  "tweblog-electron-hono"
  "tweblog-electron-vue3"
  "tweet-blog-hono"
  "tweet-blog-public-vue3"
  "tweet-blog-vue3"
)

# 遍历数组并执行操作
for dir in "${directories[@]}"; do
  cd "$dir"
  # 执行操作开始
  
# 创建标签
git tag -a 1.3.0 -m "2025-02-22

✨ **新功能**
- 新增了对于 [Bluesky](https://tweblog.com/guide/feature/import/bluesky) 的导入功能
- 新增了对于 [Bluesky](https://tweblog.com/guide/feature/forward/bluesky) 的转发功能

🧱 **改进**
- 桌面版新增了本地文档"

# 推送标签到远程仓库
git push origin 1.3.0
  
  # 执行操作结束
  cd ..
done
```



