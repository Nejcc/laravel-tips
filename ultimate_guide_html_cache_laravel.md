
# Ultimate Guide to HTML Cache in Laravel

## Table of Contents

- [Why HTML Caching?](#why-html-caching)
- [Setup and Prerequisites](#setup-and-prerequisites)
- [Basic HTML Caching](#basic-html-caching)
- [Caching HTML Pages Forever](#caching-html-pages-forever)
- [Dynamic Content with Cache](#dynamic-content-with-cache)
- [Clearing HTML Caches](#clearing-html-caches)
- [Applying Middleware to Routes](#applying-middleware-to-routes)
- [Caveats and Considerations](#caveats-and-considerations)

## Why HTML Caching?

Caching HTML pages can:
- Reduce server load
- Improve page load speed
- Enhance the user experience

## Setup and Prerequisites

1. Laravel project set up and running
2. Basic knowledge of middleware and Blade templates

## Basic HTML Caching

### Step 1: Create Middleware

Run the following command to create a new middleware.

```bash
php artisan make:middleware CachePage
```

### Step 2: Add Caching Logic

Modify `app/Http/Middleware/CachePage.php`:

```php
// ... existing import statements
use Illuminate\Support\Facades\Cache;

class CachePage
{
    public function handle($request, Closure $next)
    {
        $key = 'page_' . sha1($request->url());
        if (Cache::has($key)) {
            return response(Cache::get($key));
        }
        $response = $next($request);
        Cache::put($key, $response->getContent(), now()->addMinutes(5));
        return $response;
    }
}
```

### Step 3: Register Middleware

Register your middleware in `app/Http/Kernel.php`.

### Step 4: Apply Middleware to Routes

Apply the middleware to routes in `routes/web.php`.

```php
Route::get('/some-page', 'SomeController@someMethod')->middleware('cache.page');
```

## Caching HTML Pages Forever

### Steps 1 to 3: Same as Basic HTML Caching

Use `Cache::forever($key, $response->getContent());` for indefinite caching.

## Dynamic Content with Cache

### Step 1: Blade Directives

Create Blade directives in `AppServiceProvider.php` to exclude sections from being cached.

```php
// AppServiceProvider.php

Blade::directive('notcached', function ($expression) {
    return "<?php ob_start(); ?>";
});

Blade::directive('endnotcached', function ($expression) {
    return "<?php ob_end_clean(); ?>";
});
```

### Step 2: Use Blade Directives in Views

Wrap dynamic content between `@notcached` and `@endnotcached`.

```blade
@notcached
    <p>This is Dynamic: {{ time() }}</p>
@endnotcached
```

## Clearing HTML Caches

### Method 1: Cache Prefix

Use a specific prefix for all HTML cache keys and manually loop through to clear them.

### Method 2: Cache Tags

Utilize cache tags to easily flush out specific caches.

```php
Cache::tags(['html_page_cache'])->flush();
```

## Applying Middleware to Routes

### Option 1: Cache All Web Pages Globally

Apply middleware globally in `RouteServiceProvider`.

```php
// RouteServiceProvider.php

protected function mapWebRoutes()
{
    Route::middleware(['web', 'cache.page.forever'])
        ->namespace($this->namespace)
        ->group(base_path('routes/web.php'));
}
```

### Option 2: Cache Specific Routes

Apply the middleware directly to specific routes.

```php
// Cache for a limited time
Route::get('/some-page', 'SomeController@someMethod')->middleware('cache.page');

// Cache indefinitely
Route::get('/another-page', 'AnotherController@anotherMethod')->middleware('cache.page.forever');
```

### Option 3: Cache Group of Routes

Group several routes and apply the middleware to all.

```php
Route::middleware(['cache.page.forever'])->group(function () {
    Route::get('/page1', 'PageController@page1');
    Route::get('/page2', 'PageController@page2');
    // ... more routes
});
```

## Caveats and Considerations

- Be cautious when caching pages with user-specific data.
- Use cache drivers that support the features you need (like tagging).

