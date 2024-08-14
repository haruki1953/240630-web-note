在 `src\systems\admin\init.ts` 中，进行初始化操作，并导出 **useSetup** 函数，其返回 AdminSystem 所必要的变量与函数

在 `src\systems\admin\srevices.ts` 中，进行业务函数的编写，其会到导入执行 **useSetup** 并使用其返回的内容。可以将此文件变为文件夹

在 `src\systems\admin\index.ts` 中，导出函数 **useAdminSystem**，为保持 `init.ts` 一定执行，即使不需使用也尽量导入执行 **useSetup**
```ts
// src\systems\admin\index.ts
import { useSetup } from './init'
import { confirmAuth, updateAuth } from './srevices'

useSetup()

export const useAdminSystem = () => {
  return {
    confirmAuth,
    updateAuth
  }
}
```


### init.ts
注意：需要修改 `AdminStore` 类型 时要修改三个地方
- `src\types\system.ts` 修改类型
- `src\systems\admin\init.ts` 修改 **defaultStore**
- `src\schemas\types.ts` 修改 **typesAdminStoreSchema**
```ts
// src\systems\admin\init.ts
import fs from 'fs'
import { systemAdminConfig, systemDataPath } from '@/configs'
import type { AdminStore } from '@/types'
import { typesAdminStoreSchema } from '@/schemas'
import { confirmSaveFolderExists, generateRandomKey } from '@/utils'
import { AppError } from '@/classes'

let store: null | AdminStore = null

const filePath = systemAdminConfig.storeFile

const init = () => {
  try {
    confirmSaveFolderExists(systemDataPath)
    load()
  } catch (error) {
    throw new AppError('admin系统初始化失败')
  }
}

const defaultStore = () => {
  return {
    username: systemAdminConfig.defaultUsername,
    password: systemAdminConfig.defaultUsername,
    jwtAdminSecretKey: generateRandomKey()
  }
}

// load data from file
const load = () => {
  try {
    const dataJson = fs.readFileSync(filePath, 'utf8')
    const dataObj = JSON.parse(dataJson)
    store = typesAdminStoreSchema.parse(dataObj)
  } catch (error) {
    store = defaultStore()
    save()
  }
}

const save = () => {
  const data = JSON.stringify(store, null, 2)
  fs.writeFileSync(filePath, data, 'utf8')
}

init()

export const useSetup = () => {
  return {
    // eslint-disable-next-line @typescript-eslint/non-nullable-type-assertion-style
    store: store as AdminStore,
    save
  }
}

```