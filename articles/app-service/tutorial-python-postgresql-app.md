---
title: 'Tutoriel : Déployer une application Python Django avec Postgres'
description: Créez une application web Python avec une base de données PostgreSQL, puis déployez-la sur Azure. Le tutoriel utilise l’infrastructure Django, et l’application est hébergée dans Azure App Service sur Linux.
ms.devlang: python
ms.topic: tutorial
ms.date: 11/02/2020
ms.custom:
- mvc
- seodec18
- seo-python-october2019
- cli-validate
- devx-track-python
- devx-track-azurecli
ms.openlocfilehash: b106b403022f3407a3838b7f65222baf41cbfff5
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96852963"
---
# <a name="tutorial-deploy-a-django-web-app-with-postgresql-in-azure-app-service"></a>Tutoriel : Déployer une application web Django avec PostgreSQL dans Azure App Service

Ce tutoriel montre comment déployer une application web Python [Django](https://www.djangoproject.com/) pilotée par les données dans [Azure App Service](overview.md) et la connecter à une base de données Azure Database pour Postgres. App Service offre un service d’hébergement hautement évolutif appliquant des mises à jour correctives automatiques.

Ce tutoriel explique comment utiliser Azure CLI pour effectuer les tâches suivantes :

> [!div class="checklist"]
> * Configurer votre environnement initial avec Python et Azure CLI
> * Créer une base de données Azure Database pour PostgreSQL
> * Déployer du code pour Azure App Service et se connecter à PostgreSQL
> * Mettre à jour votre code et effectuer un redéploiement
> * Afficher les journaux de diagnostic
> * Gérer l’application web dans le portail Azure.

Vous pouvez également utiliser la [version pour le portail Azure de ce tutoriel](/azure/developer/python/tutorial-python-postgresql-app-portal).


## <a name="1-set-up-your-initial-environment"></a>1. Configurer votre environnement initial

1. Vous devez disposer d’un compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
1. Installez <a href="https://www.python.org/downloads/" target="_blank">Python 3.6 ou une version ultérieure</a>.
1. Installez l’interface <a href="/cli/azure/install-azure-cli" target="_blank">Azure CLI</a>2.0.80 ou ultérieur, avec laquelle vous exécutez des commandes dans n’importe quel interpréteur de commandes pour provisionner et configurer des ressources Azure.

Ouvrez une fenêtre de terminal et vérifiez que la version de Python est bien la version 3.6 ou une version ultérieure :

# <a name="bash"></a>[Bash](#tab/bash)

```bash
python3 --version
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```cmd
py -3 --version
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
py -3 --version
```

---

Vérifiez également que votre version d’Azure CLI est la version 2.0.80 ou une version ultérieure :

```azurecli
az --version
```

Connectez-vous ensuite à Azure par le biais de l’interface CLI :

```azurecli
az login
```

Cette commande ouvre un navigateur pour recueillir vos informations d’identification. Une fois l’exécution de la commande terminée, elle affiche une sortie JSON contenant des informations sur vos abonnements.

Une fois connecté, vous pouvez exécuter des commandes Azure avec l’interface de ligne de commande Azure CLI pour utiliser des ressources de votre abonnement.

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

## <a name="2-clone-or-download-the-sample-app"></a>2. Cloner ou télécharger l’exemple d’application

# <a name="git-clone"></a>[Clonage Git](#tab/clone)

Clonez l’exemple de dépôt :

```terminal
git clone https://github.com/Azure-Samples/djangoapp
```

Accédez ensuite à ce dossier :

```terminal
cd djangoapp
```

# <a name="download"></a>[Télécharger](#tab/download)

Accédez à [https://github.com/Azure-Samples/djangoapp](https://github.com/Azure-Samples/djangoapp), sélectionnez **Cloner**, puis sélectionnez **Télécharger le fichier ZIP**. 

Décompressez le fichier ZIP dans un dossier nommé *djangoapp*. 

Ouvrez ensuite une fenêtre de terminal dans ce dossier *djangoapp*.

---

L’exemple de dossier djangoapp contient l’application de sondage Django pilotée par les données que vous obtenez en suivant les étapes décrivant l’[écriture de votre première application Django](https://docs.djangoproject.com/en/3.1/intro/tutorial01/) dans la documentation de Django. L’application terminée est fournie ici à des fins de commodité.

L’exemple est également modifié pour s’exécuter dans un environnement de production tel qu’App Service :

- Les paramètres de production se trouvent dans le fichier *azuresite/production.py*. Les paramètres de développement se trouvent dans *azuresite/settings.py*.
- L’application utilise les paramètres de production quand la variable d’environnement `WEBSITE_HOSTNAME` est définie. Azure App Service définit automatiquement cette variable avec l’URL de l’application web, par exemple `msdocs-django.azurewebsites.net`.

Les paramètres de production sont propres à la configuration de Django pour s’exécuter dans n’importe quel environnement de production. Ils ne sont pas propres à App Service. Pour plus d’informations, consultez la [liste de contrôle du déploiement de Django](https://docs.djangoproject.com/en/3.1/howto/deployment/checklist/). Pour obtenir plus d’informations sur certaines modifications, consultez également [Paramètres de production pour Django sur Azure](configure-language-python.md#production-settings-for-django-apps).

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

## <a name="3-create-postgres-database-in-azure"></a>3. Créer une base de données Postgres dans Azure

<!-- > [!NOTE]
> Before you create an Azure Database for PostgreSQL server, check which [compute generation](../postgresql/concepts-pricing-tiers.md#compute-generations-and-vcores) is available in your region. -->

Installez l’extension de `db-up` pour Azure CLI :

```azurecli
az extension add --name db-up
```

Si la commande `az` n’est pas reconnue, vérifiez qu’Azure CLI est installé, comme décrit dans [Configurer votre environnement initial](#1-set-up-your-initial-environment).

Ensuite, créez la base de données Postgres dans Azure avec la commande [`az postgres up`](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up) :

```azurecli
az postgres up --resource-group DjangoPostgres-tutorial-rg --location westus2 --sku-name B_Gen5_1 --server-name <postgres-server-name> --database-name pollsdb --admin-user <admin-username> --admin-password <admin-password> --ssl-enforcement Enabled
```

- **Remplacez** *\<postgres-server-name>* par un nom **unique dans Azure** (le point de terminaison de serveur devient `https://<postgres-server-name>.postgres.database.azure.com`). Un bon modèle consiste à utiliser une combinaison du nom de votre société et d’une autre valeur unique.
- Pour *\<admin-username>* et *\<admin-password>* , spécifiez les informations d’identification afin de créer un utilisateur administrateur pour ce serveur Postgres. Le nom d’utilisateur de l’administrateur ne doit pas être *azure_superuser*, *azure_pg_admin*, *admin*, *administrator*, *root*, *guest* ou *public*. Il ne peut pas commencer par *pg_* . Le mot de passe doit contenir entre **8 et 128 caractères** de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres (0 à 9) et caractères non alphanumériques (par exemple, !, #, %). Le mot de passe ne peut pas contenir le nom d’utilisateur.
- N’utilisez pas le caractère `$` dans le nom d’utilisateur ou le mot de passe. Par la suite, vous créerez des variables d’environnement avec ces valeurs où le caractère `$` a une signification spéciale dans le conteneur Linux utilisé pour exécuter des applications Python.
- Le [niveau tarifaire](../postgresql/concepts-pricing-tiers.md) de B_Gen5_1 (De base, Gen5, 1 cœur) utilisé ici est le moins onéreux. Pour les bases de données de production, omettez l’argument `--sku-name` pour utiliser le niveau GP_Gen5_2 (Usage général, Gen 5, 2 cœurs) à la place.

Cette commande effectue les actions suivantes qui peuvent prendre quelques minutes :

- Créez un [groupe de ressources](../azure-resource-manager/management/overview.md#terminology) nommé `DjangoPostgres-tutorial-rg` s’il n’existe pas.
- Créez un serveur Postgres nommé par l’argument `--server-name`.
- Créez un compte d’administrateur à l’aide des arguments `--admin-user` et `--admin-password`. Vous pouvez omettre ces arguments pour permettre à la commande de générer des informations d’identification uniques pour vous.
- Créez une base de données `pollsdb` nommée par l’argument `--database-name`.
- Activez l’accès à partir de votre adresse IP locale.
- Activez l’accès à partir des services Azure.
- Créez un utilisateur de base de données avec un accès à la base de données `pollsdb`.

Vous pouvez effectuer toutes les étapes séparément avec d’autres commandes `az postgres` et `psql`, mais `az postgres up` effectue toutes les étapes ensemble.

La commande génère un objet JSON contenant des chaînes de connexion différentes pour la base de données, ainsi que l’URL du serveur, un nom d’utilisateur généré (tel que « joyfulKoala@msdocs-djangodb-12345 ») et un mot de passe GUID. **Copiez le nom d’utilisateur et le mot de passe dans un fichier texte temporaire**, car vous en aurez besoin plus loin dans ce tutoriel.

<!-- not all locations support az postgres up -->
> [!TIP]
> `-l <location-name>` peut être défini sur n’importe quelle [région Azure](https://azure.microsoft.com/global-infrastructure/regions/). Vous pouvez récupérer les régions disponibles pour votre abonnement à l’aide de la commande [`az account list-locations`](/cli/azure/account#az-account-list-locations). Pour les applications de production, placez votre base de données et votre application dans le même emplacement.

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

## <a name="4-deploy-the-code-to-azure-app-service"></a>4. Déployer l’image dans Azure App Service

Dans cette section, vous allez créer un hôte d’application dans l’application App Service, connecter cette application à la base de données Postgres, puis déployer votre code sur cet hôte.

### <a name="41-create-the-app-service-app"></a>4.1 Créer l’application App Service

Dans le terminal, vérifiez que vous êtes dans le dossier référentiel *djangoapp* qui contient le code de l’application.

Créez une application App Service (processus hôte) avec la commande [`az webapp up`](/cli/azure/webapp#az-webapp-up) :

```azurecli
az webapp up --resource-group DjangoPostgres-tutorial-rg --location westus2 --plan DjangoPostgres-tutorial-plan --sku B1 --name <app-name>
```
<!-- without --sku creates PremiumV2 plan -->

- Pour l’argument `--location`, utilisez le même emplacement que pour la base de données dans la section précédente.
- **Remplacez** *\<app-name>* par un nom unique dans l’ensemble d’Azure (le point de terminaison de serveur est `https://<app-name>.azurewebsites.net`). Les caractères autorisés pour *\<app-name>* sont `A`-`Z`, `0`-`9` et `-`. Un bon modèle consiste à utiliser une combinaison du nom de votre société et d’un identificateur d’application.

Cette commande effectue les actions suivantes qui peuvent prendre quelques minutes :

<!-- - Create the resource group if it doesn't exist. `--resource-group` is optional. -->
<!-- No it doesn't. az webapp up doesn't respect --resource-group -->
- S’il n’existe pas encore, créez le [groupe de ressources](../azure-resource-manager/management/overview.md#terminology) (dans cette commande, vous utilisez le groupe de ressources dans lequel vous avez créé la base de données précédemment).
- S’il n’existe pas, créez le [plan de App Service](overview-hosting-plans.md) *DjangoPostgres-Tutorial-plan* dans le niveau tarifaire de base (B1). Les valeurs `--plan` et `--sku` sont facultatives.
- Si elle n’existe pas, créez l’application App Service.
- Activez la journalisation par défaut pour l’application, si elle ne l’est pas déjà.
- Chargez le dépôt à l’aide du déploiement ZIP avec l’automatisation de la génération activée.
- Mettez en cache les paramètres communs, tels que le nom du groupe de ressources et le plan App Service, dans le fichier *.azure/config*. Par conséquent, vous n’avez pas besoin de spécifier le même paramètre avec les commandes ultérieures. Par exemple, pour redéployer l’application après avoir apporté des modifications, vous pouvez simplement exécuter `az webapp up` sans paramètre. Toutefois, des commandes provenant d’extensions CLI, telles que `az postgres up`, n’utilisant actuellement pas le cache, vous devez spécifier le groupe de ressources et l’emplacement ici avec l’utilisation initiale de `az webapp up`.

Une fois le déploiement réussi, la commande génère une sortie JSON comme dans l’exemple suivant :

![Exemple de sortie de la commande az webapp up](./media/tutorial-python-postgresql-app/az-webapp-up-output.png)

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

### <a name="42-configure-environment-variables-to-connect-the-database"></a>4.2 Configurer des variables d’environnement pour connecter la base de données

Le code étant maintenant déployé sur App Service, l’étape suivante consiste à connecter l’application à la base de données Postgres dans Azure.

Le code de l’application s’attend à trouver des informations sur la base de données dans quatre variables d’environnement nommées `DBHOST`, `DBNAME`, `DBUSER` et `DBPASS`.

Pour définir des variables d’environnement dans App Service, créez des « paramètres d’application » à l’aide de la commande suivante [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set).

```azurecli
az webapp config appsettings set --settings DBHOST="<postgres-server-name>" DBNAME="pollsdb" DBUSER="<username>" DBPASS="<password>"
```

- Remplacez *\<postgres-server-name>* par le nom que vous avez utilisé précédemment à l’aide de la commande `az postgres up`. Le code de *azuresite/production.py* ajoute automatiquement `.postgres.database.azure.com` pour créer l’URL complète du serveur Postgres.
- Remplacez *\<username>* et *\<password>* par les informations d’identification d’administrateur que vous avez utilisées avec la commande `az postgres up` précédente, ou celles que `az postgres up` a générées pour vous. Le code situé dans *azuresite/production.py* construit automatiquement le nom d’utilisateur Postgres complet à partir de `DBUSER` et `DBHOST`. Vous n’avez donc pas besoin d’inclure la partie `@server`. (De même, comme nous l’avons vu précédemment, évitez d’utiliser le caractère `$` dans l’une ou l’autre des valeurs, car il a une signification particulière pour les variables d’environnement Linux.)
- Les noms du groupe de ressources et de l’application sont tirés des valeurs mises en cache dans le fichier *.azure/config*.

Dans votre code Python, vous accédez à ces paramètres comme à des variables d’environnement avec des instructions telles que `os.environ.get('DBHOST')`. Pour en savoir plus, consultez [Accéder aux variables d’environnement](configure-language-python.md#access-environment-variables).

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

### <a name="43-run-django-database-migrations"></a>4.3 Exécuter des migrations de base de données Django

Les migrations de base de données Django garantissent que le schéma de la base de données PostgreSQL sur Azure correspond à ceux décrits dans votre code.

1. Ouvrez une session SSH **dans le navigateur** en accédant à l’URL suivante et vous connectant avec vos informations d’identification de compte Azure (et non les informations d’identification du serveur de base de données).

    ```
    https://<app-name>.scm.azurewebsites.net/webssh/host
    ```

    Remplacez `<app-name>` par le nom que vous avez utilisé précédemment dans la commande `az webapp up`.

    Sur macOS et Linux, vous pouvez également vous connecter à une session SSH avec la commande [`az webapp ssh`](/cli/azure/webapp?view=azure-cli-latest&preserve-view=true#az_webapp_ssh).

    Si vous ne pouvez pas vous connecter à la session SSH, c’est que l’application elle-même n’a pas pu démarrer. Pour plus d’informations, [consultez les journaux de diagnostic](#6-stream-diagnostic-logs). Par exemple, si vous n’avez pas créé les paramètres d’application nécessaires dans la section précédente, les journaux indiquent `KeyError: 'DBNAME'`.

1. Dans la session SSH, exécutez les commandes suivantes (vous pouvez coller des commandes en utilisant la combinaison de touches **Ctrl**+**Maj**+**V**) :

    ```bash
    # Change to the folder where the app code is deployed
    cd site/wwwroot
    
    # Activate default virtual environment in App Service container
    source /antenv/bin/activate

    # Install packages
    pip install -r requirements.txt

    # Run database migrations
    python manage.py migrate

    # Create the super user (follow prompts)
    python manage.py createsuperuser
    ```

1. La commande `createsuperuser` vous invite à entrer vos informations d’identification de superutilisateur. Pour les besoins de ce tutoriel, utilisez le nom d’utilisateur par défaut `root`, appuyez sur **Entrée** pour l’adresse a-mail afin de laisser le champ vide, puis entrez `Pollsdb1` en tant que mot de passe.

1. Si vous voyez une erreur indiquant que la base de données est verrouillée, assurez-vous que vous avez exécuté la commande `az webapp settings` dans la section précédente. Sans ces paramètres, la commande de migration ne peut pas communiquer avec la base de données, ce qui provoque l’erreur.

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).
    
### <a name="44-create-a-poll-question-in-the-app"></a>4.4 Créer une question de sondage dans l’application

1. Dans un navigateur, ouvrez l’URL `http://<app-name>.azurewebsites.net`. L’application doit afficher le message « No polls are available » (Aucun sondage n’est disponible), car il n’existe pas encore de sondages spécifiques dans la base de données.

    Si le message « Erreur d’application » s’affiche, vous n’avez probablement pas créé les paramètres nécessaires lors de l’étape précédente, [Configurer des variables d’environnement pour connecter la base de données](#42-configure-environment-variables-to-connect-the-database), ou que ces valeurs contiennent des erreurs. Exécutez la commande `az webapp config appsettings list` pour vérifier les paramètres. Vous pouvez également [consulter les journaux de diagnostic](#6-stream-diagnostic-logs) pour examiner les erreurs particulières survenues au démarrage de l’application. Par exemple, si vous n’avez pas créé les paramètres, les journaux affichent l’erreur `KeyError: 'DBNAME'`.

    Après avoir mis à jour les paramètres pour corriger les erreurs, donnez à l’application une minute pour redémarrer, puis actualisez le navigateur.

1. Accédez à `http://<app-name>.azurewebsites.net/admin`. Connectez-vous à l’aide des informations d’identification de superutilisateur de la section précédente (`root` et `Pollsdb1`). Sous **Sondages**, sélectionnez **Ajouter** en regard de **Questions**, puis créez une question de sondage avec quelques options.

1. Accédez de nouveau à `http://<app-name>.azurewebsites.net` pour vérifier que les questions sont maintenant présentées à l’utilisateur. Répondez aux questions en fonction de la manière dont vous voulez générer des données dans la base de données.

**Félicitations !** Vous exécutez une application web Python Django dans Azure App Service pour Linux, avec une base de données Postgres active.

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

> [!NOTE]
> App Service détecte un projet Django en recherchant un fichier *wsgi.py* dans chaque sous-dossier que `manage.py startproject` crée par défaut. Quand App Service trouve ce fichier, il charge l’application web Django. Pour plus d’informations, consultez [Configurer une image Python intégrée](configure-language-python.md).

## <a name="5-make-code-changes-and-redeploy"></a>5. Apporter des changements de code et redéployer

Dans cette section, vous apportez des modifications locales à l’application, et redéployez le code vers App Service. Au cours du processus, vous configurez un environnement virtuel Python qui prend en charge le travail en cours.

### <a name="51-run-the-app-locally"></a>5.1 Exécuter l’application localement

Exécutez les commandes suivantes dans une fenêtre de terminal. Veillez à suivre les invites lors de la création du superutilisateur :

# <a name="bash"></a>[bash](#tab/bash)

```bash
# Configure the Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install packages
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
# Configure the Python virtual environment
py -3 -m venv venv
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
venv\scripts\activate

# Install packages
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```

# <a name="cmd"></a>[CMD](#tab/cmd)

```cmd
:: Configure the Python virtual environment
py -3 -m venv venv
venv\scripts\activate

:: Install packages
pip install -r requirements.txt
:: Run Django migrations
python manage.py migrate
:: Create Django superuser (follow prompts)
python manage.py createsuperuser
:: Run the dev server
python manage.py runserver
```
---

Une fois l’application web entièrement chargée, le serveur de développement Django fournit l’URL de l’application locale dans le message « Starting development server at http://127.0.0.1:8000/. Quit the server with CTRL-BREAK » (Démarrage du serveur de développement à l’adresse http://127.0.0.1:8000/. Quittez le serveur en appuyant sur CTRL-PAUSE)

![Exemple de sortie du serveur de développement Django](./media/tutorial-python-postgresql-app/django-dev-server-output.png)

Testez l’application localement en procédant comme suit :

1. Accédez à l’URL `http://localhost:8000` dans un navigateur, ce qui doit afficher le message « No polls are available » (Aucun sondage n’est disponible). 

1. Accédez à `http:///localhost:8000/admin`, puis connectez-vous à l’aide de l’utilisateur administrateur que vous avez créé précédemment. Sous **Sondages**, sélectionnez de nouveau **Ajouter** en regard de **Questions**, puis créez une question de sondage avec quelques options. 

1. Accédez de nouveau à l’URL *http:\//localhost:8000* et répondez à la question pour tester l’application. 

1. Arrêtez le serveur Django en appuyant sur **Ctrl**+**C**.

Lors d’une exécution locale, l’application utilise une base de données sqlite3 locale et n’interfère pas avec votre base de données de production. Vous pouvez également utiliser une base de données PostgreSQL locale, si vous le souhaitez, pour mieux simuler votre environnement de production.

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

### <a name="52-update-the-app"></a>5.2 Mettre à jour l’application

Dans `polls/models.py`, localisez la ligne qui commence par `choice_text` et définissez le paramètre `max_length` sur 100 :

```python
# Find this lie of code and set max_length to 100 instead of 200
choice_text = models.CharField(max_length=100)
```

Étant donné que vous avez modifié le modèle de données, créez une migration Django et migrez la base de données :

```
python manage.py makemigrations
python manage.py migrate
```

Réexécutez le serveur de développement avec la commande `python manage.py runserver`, puis testez l’application à l’URL *http:\//localhost:8000/admin*:

Arrêtez de nouveau le serveur web Django en appuyant sur **Ctrl**+**C**.

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

### <a name="53-redeploy-the-code-to-azure"></a>5.3 Redéployer le code dans Azure

Exécutez la commande suivante dans la racine du dépôt :

```azurecli
az webapp up
```

Cette commande utilise les paramètres mis en cache dans le fichier *.azure/config*. Étant donné qu’App Service détecte que l’application existe déjà, le service redéploie simplement le code.

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

### <a name="54-rerun-migrations-in-azure"></a>5.4 Réexécuter des migrations dans Azure

Étant donné que vous avez apporté des changements au modèle de données, vous devez réexécuter les migrations de base de données dans App Service.

Ouvrez une nouvelle session SSH dans le navigateur en accédant à `https://<app-name>.scm.azurewebsites.net/webssh/host`. Exécutez ensuite les commande suivantes :

```
cd site/wwwroot

# Activate default virtual environment in App Service container
source /antenv/bin/activate
# Run database migrations
python manage.py migrate
```

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

### <a name="55-review-app-in-production"></a>5.5 Examiner une application en production

Accédez à `http://<app-name>.azurewebsites.net` et testez à nouveau l’application en production. (Étant donné que vous avez uniquement modifié la longueur d’un champ de base de données, la modification n’est visible que si vous essayez d’entrer une réponse plus longue lors de la création d’une question.)

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

## <a name="6-stream-diagnostic-logs"></a>6. Diffuser les journaux de diagnostic

Vous pouvez accéder aux journaux de la console générés à l’intérieur du conteneur hébergeant l’application sur Azure.

Exécutez la commande Azure CLI suivante dans le flux de journal. Cette commande utilise des paramètres mis en cache dans le fichier *.azure/config*.

```azurecli
az webapp log tail
```

Si vous ne voyez pas les journaux d’activité de la console, attendez 30 secondes et vérifiez à nouveau.

Pour arrêter le streaming de journaux à tout moment, appuyez sur **Ctrl**+**C**.

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

> [!NOTE]
> Vous pouvez également inspecter les fichiers journaux à partir du navigateur sur `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.
>
> `az webapp up` active automatiquement la journalisation par défaut. Pour des raisons de performances, cette journalisation se désactive après un certain temps, mais se réactive chaque fois que vous réexécutez `az webapp up`. Pour l’activer manuellement, exécutez la commande suivante :
>
> ```azurecli
> az webapp log config --docker-container-logging filesystem
> ```

## <a name="7-manage-your-app-in-the-azure-portal"></a>7. Gérer votre application dans le portail Azure

Dans le [portail Azure](https://portal.azure.com), recherchez le nom de l’application et sélectionnez celle-ci dans les résultats.

![Accéder à votre application Python Django dans le portail Azure](./media/tutorial-python-postgresql-app/navigate-to-django-app-in-app-services-in-the-azure-portal.png)

Par défaut, le portail affiche la page **Vue d’ensemble** de votre application, qui présente une vue générale des performances. Elle vous permet d’exécuter des tâches de gestion de base telles que parcourir, arrêter, redémarrer et supprimer. Les onglets sur le côté gauche de la page affichent les différentes pages de configuration que vous pouvez ouvrir.

![Gérer votre application Python Django dans la page Vue d’ensemble du portail Azure](./media/tutorial-python-postgresql-app/manage-django-app-in-app-services-in-the-azure-portal.png)

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

## <a name="8-clean-up-resources"></a>8. Nettoyer les ressources

Si vous souhaitez conserver l’application ou passer aux autres tutoriels, allez directement à [Étapes suivantes](#next-steps). Autrement, pour éviter des frais récurrents, vous pouvez supprimer le groupe de ressources créé pour ce tutoriel :

```azurecli
az group delete --no-wait
```

La commande utilise le nom du groupe de ressources mis en cache dans le fichier *.azure/config*. En supprimant le groupe de ressources, vous désallouez et supprimez également toutes les ressources qu’il contient.

La suppression de toutes les ressources peut prendre un certain temps. L’argument `--no-wait` permet à la commande de revenir immédiatement.

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/DjangoCLITutorialHelp).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment mapper un nom DNS personnalisé à votre application :

> [!div class="nextstepaction"]
> [Tutoriel : Mappage d’un nom DNS personnalisé à une application](app-service-web-tutorial-custom-domain.md)

Découvrez comment App Service exécute une application Python :

> [!div class="nextstepaction"]
> [Configurer une application Python](configure-language-python.md)
