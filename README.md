# Laravel - React stack using Inertia.js
## Table of Contents
- [Installation](#installation)
- [Pages](#pages)
    - [Rendering a page](#rendering-a-page)
    - [Passing through variables](#passing-through-variables)
- [Links](#links---forget-about-anchor-tags)
    - [Link components](#link-components)
    - [Shared components](#shared-components)
- [Layouts](#layouts)
    - [Layout files](#layout-files)
    - [Persistent layouts](#persistent-layouts)
    - [Default layouts](#default-layouts)
- [Progress bar](#progress-bar-on-loading-pages)
- [Non-GET requests](#non-get-requests-through-links)
    - [Data](#passing-through-data)
- [Scroll preservation](#preventing-a-page-from-scrolling-up-when-a-link-is-clicked)
- [Active links](#styling-active-links)
- [Shared data](#shared-data-across-all-pages-and-components)
- [Head & Title](#head--title)
    - [Multiple Heads](#what-if-we-have-multiple-head-instances)

## Installation
1. Create a Laravel project
```
composer create-project laravel/laravel .
```
2. Add breeze (Laravel starterkit) to your project
```
composer require laravel/breeze --dev
```
3. Let breeze install react into your project (using inertia.js)
```
php artisan breeze:install react
```
4. Install and run vite
```
npm i
npm run dev
```
5. Set up your database settings in .env

6. Migrate database and start your server
```
php artisan migrate
php artisan serve
```
7. Done!

## Pages
Pages are located inside the `resource/js/Pages/` directory as a .jsx file.
### Rendering a page
A page will be rendered by a route inside `routes/web.php`. 
There's two ways to render a page. Both ways, inertia will look in the Pages directory by default, and look for {name}.jsx (just like using view() in Laravel!)
```php
//using the inertia() helper function
Route::get('/', function () {
    return inertia('Welcome');
});

//or

//using the Inertia object and render method
use Inertia\Inertia;

Route::get('/', function () {
    return Inertia::render('Welcome');
});
```
### Passing through variables
We can pass variables through to the Page in the same way as we can in Laravel
```php
Route::get('/', function () {
    return Inertia::render('Welcome', [
        'name' => 'Xander'
    ]);
});
```
The name variable can then be accessed inside the Page file using `props` as a variable passed through inside the function, just like in react
```js
export default function Welcome(props) {
    return (
        <h1>Hello {props.name}</h1>
    );
}
```
## Links - forget about anchor tags
Anchor tags will reload the page, which will defeat the purpose of making a single-page app using React!
### Link components
Inertia will handle anchor tags differently in order to not break the React flow, by using the Link component, which we can import wherever needed
```js
import { Link } from "@inertiajs/react";
```
Now we can replace all `<a>` tags with `<Link>` tags
```html
<nav>
    <ul>
        <li><Link href="/">Home</Link></li>
        <li><Link href="/about">About</Link></li>
        <li><Link href="/contact">Contact</Link></li>
    </ul>
</nav>
```
### Shared components
These are components that we will be using across multiple pages. Instead of repeating the same code for the nav on each page, we keep our code DRY and write it once, then import it to other pages. Shared components will be stored inside `resources/js/Shared/`

We can write the component like we would in React, and store it inside the Shared directory
```js
import { Link } from "@inertiajs/react"
export default function Nav() {
    return (
        <nav>
            <ul>
                <li><Link href="/">Home</Link></li>
                <li><Link href="/about">About</Link></li>
                <li><Link href="/contact">Contact</Link></li>
            </ul>
        </nav>   
    )
}
```
Now we are able to import this component on several pages, using the import statement in the right Page file
```js
import Nav from "@/Shared/Nav";
```
And then use it like we would in React
```js
export default function Welcome() {
    return (
        <>
            <Nav />
            <h1>Hello World - Welcome page</h1>
        </>
    );
}
```
## Layouts
### Layout files
Inertia uses normal React components to define a layout, giving the option to append anything between the component tags at a certain place within the layout

In the layout.jsx file
```js
const Layout = ({children}) => {
    return (
        <>
            <div>This is a layout component used on every single page</div>
            <div>We can choose to show the tags nested inside the Layout component tags at any place</div>
            <div>Here for example:</div>
            {children}
            <div>This is the end of the layout component</div>
        </>
    )
}
export default Layout;
```
On the page:
```js
export default function Welcome() {
    return (
        <>
            <div className="border-red-500 border-4">
                <Layout>
                    <Nav />
                    <p>Extra content</p>
                </Layout>
            </div>
            <h1 className="font-bold text-3xl">Hello World - Welcome page</h1>
        </>
    );
}
```
The Nav component, as well as the extra content inside the paragraph tag will both be placed on where `{children}` is used in the Layout file
### Persistent layouts
Because of the Layout component being a child of the Page component, it gets destroyed and rebuilt every time the user moves onto a different page. If we want to prevent this (for several reasons, maybe the layout contains a video/audio file that shouldn't be restarting every time the page changes), we can use persistent layouts

To use a persistent layout, we have to change the way it gets renderd inside the page file
```js
import Layout from './Layout'

const Home = ({ user }) => {
  return (
    <>
      <H1>Welcome</H1>
      <p>Hello {user.name}, welcome to your first Inertia app!</p>
    </>
  )
}

Home.layout = page => <Layout children={page} title="Welcome" />

export default Home
```
### Default layouts
We can define a default layout to be set for all pages, if we don't want to import it on every single page. This is done in `resource/js/app.jsx`, inside the `resolve` property of the `createInertiaApp`
```js
import Layout from './Layout'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.jsx', { eager: true })
    let page = pages[`./Pages/${name}.jsx`]
    page.default.layout = page.default.layout || (page => <Layout children={page} />)
    return page
  },
  // ...
})
```

It is also possible to set a layout based on the page file's name
```js
import Layout from './Layout'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.jsx', { eager: true })
    let page = pages[`./Pages/${name}.jsx`]
    page.default.layout = name.startsWith('Public/') ? undefined : page => <Layout children={page} />
    return page
  },
  // ...
})
```

## Progress bar on loading pages
By default, there is a loading bar at the top of the page whenever the user loads a different page (to simulate this, there's a `sleep(2)` in the welcome route)

If you want to disable this loading bar, we can modify the `progress` property inside the `createInertiaApp` function
```js
createInertiaApp({
  progress: false,
  // ...
})
```
There are several options to customize the look of the progress bar
```js
createInertiaApp({
  progress: {
    // The delay after which the progress bar will appear, in milliseconds...
    delay: 250,

    // The color of the progress bar...
    color: '#29d',

    // Whether to include the default NProgress styles...
    includeCSS: true,

    // Whether the NProgress spinner will be shown...
    showSpinner: false,
  },
  // ...
})
```

## Non-GET requests through Links
If, for example, we want to log a user out, we have to do this through a POST request. In Inertia, we are able to make a Link use the POST method instead of the default GET request

We can also show a Link component as a different tag, using the `as` attribute
```html
<Link method="POST" as="button" href="/logout">Log out</Link>
```
In the `web.php` file
```php
Route::post("/logout", function() {
    dd("handle logout");
});
```
### Passing through data
We can pass through data this way too, using the `:data` attribute
```html
<Link method="POST" data={{foo: "bar"}} as="button" href="/logout">Log out</Link>
```
And use it in the controller as a request
```php
Route::post("/logout", function() {
    dd(request('foo')); //"bar"
});
```

## Preventing a page from scrolling up when a link is clicked
Whenever a link is clicked, and it redirects to the same page, the page will be scrolled back to the top. In Inertia, we can prevent this from happening, by using the `preserveScroll` attribute inside a Link component
```html
<Link method="POST" 
      as="button"
      data={{post-id: 1, user-id: 5}}  
      preserveScroll>
Like the post
</Link>
```

## Styling active Links
When we're on a page, we have the ability to show the user what page they're currently on inside the nav, using conditional classes

Inertia will check which component is currently rendered, and style the link based on that, so we'll have to import `usePage()` in order for intertia to know
```js
import { usePage } from '@inertiajs/react'
const { url, component } = usePage()

//inside the render
<Link href="/users" className={component.startsWith('Users') ? 'active' : ''}>Users</Link>
```

## Shared data across all pages and components
If we want data to be accessible on every page/component (for example, the data for a logged in user), we can use the share function inside the Inertia middleware, located in `app/Http/Middleware/HandleInertiaRequest`

```js
public function share(Request $request): array
    {
        return array_merge(parent::share($request), [
            'auth' => [
                'user' => $request->user(),
            ],
            'foo' => 'bar',
        ]);
    }
```

To access the shared data, we use the `usePage()` method from Inertia again

```js
import { usePage } from '@inertiajs/react'

export default function Layout({ children }) {
  const { auth } = usePage().props
  const { foo } = usePage().props

  return (
    <main>
      <header>
        You are logged in as: {auth.user.name}
      </header>
      <content>
        {children}
      </content>
    </main>
  )
}
```

Make sure you don't pass through any sensitive information in the shared data, as this data gets shared with the client-side

## Head & Title
If we want to change the meta tags or titles of our pages, we have to use the `<Head>` component, which Inertia offers us

```js
import { Head } from '@inertiajs/react'

<Head>
  <title>Your page title</title>
  <meta name="description" content="Your page description" />
</Head>
```
Note: changing the title will still add *"- Laravel"* behind it. This is the app name, defined in the `.env` file, which you should change to your own

If you only need to add a title, you can use the shorthand
```js
import { Head } from '@inertiajs/react'

<Head title="Your page title" />
```

### What if we have multiple Head instances?
Inertia will overwrite the title tag to the one specified inside your specific page component. For meta-tags, we'll need to tell it which ones are the same and should be overwritten by the ones specified inside the page component, by passing along the `head-key` 

```js
// Layout.js
import { Head } from '@inertiajs/react'

<Head>
  <title>My app</title>
  <meta head-key="description" name="description" content="This is the default description" />
  <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
</Head>

// About.js
import { Head } from '@inertiajs/react'

<Head>
  <title>About - My app</title>
  <meta head-key="description" name="description" content="This is a page specific description" />
</Head>
```

Result
```html
<head>
  <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
  <title>About - My app</title>
  <meta name="description" content="This is a page specific description" />
</head>
```