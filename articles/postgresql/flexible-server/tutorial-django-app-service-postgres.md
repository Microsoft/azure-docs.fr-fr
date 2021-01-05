---
title: 'Tutoriel : Déployer une application Django dans un réseau virtuel avec App Service et Azure Database pour PostgreSQL - Serveur flexible (préversion)'
description: Déployer une application Django dans un réseau virtuel avec App Service et Azure Database pour PostgreSQL - Serveur flexible (préversion)
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 09/22/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: bcc9ca0175e0e03c62c2ce2b91d8ec337756a3cc
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92490099"
---
# <a name="tutorial-deploy-django-app-with-app-service-and-azure-database-for-postgresql---flexible-server-preview"></a>Tutoriel : Déployer une application Django avec App Service et Azure Database pour PostgreSQL - Serveur flexible (préversion)

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est disponible en préversion.

Dans ce tutoriel, vous allez apprendre à déployer une application Django dans un réseau virtuel Azure avec App Services et Azure Database pour PostgreSQL - Serveur flexible.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Cet article nécessite que vous exécutiez localement Azure CLI version 2.0 ou ultérieure. Pour afficher la version installée, exécutez la commande `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

Vous devrez vous connecter à votre compte à l’aide de la commande [az login](/cli/azure/authenticate-azure-cli). Notez la propriété **id** depuis la sortie de commande pour le nom d’abonnement correspondant.

```azurecli
az login
```

Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Sélectionnez l’ID d’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account). Remplacez la propriété **ID d’abonnement** de la sortie **az login** pour votre abonnement dans l’espace réservé de l’ID d’abonnement.

```azurecli
az account set --subscription <subscription id>
```
## <a name="clone-or-download-the-sample-app"></a>Cloner ou télécharger l’exemple d’application

# <a name="git-clone"></a>[Clonage Git](#tab/clone)

Clonez l’exemple de dépôt :

```terminal
git clone https://github.com/Azure-Samples/djangoapp
```

Ensuite, accédez à ce dossier :

```terminal
cd djangoapp
```

# <a name="download"></a>[Télécharger](#tab/download)

Accédez à [https://github.com/Azure-Samples/djangoapp](https://github.com/Azure-Samples/djangoapp), sélectionnez **Cloner**, puis sélectionnez **Télécharger le fichier ZIP**.

Décompressez le fichier ZIP dans un dossier nommé *djangoapp*.

Ouvrez ensuite une fenêtre de terminal dans ce dossier *djangoapp*.

---

L’exemple de dossier djangoapp contient l’application de sondage Django pilotée par les données que vous obtenez en suivant les étapes décrivant l’[écriture de votre première application Django](https://docs.djangoproject.com/en/2.1/intro/tutorial01/) dans la documentation de Django. L’application terminée est fournie ici à des fins de commodité.

L’exemple est également modifié pour s’exécuter dans un environnement de production tel qu’App Service :

- Les paramètres de production se trouvent dans le fichier *azuresite/production.py*. Les détails du développement se trouvent dans *azuresite/Settings.py*.
- L’application utilise les paramètres de production lorsque la variable d’environnement `DJANGO_ENV` est définie sur « production ». Vous allez créer cette variable d’environnement ultérieurement dans le tutoriel, ainsi que d’autres éléments utilisés pour la configuration de la base de données PostgreSQL.

Ces modifications sont spécifiques de la configuration de Django pour s’exécuter dans n’importe quel environnement de production. Elles ne sont pas propres à App Service. Pour plus d’informations, consultez la [liste de contrôle du déploiement de Django](https://docs.djangoproject.com/en/2.1/howto/deployment/checklist/).

## <a name="create-a-postgresql-flexible-server-in-a-new-virtual-network"></a>Créer un serveur flexible PostgreSQL dans un nouveau réseau virtuel

Créez un serveur flexible privé et une base de données dans un réseau virtuel à l'aide de la commande suivante :
```azurecli
# Create Flexible server in a VNET

az postgres flexible-server create --resource-group myresourcegroup --location westus2

```

Cette commande effectue les actions suivantes qui peuvent prendre quelques minutes :

- S'il n'existe pas encore, créez le groupe de ressources.
- Génère un nom de serveur s'il n'est pas fourni.
- Créez un réseau virtuel pour votre nouveau serveur PostgreSQL. **Notez le nom du réseau virtuel et le nom du sous-réseau** créés pour votre serveur car vous devez ajouter l'application web au même réseau virtuel.
- Crée un nom d'utilisateur administrateur et un mot de passe pour votre serveur, s'il n'est pas fourni. **Notez le nom d'utilisateur et le mot de passe** car vous en aurez besoin à l'étape suivante.
- Créez une base de données ```postgres``` utilisable à des fins de développement. Vous pouvez exécuter [**psql** pour vous connecter à la base de données](quickstart-create-server-portal.md#connect-to-the-postgresql-database-using-psql) afin de créer une autre base de données.

> [!NOTE]
> Notez le mot de passe généré automatiquement s'il n'est pas fourni. Si vous oubliez le mot de passe, vous devrez le réinitialiser à l'aide de la commande ``` az postgres flexible-server update```


## <a name="deploy-the-code-to-azure-app-service"></a>Déployer l’image dans Azure App Service

Dans cette section, vous allez créer un hôte d’application dans l’application App Service, connecter cette application à la base de données Postgres, puis déployer votre code sur cet hôte.


### <a name="create-the-app-service-web-app-in-a-virtual-network"></a>Créer l'application web App Service dans un réseau virtuel

Dans le terminal, assurez-vous que vous êtes dans la racine du dépôt (`djangoapp`) contenant le code de l’application.

Créez une application App Service (processus hôte) avec la commande [`az webapp up`](/cli/azure/webapp#az-webapp-up) :

```azurecli

# Create a web app
az webapp up --resource-group myresourcegroup --location westus2 --plan DjangoPostgres-tutorial-plan --sku B1 --name <app-name>

# Enable VNET integration for web app.
# Replace <vnet-name> and <subnet-name> with the virtual network and subnet name that the flexible server is using.

az webapp vnet-integration add -g myresourcegroup -n  mywebapp --vnet <vnet-name> --subnet <subnet-name>

# Configure database information as environment variables
# Use the postgres server name , database name , username , password for the database created in the previous steps

az webapp config appsettings set --settings DJANGO_ENV="production" DBHOST="<postgres-server-name>.postgres.database.azure.com" DBNAME="postgres" DBUSER="<username>" DBPASS="<password>"
```
- Pour l’argument `--location`, utilisez le même emplacement que pour la base de données dans la section précédente.
- Remplacez *\<app-name>* par un nom unique dans l’ensemble d’Azure (le point de terminaison de serveur est `https://\<app-name>.azurewebsites.net`). Les caractères autorisés pour *\<app-name>* sont `A`-`Z`, `0`-`9` et `-`. Un bon modèle consiste à utiliser une combinaison du nom de votre société et d’un identificateur d’application.
- S’il n’existe pas, créez le [plan de App Service](../../app-service/overview-hosting-plans.md) *DjangoPostgres-Tutorial-plan* dans le niveau tarifaire de base (B1). Les valeurs `--plan` et `--sku` sont facultatives.
- Si elle n’existe pas, créez l’application App Service.
- Activez la journalisation par défaut pour l’application, si elle ne l’est pas déjà.
- Chargez le dépôt à l’aide du déploiement ZIP avec l’automatisation de la génération activée.
- La commande **az webapp vnet-integration** ajoute l'application web dans le même réseau virtuel que le serveur Postgres.
- Le code de l’application s’attend à trouver des informations sur la base de données dans plusieurs variables d’environnement. Pour définir des variables d’environnement dans App Service, vous créez des « paramètres d’application » à l’aide de la commande [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set).

> [!TIP]
> De nombreuses commandes Azure CLI mettent en cache des paramètres communs, tels que le nom du groupe de ressources et le plan App Service, dans le fichier *.azure/config*. Par conséquent, vous n’avez pas besoin de spécifier le même paramètre avec les commandes ultérieures. Par exemple, pour redéployer l’application après avoir apporté des modifications, vous pouvez simplement exécuter `az webapp up` sans paramètre.

### <a name="run-django-database-migrations"></a>Exécuter des migrations de base de données Django

Les migrations de base de données Django garantissent que le schéma de la base de données PostgreSQL sur Azure correspond à ceux décrits dans votre code.

1. Ouvrez une session SSH dans le navigateur en accédant à *https://\<app-name>.scm.azurewebsites.net/webssh/host*, puis connectez-vous avec vos informations d’identification de compte Azure (et non les informations d’identification du serveur de base de données).

1. Dans la session SSH, exécutez les commandes suivantes (vous pouvez coller des commandes en utilisant la combinaison de touches **Ctrl**+**Maj**+**V**) :

    ```bash
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

1. La commande `createsuperuser` vous invite à entrer vos informations d’identification de superutilisateur. Pour les besoins de ce tutoriel, utilisez le nom d’utilisateur par défaut `root`, appuyez sur **Entrée** pour l’adresse a-mail afin de laisser le champ vide, puis entrez `postgres1` en tant que mot de passe.

### <a name="create-a-poll-question-in-the-app"></a>Créer une question de sondage dans l’application

1. Dans un navigateur, ouvrez l’URL *http:\//\<app-name>.azurewebsites.net*. L’application doit afficher le message « No polls are available » (Aucun sondage n’est disponible), car il n’existe pas encore de sondages spécifiques dans la base de données.

1. Accédez à *http:\//\<app-name>.azurewebsites.net/admin*. Connectez-vous à l’aide des informations d’identification de superutilisateur de la section précédente (`root` et `postgres1`). Sous **Sondages**, sélectionnez **Ajouter** en regard de **Questions**, puis créez une question de sondage avec quelques options.

1. Accédez à nouveau à *http:\//\<app-name>.azurewebsites.net/* pour confirmer que les questions sont à présent présentées à l’utilisateur. Répondez aux questions en fonction de la manière dont vous voulez générer des données dans la base de données.

**Félicitations !** Vous exécutez une application web Python Django dans Azure App Service pour Linux, avec une base de données Postgres active.

> [!NOTE]
> App Service détecte un projet Django en recherchant un fichier *wsgi.py* dans chaque sous-dossier que `manage.py startproject` crée par défaut. Quand App Service trouve ce fichier, il charge l’application web Django. Pour plus d’informations, consultez [Configurer une image Python intégrée](../../app-service/configure-language-python.md).

## <a name="make-code-changes-and-redeploy"></a>Apporter des changements de code et redéployer

Dans cette section, vous apportez des modifications locales à l’application, et redéployez le code vers App Service. Au cours du processus, vous configurez un environnement virtuel Python qui prend en charge le travail en cours.

### <a name="run-the-app-locally"></a>Exécutez l’application localement.

Exécutez les commandes suivantes dans une fenêtre de terminal. Veillez à suivre les invites lors de la création du superutilisateur :

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
Une fois l’application web entièrement chargée, le serveur de développement Django fournit l’URL de l’application locale dans le message « Starting development server at http://127.0.0.1:8000/. Quit the server with CTRL-BREAK » (Démarrage du serveur de développement à l’adresse http://127.0.0.1:8000/. Quittez le serveur en appuyant sur CTRL-PAUSE)

:::image type="content" source="./media/tutorial-django-app-service-postgres/django-dev-server-output.png" alt-text="Exemple de sortie du serveur de développement Django":::

Testez l’application localement en procédant comme suit :

1. Accédez à l’URL *http:\//localhost:8000* dans un navigateur qui doit afficher le message « No polls are available » (Aucun sondage n’est disponible).

1. Accédez à l’URL *http:\//localhost:8000/admin* et connectez-vous à l’aide de l’ID d’utilisateur administrateur que vous avez créé précédemment. Sous **Sondages**, sélectionnez de nouveau **Ajouter** en regard de **Questions**, puis créez une question de sondage avec quelques options.

1. Accédez de nouveau à l’URL *http:\//localhost:8000* et répondez à la question pour tester l’application.

1. Arrêtez le serveur Django en appuyant sur **Ctrl**+**C**.

Lors d’une exécution locale, l’application utilise une base de données sqlite3 locale et n’interfère pas avec votre base de données de production. Vous pouvez également utiliser une base de données PostgreSQL locale, si vous le souhaitez, pour mieux simuler votre environnement de production.



### <a name="update-the-app"></a>Mettre à jour l’application

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


### <a name="redeploy-the-code-to-azure"></a>Redéployer le code dans Azure

Exécutez la commande suivante dans la racine du dépôt :

```azurecli
az webapp up
```

Cette commande utilise les paramètres mis en cache dans le fichier *.azure/config*. Étant donné qu’App Service détecte que l’application existe déjà, le service redéploie simplement le code.



### <a name="rerun-migrations-in-azure"></a>Réexécuter des migrations dans Azure

Étant donné que vous avez apporté des changements au modèle de données, vous devez réexécuter les migrations de base de données dans App Service.

Ouvrez de nouveau une session SSH dans le navigateur en accédant à *https://\<app-name>.scm.azurewebsites.net/webssh/host*. Exécutez ensuite les commande suivantes :

```
cd site/wwwroot

# Activate default virtual environment in App Service container
source /antenv/bin/activate
# Run database migrations
python manage.py migrate
```

### <a name="review-app-in-production"></a>Examiner une application en production

Accédez à *http:\//\<app-name>.azurewebsites.net* et testez à nouveau l’application en production (étant donné que vous avez uniquement modifié la longueur d’un champ de base de données, la modification n’est visible que si vous essayez d’entrer une réponse plus longue lors de la création d’une question).

> [!TIP]
> Vous pouvez utiliser [django-storages](https://django-storages.readthedocs.io/en/latest/backends/azure.html) pour stocker des ressources statiques et des éléments multimédias dans le stockage Azure. Vous pouvez utiliser Azure CDN pour le « gzippage » des fichiers statiques.


## <a name="manage-your-app-in-the-azure-portal"></a>Gérer votre application dans le portail Azure

Dans le [portail Azure](https://portal.azure.com), recherchez le nom de l’application et sélectionnez celle-ci dans les résultats.

:::image type="content" source="./media/tutorial-django-app-service-postgres/navigate-to-django-app-in-app-services-in-the-azure-portal.png" alt-text="Accéder à votre application Python Django dans le portail Azure":::

Par défaut, le portail affiche la page **Vue d’ensemble** de votre application, qui présente une vue générale des performances. Elle vous permet d’exécuter des tâches de gestion de base telles que parcourir, arrêter, redémarrer et supprimer. Les onglets sur le côté gauche de la page affichent les différentes pages de configuration que vous pouvez ouvrir.

:::image type="content" source="./media/tutorial-django-app-service-postgres/manage-django-app-in-app-services-in-the-azure-portal.png" alt-text="Gérer votre application Python Django dans la page Vue d’ensemble du portail Azure":::


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez conserver l’application ou passer au tutoriel suivant, allez directement à [Étapes suivantes](#next-steps). Autrement, pour éviter des frais récurrents, vous pouvez supprimer le groupe de ressources créé pour ce tutoriel :

```azurecli
az group delete -g myresourcegroup
```

La commande utilise le nom du groupe de ressources mis en cache dans le fichier *.azure/config*. En supprimant le groupe de ressources, vous désallouez et supprimez également toutes les ressources qu’il contient.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment mapper un nom DNS personnalisé à votre application :

> [!div class="nextstepaction"]
> [Tutoriel : Mappage d’un nom DNS personnalisé à une application](../../app-service/app-service-web-tutorial-custom-domain.md)

Découvrez comment App Service exécute une application Python :

> [!div class="nextstepaction"]
> [Configurer une application Python](../../app-service/configure-language-python.md)
