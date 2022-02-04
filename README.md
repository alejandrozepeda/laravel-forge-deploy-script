# laravel-forge-deploy-script

```
cd /home/joinfanco/joinfan.co

if [ -f artisan ]; then
    $FORGE_PHP artisan down || true
fi 

git pull origin $FORGE_SITE_BRANCH

$FORGE_COMPOSER install --no-interaction --prefer-dist --optimize-autoloader --no-dev 

( flock -w 10 9 || exit 1
    echo 'Restarting FPM...'; sudo -S service $FORGE_PHP_FPM reload ) 9>/tmp/fpmlock

if [ -f artisan ]; then
    $FORGE_PHP artisan migrate --force
    $FORGE_PHP artisan cache:clear
    $FORGE_PHP artisan route:cache
    $FORGE_PHP artisan config:cache
    $FORGE_PHP artisan view:cache
fi

npm ci
npm run production

if [ -f artisan ]; then
    $FORGE_PHP artisan up
fi
```
