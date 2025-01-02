---
title: "Next.js初体验(未完结)"
date: 2024-12-15T17:37:00+08:00
layout: "post"
draft: false
categories: ["技术笔记"]
tags: ["前端","fullstack"]
---

# Next.js 学习向

全栈框架`Next.js`的官网学习项目记录。

### CSS &

## Image and Fonts

`Layout Shifting`是作为谷歌在评估一个页面的性能的指标，而自定义字体或者是`image`都会造成这样的布局偏移的影响。所以Next提供了一个在build之前自动下载自定义字体，以及自动host到下载好的字体并进行展示，避免了在渲染的时候，花费多余的时间来连接到和加载外部的静态资源。

- 在`app/ui/font.tsx`在这个目录中定义相关的要使用的自定义Font，并在其他的page页面import进行引入使用

  ```tsx
  import { Inter, Lusitana } from 'next/font/google'
  
  export const inter = Inter({ subsets: ['latin'] })
  export const lusitana = Lusitana({
      weight: '700',
      subsets: [
          'latin'
      ]
  })
  ```

同时，`image`也是连接静态资源，影响性能的一种，所以Next同时提供了`Image`标签，直接进行使用。

```ts
		<Image
            src={'/hero-mobile.png'}
            width={560}
            height={620}
            className='block md:hidden'
            alt='Screenshots of the dashboard project showing destop version'
          ></Image>
```

## Creating Layouts and Pages

在`Next.js`中，路由的方式，是通过`app/`下的文件夹来实现的，在主文件夹下，新建一个文件夹，就是路径中的一个新的路由。在每一个文件夹中`page.tsx`文件扮演者该路由显示的页面。

`app/dashboard/page.tsx`

```tsx
export default function Page() {
    return <p>Dashboard Page</p>
}
```

而每一个路由下的`layout.tsx`则扮演者，对当前路由下包括所有的子路由（子文件夹）共享一套排版配置。同时，在主工作文件夹中，存在着一个`RootLayout`

```tsx
import '@/app/ui/global.css'
import { inter } from 'app/ui/fonts';
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

所有在`RootLayout`下添加的UI组件都会在整个项目中的整个`Page`文件中生效。

- Use it to Modify the `<html> ` and `<body>` tags
- Add metadata on it 

`app/dashboard/layout.tsx`

```tsx	
import SideNav from '@/app/ui/dashboard/sidenav'

export default function Layout({ children } : { children : React.ReactNode }) {
    return (
        <div className='flex h-screen flex-col md:flex-row md:overflow-hidden'>
            <div className='w-full flex-none md:w-64'>
                <SideNav />
            </div>
            <div className='flex-grow p-6 md:overflow-y-auto md:p-12'>{ children }</div>
        </div>
    )
}
```

**接收一个`Children`,作为一个`Prop`，**这个`prop`可以是React中的`Node`，也就是是说，可以是一个 `Page` ，也可以是其他的`layout`

### 解构赋值

其中，函数的参数部分`{ children }: { children: React.ReactNode }`，第一个大括号的作用是解构赋值，第二个大括号的作用是**TypeScript中指定参数类型的语法。**

- prop： 是父组件，向下传递给子组件的属性。比如`prop.name` `prop.age`

这里的解构赋值，作用的对象是组件本身自带的属性`prop`（r**eact底层自身实现的**)，通过使用**大括号的解构赋值**，在函数中，就可以直接引用`children` 而不是使用`prop.children`来访问`prop`这个对象的`children`属性。

## Navigating Between Pages

### 使用Hook`usePathName()`

使用该Hook，来标明当前用户所在的`page`。依靠使用`cslx`来实现css的动态转变。

但是如果要使用该Hook，这个hook是`client-side hook`必须在用户模式下，才能进行使用。`'use client'`

```tsx
Ecmascript file had an error
  1 | // 'use client'
> 2 | import { usePathname } from 'next/navigation';
    |          ^^^^^^^^^^^
  3 | import clsx from 'clsx';
  4 | import {
  5 |   UserGroupIcon,

You're importing a component that needs `usePathname`. This React hook only works in a client component. To fix, mark the file (or its parent) with the `"use client"` directive
```

### Navigate

使用Next提供的`<Link />`组件，不需要在点击连接的时候刷新整个屏幕。

原理：在识别到`<Link>`组件之后，在后台Next就会自动加载到该连接所连接到的资源，在用户实际点击过后，就会直接获取已经加载好的资源，从而不需要刷新整个屏幕，达到用户友好。

## Setting up the Postgres database

通过在Vercel上部署项目，创建Postgres database，然后将创建好的database生成的`.env.local`文件，放在项目的根目录中`/.env`，安装好依赖，就可以通过一个`page`路由中的`route.ts`连接数据库，创建表，插入数据.....

## Fetching Data in Next

- 当处在用户层也就是，`client-side`的时候，需要通过加入一个API Layer运行在`server-side`的API，防止将数据库的secret泄漏。
- 如果只是在使用React server-side component 也就是从server端拉取数据，可以跳过API层，直接向数据库发送query请求拉取数据。

> If you are using React Server Components (fetching data on the server), you can skip the API layer, and query your database directly without risking exposing your database secrets to the client.

> 思考：既然这是一个全栈框架，那么分成client-side 和 sever-side的component是有道理的，这就类比于，前后端分离式项目。（这样一来就很好理解为什么一定要分类这么严格了）

### JavaScript Pattern

在Fetching Data的时候，使用Js中的 `await`，代表的是同步操作，也就是意味着会出现等待阻塞的情况，使用以下Pattern达到同时开始查询每个的操作

```js
    const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
    const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
    const invoiceStatusPromise = sql`SELECT
         SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
         SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
         FROM invoices`;

    const data = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);

    const numberOfInvoices = Number(data[0].rows[0].count ?? '0');
    const numberOfCustomers = Number(data[1].rows[0].count ?? '0');
    const totalPaidInvoices = formatCurrency(data[2].rows[0].paid ?? '0');
    const totalPendingInvoices = formatCurrency(data[2].rows[0].pending ?? '0');
```

##  Static and Dynamic Render

> With static rendering, data fetching and rendering happens on the server at build time (when you deploy)

> With dynamic rendering, content is rendered on the server for each user at **request time** (when the user visits the page)

## Streaming

使用流水线（pipeline）方式， fetch data， 加载页面，这样就不会有木桶效应（速度取决于最慢的那个Promise）

在Next.js中有两种方法实现流式编程，获取数据。

- `loading.tsx` 在 Page 级别
- `Suspense` 对于特定的Component

`Loading.tsx`作为的是一个特定的next文件，作为一个页面正在加载时作为代替的一个Fallback  UI

> is a special Next.js file built on top of Suspense

### RouteGroup

既然Next中，子文件夹是作为route的依据，所以当想要向当前的主文件(也就是主路由)添加逻辑而不影响到其下面的子路由，这时就可以使用`RouteGroup`，他提供了一种方法，将文件作为一个逻辑的分组，这样就能够到达该逻辑分组下的文件，只会影响到当前主文件的路由了

> Route groups allow you to organize files into logical groups without affecting the URL path structure. When you create a new folder using parentheses `()`, the name won't be included in the URL path. **So `/dashboard/(overview)/page.tsx` becomes `/dashboard`.**

### Component Stream

对于单个的Component的Stream，添加正在加载的回调页面

```jsx
<Suspense fallback={<RevenueChartSkeleton />}>
          <RevenueChart />
        </ Suspense>
```

### Stream in Component-Group

添加加载页面可以是给一整个页面添加，也可以是给单个Component添加，还可以是给多个Component添加（不过需要重新写一个Wrapper Component 将整个组给包括起来），这些都取决于页面的显示逻辑。 不过:

> In general, it's **good practice to move your data fetches down to the components** that need it, and then wrap those components in Suspense.





