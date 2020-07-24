---
translateHelp: true
---

# Convention Routing


In addition to profile routing, Umi also supports conventional routing. Conventional routing is also called file routing. It does not require handwriting configuration. The file system is routing. The routing configuration is analyzed through directories, files and their naming.

**If there is no routes configuration, Umi will enter the conventional routing mode**, and then analyze the `src/pages` directory to get the routing configuration.
For example, the following file structure：

```bash
.
  └── pages
    ├── index.tsx
    └── users.tsx
```

You will get the following routing configuration,

```js
[
  { exact: true, path: '/', component: '@/pages/index' },
  { exact: true, path: '/users', component: '@/pages/users' },
]
```

It should be noted that files meeting any of the following rules will not be registered as routes.

* Files or directories beginning with `.` or `_`
* Type definition files ending in `d.ts`
* Test files ending with `test.ts`, `spec.ts`, `e2e.ts` (applicable to `.js`, `.jsx` and `.tsx` files)
* `components` and `component` directories
* `utils` and `util` directories
* Not a `.js`, `.jsx`, `.ts` or `.tsx` file
* The content of the file does not contain JSX elements

## Dynamic routing

It is agreed that the files or folders wrapped by `[]` are dynamically routed.
such as：

* `src/pages/users/[id].tsx` Will become `/users/:id`
* `src/pages/users/[id]/settings.tsx` Will become `/users/:id/settings`

For a complete example, such as the following file structure,

```bash
.
  └── pages
    └── [post]
      ├── index.tsx
      └── comments.tsx
    └── users
      └── [id].tsx
    └── index.tsx
```

Will generate routing configuration,

```js
[
  { exact: true, path: '/', component: '@/pages/index' },
  { exact: true, path: '/users/:id', component: '@/pages/users/[id]' },
  { exact: true, path: '/:post/', component: '@/pages/[post]/index' },
  {
    exact: true,
    path: '/:post/comments',
    component: '@/pages/[post]/comments',
  },
];
```

## Dynamic optional routing

Not currently supported.

## Nested routing

It is agreed in Umi that if there is `_layout.tsx` in the directory, a nested route will be generated, with `_layout.tsx` as the layout of the directory. The layout file needs to return a React component and render the child components through `props.children`.

For example, the following directory structure,

```bash
.
└── pages
    └── users
        ├── _layout.tsx
        ├── index.tsx
        └── list.tsx
```

Will generate routing,

```js
[
  { exact: false, path: '/users', component: '@/pages/users/_layout',
    routes: [
      { exact: true, path: '/users', component: '@/pages/users/index' },
      { exact: true, path: '/users/list', component: '@/pages/users/list' },
    ]
  }
]
```

## Global layout

Convention `src/layouts/index.tsx` as the global route. Return a React component and render the child components through `props.children`.

For example, the following directory structure,

```bash
.
└── src
    ├── layouts
    │   └── index.tsx
    └── pages
        ├── index.tsx
        └── users.tsx
```

Will generate routing,

```js
[
  { exact: false, path: '/', component: '@/layouts/index',
    routes: [
      { exact: true, path: '/', component: '@/pages/index' },
      { exact: true, path: '/users', component: '@/pages/users' },
    ],
  },
]
```

A custom global `layout` is as follows:

```tsx
import { IRouteComponentProps } from 'umi'

export default function Layout({ children, location, route, history, match }: IRouteComponentProps) {
  return children
}
```

## Different global layout

You may need to output different global layouts for different routes. Umi does not support such configuration, but you can still distinguish `location.path` in `src/layouts/index.tsx` and render different layouts.

For example, if you want to output a simple layout for `/login`,

```js
export default function(props) {
  if (props.location.pathname === '/login') {
    return <SimpleLayout>{ props.children }</SimpleLayout>
  }

  return (
    <>
      <Header />
      { props.children }
      <Footer />
    </>
  );
}
```

## 404 routing

It is agreed that `src/pages/404.tsx` is a 404 page, and React components need to be returned.

For example, the following directory structure,

```bash
.
└── pages
    ├── 404.tsx
    ├── index.tsx
    └── users.tsx
```

Will generate routing,

```js
[
  { exact: true, path: '/', component: '@/pages/index' },
  { exact: true, path: '/users', component: '@/pages/users' },
  { component: '@/pages/404' },
]
```

In this way, if you visit `/foo`, neither `/` nor `/users` can match, it will fallback to the 404 route and render through `src/pages/404.tsx`.

## PrivateRoute

By specify `wrappers` property in page component.

For example, `src/pages/user`：

```js
import React from 'react'

function User() {
  return <>user profile</>
}

User.wrappers = ['@/wrappers/auth']

export default User

```

See below example as content of  `src/wrappers/auth`,

```jsx
import { Redirect } from 'umi'

export default (props) => {
  const { isLogin } = useAuth();
  if (isLogin) {
    return <div>{ props.children }</div>;
  } else {
    return <Redirect to="/login" />;
  }
}
```

With above configuration, user request of `/user` will be validated via `useAuth`. `src/pages/user` gets rendered or page redirected to `/login`.

## Extended routing attributes

Support to extend routing by exporting static attributes at the code layer.

such as:

```js
function HomePage() {
  return <h1>Home Page</h1>;
}

HomePage.title = 'Home Page';

export default HomePage;
```

The `title` will be appended to the routing configuration.
