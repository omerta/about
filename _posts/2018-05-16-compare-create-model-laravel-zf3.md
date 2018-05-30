---
#layout: post
title:  Compare development workflow - Creating a model in ZF3, Symfony, and Laravel
#date:   2018-05-16 16:16:01 -0600
categories: [ zf3, laravel, symfony ]
---

MVC is a architecture paradigm and pattern, MVC create layers for each different
concern: model, view and controller.

For Model-View-Controller all data, logic and business rules is represented
by model layer. Moreover, the model layer is the heart and soul of every MVC
application. For this reason it seems correcto to start with the model. What
is a model in ZF3, Laravel and Symfony?

In this post we describe the development workflow from create a migrations and
its corresponding database table, the model and how to populate it with fake data.

We show this workflow comparing tree PHP frameworks: Laravel, Zend Framework
and Symfony.

## Create `Room` model in Laravel

Suppose that we already installed `laravel`. Laravel is configure to use MySQL
by default. In this tutorial we use `SQLite`, in order to use this database
we change the settings in `app/config/database.php` file. The database is stored
in the `app/database` directory.

In Laravel is posible start to develop creating a migration. A migration define
the database structure through a PHP class. The database schema would be defined
in source code.

From the root of your project, run the `migrate:make` command to create the
migration:

```bash
hotel-laravel$ php artisan make:migration create_rooms_table --create=rooms
```

In the `database/migrations` directory you can find the
`2018_05_15_180115_create_rooms_table.php` file. This file have two methods,
in the `up()` method you describe the future table, for instance, the `number`
column is defined with: `$table->string('number', 10);`.

Using the `migrate` command you can run the migrations.

```bash
hotel-laravel$ php artisan migrate
```

To this point, we have created a database table using two Laravel commands.

We need interact with database. This task is achieved by an ORM. Laravel provides
Eloquent as ActiveRecord ORM.

    *Laravel Up & Running*, Matt Stauffer

    "Eloquent is one of Laravel's most popular and influential features. It's a great example
    of how Laravel is different from the majority of PHP frameworks; in a world of Data‐
    Mapper ORMs that are powerful but complex, Eloquent stands out for its simplicity."

Is time of make the model for the table `rooms`. Laravel ships with `Eloquent`
ORM. The `make:modal` command will create the `app/Room.php` file with the
model for the table `rooms`.

```bash
hotel-laravel$ php artisan make:model Room
```

In the `app/Room.php` file we specify a table to use for `Room` model by defining
a `table` property. Also, we specify a `fillable` property to specifying which
attributes should be "mass-assignable". Actually, this two properties are not
necessary. In compare of Doctrine ORM, a Eloquent model is very anaemic.

In this second stage, we might create a new record in the database using the
`Room` model.

To end we are going to create a seed file to insert a fake data.

    *Laravel Up & Running*, Matt Stauffer

    "Seeding with Laravel is so simple, it has gained widespread adoption as a
    part of normal development workflows in a way it hasn't in previous PHP
    frameworks."

The seed classes are stored in `database/seeds` directory. We use the seeder to
load sample data into database. In order to create a seeder we run the
`make:seeder` command:

```bash
hotel-laravel$ php artisan make:seeder RoomSeeder
```

This command create the `database/seeds/RoomSeeder.php` file. The class `RoomSeeder`
use the `Room` model and the method `create` to create sample data room.

Finally, we may seed the database using the `db:seed` command:

```bash
hotel-laravel$ php artisan db:seed --class=RoomSeeder
```

## Create `Room` model in ZF3

ZF3 have a component named `zend-db` to access to database, but we
use Doctrine ORM instead.

Assuming that already we have installed Zend Framework 3. We set the database
and Doctrine configuration.

To configure the database we edit the `config/autoload/local.php` file. We use
the `PDOSqliteDriver` driver and store de database in the `data/database.sql` file.

Then, we configure Doctrine to interact with the database. In the
`module/Application/config/module.config.php` we set the path where entities
will store. If we have not installed doctrine we can do it with the following
command:

```bash
hotel-zf3$ composer require doctrine/doctrine-orm-module
```

If is necessary create the `module/Application/src/Entity` directory. For this
moment we only have got Doctrine component installed and configured.

Now, we can create the entity room that will represent the room table. We create
the `module/Application/src/Entity/Room.php` file. When the class is ready we
can use the `doctrine/migrations` component to create the database schema.

We edit the `config/autoload/global.php` file in order to configure the
`doctrine/migrations` component. To install the migration component
provided by doctrine, with composer, we run the following command:

```bash
hotel-zf3$ composer required doctrine/migrations
```

If the `doctrine/migrations` component is installed and configured we are ready
to create the migration:

```bash
hotel-zf3$ php vendor/bin/doctrine-module migration:diff
```

The `migration:diff` command will create the `data/Migrations/Version20180516004920.php`
file with the instructions to create the table room. To apply the migrations
we run the following command:

```bash
php vendor/bin/doctrine-module migrations:migrate
```

## Create `Room` model in Symfony

$ php composer.phar create-project symfony/website-skeleton hotel-symfony

We need change de variable `DATABASE_URL` in the `.env`:

```
DATABASE_URL=sqlite:///%kernel.project_dir%/var/database.sql
```

```bash
hotel-symfony$ php bin/console make:entity
```

In the interactive console we type the name of the attributes, its type, if is
nullable and its length.

This command create two files `src/Entity/Room.php` and `src/Repository/RoomRepository.php`

Una vez que la entidad esta creada, generamos la migración con el siguiente
comando:

```bash
hotel-symfony$ php bin/console make:migration
```

El comando anterior crea el archivo `src/Migrations/Version20180517193124.php`
que se usará para crear la tabla en la base de datos.

```bash
php bin/console doctrine:migrations:migrate
```

The we use `DoctrineFixturesBundle` to load sample data into table `room`.

Run the following command to download the bundle:

```bash
hotel-symfony$ php composer.phar require doctrine/doctrine-fixtures-bundle
```

With the command `make:fixtures` we create a class to load the fixture. In the
interactive console we type the name of the class fixture: `RoomFixture`.

```bash
hotel-symfony$ php bin/console make:fixtures RoomFixture
```

Now, we need write a fixture.

For load the fixture we run the fallowing command:

```bash
php bin/console doctrine:fixtures:load
```
