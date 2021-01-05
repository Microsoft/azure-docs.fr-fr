---
title: Configurer des applications PHP
description: Découvrez comment configurer une application PHP dans les instances Windows natives ou dans un conteneur PHP prédéfini, dans Azure App Service. Cet article présente les tâches de configuration les plus courantes.
ms.devlang: php
ms.topic: article
ms.date: 06/02/2020
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: afac8273b5729bcf5470be471145214426dc7dab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90055297"
---
# <a name="configure-a-php-app-for-azure-app-service"></a>Configurer une application PHP pour Azure App Service

Ce guide vous explique comment configurer les applications web PHP, les back-ends mobiles et les applications API dans Azure App Service.

Ce guide fournit les concepts et instructions clés aux développeurs PHP qui déploient des applications sur App Service. Si vous n’avez jamais utilisé Azure App Service, commencez par suivre le [démarrage rapide PHP](quickstart-php.md) et le [tutoriel PHP avec MySQL](tutorial-php-mysql-app.md).

## <a name="show-php-version"></a>Afficher la version PHP

::: zone pivot="platform-windows"  

Pour afficher la version actuelle de PHP, exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query phpVersion
```

Pour afficher toutes les versions prises en charge de PHP, exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp list-runtimes | grep php
```

::: zone-end

::: zone pivot="platform-linux"

Pour afficher la version actuelle de PHP, exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

Pour afficher toutes les versions prises en charge de PHP, exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp list-runtimes --linux | grep PHP
```

::: zone-end

## <a name="set-php-version"></a>Définir la version PHP

::: zone pivot="platform-windows"  

Exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) pour définir la version 7.4 de PHP :

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --php-version 7.4
```

::: zone-end

::: zone pivot="platform-linux"

Exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) pour définir la version 7.2 de PHP :

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --linux-fx-version "PHP|7.2"
```

::: zone-end

::: zone pivot="platform-windows"  

## <a name="run-composer"></a>Exécuter Composer

Si vous souhaitez qu’App Service exécute [Composer](https://getcomposer.org/) au moment du déploiement, le moyen le plus simple consiste à inclure le Composer dans votre référentiel.

Dans une fenêtre de terminal local, accédez à la racine de votre référentiel, puis suivez les instructions de [Télécharger Composer](https://getcomposer.org/download/) pour télécharger *composer.phar* à la racine du répertoire.

Exécutez les commandes suivantes ([npm](https://www.npmjs.com/get-npm) doit être installé) :

```bash
npm install kuduscript -g
kuduscript --node --scriptType bash --suppressPrompt
```

La racine du référentiel comporte désormais deux fichiers supplémentaires : *.deployment* et *deploy.sh*.

Ouvrez *deploy.sh* et recherchez la section `Deployment`, qui ressemble à ce qui suit :

```bash
##################################################################################################################################
# Deployment
# ----------
```

Ajoutez la section de code dont vous avez besoin pour exécuter l’outil requis *à la fin* de la section `Deployment` :

```bash
# 4. Use composer
echo "$DEPLOYMENT_TARGET"
if [ -e "$DEPLOYMENT_TARGET/composer.json" ]; then
  echo "Found composer.json"
  pushd "$DEPLOYMENT_TARGET"
  php composer.phar install $COMPOSER_ARGS
  exitWithMessageOnError "Composer install failed"
  popd
fi
```

Validez toutes vos modifications et déployez votre code à l’aide de Git ou de Zip Deploy avec l’automatisation de build activée. Composer doit maintenant s’exécuter dans le cadre de l’automatisation du déploiement.

## <a name="run-gruntbowergulp"></a>Exécuter Grunt/Bower/Gulp

Si vous souhaitez qu’App Service exécute certains outils d’automatisation populaires au moment du déploiement, par exemple Grunt, Bower ou Gulp, vous devez fournir un [script de déploiement personnalisé](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script). App Service exécute ce script lorsque vous déployez avec Git, ou avec le [déploiement Zip](deploy-zip.md) avec l’automatisation de build activée. 

Pour permettre à votre référentiel d’exécuter ces outils, vous devez les ajouter aux dépendances dans *package.json*. Par exemple :

```json
"dependencies": {
  "bower": "^1.7.9",
  "grunt": "^1.0.1",
  "gulp": "^3.9.1",
  ...
}
```

Dans une fenêtre de terminal locale, remplacez le répertoire par la racine du référentiel et exécutez les commandes suivantes ([npm](https://www.npmjs.com/get-npm) doit être installé) :

```bash
npm install kuduscript -g
kuduscript --node --scriptType bash --suppressPrompt
```

La racine du référentiel comporte désormais deux fichiers supplémentaires : *.deployment* et *deploy.sh*.

Ouvrez *deploy.sh* et recherchez la section `Deployment`, qui ressemble à ce qui suit :

```bash
##################################################################################################################################
# Deployment
# ----------
```

Cette section se termine par l’exécution de `npm install --production`. Ajoutez la section de code dont vous avez besoin pour exécuter l’outil requis *à la fin* de la section `Deployment` :

- [Bower](#bower)
- [Gulp](#gulp)
- [Grunt](#grunt)

Consultez un [exemple dans l’échantillon MEAN.js](https://github.com/Azure-Samples/meanjs/blob/master/deploy.sh#L112-L135), où le script de déploiement exécute également une commande `npm install` personnalisée.

### <a name="bower"></a>Bower

Cet extrait de code exécute `bower install`.

```bash
if [ -e "$DEPLOYMENT_TARGET/bower.json" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/bower install
  exitWithMessageOnError "bower failed"
  cd - > /dev/null
fi
```

### <a name="gulp"></a>Gulp

Cet extrait de code exécute `gulp imagemin`.

```bash
if [ -e "$DEPLOYMENT_TARGET/gulpfile.js" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/gulp imagemin
  exitWithMessageOnError "gulp failed"
  cd - > /dev/null
fi
```

### <a name="grunt"></a>Grunt

Cet extrait de code exécute `grunt`.

```bash
if [ -e "$DEPLOYMENT_TARGET/Gruntfile.js" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/grunt
  exitWithMessageOnError "Grunt failed"
  cd - > /dev/null
fi
```

::: zone-end

::: zone pivot="platform-linux"

## <a name="customize-build-automation"></a>Personnaliser l’automatisation de la génération

Si vous déployez votre application à l’aide de packages Git ou zip quand l’automatisation de la génération est activée, ce processus d’automatisation d’App Service exécute pas à pas la séquence suivante :

1. Exécution du script personnalisé s’il est spécifié par `PRE_BUILD_SCRIPT_PATH`.
1. Exécutez `php composer.phar install`.
1. Exécution du script personnalisé s’il est spécifié par `POST_BUILD_SCRIPT_PATH`.

`PRE_BUILD_COMMAND` et `POST_BUILD_COMMAND` sont des variables d’environnement qui sont vides par défaut. Pour exécuter des commandes pré-build, définissez `PRE_BUILD_COMMAND`. Pour exécuter des commandes post-build, définissez `POST_BUILD_COMMAND`.

L’exemple suivant définit les deux variables sur une série de commandes, séparées par des virgules.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PRE_BUILD_COMMAND="echo foo, scripts/prebuild.sh"
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings POST_BUILD_COMMAND="echo foo, scripts/postbuild.sh"
```

Pour connaître les autres variables d’environnement permettant de personnaliser l’automatisation de la génération, consultez [Configuration d’Oryx](https://github.com/microsoft/Oryx/blob/master/doc/configuration.md).

Pour plus d’informations sur la façon dont App Service exécute et génère des applications PHP dans Linux, consultez [Documentation Oryx : Comment les applications PHP sont détectées et générées](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/php.md).

## <a name="customize-start-up"></a>Personnaliser le démarrage

Par défaut, le conteneur PHP intégré exécute le serveur Apache. Au démarrage, il s’exécute `apache2ctl -D FOREGROUND"`. Vous pouvez également exécuter une commande différente au démarrage, en exécutant la commande suivante dans [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<custom-command>"
```

::: zone-end

## <a name="access-environment-variables"></a>Accéder aux variables d’environnement

Dans App Service, vous pouvez [définir les paramètres de l’application](configure-common.md#configure-app-settings) en dehors de votre code d’application. Vous pouvez ensuite y accéder à l’aide du modèle standard [getenv()](https://secure.php.net/manual/function.getenv.php). Par exemple, pour accéder à un paramètre d’application nommé `DB_HOST`, utilisez le code suivant :

```php
getenv("DB_HOST")
```

## <a name="change-site-root"></a>Modifier la racine du site

::: zone pivot="platform-windows"  

L’infrastructure web de votre choix peut utiliser un sous-répertoire en tant que racine du site. Par exemple, [Laravel](https://laravel.com/) utilise le sous-répertoire *public/* en tant que racine du site.

Pour personnaliser la racine du site, définissez le chemin d’accès de l’application virtuelle pour l’application à l’aide de la commande [`az resource update`](/cli/azure/resource#az-resource-update). L’exemple suivant définit la racine du site sur le sous-répertoire *public/* de votre référentiel. 

```azurecli-interactive
az resource update --name web --resource-group <group-name> --namespace Microsoft.Web --resource-type config --parent sites/<app-name> --set properties.virtualApplications[0].physicalPath="site\wwwroot\public" --api-version 2015-06-01
```

Par défaut, Azure App Service pointe le chemin d’accès à l’application virtuelle ( _/_ ) vers le répertoire racine des fichiers d’application déployés (_sites\wwwroot_).

::: zone-end

::: zone pivot="platform-linux"

L’infrastructure web de votre choix peut utiliser un sous-répertoire en tant que racine du site. Par exemple, [Laravel](https://laravel.com/) utilise le sous-répertoire `public/` en tant que racine du site.

L’image par défaut PHP pour App Service utilise Apache, et ne vous permet pas de personnaliser la racine du site pour votre application. Pour contourner cette limitation, ajoutez un fichier *.htaccess* à la racine du référentiel avec le contenu suivant :

```
<IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteCond %{REQUEST_URI} ^(.*)
    RewriteRule ^(.*)$ /public/$1 [NC,L,QSA]
</IfModule>
```

Si vous préférez ne pas utiliser la réécriture *.htaccess*, vous pouvez à la place déployer votre application Laravel avec une [image Docker personnalisée](quickstart-custom-container.md).

::: zone-end

## <a name="detect-https-session"></a>Détecter une session HTTPS

Dans App Service, une [terminaison SSL](https://wikipedia.org/wiki/TLS_termination_proxy) se produit au niveau des équilibreurs de charge réseau. Toutes les requêtes HTTPS accèdent donc à votre application en tant que requêtes HTTP non chiffrées. Si votre logique d’application doit vérifier si les requêtes utilisateur sont chiffrées ou non, inspectez l’en-tête `X-Forwarded-Proto`.

```php
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
// Do something when HTTPS is used
}
```

Les frameworks web populaires vous permettent d’accéder aux informations `X-Forwarded-*` dans votre modèle d’application standard. Dans [CodeIgniter](https://codeigniter.com/), [is_https()](https://github.com/bcit-ci/CodeIgniter/blob/master/system/core/Common.php#L338-L365) vérifie la valeur de `X_FORWARDED_PROTO` par défaut.

## <a name="customize-phpini-settings"></a>Personnaliser les paramètres php.ini

S'il vous faut apporter des modifications à votre installation PHP, vous pouvez modifier les [directives php.ini](https://www.php.net/manual/ini.list.php) en suivant ces étapes.

> [!NOTE]
> La meilleure façon de consulter la version de PHP et la configuration actuelle de *php.ini* consiste à appeler [phpinfo()](https://php.net/manual/function.phpinfo.php) dans votre application.
>

### <a name="customize-non-php_ini_system-directives"></a><a name="Customize-non-PHP_INI_SYSTEM directives"></a>Directives Customize-non-PHP_INI_SYSTEM

::: zone pivot="platform-windows"  

Pour personnaliser les directives PHP_INI_USER, PHP_INI_PERDIR et PHP_INI_ALL (consultez les [directives php.ini](https://www.php.net/manual/ini.list.php)), ajoutez un fichier `.user.ini` au répertoire racine de votre application.

Ajoutez des paramètres de configuration au fichier `.user.ini` en utilisant la même syntaxe que pour le fichier `php.ini`. Par exemple, si vous souhaitez activer le paramètre `display_errors` et régler le paramètre `upload_max_filesize` sur 10M, votre fichier `.user.ini` doit contenir le texte suivant :

```
 ; Example Settings
 display_errors=On
 upload_max_filesize=10M

 ; Write errors to d:\home\LogFiles\php_errors.log
 ; log_errors=On
```

Redéployez votre application avec les modifications et redémarrez-la.

En guise d’alternative au fichier `.user.ini`, vous pouvez utiliser [ini_set()](https://www.php.net/manual/function.ini-set.php) dans votre application pour personnaliser ces directives non-PHP_INI_SYSTEM.

::: zone-end

::: zone pivot="platform-linux"

Pour personnaliser les directives PHP_INI_USER, PHP_INI_PERDIR et PHP_INI_ALL (consultez les [directives php.ini](https://www.php.net/manual/ini.list.php)), ajoutez un fichier *.htaccess* au répertoire racine de votre application.

Dans le fichier *.htaccess*, ajoutez les directives à l’aide de la syntaxe `php_value <directive-name> <value>`. Par exemple :

```
php_value upload_max_filesize 1000M
php_value post_max_size 2000M
php_value memory_limit 3000M
php_value max_execution_time 180
php_value max_input_time 180
php_value display_errors On
php_value upload_max_filesize 10M
```

Redéployez votre application avec les modifications et redémarrez-la. Si vous la déployez avec Kudu (à l’aide de [Git](deploy-local-git.md), par exemple), elle est automatiquement redémarrée au terme du déploiement.

En guise d'alternative à *.htaccess*, vous pouvez utiliser [ini_set()](https://www.php.net/manual/function.ini-set.php) dans votre application pour personnaliser ces directives non-PHP_INI_SYSTEM.

::: zone-end

### <a name="customize-php_ini_system-directives"></a><a name="customize-php_ini_system-directives"></a>Personnaliser les directives PHP_INI_SYSTEM

::: zone pivot="platform-windows"  

Pour personnaliser les directives PHP_INI_SYSTEM (consultez les [directives php.ini](https://www.php.net/manual/ini.list.php)), vous ne pouvez pas utiliser l'approche *.htaccess*. App Service propose un mécanisme distinct utilisant le paramètre d’application `PHP_INI_SCAN_DIR`.

Commencez par exécuter la commande suivante dans [Cloud Shell](https://shell.azure.com) pour ajouter un paramètre d'application appelé `PHP_INI_SCAN_DIR` :

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PHP_INI_SCAN_DIR="d:\home\site\ini"
```

Accédez à la console Kudu (`https://<app-name>.scm.azurewebsites.net/DebugConsole`) et accédez à `d:\home\site`.

Créez un répertoire dans `d:\home\site` appelé `ini`, puis créez un fichier *.ini* dans le répertoire `d:\home\site\ini` (par exemple, *settings.ini)* avec les directives que vous souhaitez personnaliser. Utilisez la même syntaxe que pour un fichier *php.ini*. 

Par exemple, pour modifier la valeur de [expose_php](https://php.net/manual/ini.core.php#ini.expose-php), exécutez les commandes suivantes :

```bash
cd /home/site
mkdir ini
echo "expose_php = Off" >> ini/setting.ini
```

Redémarrez l'application pour que les modifications soient prises en compte.

::: zone-end

::: zone pivot="platform-linux"

Pour personnaliser les directives PHP_INI_SYSTEM (consultez les [directives php.ini](https://www.php.net/manual/ini.list.php)), vous ne pouvez pas utiliser l'approche *.htaccess*. App Service propose un mécanisme distinct utilisant le paramètre d’application `PHP_INI_SCAN_DIR`.

Commencez par exécuter la commande suivante dans [Cloud Shell](https://shell.azure.com) pour ajouter un paramètre d'application appelé `PHP_INI_SCAN_DIR` :

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PHP_INI_SCAN_DIR="/usr/local/etc/php/conf.d:/home/site/ini"
```

`/usr/local/etc/php/conf.d` correspond au répertoire par défaut dans lequel se trouve *php.ini*. `/home/site/ini` correspond au répertoire personnalisé dans lequel vous ajouterez un fichier *.ini* personnalisé. Séparez les valeurs par `:`.

Accédez à la session SSH web avec votre conteneur Linux (`https://<app-name>.scm.azurewebsites.net/webssh/host`).

Créez un répertoire dans `/home/site` appelé `ini`, puis créez un fichier *.ini* dans le répertoire `/home/site/ini` (par exemple, *settings.ini)* avec les directives que vous souhaitez personnaliser. Utilisez la même syntaxe que pour un fichier *php.ini*. 

> [!TIP]
> Dans les conteneurs Linux intégrés dans App Service, */home* est utilisé en tant que stockage partagé persistant. 
>

Par exemple, pour modifier la valeur de [expose_php](https://php.net/manual/ini.core.php#ini.expose-php), exécutez les commandes suivantes :

```bash
cd /home/site
mkdir ini
echo "expose_php = Off" >> ini/setting.ini
```

Redémarrez l'application pour que les modifications soient prises en compte.

::: zone-end

## <a name="enable-php-extensions"></a>Activer les extensions PHP

::: zone pivot="platform-windows"  

Les installations PHP intégrées contiennent les extensions les plus fréquemment utilisées. Vous pouvez activer des extensions supplémentaires de la même manière que vous [personnalisez les directives php.ini](#customize-php_ini_system-directives).

> [!NOTE]
> La meilleure façon de consulter la version de PHP et la configuration actuelle de *php.ini* consiste à appeler [phpinfo()](https://php.net/manual/function.phpinfo.php) dans votre application.
>

Pour activer des extensions supplémentaires, effectuez les étapes suivantes :

Ajoutez un répertoire `bin` au répertoire racine de votre application et placez-y les fichiers d’extension `.dll` (par exemple *mongodb.dll*). Assurez-vous que les extensions sont compatibles avec la version de PHP dans Azure, ainsi qu'avec VC9 et NTS (Non-Thread Safe).

Déployez vos modifications.

Suivez les étapes décrites dans [Personnaliser les directives PHP_INI_SYSTEM](#customize-php_ini_system-directives), ajoutez les extensions au fichier *.ini* personnalisé avec l'[extension](https://www.php.net/manual/ini.core.php#ini.extension) ou les directives [zend_extension](https://www.php.net/manual/ini.core.php#ini.zend-extension).

```
extension=d:\home\site\wwwroot\bin\mongodb.dll
zend_extension=d:\home\site\wwwroot\bin\xdebug.dll
```

Redémarrez l'application pour que les modifications soient prises en compte.

::: zone-end

::: zone pivot="platform-linux"

Les installations PHP intégrées contiennent les extensions les plus fréquemment utilisées. Vous pouvez activer des extensions supplémentaires de la même manière que vous [personnalisez les directives php.ini](#customize-php_ini_system-directives).

> [!NOTE]
> La meilleure façon de consulter la version de PHP et la configuration actuelle de *php.ini* consiste à appeler [phpinfo()](https://php.net/manual/function.phpinfo.php) dans votre application.
>

Pour activer des extensions supplémentaires, effectuez les étapes suivantes :

Ajoutez un répertoire `bin` au répertoire racine de votre application et placez-y les fichiers d'extension `.so` (par exemple, *mongodb.so*). Assurez-vous que les extensions sont compatibles avec la version de PHP dans Azure, ainsi qu'avec VC9 et NTS (Non-Thread Safe).

Déployez vos modifications.

Suivez les étapes décrites dans [Personnaliser les directives PHP_INI_SYSTEM](#customize-php_ini_system-directives), ajoutez les extensions au fichier *.ini* personnalisé avec l'[extension](https://www.php.net/manual/ini.core.php#ini.extension) ou les directives [zend_extension](https://www.php.net/manual/ini.core.php#ini.zend-extension).

```ini
extension=/home/site/wwwroot/bin/mongodb.so
zend_extension=/home/site/wwwroot/bin/xdebug.so
```

Redémarrez l'application pour que les modifications soient prises en compte.

::: zone-end

## <a name="access-diagnostic-logs"></a>Accéder aux journaux de diagnostic

::: zone pivot="platform-windows"  

Utilisez l’utilitaire standard [error_log ()](https://php.net/manual/function.error-log.php) pour faire apparaître vos journaux de diagnostic dans Azure App Service.

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

::: zone-end

## <a name="troubleshooting"></a>Dépannage

Si une application PHP se comporte différemment dans App Service ou présente des erreurs, essayez ce qui suit :

- [Accéder au flux de journaux](#access-diagnostic-logs).
- Testez l’application localement en mode de production. App Service exécute votre application en mode de production et dès lors, vous devez vous assurer que votre projet fonctionne comme prévu en mode de production localement. Par exemple :
    - Selon votre fichier *composer.json*, différents packages peuvent être installés pour le mode de production (`require` et `require-dev`).
    - Certaines infrastructures web peuvent déployer des fichiers statiques différemment en mode de production.
    - Certaines infrastructures web peuvent utiliser des scripts de démarrage personnalisés lorsqu'elles s'exécutent en mode de production.
- Dans App Service, exécutez votre application en mode de débogage. Par exemple, dans [Laravel](https://meanjs.org/), vous pouvez configurer votre application de manière à envoyer les messages de débogage en production en [définissant le paramètre d'application `APP_DEBUG` sur `true`](configure-common.md#configure-app-settings).

::: zone pivot="platform-linux"

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

::: zone-end

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Application PHP avec MySQL](tutorial-php-mysql-app.md)

::: zone pivot="platform-linux"

> [!div class="nextstepaction"]
> [Questions fréquentes (FAQ) sur App Service sur Linux](faq-app-service-linux.md)

::: zone-end

