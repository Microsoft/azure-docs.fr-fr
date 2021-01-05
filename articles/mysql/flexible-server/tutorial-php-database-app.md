---
title: 'Tutoriel : Créer une application PHP (Laravel) avec un serveur flexible Azure Database pour MySQL'
description: Ce tutoriel explique comment créer une application PHP avec un serveur flexible.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: tutorial
ms.devlang: php
ms.date: 9/21/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 93e605cb20d593750100ec8e340a7ad74c4dd385
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587891"
---
# <a name="tutorial-build-a-php-laravel-and-mysql-flexible-server-preview-app-in-azure-app-service"></a>Tutoriel : Créer une application PHP (Laravel) et Serveur flexible MySQL (préversion) dans Azure App Service


:::image type="content" source="media/tutorial-php-database-app/complete-checkbox-published.png" alt-text="Application web PHP dans Azure avec serveur flexible":::

[Azure App Service](../../app-service/overview.md) fournit un service d’hébergement web hautement évolutif et appliquant des mises à jour correctives automatiques en utilisant le système d’exploitation Linux. Ce tutoriel montre comment créer une application PHP dans Azure et comment la connecter à une base de données MySQL. Quand vous aurez terminé, vous disposerez d’une application [Laravel](https://laravel.com/) s’exécutant sur Azure App Service sur Linux.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Configurer une application PHP (Laravel) avec MySQL local
> * Créer un serveur flexible MySQL (préversion)
> * Connecter une application PHP à un serveur flexible MySQL (préversion)
> * Déployer l’application sur Azure App Service
> * Mise à jour du modèle de données et redéploiement de l’application
> * Gérer l’application dans le portail Azure

Si vous n’avez pas [d’abonnement Azure](../../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

1. [Installez Git](https://git-scm.com/)
2. [Installez PHP 5.6.4 ou version ultérieure](https://php.net/downloads.php)
3. [Installez Composer](https://getcomposer.org/doc/00-intro.md)
4. Activez les extensions PHP suivantes dont Laravel a besoin : OpenSSL, PDO-MySQL, Mbstring, Tokenizer, XML
5. [Installez et démarrez MySQL](https://dev.mysql.com/doc/refman/5.7/en/installing.html)

## <a name="prepare-local-mysql"></a>Préparation du MySQL local

Dans cette étape, vous allez créer une base de données dans votre serveur MySQL local qui vous sera utile dans ce didacticiel.

### <a name="connect-to-local-mysql-server"></a>Se connecter au serveur MySQL local

Dans une fenêtre de terminal, connectez-vous à votre serveur MySQL local. Vous pouvez utiliser cette fenêtre de terminal pour exécuter toutes les commandes de ce didacticiel.

```bash
mysql -u root -p
```

Si vous êtes invité à entrer un mot de passe, tapez le mot de passe du compte `root`. Si vous avez oublié votre mot de passe de compte racine, consultez [MySQL: How to Reset the Root Password](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html) (MySQL : Réinitialisation du mot de passe racine).

Si la commande est exécutée correctement, votre serveur MySQL est en cours d’exécution. Dans le cas contraire, assurez-vous que votre serveur MySQL local est démarré en suivant ces [étapes consécutives à l’installation de MySQL](https://dev.mysql.com/doc/refman/5.7/en/postinstallation.html).

### <a name="create-a-database-locally"></a>Créer une base de données locale

À l’invite `mysql`, créez une base de données.

```sql
CREATE DATABASE sampledb;
```

Quittez votre connexion au serveur en tapant `quit`.

```sql
quit
```

<a name="step2"></a>

## <a name="create-a-php-app-locally"></a>Créer une application PHP locale
Dans cette étape, vous allez créer un exemple d’application Laravel, configurer sa connexion à la base de données et l’exécuter localement.

### <a name="clone-the-sample"></a>Clonage de l’exemple

Dans la fenêtre de terminal, accédez à un répertoire vide dans lequel vous pouvez cloner l’exemple d’application.  Exécutez la commande suivante pour cloner l’exemple de référentiel :

```bash
git clone https://github.com/Azure-Samples/laravel-tasks
```

`cd` vers votre répertoire cloné.
Installez les packages requis.

```bash
cd laravel-tasks
composer install
```

### <a name="configure-mysql-connection"></a>Configuration de la connexion MySQL

À la racine du référentiel, créez un fichier nommé *.env*. Copiez les variables suivantes dans le fichier *.env*. Remplacez l’espace réservé _&lt;root_password>_ par le mot de passe de l’utilisateur racine de MySQL.

```txt
APP_ENV=local
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=sampledb
DB_USERNAME=root
DB_PASSWORD=<root_password>
```

Pour en savoir plus sur la manière dont Laravel utilise ce fichier _.env_, consultez [Configuration de l’environnement Laravel](https://laravel.com/docs/5.4/configuration#environment-configuration).

### <a name="run-the-sample-locally"></a>Télécharger l’exemple localement

Exécutez les [migrations de base de données Laravel](https://laravel.com/docs/5.4/migrations) pour créer les tables requises par l’application. Pour voir quelles tables sont créées dans les migrations, consultez le répertoire _database/migrations_ dans le référentiel Git.

```bash
php artisan migrate
```

Générez une nouvelle clé d’application Laravel.

```bash
php artisan key:generate
```

Exécutez l'application.

```bash
php artisan serve
```

Dans un navigateur, accédez à `http://localhost:8000`. Ajoutez quelques tâches dans la page.

:::image type="content" source="media/tutorial-php-database-app/mysql-connect-success.png" alt-text="PHP se connecte correctement à MySQL":::

Pour arrêter PHP, tapez `Ctrl + C` dans le terminal.

## <a name="create-a-mysql-flexible-server-preview"></a>Créer un serveur flexible MySQL (préversion)
Au cours de cette étape, vous allez créer une base de données MySQL dans [Azure Database pour MySQL - Serveur flexible](../index.yml), qui est en préversion publique. Ensuite, vous configurerez l’application PHP pour la connexion à cette base de données. Dans [Azure Cloud Shell](../../cloud-shell/overview.md), créez un serveur avec la commande [`az flexible-server create`](/cli/azure/mysql/server#az-mysql-flexible-server-create).

```azurecli-interactive
az mysql flexible-server create  --resource-group myResourceGroup --public-access <IP-Address>
```

> [!IMPORTANT]
> - Notez le **nom du serveur** et la **chaîne de connexion** pour les utiliser à l’étape suivante afin de connecter et d’exécuter la migration des données laravel.
> - Pour l’argument **IP-Address**, indiquez l’adresse IP de votre ordinateur client. Le serveur est verrouillé lors de sa création et vous devez autoriser l’accès à votre ordinateur client pour gérer le serveur localement.

### <a name="configure-server-firewall-to-allow-web-app-to-connect-to-the-server"></a>Configurer le pare-feu du serveur pour autoriser l’application web à se connecter au serveur

Dans Cloud Shell, créez une règle de pare-feu pour votre serveur MySQL afin d’autoriser les connexions client à l’aide de la commande az mysql server firewall-rule create. Lorsque les adresses IP de début et de fin sont définies sur ```0.0.0.0```, le pare-feu est uniquement ouvert pour les autres services Azure ne disposant pas d’une adresse IP statique pour se connecter au serveur.

```azurecli
az mysql flexible-server firewall-rule create --name allanyAzureIPs --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

### <a name="connect-to-production-mysql-server-locally"></a>Se connecter au serveur de production MySQL localement

Dans la fenêtre du terminal local, connectez-vous au serveur MySQL dans Azure. Utilisez la valeur que vous avez spécifiée précédemment pour ```<admin-user>``` et ```<mysql-server-name>```. Lorsqu’une invite de mot de passe apparaît, utilisez le mot de passe que vous avez spécifié lors de la création de la base de données dans Azure.

```bash
mysql -u <admin-user> -h <mysql-server-name>.mysql.database.azure.com -P 3306 -p
```

### <a name="create-a-production-database"></a>Création d’une base de données de production

À l’invite `mysql`, créez une base de données.

```sql
CREATE DATABASE sampledb;
```

### <a name="create-a-user-with-permissions"></a>Création d’un utilisateur avec des autorisations

Créez un utilisateur de base de données nommé _phpappuser_ et accordez-lui tous les privilèges dans la base de données `sampledb`. Par souci de simplicité, dans le cadre de ce tutoriel, utilisez _MySQLAzure2020_ comme mot de passe.

```sql
CREATE USER 'phpappuser' IDENTIFIED BY 'MySQLAzure2020';
GRANT ALL PRIVILEGES ON sampledb.* TO 'phpappuser';
```

Quittez la connexion au serveur en tapant `quit`.

```sql
quit
```

## <a name="connect-app-to-mysql-flexible-server"></a>Connecter l’application au serveur flexible MySQL

Dans cette étape, vous allez connecter l’application PHP à la base de données MySQL que vous avez créée dans Azure Database pour MySQL.

<a name="devconfig"></a>

### <a name="configure-the-database-connection"></a>Configurer la connexion à la base de données

À la racine du référentiel, créez un fichier _.env.production_ et copiez-y les variables suivantes. Remplacez l’espace réservé _&lt;mysql-server-name>_ dans *DB_HOST* et *DB_USERNAME*.

```
APP_ENV=production
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=<mysql-server-name>.mysql.database.azure.com
DB_DATABASE=sampledb
DB_USERNAME=phpappuser
DB_PASSWORD=MySQLAzure2017
MYSQL_SSL=true
```

Enregistrez les modifications.

> [!TIP]
> Pour sécuriser vos informations de connexion MySQL, ce fichier est déjà exclu du référentiel Git (consultez _.gitignore_ dans la racine du référentiel). Vous apprendrez ultérieurement à configurer les variables d’environnement dans App Service pour vous connecter à votre base de données dans Azure Database pour MySQL. Avec les variables d’environnement, vous n’avez pas besoin du fichier *.env* dans App Service.
>

### <a name="configure-tlsssl-certificate"></a>Configurer le certificat TLS/SSL

Par défaut, le serveur flexible MySQL exige des connexions TLS de la part des clients. Pour vous connecter à votre base de données MySQL dans Azure, vous devez utiliser le certificat [ _.pem_ fourni par Azure Database pour MySQL - Serveur flexible](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem). Téléchargez [ce certificat](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) et placez-le dans le dossier **SSL** dans la copie locale du référentiel de l’exemple d’application.

Ouvrez _config/database.php_ et ajoutez les paramètres `sslmode` et `options` à `connections.mysql`, comme illustré dans le code suivant.

```php
'mysql' => [
    ...
    'sslmode' => env('DB_SSLMODE', 'prefer'),
    'options' => (env('MYSQL_SSL') && extension_loaded('pdo_mysql')) ? [
        PDO::MYSQL_ATTR_SSL_KEY    => '/ssl/DigiCertGlobalRootCA.crt.pem',
    ] : []
],
```

### <a name="test-the-application-locally"></a>Tester localement l’application

Exécutez les migrations de base de données Laravel avec _.env.production_ comme fichier d’environnement pour créer les tables dans votre base de données MySQL dans Azure Database pour MySQL. N’oubliez pas que _. env.production_ contient les informations de connexion à votre base de données MySQL dans Azure.

```bash
php artisan migrate --env=production --force
```

_.env.production_ n’a pas encore de clé d’application valide. Générez-en une pour lui dans le terminal.

```bash
php artisan key:generate --env=production --force
```

Exécutez l’exemple d’application avec _.env.production_ comme fichier d’environnement.

```bash
php artisan serve --env=production
```

Accédez à `http://localhost:8000`. Si la page se charge sans erreur, l’application PHP se connecte à la base de données MySQL dans Azure.

Ajoutez quelques tâches dans la page.

:::image type="content" source="media/tutorial-php-database-app/mysql-connect-success.png" alt-text="PHP se connecte correctement à Azure Database pour MySQL":::

Pour arrêter PHP, tapez `Ctrl + C` dans le terminal.

### <a name="commit-your-changes"></a>Validation de vos modifications

Exécutez les commandes Git suivantes pour valider vos modifications :

```bash
git add .
git commit -m "database.php updates"
```

Votre application est prête à être déployée.

## <a name="deploy-to-azure"></a>Déployer dans Azure

Dans cette étape, vous allez déployer l’application PHP connectée à MySQL dans Azure App Service.

### <a name="configure-a-deployment-user"></a>Configuration d’un utilisateur de déploiement

Vous pouvez déployer le protocole FTP et Git local sur une application web Azure en faisant appel à un utilisateur de déploiement. Une fois que vous avez créé votre utilisateur de déploiement, vous pouvez l’utiliser pour tous vos déploiements Azure. Votre nom d’utilisateur et votre mot de passe de déploiement au niveau du compte sont différents de vos informations d’identification de l’abonnement Azure.

Pour configurer l’utilisateur de déploiement, exécutez la commande [az webapp deployment user set](/cli/azure/webapp/deployment/user#az-webapp-deployment-user-set) dans Azure Cloud Shell. Remplacez _&lt;username>_ and _&lt;password>_ par le nom d’utilisateur et le mot de passe de votre déploiement.

Le nom d’utilisateur doit être unique dans Azure et, pour les push Git locaux, ne doit pas contenir le symbole « @ ».
Le mot de passe doit comporter au moins huit caractères et inclure deux des trois éléments suivants : lettres, chiffres et symboles.

```bash
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku F1 --is-linux
```

La sortie JSON affiche le mot de passe comme étant null. Si vous obtenez une erreur « Conflit ». Détails : Erreur 409, modifiez le nom d’utilisateur. Si vous obtenez une erreur « Requête incorrecte ». Détails : Erreur 400, utilisez un mot de passe plus fort. **Enregistrez le nom d’utilisateur et le mot de passe à utiliser pour déployer vos applications web.**

### <a name="create-an-app-service-plan"></a>Créer un plan App Service

Dans Cloud Shell, créez un plan App Service dans le groupe de ressources avec la commande [az appservice plan create](/cli/azure/appservice/plan#az-appservice-plan-create). L’exemple suivant crée un plan App Service nommé myAppServicePlan au niveau tarifaire Gratuit (--sku F1) et dans un conteneur Linux (--is-linux).

az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku F1 --is-linux

<a name="create"></a>

### <a name="create-a-web-app"></a>Créer une application web

Créez une [application web](../../app-service/overview.md#app-service-on-linux) dans le plan App Service myAppServicePlan.

Dans Cloud Shell, vous pouvez utiliser la commande [az webapp create](/cli/azure/webapp#az-webapp-create). Dans l’exemple suivant, remplacez _&lt;app-name>_ par un nom d’application unique (les caractères autorisés sont `a-z`, `0-9` et `-`). Le runtime est défini sur `PHP|7.0`. Pour afficher tous les runtimes pris en charge, exécutez [az webapp list-runtimes --linux](/cli/azure/webapp#az-webapp-list-runtimes).

```bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "PHP|7.3" --deployment-local-git
```

Une fois l’application web créée, Azure CLI affiche une sortie similaire à l’exemple suivant :

```
Local git is configured with url of 'https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app-name>.azurewebsites.net",
  "deploymentLocalGitUrl": "https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

Vous avez créé une application web vide, avec le déploiement Git activé.

> [!NOTE]
> L’URL du Git distant est indiquée dans la propriété deploymentLocalGitUrl, au format https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git. Enregistrez cette URL, car vous en aurez besoin ultérieurement.

### <a name="configure-database-settings"></a>Configuration des paramètres de la base de données

Dans App Service, vous définissez les variables d’environnement en tant que _paramètres d’application_ à l’aide de la commande [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set).

La commande suivante configure les paramètres d’application `DB_HOST`, `DB_DATABASE`, `DB_USERNAME` et `DB_PASSWORD`. Remplacez les espaces réservés _&lt;app-name>_ et _&lt;mysql-server-name>_ .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings DB_HOST="<mysql-server-name>.mysql.database.azure.com" DB_DATABASE="sampledb" DB_USERNAME="phpappuser" DB_PASSWORD="MySQLAzure2017" MYSQL_SSL="true"
```

Vous pouvez utiliser la méthode PHP [getenv](https://www.php.net/manual/en/function.getenv.php) pour accéder aux paramètres. Votre code Laravel utilise un wrapper [env](https://laravel.com/docs/5.4/helpers#method-env) sur le PHP `getenv`. Par exemple, la configuration MySQL dans _config/database.php_ ressemble au code suivant :

```php
'mysql' => [
    'driver'    => 'mysql',
    'host'      => env('DB_HOST', 'localhost'),
    'database'  => env('DB_DATABASE', 'forge'),
    'username'  => env('DB_USERNAME', 'forge'),
    'password'  => env('DB_PASSWORD', ''),
    ...
],
```

### <a name="configure-laravel-environment-variables"></a>Configuration des variables d’environnement Laravel

Laravel a besoin d’une clé d’application dans App Service. Vous pouvez la configurer avec les paramètres d’application.

Dans la fenêtre de terminal local, utilisez `php artisan` pour générer une nouvelle clé d’application sans l’enregistrer dans _.env_.

```bash
php artisan key:generate --show
```

Dans Cloud Shell, définissez la clé d’application dans l’application App Service en utilisant la commande [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set). Remplacez les espaces réservés _&lt;app-name>_ et _&lt;outputofphpartisankey:generate>_ .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings APP_KEY="<output_of_php_artisan_key:generate>" APP_DEBUG="true"
```

`APP_DEBUG="true"` indique à Laravel de renvoyer les informations de débogage quand l’application déployée rencontre des erreurs. Lorsque vous exécutez une application de production, affectez-lui la valeur `false`, qui est plus sécurisée.

### <a name="set-the-virtual-application-path"></a>Définition du chemin d’accès de l’application virtuelle

Le [cycle de vie de l’application Laravel](https://laravel.com/docs/5.4/lifecycle) commence dans le répertoire _public_, et non pas dans le répertoire racine de l’application. L’image par défaut Docker PHP pour App Service utilise Apache, et ne vous permet pas de personnaliser `DocumentRoot` pour Laravel. Toutefois, vous pouvez utiliser `.htaccess` pour réécrire toutes les requêtes pour pointer vers _/public_ au lieu du répertoire racine. Dans le répertoire racine, l’élément `.htaccess` a déjà été ajouté à cet effet. Ainsi, votre application Laravel est prête à être déployée.

Pour plus d’informations, consultez [Modifier la racine du site](../../app-service/configure-language-php.md?pivots=platform-linux#change-site-root).

### <a name="push-to-azure-from-git"></a>Effectuer une transmission de type push vers Azure à partir de Git

De retour dans la fenêtre de terminal locale, ajoutez un dépôt distant Azure dans votre dépôt Git local. Remplacez _&lt;deploymentLocalGitUrl-from-create-step>_ par l’URL du Git distant que vous avez enregistrée à la section [Créer une app web](#create-a-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Effectuez une transmission de type push vers le référentiel distant Azure pour déployer votre application à l’aide de la commande suivante. Quand Git Credential Manager vous invite à entrer vos informations d’identification, veillez à entrer celles que vous avez créées dans la section **Configurer un utilisateur de déploiement**, et non pas celles vous permettant de vous connecter au portail Azure.

```bash
git push azure main
```

L’exécution de cette commande peut prendre quelques minutes. Pendant son exécution, des informations semblables à ce qui suit s’affichent :

<pre>
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 291 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'a5e076db9c'.
remote: Running custom deployment command...
remote: Running deployment command...
...
&lt; Output has been truncated for readability &gt;
</pre>

### <a name="browse-to-the-azure-app"></a>Accéder à l’application Azure

Accédez à `http://<app-name>.azurewebsites.net` et ajoutez quelques tâches à la liste.

:::image type="content" source="media/tutorial-php-database-app/php-mysql-in-azure.png" alt-text="Application web PHP dans Azure":::

Félicitations, vous exécutez une application PHP orientée données dans Azure App Service.

## <a name="update-model-locally-and-redeploy"></a>Mettre à jour et redéployer un modèle localement

Dans cette étape, vous allez apporter une modification simple au modèle de données `task` et à l’application web, puis publier la mise à jour sur Azure.

Pour le scénario des tâches, vous modifiez l’application afin de pouvoir marquer une tâche comme terminée.

### <a name="add-a-column"></a>Ajout d’une colonne

Dans la fenêtre de terminal local, accédez à la racine du référentiel Git.

Générez une nouvelle migration de base de données pour la table `tasks` :

```bash
php artisan make:migration add_complete_column --table=tasks
```

Cette commande affiche le nom du fichier de migration qui est généré. Recherchez ce fichier dans _database/migrations_ et ouvrez-le.

Remplacez la méthode `up` par le code suivant :

```php
public function up()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->boolean('complete')->default(False);
    });
}
```

Le code ci-dessus ajoute une colonne booléenne dans la table `tasks` nommée `complete`.

Remplacez la méthode `down` par le code suivant pour l’action de restauration :

```php
public function down()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropColumn('complete');
    });
}
```

Dans la fenêtre de terminal local, exécutez les migrations de base de données Laravel pour apporter la modification dans la base de données locale.

```bash
php artisan migrate
```

Selon la [convention d’affectation de noms Laravel](https://laravel.com/docs/5.4/eloquent#defining-models), le modèle `Task` (voir _app/Task.php_) est mappé à la table `tasks` par défaut.

### <a name="update-application-logic"></a>Mise à jour de la logique d’application

Ouvrez le fichier *routes/web.php*. L’application définit ici ses itinéraires et sa logique métier.

À la fin du fichier, ajoutez un itinéraire avec le code suivant :

```php
/**
 * Toggle Task completeness
 */
Route::post('/task/{id}', function ($id) {
    error_log('INFO: post /task/'.$id);
    $task = Task::findOrFail($id);

    $task->complete = !$task->complete;
    $task->save();

    return redirect('/');
});
```

Le code ci-dessus effectue une simple mise à jour du modèle de données en basculant la valeur de `complete`.

### <a name="update-the-view"></a>Mise à jour de la vue

Ouvrez le fichier *resources/views/tasks.blade.php*. Recherchez la balise d’ouverture `<tr>` et remplacez-la par :

```html
<tr class="{{ $task->complete ? 'success' : 'active' }}" >
```

Le code ci-dessus modifie la couleur de la ligne selon que la tâche est terminée ou non.

La ligne suivante contient le code suivant :

```html
<td class="table-text"><div>{{ $task->name }}</div></td>
```

Remplacez la ligne entière par le code suivant :

```html
<td>
    <form action="{{ url('task/'.$task->id) }}" method="POST">
        {{ csrf_field() }}

        <button type="submit" class="btn btn-xs">
            <i class="fa {{$task->complete ? 'fa-check-square-o' : 'fa-square-o'}}"></i>
        </button>
        {{ $task->name }}
    </form>
</td>
```

Le code ci-dessus ajoute le bouton Envoyer qui fait référence à l’itinéraire défini précédemment.

### <a name="test-the-changes-locally"></a>Test des modifications en local

Dans la fenêtre de terminal local, exécutez le serveur de développement à partir du répertoire racine du référentiel Git.

```bash
php artisan serve
```

Pour voir l’évolution de l’état de la tâche, accédez à `http://localhost:8000` et activez la case à cocher.

:::image type="content" source="media/tutorial-php-database-app/complete-checkbox.png" alt-text="Case à cocher ajoutée à la tâche":::

Pour arrêter PHP, tapez `Ctrl + C` dans le terminal.

### <a name="publish-changes-to-azure"></a>Publier les modifications dans Azure

Dans la fenêtre de terminal local, exécutez les migrations de base de données Laravel avec la chaîne de connexion de production pour apporter la modification dans la base de données Azure.

```bash
php artisan migrate --env=production --force
```

Validez toutes les modifications dans Git, puis envoyez les modifications de code à Azure.

```bash
git add .
git commit -m "added complete checkbox"
git push azure main
```

Une fois le `git push` terminé, accédez à l’application Azure et testez la nouvelle fonctionnalité.

:::image type="content" source="media/tutorial-php-database-app/complete-checkbox-published.png" alt-text="Modifications du modèle et de la base de données publiées dans Azure":::

Si vous avez ajouté des tâches, celles-ci sont conservées dans la base de données. Les mises à jour appliquées au schéma de données n’affectent pas les données existantes.

## <a name="clean-up-resources"></a>Nettoyer les ressources
Au cours des étapes précédentes, vous avez créé des ressources Azure au sein d’un groupe de ressources. Si vous ne pensez pas avoir besoin de ces ressources à l’avenir, supprimez le groupe de ressources en exécutant la commande suivante dans Cloud Shell :

```bash
az group delete --name myResourceGroup
```

<a name="next"></a>

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gérer vos ressources dans le portail Azure](../../azure-resource-manager/management/manage-resources-portal.md) <br/>
> [!div class="nextstepaction"]
> [Gérer votre serveur](how-to-manage-server-cli.md)
