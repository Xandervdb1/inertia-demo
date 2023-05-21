# Laravel - React stack using Inertia.js
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