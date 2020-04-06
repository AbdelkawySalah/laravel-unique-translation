# Laravel Unique Translation

[![GitHub release](https://img.shields.io/github/release/codezero-be/laravel-unique-translation.svg)]()
[![License](https://img.shields.io/packagist/l/codezero/laravel-unique-translation.svg)]()
[![Build Status](https://scrutinizer-ci.com/g/codezero-be/laravel-unique-translation/badges/build.png?b=master)](https://scrutinizer-ci.com/g/codezero-be/laravel-unique-translation/build-status/master)
[![Code Coverage](https://scrutinizer-ci.com/g/codezero-be/laravel-unique-translation/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/codezero-be/laravel-unique-translation/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/codezero-be/laravel-unique-translation/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/codezero-be/laravel-unique-translation/?branch=master)
[![Total Downloads](https://img.shields.io/packagist/dt/codezero/laravel-unique-translation.svg)](https://packagist.org/packages/codezero/laravel-unique-translation)

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/R6R3UQ8V)

#### Check if a translated value in a JSON column is unique in the database.

Imagine you want store a `slug` for a `Post` model in different languages.

The amazing [`spatie/laravel-translatable`](https://github.com/spatie/laravel-translatable) package makes this a cinch!

But then you want to make sure each translation is unique for its language.

That's where this package comes in to play.

This package also supports [`spatie/nova-translatable`](https://github.com/spatie/nova-translatable/) in case you are using [Laravel Nova](https://nova.laravel.com/).

## ✅ Requirements

-   PHP ^7.2
-   MySQL >= 5.7
-   [Laravel](https://laravel.com/) >= 6 (use v1.* for Laravel 5.*)
-   [spatie/laravel-translatable](https://github.com/spatie/laravel-translatable) ^4.2
-   [spatie/nova-translatable](https://github.com/spatie/nova-translatable/) ^3.0

## 📦 Installation

Require the package via Composer:

```
composer require codezero/laravel-unique-translation
```
Laravel will automatically register the [ServiceProvider](https://github.com/codezero-be/laravel-unique-translation/blob/master/src/UniqueTranslationServiceProvider.php).

## 🛠 Usage

For the following examples, I will use a `slug` in a `posts` table as the subject of our validation.

### ☑️ Validate a Single Translation

Your form can submit a single slug:

```html
<input name="slug">
```

We can then check if it is unique **in the current locale**:

```php
$attributes = request()->validate([
    'slug' => 'required|unique_translation:posts',
]);
```

You could also use the Rule instance:

```php
use CodeZero\UniqueTranslation\UniqueTranslationRule;

$attributes = request()->validate([
    'slug' => ['required', UniqueTranslationRule::for('posts')],
]);
```

### ☑️ Validate an Array of Translations

Your form can also submit an array of slugs.

```html
<input name="slug[en]">
<input name="slug[nl]">
```

We need to validate the entire array in this case. Mind the `slug.*` key.

```php
$attributes = request()->validate([
    'slug.*' => 'unique_translation:posts',
    // or...
    'slug.*' => UniqueTranslationRule::for('posts'),
]);
```

### ☑️ Specify a Column

Maybe your form field has a name of `post_slug` and your database field `slug`:

```php
$attributes = request()->validate([
    'post_slug.*' => 'unique_translation:posts,slug',
    // or...
    'post_slug.*' => UniqueTranslationRule::for('posts', 'slug'),
]);
```

### ☑️ Specify a Database Connection

If you are using multiple database connections, you can specify which one to use by prepending it to the table name, separated by a dot:

```php
$attributes = request()->validate([
    'slug.*' => 'unique_translation:db_connection.posts',
    // or...
    'slug.*' => UniqueTranslationRule::for('db_connection.posts'),
]);
```

### ☑️ Ignore a Record with ID

If you're updating a record, you may want to ignore the post itself from the unique check.

```php
$attributes = request()->validate([
    'slug.*' => "unique_translation:posts,slug,{$post->id}",
    // or...
    'slug.*' => UniqueTranslationRule::for('posts')->ignore($post->id),
]);
```

### ☑️ Ignore Records with a Specific Column and Value

If your ID column has a different name, or you just want to use another column:

```php
$attributes = request()->validate([
    'slug.*' => 'unique_translation:posts,slug,ignore_value,ignore_column',
    // or...
    'slug.*' => UniqueTranslationRule::for('posts')->ignore('ignore_value', 'ignore_column'),
]);
```

### ☑️ Use Additional Where Clauses

You can add 4 types of where clauses to the rule.

#### `where`

```php
$attributes = request()->validate([
    'slug.*' => "unique_translation:posts,slug,null,null,column,value",
    // or...
    'slug.*' => UniqueTranslationRule::for('posts')->where('column', 'value'),
]);
```

#### `whereNot`

```php
$attributes = request()->validate([
    'slug.*' => "unique_translation:posts,slug,null,null,column,!value",
    // or...
    'slug.*' => UniqueTranslationRule::for('posts')->whereNot('column', 'value'),
]);
```

#### `whereNull`

```php
$attributes = request()->validate([
    'slug.*' => "unique_translation:posts,slug,null,null,column,NULL",
    // or...
    'slug.*' => UniqueTranslationRule::for('posts')->whereNull('column'),
]);
```

#### `whereNotNull`

```php
$attributes = request()->validate([
    'slug.*' => "unique_translation:posts,slug,null,null,column,NOT_NULL",
    // or...
    'slug.*' => UniqueTranslationRule::for('posts')->whereNotNull('column'),
]);
```

### ☑️ Laravel Nova

If you are using [Laravel Nova](https://nova.laravel.com/) in combination with  [`spatie/nova-translatable`](https://github.com/spatie/nova-translatable/), then you can add the validation rule like this:

```php
Text::make(__('Slug'), 'slug')
  ->creationRules('unique_translation:posts,slug')
  ->updateRules('unique_translation:posts,slug,{{resourceId}}');
```

## 🖥 Example

Your existing `slug`  column (JSON) in a `posts` table:

```json
{
  "en":"not-abc",
  "nl":"abc"
}
```

Your form input to create a new record:


```html
<input name="slug[en]" value="abc">
<input name="slug[nl]" value="abc">
```

Your validation logic:

```php
$attributes = request()->validate([
    'slug.*' => 'unique_translation:posts',
]);
```

The result is that `slug[en]` is valid, since the only `en` value in the database is `not-abc`.

And `slug[nl]` would fail, because there already is a `nl` value of `abc`.

## ⚠️ Error Messages

You can pass your own error messages as normal.

When validating a single form field:

```html
<input name="slug">
```

```php
$attributes = request()->validate([
    'slug' => 'unique_translation:posts',
], [
    'slug.unique_translation' => 'Your custom :attribute error.',
]);
```

In your view you can then get the error with `$errors->first('slug')`.

Or when validation an array:

```html
<input name="slug[en]">
```

```php
$attributes = request()->validate([
    'slug.*' => 'unique_translation:posts',
], [
    'slug.*.unique_translation' => 'Your custom :attribute error.',
]);
```

In your view you can then get the error with `$errors->first('slug.en')` (`en` being your array key).

## 🚧 Testing

```
vendor/bin/phpunit
```

## ☕️ Credits

- [Ivan Vermeyen](https://byterider.io)
- [All contributors](../../contributors)

## 🔓 Security

If you discover any security related issues, please [e-mail me](mailto:ivan@codezero.be) instead of using the issue tracker.

## 📑 Changelog

See a list of important changes in the [changelog](https://github.com/codezero-be/laravel-unique-translation/blob/master/CHANGELOG.md).

## 📜 License

The MIT License (MIT). Please see [License File](https://github.com/codezero-be/laravel-unique-translation/blob/master/LICENSE.md) for more information.
