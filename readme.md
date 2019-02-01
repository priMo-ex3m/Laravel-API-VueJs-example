# Larticles Laravel/Vue App

> Laravel 5.7 API that uses the API resources with a Vue.js frontend

## Quick Start

``` bash
# Install Dependencies
composer install

# Connect to your DB in .env file

# Run Migrations
php artisan migrate

# Import Articles
php artisan db:seed

# If you get an error about an encryption key
php artisan key:generate

# Install JS Dependencies
npm install

# Watch Files
npm run watch

# Open website in browser
If you use laravel built-in server (php artisan serve) go to localhost:8000
```

## Endpoints

### List all articles with links and meta
``` bash
GET api/articles
```
### Get single article
``` bash
GET api/articles/{article}
```

### Delete article
``` bash
DELETE api/articles/{article}
```

### Add article
``` bash
POST api/articles
title/body
```

### Update article
``` bash
PUT api/articles/{article}
title/body
```


