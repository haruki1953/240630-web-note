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
在其他文件中封装组合式api，在Store中use，像postList这样变量可以通过use的参数传递。

不知道这样是否会影响响应式？来试验一下，应该可以吧？