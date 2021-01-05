---
title: Configurer une fonction Azure pour le traitement des données
titleSuffix: Azure Digital Twins
description: Découvrez comment créer une fonction Azure qui peut accéder à des jumeaux numériques et être déclenchée par ceux-ci.
author: baanders
ms.author: baanders
ms.date: 8/27/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 5352a95b865851be937af7b9f19268afd23148db
ms.sourcegitcommit: 58f12c358a1358aa363ec1792f97dae4ac96cc4b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93280031"
---
# <a name="connect-azure-functions-apps-for-processing-data"></a>Connecter des applications Azure Functions pour le traitement des données

La mise à jour des jumeaux numériques en fonction des données s’effectue à l’aide de [**routes d’événements**](concepts-route-events.md), par le biais de ressources de calcul telles qu’ [Azure Functions](../azure-functions/functions-overview.md). Une fonction Azure peut être utilisée pour mettre à jour un jumeau numérique en réponse à :
* Des données de télémétrie d’appareil provenant d’IoT Hub
* La modification d’une propriété ou autres données provenant d’un autre jumeau numérique au sein du graphe jumeau

Cet article explique comment créer une fonction Azure en vue de l’utiliser avec Azure Digital Twins. 

Voici un aperçu des étapes qui sont présentées :

1. Créer une application Azure Functions dans Visual Studio
2. Écrire une fonction Azure avec un déclencheur [Event Grid](../event-grid/overview.md)
3. Ajouter du code d’authentification à la fonction (pour pouvoir accéder à Azure Digital Twins)
4. Publier l’application de fonction dans Azure
5. Configurer l’accès de [sécurité](concepts-security.md) pour l’application de fonction Azure

## <a name="prerequisite-set-up-azure-digital-twins-instance"></a>Condition préalable : Configurer l’instance Azure Digital Twins

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

## <a name="create-an-azure-functions-app-in-visual-studio"></a>Créer une application Azure Functions dans Visual Studio

Dans Visual Studio 2019, sélectionnez _Fichier > Nouveau > Projet_. Recherchez et sélectionnez le modèle _Azure Functions_ , puis sélectionnez _Suivant_.

:::image type="content" source="media/how-to-create-azure-function/create-azure-function-project.png" alt-text="Visual Studio : boîte de dialogue Nouveau projet":::

Spécifiez un nom pour l’application de fonction, puis sélectionnez _Créer_.

:::image type="content" source="media/how-to-create-azure-function/configure-new-project.png" alt-text="Visual Studio : Configurer un nouveau projet":::

Sélectionnez le type d’application de fonction *Déclencheur Event Grid* et sélectionnez _Créer_.

:::image type="content" source="media/how-to-create-azure-function/eventgridtrigger-function.png" alt-text="Visual Studio : boîte de dialogue montrant le déclencheur de projet Azure Functions":::

Une fois l’application de fonction créée, Visual Studio dispose d’un exemple de code rempli automatiquement dans le fichier **function.cs** qui se trouve dans le dossier de votre projet. Cette courte fonction Azure est utilisée pour consigner les événements.

:::image type="content" source="media/how-to-create-azure-function/visual-studio-sample-code.png" alt-text="Visual Studio : fenêtre du projet avec l’exemple de code":::

## <a name="write-an-azure-function-with-an-event-grid-trigger"></a>Écrire une fonction Azure avec un déclencheur Event Grid

Vous pouvez écrire une fonction Azure en ajoutant le kit SDK à votre application de fonction. L’application de fonction interagit avec Azure Digital Twins à l’aide du [Kit de développement logiciel (SDK) Azure Digital Twins pour .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true). 

Pour utiliser le kit SDK, vous devez inclure les packages suivants dans votre projet. Vous pouvez installer les packages à l’aide du gestionnaire de package NuGet de Visual Studio, ou ajouter les packages avec l’outil en ligne de commande `dotnet`. Choisissez une de ces méthodes : 

**Option 1. Ajouter des packages à l’aide du gestionnaire de package Visual Studio :**
    
Pour ce faire, cliquez avec le bouton droit sur votre projet et sélectionnez _Gérer les packages NuGet_ dans la liste. Ensuite, dans la fenêtre qui s’ouvre, sélectionnez l’onglet _Parcourir_ et recherchez les packages suivants. Sélectionnez _Installer_ et _Accepter_ pour approuver le contrat de licence et installer les packages.

* `Azure.DigitalTwins.Core`
* `Azure.Identity` 

Pour que la configuration du pipeline du kit SDK Azure soit correctement effectuée pour Azure Functions, vous aurez également besoin des packages suivants. Répétez le même processus que celui décrit ci-dessus pour installer tous les packages.

* `System.Net.Http`
* `Azure.Core.Pipeline`

**Option 2. Ajouter des packages à l’aide de `dotnet`l’outil en ligne de commande :**

```cmd/sh
dotnet add package Azure.DigitalTwins.Core --version 1.0.0-preview.3
dotnet add package Azure.identity --version 1.2.2
dotnet add package System.Net.Http
dotnet add package Azure.Core.Pipeline
```
Ensuite, dans votre Explorateur de solutions Visual Studio, ouvrez le fichier _function.cs_ , dans lequel se trouve l’exemple de code, et ajoutez les instructions _using_ suivantes à votre fonction Azure. 

```csharp
using Azure.DigitalTwins.Core;
using Azure.Identity;
using System.Net.Http;
using Azure.Core.Pipeline;
```
## <a name="add-authentication-code-to-the-azure-function"></a>Ajouter du code d’authentification à la fonction Azure

Vous allez à présent déclarer les variables au niveau de la classe, et ajoutez le code d’authentification qui permettra à la fonction d’accéder à Azure Digital Twins. Vous allez ajouter ce qui suit à votre fonction Azure dans le fichier {nom de votre fonction}.cs.

* Lisez l’URL du service ADT en tant que variable d’environnement. Il est recommandé de lire l’URL du service à partir d’une variable d’environnement, plutôt que de la coder en dur dans la fonction.
```csharp     
private static readonly string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
```
* Une variable statique destinée à contenir une instance HttpClient. HttpClient est relativement coûteux à créer. Nous voulons donc éviter d’avoir à le faire pour chaque appel de fonction.
```csharp
private static readonly HttpClient httpClient = new HttpClient();
```
* Vous pouvez utiliser les informations d’identification de l’identité managée dans la fonction Azure.
```csharp
ManagedIdentityCredential cred = new ManagedIdentityCredential("https://digitaltwins.azure.net");
```
* Ajoutez une variable locale _DigitalTwinsClient_ à l’intérieur de votre fonction afin de stocker votre instance du client Azure Digital Twins dans le projet de fonction. Vous *ne devez pas* rendre cette variable statique dans votre classe.
```csharp
DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
```
* Ajoutez un contrôle de valeur Null pour _adtInstanceUrl_ et incluez dans un wrapper la logique de votre fonction dans un bloc try catch pour intercepter toutes les exceptions.

Une fois ces modifications effectuées, le code de votre fonction doit ressembler à ce qui suit :

```csharp
// Default URL for triggering event grid function in the local environment.
// http://localhost:7071/runtime/webhooks/EventGrid?functionName={functionname}
using System;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Extensions.Logging;
using Azure.DigitalTwins.Core;
using Azure.Identity;
using System.Net.Http;
using Azure.Core.Pipeline;

namespace adtIngestFunctionSample
{
    public class Function1
    {
        //Your Digital Twin URL is stored in an application setting in Azure Functions
        private static readonly string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
        private static readonly HttpClient httpClient = new HttpClient();

        [FunctionName("TwinsFunction")]
        public void Run([EventGridTrigger] EventGridEvent eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.Data.ToString());
            if (adtInstanceUrl == null) log.LogError("Application setting \"ADT_SERVICE_URL\" not set");
            try
            {
                //Authenticate with Digital Twins
                ManagedIdentityCredential cred = new ManagedIdentityCredential("https://digitaltwins.azure.net");
                DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
                log.LogInformation($"ADT service client connection created.");
                /*
                * Add your business logic here
                */
            }
            catch (Exception e)
            {
                log.LogError(e.Message);
            }

        }
    }
}
```

## <a name="publish-the-function-app-to-azure"></a>Publier l’application de fonction dans Azure

Pour publier l’application de fonction dans Azure, cliquez avec le bouton droit sur le projet de fonction (et non sur la solution) dans l’Explorateur de solutions, puis sélectionnez **Publier**.

> [!IMPORTANT] 
> La publication d’une fonction Azure entraîne des frais supplémentaires dans votre abonnement, indépendamment d’Azure Digital Twins.

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function.png" alt-text="Visual Studio : publication de la fonction Azure":::

Sélectionnez **Azure** comme cible de publication, puis sélectionnez **Suivant**.

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-1.png" alt-text="Visual Studio : boîte de dialogue de publication de la fonction Azure, sélection de Azure":::

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-2.png" alt-text="Visual Studio : boîte de dialogue de publication de la fonction, sélection de Application de fonction Azure (Windows) ou (Linux) en fonction de votre machine":::

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-3.png" alt-text="Visual Studio : boîte de dialogue de publication de la fonction – Créer une fonction Azure":::

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-4.png" alt-text="Visual Studio : boîte de dialogue de publication de la fonction – Renseignement des champs et sélection de Créer":::

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-5.png" alt-text="Visual Studio : boîte de dialogue de publication de la fonction – Sélection de votre application de fonction dans la liste, puis de Terminer":::

Dans la page suivante, entrez le nom souhaité pour la nouvelle application de fonction, un groupe de ressources, ainsi que d’autres informations.
Pour que votre application Functions puisse accéder à Azure Digital Twins, elle doit disposer d’une identité managée par le système et disposer des autorisations nécessaires pour accéder à votre instance Azure Digital Twins.

Ensuite, vous pouvez configurer l’accès de sécurité pour la fonction à l’aide de l’interface CLI ou du portail Azure. Choisissez une de ces méthodes :

## <a name="set-up-security-access-for-the-azure-function-app"></a>Configurer l’accès de sécurité pour l’application de fonction Azure
Vous pouvez configurer l’accès de sécurité pour l’application de fonction Azure en utilisant l’une des options suivantes :

### <a name="option-1-set-up-security-access-for-the-azure-function-app-using-cli"></a>Option 1 : Configurer l’accès de sécurité pour l’application de fonction Azure avec l’interface CLI

Le squelette de fonction Azure des exemples précédents nécessite qu’un jeton de porteur lui soit transmis, afin de pouvoir s’authentifier auprès d’Azure Digital Twins. Pour garantir que ce jeton du porteur sera transmis, vous devez configurer [Managed Service Identity (MSI)](../active-directory/managed-identities-azure-resources/overview.md) pour l’application de fonction. Cette opération ne doit être effectuée qu’une seule fois pour chaque application de fonction.

Vous pouvez créer une identité managée par le système et attribuer l’identité de l’application de fonction au rôle _**Propriétaire des données Azure Digital Twins**_ pour votre instance Azure Digital Twins. L'instance sera ainsi autorisée à effectuer des activités de plan de données. Ensuite, rendez l’URL de l’instance Azure Digital Twins accessible à votre fonction en définissant une variable d’environnement.

[!INCLUDE [digital-twins-role-rename-note.md](../../includes/digital-twins-role-rename-note.md)]

Utilisez [Azure Cloud Shell](https://shell.azure.com) pour exécuter les commandes.

Utilisez la commande suivante pour créer l’identité gérée par le système. Prenez note du champ _principalId_ dans la sortie.

```azurecli-interactive 
az functionapp identity assign -g <your-resource-group> -n <your-App-Service-(function-app)-name>   
```
Utilisez la valeur _principalId_ dans la commande suivante afin d’attribuer l’identité de l’application de fonction au rôle _Propriétaire des données Azure Digital Twins_ pour votre instance Azure Digital Twins.

```azurecli-interactive 
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<principal-ID>" --role "Azure Digital Twins Data Owner"
```
Enfin, vous pouvez rendre l’URL de votre instance Azure Digital Twins accessible à votre fonction en définissant une variable d’environnement. Pour plus d’informations sur la définition d’une variable d’environnement, consultez [*Variables d’environnement*](/sandbox/functions-recipes/environment-variables). 

> [!TIP]
> L’URL de l’instance Azure Digital Twins est créée en ajoutant *https://* au début du *nom d’hôte* de votre instance Azure Digital Twins. Pour afficher le nom d’hôte, ainsi que toutes les propriétés de votre instance, vous pouvez exécuter `az dt show --dt-name <your-Azure-Digital-Twins-instance>`.

```azurecli-interactive 
az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "ADT_SERVICE_URL=https://<your-Azure-Digital-Twins-instance-hostname>"
```
### <a name="option-2-set-up-security-access-for-the-azure-function-app-using-azure-portal"></a>Option n°2 : Configurer l’accès de sécurité pour l’application de fonction Azure avec le portail Azure

Une identité managée affectée par le système permet aux ressources Azure de s’identifier auprès des services cloud (comme Azure Key Vault, par exemple), sans stocker les informations d’identification dans le code. Une fois activées, toutes les autorisations nécessaires peuvent être accordées via le contrôle d’accès en fonction du rôle Azure. Le cycle de vie de ce type d’identité managée est lié au cycle de vie de cette ressource. De plus, chaque ressource (Machines virtuelles, par exemple) peut n’avoir qu’une identité managée affectée par le système.

Dans le [portail Azure](https://portal.azure.com/), recherchez _Application de fonction_ dans la barre de recherche, avec le nom de l’application de fonction que vous avez créée précédemment. Sélectionnez l’ *application de fonction* dans la liste. 

:::image type="content" source="media/how-to-create-azure-function/portal-search-for-functionapp.png" alt-text="Portail Azure : recherche de l’application de fonction":::

Dans la fenêtre de l’application de fonction, sélectionnez _Identité_ dans la barre de navigation sur la gauche pour activer l’identité managée.
Sous l’onglet _Affectée par le système_ , basculez l’ _État_ sur Activé et _Enregistrez_. Une fenêtre contextuelle s’affiche pour _Activer l’identité managée affectée par le système_.
Sélectionnez le bouton _Oui_. 

:::image type="content" source="media/how-to-create-azure-function/enable-system-managed-identity.png" alt-text="Portail Azure : activation de l’identité managée affectée par le système":::

Dans les notifications, vous pouvez vérifier que votre fonction est correctement inscrite auprès d’Azure Active Directory.

:::image type="content" source="media/how-to-create-azure-function/notifications-enable-managed-identity.png" alt-text="Portail Azure : notifications":::

Notez également l’ **ID d’objet** qui figure dans la page _Identité_ , car vous en aurez besoin à la section suivante.

:::image type="content" source="media/how-to-create-azure-function/object-id.png" alt-text="Copier l’ID d’objet à utiliser à l’avenir":::

### <a name="assign-access-roles-using-azure-portal"></a>Affecter des rôles d’accès à l’aide du portail Azure

Sélectionnez le bouton _Attributions de rôles Azure_ qui ouvre la page *Attributions de rôles Azure*. Ensuite, sélectionnez _+ Ajouter une attribution de rôle (préversion)_ .

:::image type="content" source="media/how-to-create-azure-function/add-role-assignments.png" alt-text="Portail Azure : ajout d’une attribution de rôle":::

Dans la page _Ajouter une attribution de rôle (préversion)_ qui s’ouvre, renseignez les champs suivants :

* _Portée_  : groupe de ressources
* _Abonnement_  : sélectionnez votre abonnement Azure
* _Groupe de ressources_  : sélectionnez votre groupe de ressources dans la liste déroulante
* _Rôle_  : sélectionnez _Propriétaire des données Azure Digital Twins_ dans la liste déroulante

Ensuite, enregistrez vos informations en cliquant sur le bouton _Enregistrer_.

:::image type="content" source="media/how-to-create-azure-function/add-role-assignment.png" alt-text="Portail Azure : ajout d’une affectation de rôle (préversion) ":::

### <a name="configure-application-settings-using-azure-portal"></a>Configurer les paramètres de l’application avec le portail Azure

Vous pouvez rendre l’URL de votre instance Azure Digital Twins accessible à votre fonction en définissant une variable d’environnement. Pour plus d’informations, consultez [*Variables d’environnement*](/sandbox/functions-recipes/environment-variables). Les paramètres d’application sont exposés en tant que variables d’environnement pour accéder à l’instance Digital Twins. 

Vous avez besoin de l’URL_INSTANCE_ADT pour créer un paramètre d’application.

Vous pouvez obtenir l’URL_INSTANCE_ADT en ajoutant **_https://_** au nom d’hôte de votre instance. Dans le portail Azure, vous pouvez trouver le nom d’hôte de votre instance Digital Twins en recherchant celle-ci avec la barre de recherche. Ensuite, sélectionnez _Vue d’ensemble_ dans la barre de navigation de gauche pour afficher le _Nom d’hôte_. Copiez cette valeur pour créer un paramètre d’application.

:::image type="content" source="media/how-to-create-azure-function/adt-hostname.png" alt-text="Portail Azure : Vue d’ensemble-> Copie du nom d’hôte à utiliser dans le champ _Valeur_.":::

Vous pouvez désormais créer un paramètre d’application en suivant les étapes ci-dessous :

* Recherchez votre fonction Azure à l’aide de son nom dans la barre de recherche, puis sélectionnez la fonction dans la liste
* Sélectionnez _Configuration_ dans la barre de navigation à gauche pour créer un paramètre d’application
* Sous l’onglet _Paramètres d’application_ , sélectionnez _+ Nouveau paramètre d’application_

:::image type="content" source="media/how-to-create-azure-function/search-for-azure-function.png" alt-text="Portail Azure : recherche d’une fonction Azure existante":::

:::image type="content" source="media/how-to-create-azure-function/application-setting.png" alt-text="Portail Azure : configuration des paramètres de l’application":::

Dans la fenêtre qui s’ouvre, utilisez la valeur copiée ci-dessus pour créer un paramètre d’application. \
_Nom_  : URL_SERVICE_ADT \
_Valeur_  : https://{votre-nom-d’hôte-azure-digital-twins}

Sélectionnez _OK_ pour créer un paramètre d’application.

:::image type="content" source="media/how-to-create-azure-function/add-application-setting.png" alt-text="Portail Azure : ajout de paramètres d’application.":::

Vous pouvez afficher les paramètres de votre application, avec le nom de l’application sous le champ _Nom_. Ensuite, enregistrez les paramètres de votre application en sélectionnant le bouton _Enregistrer_.

:::image type="content" source="media/how-to-create-azure-function/application-setting-save-details.png" alt-text="Portail Azure : affichage de l’application créée et redémarrage de l’application":::

Toute modification apportée aux paramètres de l’application nécessite un redémarrage de l’application. Sélectionnez _Continuer_ pour redémarrer votre application.

:::image type="content" source="media/how-to-create-azure-function/save-application-setting.png" alt-text="Portail Azure : enregistrement des paramètres de l’application":::

Vous pouvez constater que les paramètres de l’application sont mis à jour en sélectionnant l’icône _Notifications_. Si le paramètre de votre application n’est pas créé, vous pouvez essayer de nouveau d’ajouter un paramètre d’application en suivant la procédure décrite ci-dessus.

:::image type="content" source="media/how-to-create-azure-function/notifications-update-web-app-settings.png" alt-text="Portail Azure : notifications de mise à jour des paramètres d’application":::

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez vu les étapes permettant de configurer une fonction Azure en vue d’une utilisation avec Azure Digital Twins. Ensuite, vous pouvez vous abonner à votre fonction Azure pour Event Grid afin d’écouter un point de terminaison. Ce point de terminaison peut être :
* Un point de terminaison Event Grid attaché à Azure Digital Twins pour traiter les messages provenant d’Azure Digital Twins (tels que les messages de changement de propriété, les messages de données de télémétrie générés par les [jumeaux numériques](concepts-twins-graph.md) dans le graphique jumeau ou les messages concernant le cycle de vie)
* Des rubriques du système IoT utilisées par IoT Hub pour envoyer des données de télémétrie et d’autres événements d’appareil
* Un point de terminaison Event Grid qui reçoit des messages d’autres services

Dans la suite, vous allez voir comment créer une fonction Azure de base pour ingérer des données IoT Hub dans Azure Digital Twins :
* [*Guide pratique : Ingérer la télémétrie depuis IoT Hub*](how-to-ingest-iot-hub-data.md)