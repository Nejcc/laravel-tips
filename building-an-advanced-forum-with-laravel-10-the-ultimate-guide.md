
# Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Setting up Laravel 10](#setting-up-laravel-10)
    - [Installing Laravel](#installing-laravel)
    - [Setting up the Database](#setting-up-the-database)
    - [Basic Configuration](#basic-configuration)
4. [Models and Migrations Continued](#models-and-migrations-continued)
5. [Controllers](#controllers)
    - [UserController](#usercontroller)
    - [PostController](#postcontroller)
    - [CommentController](#commentcontroller)
6. [Policies and Authorization](#policies-and-authorization)
7. [Form Requests for Validation](#form-requests-for-validation)
8. [Services](#services)
9. [Views](#views)
10. [Implementing Permissions with Spatie Laravel Permission](#implementing-permissions-with-spatie-laravel-permission)
11. [Testing](#testing)

# Code Structure Tree

```
advanced-forum/
|-- app/
|   |-- Http/
|   |   |-- Controllers/
|   |   |   |-- UserController.php
|   |   |   |-- PostController.php
|   |   |   |-- CommentController.php
|   |-- Providers/
|   |   |-- AuthServiceProvider.php
|   |-- Services/
|   |   |-- UserService.php
|-- database/
|   |-- migrations/
|   |   |-- create_users_table.php
|   |   |-- create_posts_table.php
|   |   |-- create_comments_table.php
|-- resources/
|   |-- views/
|   |   |-- posts/
|   |   |   |-- index.blade.php
|   |   |   |-- show.blade.php
|   |   |   |-- create.blade.php
|-- routes/
|   |-- web.php
|-- tests/
|   |-- Feature/
|   |   |-- PostControllerTest.php
|   |-- Unit/
|   |   |-- UserModelTest.php
|   |   |-- PostPolicyTest.php
```

# Building an Advanced Forum with Laravel 10: The Ultimate Guide

---

## Introduction

Welcome to the ultimate guide on building an advanced forum using Laravel 10. In this tutorial, we'll leverage the power of Laravel 10 and Spatie's Laravel Permission package to create a full-fledged forum with advanced features like roles and permissions.

---

## Prerequisites

Before you begin, make sure you have the following:

- PHP >= 7.4
- Composer
- Laravel CLI
- A MySQL database

---

## Setting up Laravel 10

### Installing Laravel

Run the following command to install a new Laravel project.

```bash
composer create-project laravel/laravel advanced-forum
```

This command installs a fresh Laravel project named `advanced-forum`.

### Setting up the Database

Open your `.env` file and set up your database credentials.

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=forum
DB_USERNAME=root
DB_PASSWORD=
```

### Basic Configuration

After setting up the database, run migrations to set up the basic tables.

```bash
php artisan migrate
```

---

## Models and Migrations Continued

### Enhancing the `User` Model

Let's add a `username` field to the `User` model for better identification on the forum.

Update the `User` migration:

```php
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('username')->unique();
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamp('email_verified_at')->nullable();
        $table->string('password');
        $table->rememberToken();
        $table->timestamps();
    });
}
```

In the `User` model, define the relationships:

```php
public function posts()
{
    return $this->hasMany(Post::class);
}

public function comments()
{
    return $this->hasMany(Comment::class);
}
```

### Enhancing the `Post` Model

In addition to the basic fields, let's include a `status` field to indicate if a post is published, draft, or archived.

Update the `Post` migration:

```php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained();
        $table->string('title');
        $table->text('content');
        $table->enum('status', ['draft', 'published', 'archived']);
        $table->softDeletes();
        $table->timestamps();
    });
}
```

In the `Post` model, define the relationships and soft deletes:

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes;

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

### Creating the `Comment` Model

Run the following command to create the `Comment` model and its migration:

```bash
php artisan make:model Comment -m
```

Define the migration schema for the `Comment` table:

```php
public function up()
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained();
        $table->foreignId('post_id')->constrained();
        $table->text('body');
        $table->timestamps();
    });
}
```

In the `Comment` model, define the relationships:

```php
public function user()
{
    return $this->belongsTo(User::class);
}

public function post()
{
    return $this->belongsTo(Post::class);
}
```

### Running Migrations

After making these changes, roll back and rerun the migrations.

```bash
php artisan migrate:refresh
```

---

With these enhancements, our models and migrations are now more suited for an advanced forum. They include user-posts and posts-comments relationships, soft deletes for posts, and additional fields like `username` and `status`.



---

## Controllers

### UserController

#### Create a New User

To create a new user, we'll need a controller method that handles the logic.

Run the artisan command to create a new controller:

```bash
php artisan make:controller UserController
```

In `UserController.php`, add the `store` method:

```php
public function store(Request $request)
{
    $validatedData = $request->validate([
        'username' => 'required|unique:users',
        'name' => 'required',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8',
    ]);

    $user = User::create($validatedData);
    return response()->json(['message' => 'User created successfully', 'data' => $user]);
}
```

### PostController

#### Create a New Post

Run the following command to create a new controller:

```bash
php artisan make:controller PostController
```

In `PostController.php`, add the `store` method:

```php
public function store(Request $request)
{
    $validatedData = $request->validate([
        'title' => 'required',
        'content' => 'required',
        'status' => 'required|in:draft,published,archived',
    ]);

    $post = Post::create($validatedData);
    return response()->json(['message' => 'Post created successfully', 'data' => $post]);
}
```

### CommentController

#### Add Comment to Post

Run the following command to create a new controller:

```bash
php artisan make:controller CommentController
```

In `CommentController.php`, add the `store` method:

```php
public function store(Request $request, Post $post)
{
    $validatedData = $request->validate([
        'body' => 'required',
    ]);

    $comment = new Comment($validatedData);
    $comment->user_id = auth()->id();
    $post->comments()->save($comment);

    return response()->json(['message' => 'Comment added successfully', 'data' => $comment]);
}
```

---

## Policies and Authorization

### Creating Policies

Let's create a policy for the `Post` model.

```bash
php artisan make:policy PostPolicy --model=Post
```

In `PostPolicy.php`, define a method to check if a user is authorized to update a post.

```php
public function update(User $user, Post $post)
{
    return $user->id === $post->user_id;
}
```

### Authorization in Controllers

Now, let's integrate the policy into the `PostController`.

```php
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);

    // Update logic here
}
```

### Using Gates

Create a gate to define a general authorization logic. In your `AuthServiceProvider`, add:

```php
Gate::define('create-post', function ($user) {
    return $user->role === 'admin';
});
```

You can now use this gate in any of your controllers:

```php
if (Gate::allows('create-post')) {
    // The current user can create posts
}
```

---



---

## Form Requests for Validation

### CreatePostRequest

Instead of validating within the controller, we'll use form request validation.

Create a new form request:

```bash
php artisan make:request CreatePostRequest
```

In `CreatePostRequest.php`, specify the rules:

```php
public function rules()
{
    return [
        'title' => 'required|max:255',
        'content' => 'required',
        'status' => 'required|in:draft,published,archived',
    ];
}
```

Update the `store` method in `PostController`:

```php
public function store(CreatePostRequest $request)
{
    $post = Post::create($request->validated());
    return response()->json(['message' => 'Post created successfully', 'data' => $post]);
}
```

### CreateUserRequest

Similarly, create a request for user creation:

```bash
php artisan make:request CreateUserRequest
```

Specify the rules in `CreateUserRequest.php`:

```php
public function rules()
{
    return [
        'username' => 'required|unique:users',
        'name' => 'required',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8',
    ];
}
```

---

## Services

### UserService

Create a service class to handle business logic for users.

```bash
php artisan make:service UserService
```

In `UserService.php`, add a method for creating a user:

```php
public function createUser(array $data)
{
    return User::create($data);
}
```

Inject this service into `UserController` and use it in the `store` method:

```php
public function __construct(UserService $userService)
{
    $this->userService = $userService;
}

public function store(CreateUserRequest $request)
{
    $user = $this->userService->createUser($request->validated());
    return response()->json(['message' => 'User created successfully', 'data' => $user]);
}
```

---

## Views

### Blade Templates

Create Blade templates for listing posts, showing individual posts, and creating new posts.

- `resources/views/posts/index.blade.php`
- `resources/views/posts/show.blade.php`
- `resources/views/posts/create.blade.php`

#### Listing Posts (`index.blade.php`)

```blade
@foreach ($posts as $post)
    <h2>{{ $post->title }}</h2>
    <p>{{ $post->content }}</p>
@endforeach
```

#### Showing Individual Posts (`show.blade.php`)

```blade
<h1>{{ $post->title }}</h1>
<p>{{ $post->content }}</p>
```

#### Creating New Posts (`create.blade.php`)

```blade
<!-- Form to create a new post -->
```

---



### Creating New Posts (`create.blade.php`)

In this section, we'll create a Blade form for adding new posts.

Create a new Blade file at `resources/views/posts/create.blade.php` and add the following content:

```blade
<form method="POST" action="{{ route('posts.store') }}">
    @csrf
    <div class="form-group">
        <label for="title">Title</label>
        <input type="text" class="form-control" id="title" name="title" required>
    </div>
    <div class="form-group">
        <label for="content">Content</label>
        <textarea class="form-control" id="content" name="content" rows="4" required></textarea>
    </div>
    <div class="form-group">
        <label for="status">Status</label>
        <select class="form-control" id="status" name="status">
            <option value="draft">Draft</option>
            <option value="published">Published</option>
            <option value="archived">Archived</option>
        </select>
    </div>
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

This Blade form includes fields for the title, content, and status of the post. The form sends a POST request to the `store` method in `PostController`, which we previously defined.



---

## Implementing Permissions with Spatie Laravel Permission

### Installing the Package

To get started with permissions and roles, we'll use the Spatie Laravel Permission package. Install it using Composer:

```bash
composer require spatie/laravel-permission
```

### Setting up Roles and Permissions

After installing the package, run its migrations:

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="migrations"
php artisan migrate
```

Let's create some roles and permissions. You can do this in a seeder or directly in your code:

```php
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

// Creating roles
$adminRole = Role::create(['name' => 'admin']);
$userRole = Role::create(['name' => 'user']);

// Creating permissions
$postCreate = Permission::create(['name' => 'create posts']);
$postEdit = Permission::create(['name' => 'edit posts']);

// Assigning permissions to roles
$adminRole->givePermissionTo($postCreate, $postEdit);
$userRole->givePermissionTo($postCreate);
```

### Assigning Roles and Permissions

To assign roles to a user, you can use the `assignRole` method:

```php
$user->assignRole('admin');
```

To assign permissions directly to a user:

```php
$user->givePermissionTo('edit posts');
```

### Checking Permissions in Blade Views

In Blade views, you can check permissions and roles like this:

```blade
@can('edit posts')
    <!-- Show edit button -->
@endcan

@role('admin')
    <!-- Show admin panel link -->
@endrole
```

---

## Testing

### Unit Testing Models

To test models, you can use Laravel's built-in testing features.

```bash
php artisan make:test UserModelTest --unit
```

```php
public function test_a_user_can_have_many_posts()
{
    $user = User::factory()->create();
    $post1 = Post::factory()->create(['user_id' => $user->id]);
    $post2 = Post::factory()->create(['user_id' => $user->id]);

    $this->assertEquals(2, $user->posts->count());
}
```

### Feature Testing Controllers

Create a feature test for your controllers.

```bash
php artisan make:test PostControllerTest
```

```php
public function test_post_can_be_created()
{
    $this->post('/posts', [
        'title' => 'New Post',
        'content' => 'Post content',
        'status' => 'published',
    ])->assertStatus(201);
}
```

### Policy Tests

You can also test your policies to ensure they are working as expected.

```bash
php artisan make:test PostPolicyTest
```

```php
public function test_only_post_owner_can_update_post()
{
    $user = User::factory()->create();
    $post = Post::factory()->create(['user_id' => $user->id]);

    $this->actingAs($user);

    $response = $this->put("/posts/{$post->id}", [
        'title' => 'Updated',
    ]);

    $response->assertStatus(200);
}
```

---

