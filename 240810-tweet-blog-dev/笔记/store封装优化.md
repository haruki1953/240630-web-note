关于vue3项目中的Pinia
对于中大型项目，`src\stores\modules` 的文件中的函数很多，不利于维护
```ts
// src\stores\modules\post.ts
// ...
export const usePostStore = defineStore(
  'tweet-post',
  () => {
    // home
    const cursor = ref(0)
    const postList = ref<PostData[][]>([])
    const haveMoreMark = ref(false)
    const isHaveMore = computed(() => {
      return haveMoreMark.value
    })
    const isFirstRequest = computed(() => {
      return cursor.value === 0
    })
	// ...
	return {
	  postList,
	  // ...
	}
  },
  {
    persist: true // 持久化
  }
)
```

虽然可以将某些处理数据的函数封装在其他文件，但这样也不是所有情况都适用

比如很多函数中都需要访问postList，这就没办法封装在其他文件


### 想到了一个办法：封装组合式api
在其他文件中封装组合式api（自己起名叫“module”），在Store中use
- 像postList这样变量可以通过use的参数传递（自己想将其称为“引用”）。
- 如果module中需要用到其他module的函数，也可以通过参数传递（通过父级控制）

> 不知道这样是否会影响响应式？来试验一下，应该是可以的

操作细节
```
改变文件节结构，以postStore为例
在stores目录下创建post文件夹
post文件夹中保存原先的post.ts文件，并创建index.ts来控制导出
创建modules文件夹，将post.ts分为多个模块
listModule用于postList中的数据管理，引用postList
	其中包含游标查询无限滚动相关，可尝试进一步封装
	listManageModule专门用来操作postList中的数据
poolModule用于postPool的数据管理
controlModule用控制跳转至发送页面等操作

对于Module导出地内容，其首为module名最好（如list）
不过对于现在已经写好的，再改有些麻烦，以后再注意
```

已提交代码：
- [完成了postStore的封装优化](https://github.com/haruki1953/tweet-blog-vue3/commit/2ad2edb8dae69917305f040605860e5e010e2bcc)
- [完成了imageStore的封装优化](https://github.com/haruki1953/tweet-blog-vue3/commit/0d1248c74048ff974186d1ad1fbcf2eecc812d6c)



