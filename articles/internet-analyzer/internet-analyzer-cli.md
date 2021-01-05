---
title: Créer un test Internet Analyzer avec l’interface de ligne de commande | Microsoft Docs
description: Dans cet article, vous allez voir comment créer votre premier test Internet Analyzer à l’aide d’Azure CLI.
services: internet-analyzer
author: diego-perez-botero
ms.service: internet-analyzer
ms.topic: tutorial
ms.date: 10/16/2019
ms.author: mebeatty
ms.openlocfilehash: 23a76e4007965c263671554bbcd937dbf3b4b0a6
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92676205"
---
# <a name="create-an-internet-analyzer-test-using-cli-preview"></a>Créer un test Internet Analyzer avec l’interface de ligne de commande (préversion)

Il existe deux façons de créer une ressource Internet Analyzer : avec le portail Azure et avec l’[interface CLI](internet-analyzer-create-test-portal.md). Cette section va vous aider à créer une ressource Azure Internet Analyzer à l’aide de l’interface de ligne de commande. 


> [!IMPORTANT]
> Cette préversion publique est fournie sans contrat de niveau de service et ne doit pas être utilisée pour les charges de travail de production. Certaines fonctionnalités peuvent ne pas être prises en charge, disposer de capacités limitées ou ne pas être disponibles dans tous les emplacements Azure. Consultez les [Conditions d’utilisation supplémentaires des préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="before-you-begin"></a>Avant de commencer

La préversion publique est disponible dans toutes les régions. Toutefois, le stockage de données est seulement disponible dans la région *USA Ouest 2* dans la préversion.

## <a name="object-model"></a>Modèle objet
L’interface CLI d’Internet Analyzer expose les types de ressources suivants :
* **Tests**  : un test compare les performances des utilisateurs finaux de deux points de terminaison Internet (A et B) au fil du temps.
* **Profils** : les tests sont créés sous un profil Internet Analyzer. Les profils permettent de regrouper les tests associés ; un profil unique peut contenir un ou plusieurs tests.
* **Points de terminaison préconfigurés**  : nous avons préparé des points de terminaison avec diverses configurations (régions, technologies d’accélération, etc.). Vous pouvez utiliser un de ces points de terminaison préconfigurés dans vos tests.
* **Cartes de performance** : une carte de performance fournit des résumés rapides et pertinents des résultats des mesures. Consultez [Interprétation de votre carte de performance](internet-analyzer-scorecard.md).
* **Série chronologique** : Une série chronologique montre comment une métrique change dans le temps.

## <a name="profile-and-test-creation"></a>Création de profils et de tests
1. Obtenez un accès à la préversion d’Internet Analyzer en suivant les instructions fournies dans la section **Comment faire pour participer à la préversion ?** des [Questions fréquentes (FAQ) sur Internet Analyzer](internet-analyzer-faq.md).
2. [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).
3. Exécutez la commande `login` pour démarrer une session CLI :
    ```azurecli-interactive
    az login
    ```

    Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fait et charge une page de connexion Azure par la même occasion.
    Sinon, ouvrez une page de navigateur à l’adresse https://aka.ms/devicelogin et entrez le code d’autorisation affiché dans votre terminal.

4. Dans le navigateur, connectez-vous avec les informations d’identification de votre compte.

5. Sélectionnez votre ID d’abonnement qui a obtenu l’accès à la préversion publique d’Internet Analyzer.

    Après vous être connecté, vous voyez une liste des abonnements associés à votre compte Azure. Les informations d’abonnement avec `isDefault: true` correspond à l’abonnement actuellement activé après vous être connecté. Pour sélectionner un autre abonnement, utilisez la commande [az account set](/cli/azure/account#az-account-set) avec l’ID d’abonnement auquel vous voulez passer. Pour plus d’informations sur la sélection d’abonnements, consultez [Utiliser plusieurs abonnements Azure](/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest).

    Il existe des méthodes pour se connecter de manière non interactive, qui sont dévelopées en détail dans [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

6. **[Facultatif]** créer un groupe de ressources Azure :
    ```azurecli-interactive
    az group create --location eastus --name "MyInternetAnalyzerResourceGroup"
    ```

7. Installez l’extension Internet Analyzer pour Azure CLI :
     ```azurecli-interactive
    az extension add --name internet-analyzer
    ```

8. Créez un nouveau profil Internet Analyzer :
    ```azurecli-interactive
    az internet-analyzer profile create --location eastus --resource-group "MyInternetAnalyzerResourceGroup" --name "MyInternetAnalyzerProfile" --enabled-state Enabled
    ```

9. Répertorie tous les points de terminaison préconfigurés disponibles pour le profil nouvellement créé :
    ```azurecli-interactive
    az internet-analyzer preconfigured-endpoint list --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile"
    ```

10. Créez un nouveau test sous le profil Internet Analyzer nouvellement créé :
    ```azurecli-interactive
    az internet-analyzer test create --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --endpoint-a-name "contoso" --endpoint-a-endpoint "www.contoso.com/some/path/to/trans.gif" --endpoint-b-name "microsoft" --endpoint-b-endpoint "www.microsoft.com/another/path/to/trans.gif" --name "MyFirstInternetAnalyzerTest" --enabled-state Enabled
    ```

    La commande ci-dessus suppose que `www.contoso.com` et `www.microsoft.com` hébergent l’image d’un pixel ([trans.gif](https://fpc.msedge.net/apc/trans.gif)) sous des chemins personnalisés. Si un chemin d’accès à l’objet n’est pas spécifié explicitement, Internet Analyzer utilise `/apc/trans.gif` comme chemin d’accès à l’objet par défaut, à savoir l’emplacement où les points de terminaison préconfigurés hébergent l’image d’un pixel. Notez également que le schéma (https/http) n’a pas besoin d’être spécifié. Internet Analyzer ne prend en charge que les points de terminaison HTTPS ; par conséquent, le protocole HTTPS est utilisé.

11. Le nouveau test doit apparaître sous le profil Internet Analyzer :
    ```azurecli-interactive
    az internet-analyzer test list --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile"
    ```

    Exemple de sortie :
    ````
    [
        {
            "description": null,
            "enabledState": "Enabled",
            "endpointA": {
            "endpoint": "www.contoso.com/some/path/to/1k.jpg",
            "name": "contoso"
            },
            "endpointB": {
            "endpoint": "www.microsoft.com/another/path/to/1k.jpg",
            "name": "microsoft"
            },
            "id": "/subscriptions/faa9ddd0-9137-4659-99b7-cdc55a953342/resourcegroups/MyInternetAnalyzerResourceGroup/providers/Microsoft.Network/networkexperimentprofiles/MyInternetAnalyzerProfile/experiments/MyFirstInternetAnalyzerTest",
            "location": null,
            "name": "MyFirstInternetAnalyzerTest",
            "resourceGroup": "MyInternetAnalyzerResourceGroup",
            "resourceState": "Enabled",
            "scriptFileUri": "https://fpc.msedge.net/client/v2/d8c6fc64238d464c882cee4a310898b2/ab.min.js",
            "status": "Created",
            "tags": null,
            "type": "Microsoft.Network/networkexperimentprofiles/experiments"
        }
    ]
    ````

12. Pour commencer à générer des mesures, le fichier JavaScript vers lequel pointe la valeur **scriptFileUri** du test doit être incorporé dans votre application web. Pour obtenir des instructions, consultez la page [Incorporer le client Internet Analyzer](internet-analyzer-embed-client.md).

13. Vous pouvez surveiller la progression du test en gardant une trace de sa valeur « État » :
    ```azurecli-interactive
    az internet-analyzer test show --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest"
    ```

14. Vous pouvez inspecter les résultats collectés par le test en générant des séries chronologiques ou des cartes de performances pour celui-ci :
    ```azurecli-interactive
    az internet-analyzer show-scorecard --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest" --aggregation-interval "Daily" --end-date-time-utc "2019-10-24T00:00:00"
    ```

    ```azurecli-interactive
    az internet-analyzer show-timeseries --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest" --aggregation-interval "Hourly" --start-date-time-utc "2019-10-23T00:00:00" --end-date-time-utc "2019-10-24T00:00:00" --timeseries-type MeasurementCounts
    ```


## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir la liste complète des commandes prises en charge et des exemples d’utilisation, parcourez les [Informations de référence sur l’interface CLI d’Internet Analyzer](/cli/azure/ext/internet-analyzer/internet-analyzer?view=azure-cli-latest).
* Consultez la [FAQ sur Internet Analyzer](internet-analyzer-faq.md).
* En savoir plus sur l’incorporation du [client Internet Analyzer](internet-analyzer-embed-client.md) et sur la création d’un [point de terminaison personnalisé](internet-analyzer-custom-endpoint.md).