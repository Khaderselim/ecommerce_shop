# Setup & Start (quick)

1. Copy project to local server folder
   - Laragon: C:/laragon/www/ecommerce_shop

2. Open terminal
   cd C:/laragon/www/ecommerce_shop

3. Install deps
   composer install

4. Start server
   symfony server:start

5. Database (if needed)
   symfony console doctrine:database:create
   symfony console doctrine:migrations:migrate
