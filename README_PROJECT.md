# E-commerce Shop (Symfony 6.4)

This README is a step-by-step guide to building the project from scratch.

## Table of Contents


1. [Create the project](#1-create-the-project)
2. [Configure the database](#2-configure-the-database)
3. [Create the Product entity](#3-create-the-product-entity)
4. [Create and run the migration](#4-create-and-run-the-migration)
5. [Generate the Product CRUD](#5-generate-the-product-crud)
6. [Run the server](#6-run-the-server)
7. [Integrate the panel theme](#7-integrate-the-panel-theme)
8. [Create the Category entity](#8-create-the-category-entity)
9. [Link Product to Category](#9-link-product-to-category)
10. [Generate the Category CRUD](#10-generate-the-category-crud)
11. [Show the category in the Product pages](#11-show-the-category-in-the-product-pages)


---



## 1. Create the project

```bash
composer create-project symfony/skeleton:"6.4.*" ecommerce_shop
cd ecommerce_shop
composer require webapp
```

`webapp` installs the full-stack pack: Doctrine ORM, Twig, Forms, Security, Maker bundle, etc.

---

## 2. Configure the database

Open the `.env` file at the project root.

1. **Comment out** the PostgreSQL `DATABASE_URL` line.
2. **Uncomment** the MySQL one and adjust the credentials and database name.

```dotenv
# DATABASE_URL="postgresql://app:!ChangeMe!@127.0.0.1:5432/app?serverVersion=16&charset=utf8"
DATABASE_URL="mysql://root:@127.0.0.1:3306/ecommerce_shop?serverVersion=8.0.32&charset=utf8mb4"
```

Format: `mysql://USER:PASSWORD@HOST:PORT/DATABASE_NAME?serverVersion=VERSION&charset=utf8mb4`

> The default Laragon user is `root` with an empty password. If you use MariaDB, set `serverVersion` accordingly (e.g. `10.11.2-MariaDB`).

Create the database:

```bash
symfony console doctrine:database:create
```

---

## 3. Create the Product entity

```bash
symfony console make:entity Product
```

Answer the prompts as follows:

| Property      | Type      | Options                                      | Nullable |
|---------------|-----------|----------------------------------------------|----------|
| `name`        | `string`  | length: `55`                                 | no       |
| `price`       | `decimal` | precision: `10`, scale: `2` (see note below) | yes      |
| `description` | `text`    | -                                            | yes      |

Example session:

```text
New property name (press <return> to stop adding fields):
> name
Field type (enter ? to see all types) [string]:
> string
Field length [255]:
> 55
Can this field be null in the database (nullable) (yes/no) [no]:
> no

Add another property? Enter the property name (or press <return> to stop adding fields):
> price
Field type (enter ? to see all types) [string]:
> decimal
Precision (total number of digits stored: 100.00 would be 5) [10]:
> 10
Scale (number of decimals to store: 100.00 would be 2) [0]:
> 2
Can this field be null in the database (nullable) (yes/no) [no]:
> yes

Add another property? Enter the property name (or press <return> to stop adding fields):
> description
Field type (enter ? to see all types) [string]:
> text
Can this field be null in the database (nullable) (yes/no) [no]:
> yes

Add another property? Enter the property name (or press <return> to stop adding fields):
>   (press Enter to finish)
```

> **Note:** the default scale is `0`, which stores prices as whole numbers (`100`). Use `2` so prices keep their cents (`100.50`).

Files created:

```text
src/Entity/Product.php
src/Repository/ProductRepository.php
```

---

## 4. Create and run the migration

A migration is like a commit for your database structure: each one is a versioned file, and you can move back to an earlier version at any time.

Generate the migration from the current entities:

```bash
symfony console make:migration
```

Execute it:

```bash
symfony console doctrine:migrations:migrate
```

Type `yes` when asked to confirm. The `product` table now exists in the database.

---

## 5. Generate the Product CRUD

```bash
symfony console make:crud Product
```

Accept the defaults (controller name `ProductController`, no PHPUnit tests).

Files created:

```text
src/Controller/ProductController.php
src/Form/ProductType.php
templates/product/_delete_form.html.twig
templates/product/_form.html.twig
templates/product/edit.html.twig
templates/product/index.html.twig
templates/product/new.html.twig
templates/product/show.html.twig
```

> **Heads up:** if the generated form class is named `Product1Type.php` instead of `ProductType.php`, a `ProductType` file already existed. Either rename `Product1Type` to `ProductType` (in the file and in `ProductController.php`), or use `Product1Type` everywhere this guide says `ProductType`.

---

## 6. Run the server

```bash
symfony server:start
```

Open the URL printed in the terminal (usually `https://127.0.0.1:8000`) and go to `/product/`.

---

## 7. Integrate the panel theme

Now we improve the UI by wiring an admin panel theme into the project.

### 7.1 Copy the theme assets into `public/`

The first thing we do is take the CSS, JavaScript, images, and fonts from the downloaded panel template and place them inside the project assets folder.

The structure used in this project is:
```text
public/
└── assets/
    ├── css/
    ├── imgs/
    ├── js/
    └── fonts/
```

This is important because Symfony loads static files from the `public` directory, and the template will reference them using the `asset()` function.
> Keep the theme's internal folder structure. For example, Font Awesome is loaded from `assets/css/fonts/css/font-awesome.css`, so that path must exist.

Example of final asset usage:

```twig
<link rel="stylesheet" href="{{ asset('assets/css/bootstrap.css') }}"/>
<link rel="stylesheet" href="{{ asset('assets/css/main.css') }}"/>
<script src="{{ asset('assets/js/jquery-2.0.3.min.js') }}"></script>
<script src="{{ asset('assets/js/bootstrap.min.js') }}"></script>
```

### 7.2 Configure Twig to use the Bootstrap 3 form theme

Edit `config/packages/twig.yaml`:

```yaml
twig:
    file_name_pattern: '*.twig'
    form_themes: ['bootstrap_3_layout.html.twig']

when@test:
    twig:
        strict_variables: true
```

`form_themes` makes every Symfony form render with Bootstrap 3 markup, matching the panel theme.
### 7.3 Add the panel CSS and JS files to the base template
The global layout of the application is defined in `templates/base.html.twig`. This is the master template used by all pages.

We add the theme stylesheet links inside the `<head>` section:

```twig
<meta content="width=device-width, initial-scale=1.0" name="viewport"/>
<link rel="stylesheet" href="{{ asset('assets/css/bootstrap.css') }}"/>
<link rel="stylesheet" href="{{ asset('assets/css/fonts/css/font-awesome.css') }}"/>
<link rel="stylesheet" href="{{ asset('assets/css/main.css') }}"/>
```

Then we add the required JavaScript before the closing `</html>` tag:

```twig
<script src="{{ asset('assets/js/jquery-2.0.3.min.js') }}"></script>
<script src="{{ asset('assets/js/bootstrap.min.js') }}"></script>
```

This gives the project access to the Bootstrap framework and the custom admin panel styling.

The final HTML structure in `base.html.twig` looks like this:

```twig
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Welcome!{% endblock %}</title>
    <meta content="width=device-width, initial-scale=1.0" name="viewport"/>
    <link rel="stylesheet" href="{{ asset('assets/css/bootstrap.css') }}"/>
    <link rel="stylesheet" href="{{ asset('assets/css/fonts/css/font-awesome.css') }}"/>
    <link rel="stylesheet" href="{{ asset('assets/css/main.css') }}"/>
</head>
...
<script src="{{ asset('assets/js/jquery-2.0.3.min.js') }}"></script>
<script src="{{ asset('assets/js/bootstrap.min.js') }}"></script>
```

### 7.4 Apply the panel layout in the main page structure
Once the assets are available, the layout is organized inside `base.html.twig` using the same structure as the theme.

The template includes:

- a fixed top navigation bar
- a sidebar menu
- a main content panel area
- a page heading section
- a panel body container

Example structure:

```twig
    <body class="padTop53 ">
    <div id="wrap">
        <div id="top">
            <nav class="navbar navbar-inverse navbar-fixed-top " style="padding-top: 10px;">
                <a data-original-title="Show/Hide Menu" data-placement="bottom" data-tooltip="tooltip" class="accordion-toggle btn btn-primary btn-sm visible-xs" data-toggle="collapse" href="#menu" id="menu-toggle">
                    <i class="icon-align-justify"></i>
                </a>
                <header class="navbar-header">
                    <a href="index.html" class="navbar-brand">
                        <img src="{{ asset('assets/imgs/logo.png') }}" alt=""/></a>
                </header>
                <ul class="nav navbar-top-links navbar-right">
                    <li class="dropdown">
                        <a class="dropdown-toggle" data-toggle="dropdown" href="#">
                            <i class="icon-user "></i>&nbsp; <i class="icon-chevron-down "></i>
                        </a>
                        <ul class="dropdown-menu dropdown-user">
                            <li>
                                <a href="#">
                                    <i class="icon-user"></i> Profile
                                </a>
                            </li>
                            <li class="divider"></li>
                            <li>
                                <a href="login.html">
                                    <i class="icon-signout"></i> Logout
                                </a>
                            </li>
                        </ul>
                    </li>
                </ul>
            </nav>
        </div>
        <div id="left">
            <div class="media user-media well-small">
                <a class="user-link" href="#">
                    <img class="media-object img-thumbnail user-img" alt="User Picture" src="{{ asset('assets/imgs/user.gif') }}"/>
                </a>
                <br/>
                <div class="media-body">
                    <h5 class="media-heading"> Aziz</h5>
                    <ul class="list-unstyled user-info">
                        <li>
                            <a class="btn btn-success btn-xs btn-circle" style="width: 10px;height: 12px;"></a> online

                        </li>
                    </ul>
                </div>
                <br/>
            </div>
            <ul id="menu" class="collapse">
                <li class="panel">
                    <a href="{{ path('app_product_index') }}" >
                        <i class="icon-list"></i> Our Products
                    </a>
                </li>
                <li class="panel">
                    <a href="{{ path('app_product_new') }}">
                        <i class="icon-edit"></i> Create Product
                    </a>
                </li>
            </ul>
        </div>
        <div id="content">
            <div class="inner" style="min-height:1200px;">
                <div class="row">
                    <div class="col-lg-12">
                        <h2>Product Management</h2>
                    </div>
                </div>
                <hr/>
                <div class="row">
                    <div class="col-lg-12">
                        <div class="panel panel-default">
                            <div class="panel-heading">
                                {% block panel_heading %}{% endblock %}
                            </div>
                            <div class="panel-body">
                                <div class="table-responsive">
                                    {% block body %}{% endblock %}
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div id="footer">
        <p>
            &copy; <a href="https://menchari.codeberg.page">menchari</a> &nbsp;2024 &nbsp;
        </p>
    </div>

    </body>

```

This is the key part: the panel theme is not applied manually on every page; instead, the global layout provides the panel container and each page only fills the content blocks.
### 7.5 Modify the class of the table in page templates
In `templates/product/index.html.twig`:
```twig
    <table class="table table-striped table-bordered table-hover" id="dataTables-example">
        <thead>
            <tr>
                <th>Id</th>
                <th>Name</th>
                <th>Price</th>
                <th>Description</th>
                <th>actions</th>
            </tr>
        </thead>
        <tbody>
        {% for product in products %}
            <tr>
                <td>{{ product.id }}</td>
                <td>{{ product.name }}</td>
                <td>{{ product.price }}</td>
                <td>{{ product.description }}</td>
                <td>
                    <a class="btn btn-primary" href="{{ path('app_product_show', {'id': product.id}) }}">show</a>
                    <a  class="btn btn-danger" href="{{ path('app_product_edit', {'id': product.id}) }}">edit</a>
                </td>
            </tr>
        {% else %}
            <tr>
                <td colspan="5">no records found</td>
            </tr>
        {% endfor %}
        </tbody>
    </table>

```
### 7.6 Use the panel blocks in page templates

Example: `templates/product/show.html.twig`

```twig
{% extends 'base.html.twig' %}

{% block title %}Product{% endblock %}

{% block panel_heading %}Product: {{ product.name }}{% endblock %}

{% block body %}
    <table class="table">
        <tbody>
            <tr>
                <th>Id</th>
                <td>{{ product.id }}</td>
            </tr>
            <tr>
                <th>Name</th>
                <td>{{ product.name }}</td>
            </tr>
        </tbody>
    </table>
{% endblock %}
```

Apply the same pattern (`extends`, `title`, `panel_heading`, `body`) to the other CRUD templates (`index`, `new`, `edit`) so they all render inside the theme.

---

## 8. Create the Category entity

```bash
symfony console make:entity Category
```

Add one property:

```text
New property name (press <return> to stop adding fields):
> title
Field type (enter ? to see all types) [string]:
>   (press Enter for string)
Field length [255]:
> 55
Can this field be null in the database (nullable) (yes/no) [no]:
> yes
```

Files created:

```text
src/Entity/Category.php
src/Repository/CategoryRepository.php
```

---

## 9. Link Product to Category

Each product belongs to one category, and a category has many products (**ManyToOne** from Product to Category). Re-run `make:entity` on `Product` to add the relation:

```bash
symfony console make:entity Product
```

```text
New property name (press <return> to stop adding fields):
> category
Field type (enter ? to see all types) [string]:
> relation
What class should this entity be related to?:
> Category
Relation type? [ManyToOne, OneToMany, ManyToMany, OneToOne]:
> ManyToOne
Is the Product.category property allowed to be null (nullable)? (yes/no) [yes]:
> yes
Do you want to add a new property to Category so that you can access/update Product objects from it? (yes/no) [yes]:
> yes
New field name inside Category [products]:
>   (press Enter)
Do you want to activate orphanRemoval on your relationship? (yes/no) [no]:
> no
```

Then create and run a migration for both the `category` table and the new relation:

```bash
symfony console make:migration
symfony console doctrine:migrations:migrate
```

A new file appears in `migrations/` (e.g. `migrations/Version20261015124009.php`).

---

## 10. Generate the Category CRUD

```bash
symfony console make:crud Category
```

Files created:

```text
src/Controller/CategoryController.php
src/Form/CategoryType.php
templates/category/_delete_form.html.twig
templates/category/_form.html.twig
templates/category/edit.html.twig
templates/category/index.html.twig
templates/category/new.html.twig
templates/category/show.html.twig
```

Categories are now manageable at `/category/`.

> **Create a few categories first**, otherwise the category dropdown in the product form will be empty.

---

## 11. Show the category in the Product pages

### 11.1 Add the category field to the product form

Edit `src/Form/ProductType.php`:

```php
<?php

namespace App\Form;

use App\Entity\Category;
use App\Entity\Product;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name')
            ->add('price')
            ->add('description')
            ->add('category', EntityType::class, [
                'class' => Category::class,
                'choice_label' => 'title',
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Product::class,
        ]);
    }
}
```

`EntityType` renders a dropdown filled with categories from the database, showing each one's `title`.

### 11.2 Show the category in the product list

In `templates/product/index.html.twig`, add a header cell and a data cell:

```twig
<th>Category</th>
...
<td>{{ product.category?.title }}</td>
```

### 11.3 Show the category on the product details page

In `templates/product/show.html.twig`, add a row:

```twig
<tr>
    <th>Category</th>
    <td>{{ product.category ? product.category.title : 'No category selected' }}</td>
</tr>
```

=

