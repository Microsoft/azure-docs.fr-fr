---
title: 'Démarrage rapide : Créer une application Python'
description: Commencez à utiliser Azure App Service en déployant votre première application Python sur un conteneur Linux dans App Service.
ms.topic: quickstart
ms.date: 11/10/2020
ms.custom: seo-python-october2019, cli-validate, devx-track-python, devx-track-azurecli
zone_pivot_groups: python-frameworks-01
adobe-target: true
adobe-target-activity: DocsExp–377467–A/B– Quickstarts/Python App–12.11
adobe-target-experience: Experience B
adobe-target-content: ./quickstart-python-1
ms.openlocfilehash: 29c2e9e892b2c5f51907cc63d9b2bf1a6f2e7979
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561526"
---
# <a name="quickstart-create-a-python-app-in-azure-app-service-on-linux"></a>Démarrage rapide : Créer une application Python dans Azure App Service sur Linux

Dans ce démarrage rapide, vous déployez une application web Python dans [App Service sur Linux](overview.md#app-service-on-linux), le service d’hébergement web hautement scalable et autocorrectif d’Azure. Vous utilisez l’[interface de ligne de commande (CLI) Azure](/cli/azure/install-azure-cli) locale sur un ordinateur Mac, Linux ou Windows pour déployer un exemple avec les frameworks Flask ou Django. L’application web que vous configurez utilise un niveau App Service gratuit. Vous n’êtes donc pas facturé quand vous suivez la procédure décrite dans cet article.

Pour les **définitions des termes courants**, consultez [Terminologie Azure en bref](/azure/developer/python/cloud-azure-terminology?toc=/azure/app-service/toc.json).

> [!TIP]
> Si vous préférez plutôt utiliser Visual Studio Code, suivez notre **[démarrage rapide Visual Studio Code App Service](/azure/developer/python/tutorial-deploy-app-service-on-linux-01)** .

## <a name="set-up-your-initial-environment"></a>Configurer votre environnement initial

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

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

## <a name="clone-the-sample"></a>Clonage de l’exemple

Clonez l’exemple de dépôt à l’aide de la commande suivante et accédez au dossier de l’exemple. ([Installez Git](https://git-scm.com/downloads) si vous ne l’avez pas déjà fait.)

::: zone pivot="python-framework-flask"
```terminal
git clone https://github.com/Azure-Samples/python-docs-hello-world
```
::: zone-end

::: zone pivot="python-framework-django"
```terminal
git clone https://github.com/Azure-Samples/python-docs-hello-django
```
::: zone-end

L’exemple contient du code propre au framework qu’Azure App Service reconnaît au démarrage de l’application. Pour plus d’informations, consultez [Processus de démarrage du conteneur](configure-language-python.md#container-startup-process).

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

## <a name="run-the-sample"></a>Exécution de l'exemple

::: zone pivot="python-framework-flask"
1. Accédez au dossier *python-docs-hello-world* :

    ```terminal
    cd python-docs-hello-world
    ```

1. Créez un environnement virtuel et installez les dépendances :

    [!include [virtual environment setup](../../includes/app-service-quickstart-python-venv.md)]

    Si vous rencontrez une erreur du type « [Errno 2] Pas de fichier ou de répertoire correspondant : 'requirements.txt'. », vérifiez que vous êtes dans le dossier *python-docs-hello-world*.

1. Lancez le serveur de développement.

    ```terminal  
    flask run
    ```
    
    Par défaut, le serveur suppose que le module d’entrée de l’application se trouve dans *app.py*, comme dans l’exemple.

    Si vous utilisez un autre nom de module, définissez la variable d’environnement `FLASK_APP` sur ce nom.

    Si vous rencontrez l’erreur « Impossible de localiser une application Flask. Vous n’avez pas fourni la variable d’environnement « FLASK_APP », et un module « wsgi.py » ou « app.py » n’a pas été trouvé dans le répertoire actif. », veillez à être dans le dossier `python-docs-hello-world` qui contient l’exemple.

1. Ouvrez un navigateur web et accédez à l’exemple d’application sur `http://localhost:5000/`. L’application affiche le message **Hello, World !** .

    ![Exécuter un exemple d’application Python localement](./media/quickstart-python/run-hello-world-sample-python-app-in-browser-localhost.png)
    
1. Dans la fenêtre de terminal, appuyez sur **Ctrl**+**C** pour quitter le serveur de développement.
::: zone-end

::: zone pivot="python-framework-django"
1. Accédez au dossier *python-docs-hello-django* :

    ```terminal
    cd python-docs-hello-django
    ```

1. Créez un environnement virtuel et installez les dépendances :

    [!include [virtual environment setup](../../includes/app-service-quickstart-python-venv.md)]

    Si vous rencontrez une erreur du type « [Errno 2] Pas de fichier ou de répertoire correspondant : 'requirements.txt'. », vérifiez que vous êtes dans le dossier *python-docs-hello-django*.
    
1. Lancez le serveur de développement.

    ```terminal
    python manage.py runserver
    ```

1. Ouvrez un navigateur web et accédez à l’exemple d’application sur `http://localhost:8000/`. L’application affiche le message **Hello, World !** .

    ![Exécuter un exemple d’application Python localement](./media/quickstart-python/run-hello-world-sample-python-app-in-browser-localhost.png)
    
1. Dans la fenêtre de terminal, appuyez sur **Ctrl**+**C** pour quitter le serveur de développement.
::: zone-end

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

## <a name="deploy-the-sample"></a>Déployer l'exemple

Déployez le code dans votre dossier local (*python-docs-hello-world*) à l’aide de la commande `az webapp up` :

```azurecli
az webapp up --sku F1 --name <app-name>
```

- Si la commande `az` n’est pas reconnue, vérifiez qu’Azure CLI est installé, comme décrit dans [Configurer votre environnement initial](#set-up-your-initial-environment).
- Si la commande `webapp` n’est pas reconnue, vérifiez que vous utilisez Azure CLI version 2.0.80 ou ultérieure. Si ce n’est pas le cas, [installez la dernière version](/cli/azure/install-azure-cli).
- Remplacez `<app_name>` par un nom unique sur l’ensemble d’Azure (*les caractères valides sont `a-z`, `0-9` et `-`* ). Un bon modèle consiste à utiliser une combinaison du nom de votre société et d’un identificateur d’application.
- L’argument `--sku F1` crée l’application web sur le niveau tarifaire Gratuit. Omettez cet argument pour utiliser un niveau Premium plus rapide, ce qui entraîne un coût horaire.
- Vous pouvez éventuellement inclure l’argument `--location <location-name>` où `<location_name>` est une région Azure disponible. Vous pouvez récupérer une liste de régions autorisées pour votre compte Azure en exécutant la commande [`az account list-locations`](/cli/azure/appservice#az-appservice-list-locations).
- Si vous voyez l’erreur « Impossible de détecter automatiquement la pile d’exécution de votre application », vérifiez que vous exécutez la commande dans le dossier *python-docs-hello-world* (Flask) ou dans le dossier *python-docs-hello-django* (Django) qui contient le fichier *requirements.txt*. (Consultez [Résolution des problèmes de détection automatique avec az webapp up](https://github.com/Azure/app-service-linux-docs/blob/master/AzWebAppUP/runtime_detection.md) (GitHub).)

La commande peut prendre quelques minutes. Lors de son exécution, elle fournit des messages sur la création du groupe de ressources, le plan App Service et l’application d’hébergement, la configuration de la journalisation, puis le déploiement ZIP. Elle affiche ensuite le message « Vous pouvez lancer l’application sur http://&lt;app-name&gt;.azurewebsites.net », qui est l’URL de l’application sur Azure.

![Exemple de sortie de la commande az webapp up](./media/quickstart-python/az-webapp-up-output.png)

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

[!include [az webapp up command note](../../includes/app-service-web-az-webapp-up-note.md)]

## <a name="browse-to-the-app"></a>Accéder à l’application

Accédez à l’application déployée à l’aide de votre navigateur web à l’URL `http://<app-name>.azurewebsites.net`. Le démarrage de l’application pouvant prendre une ou deux minutes, si vous voyez une page d’application par défaut, attendez une minute et actualisez le navigateur.

L’exemple de code Python exécute un conteneur Linux dans App Service avec une image intégrée.

![Exécuter un exemple d’application Python dans Azure](./media/quickstart-python/run-hello-world-sample-python-app-in-browser.png)

**Félicitations !** Vous venez de déployer votre application Python sur App Service.

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

## <a name="redeploy-updates"></a>Redéployer les mises à jour

Dans cette section, vous apportez une petite modification au code, puis vous redéployez le code dans Azure. La modification du code insère une instruction `print` pour générer la sortie de journalisation que vous allez utiliser à la section suivante.

::: zone pivot="python-framework-flask"
Ouvrez *app.py* dans un éditeur et mettez à jour la fonction `hello` d’après le code suivant. 

```python
def hello():
    print("Handling request to home page.")
    return "Hello, Azure!"
```
::: zone-end
::: zone pivot="python-framework-django"
Ouvrez *hello/views.py* dans un éditeur et mettez à jour la fonction `hello` selon le code ci-dessous.

```python
def hello(request):
    print("Handling request to home page.")
    return HttpResponse("Hello, Azure!")
```
::: zone-end
    
Enregistrez vos modifications, puis redéployez l’application en réexécutant la commande `az webapp up` :

```azurecli
az webapp up
```

Cette commande utilise des valeurs qui sont mises en cache localement dans le fichier *.azure/config*, notamment le nom de l’application, le groupe de ressources et le plan App Service.

Une fois le déploiement terminé, revenez à la fenêtre du navigateur ouverte sur `http://<app-name>.azurewebsites.net`. Actualisez la page, qui doit afficher le message modifié :

![Exécuter un exemple d’application Python mis à jour dans Azure](./media/quickstart-python/run-updated-hello-world-sample-python-app-in-browser.png)

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

> [!TIP]
> Visual Studio Code fournit des extensions puissantes pour Python et Azure App Service, qui simplifient le processus de déploiement d’applications web Python sur App Service. Pour plus d’informations, consultez [Déployer des applications Python sur App Service à partir de Visual Studio Code](/azure/python/tutorial-deploy-app-service-on-linux-01).

## <a name="stream-logs"></a>Diffuser les journaux d’activité en continu

Vous pouvez accéder aux journaux de la console générés à l’intérieur de l’application et au conteneur dans lequel elle s’exécute. Les journaux incluent toute sortie générée à l’aide d’instructions `print`.

Pour activer le streaming des journaux, exécutez la commande [az webapp log tail](/cli/azure/webapp/log?view=azure-cli-latest&preserve-view=true#az_webapp_log_tail) :

```azurecli
az webapp log tail
```

Vous pouvez aussi inclure le paramètre `--logs` dans la commande `az webapp up` pour ouvrir automatiquement le stream des journaux lors du déploiement.

Actualisez l’application dans le navigateur pour générer des journaux de console, qui incluent des messages décrivant les requêtes HTTP envoyées à l’application. Si aucune sortie ne s’affiche immédiatement, réessayez dans 30 secondes.

Vous pouvez également inspecter les fichiers journaux à partir du navigateur sur `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.

Pour arrêter le streaming des journaux à tout moment, appuyez sur **Ctrl**+**C** dans le terminal.

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

## <a name="manage-the-azure-app"></a>Gérer l’application Azure

Accédez au <a href="https://portal.azure.com" target="_blank">portail Azure</a> pour gérer l’application que vous avez créée. Recherchez et sélectionnez **App Services**.

![Accéder à App Service dans le portail Azure](./media/quickstart-python/navigate-to-app-services-in-the-azure-portal.png)

Sélectionnez le nom de votre application Azure.

![Accéder à votre application Python dans App Services dans le portail Azure](./media/quickstart-python/navigate-to-app-in-app-services-in-the-azure-portal.png)

La sélection de l’application ouvre la page **Vue d’ensemble**, dans laquelle vous pouvez effectuer des tâches de gestion de base (parcourir, arrêter, démarrer, redémarrer et supprimer).

![Gérer votre application Python dans la page Vue d’ensemble du portail Azure](./media/quickstart-python/manage-an-app-in-app-services-in-the-azure-portal.png)

Le menu App Service fournit différentes pages vous permettant de configurer votre application.

Vous rencontrez des problèmes ? Consultez d’abord le [Guide de résolution des problèmes](configure-language-python.md#troubleshooting) ; autrement, [faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Au cours des étapes précédentes, vous avez créé des ressources Azure au sein d’un groupe de ressources. Le groupe de ressources a un nom tel que « appsvc_rg_Linux_CentralUS » en fonction de votre emplacement. Si vous utilisez une référence SKU App Service autre que le niveau F1 gratuit, ces ressources entraînent des coûts récurrents (consultez [Tarifs App Service](https://azure.microsoft.com/pricing/details/app-service/linux/)).

Si vous ne pensez pas avoir besoin de ces ressources à l’avenir, supprimez le groupe de ressources en exécutant la commande suivante :

```azurecli
az group delete --no-wait
```

La commande utilise le nom du groupe de ressources mis en cache dans le fichier *.azure/config*.

Avec l’argument `--no-wait`, la commande peut retourner une sortie avant la fin de l’opération.

Vous rencontrez des problèmes ? [Faites-le nous savoir](https://aka.ms/FlaskCLIQuickstartHelp).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Application web Python (Django) avec PostgreSQL](tutorial-python-postgresql-app.md)

> [!div class="nextstepaction"]
> [Configurer une application Python](configure-language-python.md)

> [!div class="nextstepaction"]
> [Ajouter une connexion utilisateur à une application web Python](../active-directory/develop/quickstart-v2-python-webapp.md)

> [!div class="nextstepaction"]
> [Tutoriel : Exécuter une application Python dans un conteneur personnalisé](tutorial-custom-container.md)
