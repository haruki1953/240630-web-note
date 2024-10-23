比如 storeSchema 就是用z.object生成的，所以直接用z.object的ReturnType即可

```ts
import { z } from 'zod'

const defineSystemStoreTest = <
  StoreSchema extends ReturnType<typeof z.object>
>
  (
    dependencies: {
      defaultStore: () => z.infer<StoreSchema>
      storeSchema: StoreSchema
    }
  ) => {
  const { defaultStore, storeSchema } = dependencies

  const getStore = (): z.infer<StoreSchema> => storeSchema.parse(defaultStore())

  return {
    getStore
  }
}

const storeSchema = z.object({
  username: z.string(),
  password: z.string(),
  jwtAdminSecretKey: z.string(),
  jwtAdminExpSeconds: z.number().int().positive(),
  loginMaxFailCount: z.number().int().positive(),
  loginLockSeconds: z.number().int().positive()
})

const defaultStore = () => {
  return {
    username: 'admin',
    password: 'adminadmin',
    jwtAdminSecretKey: 'aaaaaa',
    jwtAdminExpSeconds: 100 * 24 * 60 * 60, // Token expires in 100 days
    loginMaxFailCount: 10,
    loginLockSeconds: 1 * 60 * 60 // 1 hour
  }
}

const systemStore = defineSystemStoreTest({
  defaultStore,
  storeSchema
})

const store = systemStore.getStore()
/*
const store: {
  username: string;
  password: string;
  jwtAdminSecretKey: string;
  jwtAdminExpSeconds: number;
  loginMaxFailCount: number;
  loginLockSeconds: number;
}
*/

```