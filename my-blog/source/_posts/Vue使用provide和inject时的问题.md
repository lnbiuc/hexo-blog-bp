---
title: Vue使用provide和inject时的问题
date: 2023/5/19 04:39:37
updated: 2023/5/19 04:39:37
categories:
- FrontEnd
tags:
- Vue.js
---

## 需求

父组件provide用户信息，子组件用户设置inject用户信息。

父组件需要通过api获取用户信息，所以注入的用户信息需要是响应式的

## 解决方案 & 原因

子组件在setup中inject之后无法访问属性，原因是在将inject的值赋值给params是，inject的值还没有被注入进来，此时赋值给params的为空，所以访问不到

```js
// error
const injectInfo = inject<Ref<UserProfileEntity>>('userInfo');

onMounted(() => {
    params.value.userId = injectInfo!.value.userId;
    console.log('onMounted params__' + params.value.userId);
    console.log('onMounted injectInfo__' + injectInfo!.value.userId);
});

// 可以获得属性
setTimeout(() => {
    console.log('setTimeout_injectInfo_' + injectInfo!.value.userId);
    params.value.userId = injectInfo!.value.userId;
    console.log('setTimeout_params_' + params.value.userId);
}, 1000);
```

不能通过watch injectInfo来判断是否inject的值发生了变化，所以watch中的内容不会执行

```js
watch(injectInfo!.value, () => {
    params.value = injectInfo!.value;
    console.log('watch__' + params.value.userId);
});
```

需要在注入时添加默认值，通过监听默认值和api获取到数据，发生的变化，再将值赋给params即可解决

```js
const injectInfo = inject<Ref<UserProfileEntity>>('userInfo', params);

watch(injectInfo, () => {
    params.value = injectInfo!.value;
});
```

## 路由信息

Settings提供用户信息，Settings | Account注入用户信息。

```js
{
	path: '/u/s/:userId',
	name: 'Settings',
	component: () => import('../views/user/UserSettings.vue'),
	meta: { title: 'Settings' },
	redirect: (to: any) => {
		const { userId } = to.params; 
		return `/u/s/account/${userId}`;},
	children: [
		{
			path: '/u/s/account/:userId',
			name: 'Settings | Account',
            meta: { title: 'Settings | Account' },
            component: () => import('../views/user/settings/Account.vue'),
		},
	],
},
```

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/47201684441825783.png)

## 完整代码

父组件

> provide必须在setup中使用

```js
<script setup lang="ts">
const user: Ref<UserProfileEntity> = ref({
    userId: 'old',
    username: '',
    phone: '',
    mail: '',
    signature: '',
    avatar: 'https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png',
    level: 0,
    isBlock: false,
    permissionLevel: 0,
    registerTime: '',
    isMailNotice: false,
    isPhoneNotice: false,
    theme: 'auto',
    lastLogin: '',
    ipAddr: '',
});
onMounted(() => {
    // 从api获取数据
    getUserProfile(userId.value).then((res) => {
        user.value = res.data.data;
    });
});
// 注入
provide('userInfo', user);
</script>
```

子组件

```js {highlight=1,3-5}
<script setup lang="ts">
const params: Ref<UserProfileEntity> = ref({
    userId: 'new',
    username: '',
    phone: '',
    mail: '',
    signature: '',
    avatar: 'https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png',
    level: 0,
    isBlock: false,
    permissionLevel: 0,
    registerTime: '',
    isMailNotice: false,
    isPhoneNotice: false,
    theme: 'auto',
    lastLogin: '',
    ipAddr: '',
});

const injectInfo = inject<Ref<UserProfileEntity>>('userInfo', params);

watch(injectInfo, () => {
    params.value = injectInfo!.value;
});
</script>
```
