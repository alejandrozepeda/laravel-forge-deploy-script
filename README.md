ZERO DOWNTIME SCRIPT
Based on https://vincek.co/zero-downtime-deployments-for-laravel-forge/
```
BASE="devjoinfanco"
SITE="dev.joinfan.co"
DEPLOYMENTS="/home/${BASE}/deployments/${SITE}"

mkdir -p ${DEPLOYMENTS}

CURRENT="/home/${BASE}/${SITE}"
NEW="${DEPLOYMENTS}/new"
BACKUP="${DEPLOYMENTS}/backup"

if [ -d ${NEW} ]; then
 rm -rf ${NEW}
fi

cp -R ${CURRENT} ${NEW}

cd ${NEW}

git pull origin $FORGE_SITE_BRANCH

$FORGE_COMPOSER install --no-interaction --prefer-dist --optimize-autoloader --no-dev 

( flock -w 10 9 || exit 1
    echo 'Restarting FPM...'; sudo -S service $FORGE_PHP_FPM reload ) 9>/tmp/fpmlock

npm ci
npm run production

if [ -f artisan ]; then
    $FORGE_PHP artisan migrate --force
    $FORGE_PHP artisan optimize:clear
    $FORGE_PHP artisan queue:restart
    $FORGE_PHP artisan auth:clear-resets
fi

rm -rf node_modules

if [ -d ${BACKUP} ]; then
 rm -rf ${BACKUP}
fi

mv ${CURRENT} ${BACKUP}
mv ${NEW} ${CURRENT}

cd ${CURRENT}

if [ -f public/storage ]; then
    rm public/storage
fi

if [ -f artisan ]; then
    $FORGE_PHP artisan storage:link
    $FORGE_PHP artisan optimize
    $FORGE_PHP artisan config:cache
    $FORGE_PHP artisan route:cache
    $FORGE_PHP artisan view:cache
fi
```

OLD SCRIPT
```
cd /home/devjoinfanco/dev.joinfan.co

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
