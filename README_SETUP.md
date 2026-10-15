# Setup & Start (quick)

### 1. Copy the project into your local server folder

If you are using Laragon, place the application in the `www` folder:

```text
C:/laragon/www/ecommerce_shop
```

If the project is stored on GitHub or another platform, clone or download it into the same folder structure.

### 2. Open the project in the terminal

```bash
cd C:/laragon/www/ecommerce_shop
```

### 3. Install the dependencies

```bash
composer install
```

This installs all Symfony and PHP dependencies required for the application to run.

### 4. Start the application

You can start it with:

```bash
symfony server:start
```

### 5. Update the database 

Run:

```bash
symfony console doctrine:database:create
symfony console doctrine:migrations:migrate
```
