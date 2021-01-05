---
title: Intégrer Gestion des API à Service Fabric dans Azure
description: Découvrez comment démarrer rapidement avec Gestion des API Azure et comment router le trafic vers un service backend dans Service Fabric.
ms.topic: conceptual
ms.date: 07/10/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: b89204e2df5c8e79200b780de38d06b1a93c1fa3
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97702189"
---
# <a name="integrate-api-management-with-service-fabric-in-azure"></a>Intégrer Gestion des API à Service Fabric dans Azure

Le déploiement du service Gestion des API Azure avec Service Fabric représente un scénario avancé.  Le service Gestion des API est utile lorsque vous avez besoin de publier des API avec un ensemble complet de règles d’acheminement pour vos services Service Fabric principaux. Les applications cloud ont généralement besoin d’une passerelle frontale afin de fournir un point d’entrée unique pour les utilisateurs, les appareils ou d’autres applications. Dans Service Fabric, une passerelle peut être n’importe quel service sans état conçu pour l’entrée de trafic, comme une application ASP.NET Core, Event Hubs, IoT Hub ou Gestion des API Azure.

Cet article vous montre comment configurer [Gestion des API Azure](../api-management/api-management-key-concepts.md) avec Service Fabric pour router le trafic vers un service backend dans Service Fabric.  Lorsque vous avez terminé, vous avez déployé la Gestion des API sur un réseau virtuel et configuré une opération d’API pour qu’elle envoie le trafic aux services sans état principaux. Pour en savoir plus sur les scénarios de gestion des API Azure avec Service Fabric, consultez l’article [Vue d’ensemble](service-fabric-api-management-overview.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="availability"></a>Disponibilité

> [!IMPORTANT]
> Cette fonctionnalité est disponible aux niveaux **Premium** et **Développeur** de Gestion des API en raison de la prise en charge de réseau virtuel requise.

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Installez [Azure PowerShell](/powershell/azure/install-az-ps) ou [Azure CLI](/cli/azure/install-azure-cli).
* Créez un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé dans un groupe de sécurité réseau.
* Si vous déployez un cluster Windows, configurez un environnement de développement Windows. Installez les charges de travail [Visual Studio 2019](https://www.visualstudio.com) et le **développement Azure**, **ASP.NET et le développement web**, ainsi que **Développement multiplateforme .NET Core**.  Ensuite, configurez un [environnement de développement .NET](service-fabric-get-started.md).

## <a name="network-topology"></a>Topologie du réseau

Maintenant que vous avez un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé sur Azure, déployez Gestion des API sur le réseau virtuel (VNET) dans le sous-réseau et le groupe de sécurité réseau désignés pour Gestion des API. Pour cet article, le modèle Resource Manager de Gestion des API est préconfiguré pour utiliser les noms du réseau virtuel, du sous-réseau et du groupe de sécurité réseau que vous avez configurés dans le [tutoriel sur le cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md). Cet article explique comment déployer la topologie suivante sur Azure, où Gestion des API et Service Fabric se trouvent dans des sous-réseaux du même réseau virtuel :

 ![Légende de l’image][sf-apim-topology-overview]

## <a name="sign-in-to-azure-and-select-your-subscription"></a>Connexion à Azure et sélection de votre abonnement

Connectez-vous à votre compte Azure, sélectionnez votre abonnement avant d’exécuter des commandes Azure.

```powershell
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>
```

```azurecli
az login
az account set --subscription <guid>
```

## <a name="deploy-a-service-fabric-back-end-service"></a>Déploiement d’un service principal de Service Fabric

Pour configurer le service Gestion des API afin d’acheminer le trafic vers un service principal Service Fabric, vous avez besoin d’un service en cours d’exécution pour accepter les requêtes.  

Créez un service fiable ASP.NET Core de base sans état en utilisant le modèle de projet d’API web par défaut. Vous créez ainsi un point de terminaison HTTP pour votre service, que vous exposez via le service Gestion des API Azure.

Démarrez Visual Studio en tant qu’administrateur et créez un service ASP.NET Core :

 1. Dans Visual Studio, sélectionnez Fichier -> Nouveau projet.
 2. Sélectionnez le modèle d’application Service Fabric sous Cloud et nommez-le **« ApiApplication »** .
 3. Sélectionnez le modèle de service ASP.NET Core et nommez le projet **« WebApiService »** .
 4. Sélectionnez le modèle de projet API web ASP.NET Core 2.1.
 5. Une fois le projet créé, ouvrez `PackageRoot\ServiceManifest.xml` et supprimez l’attribut `Port` de la configuration des ressources du point de terminaison :

    ```xml
    <Resources>
      <Endpoints>
        <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" />
      </Endpoints>
    </Resources>
    ```

    La suppression du port permet à Service Fabric de spécifier un port dynamique à partir de la plage de ports d’application, ouverte via le Groupe de sécurité réseau dans le modèle Cluster Resource Manager, autorisant le trafic à y accéder à partir du service Gestion des API.

 6. Appuyez sur F5 dans Visual Studio pour vérifier que l’API web est disponible localement.

    Ouvrez Service Fabric Explorer et accédez à une instance spécifique du service ASP.NET Core pour voir l’adresse de base écoutée par le service. Ajoutez `/api/values` à l’adresse de base et ouvrez-la dans un navigateur. Cette action appelle la méthode Get sur ValuesController dans le modèle d’API web. Elle retourne la réponse par défaut fournie par le modèle, un tableau JSON contenant deux chaînes :

    ```json
    ["value1", "value2"]`
    ```

    Il s’agit du point de terminaison que vous exposez via Gestion des API dans Azure.

 7. Enfin, déployez l’application sur votre cluster dans Azure. Dans Visual Studio, cliquez avec le bouton droit sur le projet Application, puis sélectionnez **Publier**. Fournissez le point de terminaison de votre cluster (par exemple, `mycluster.southcentralus.cloudapp.azure.com:19000`) pour déployer l’application sur votre cluster Service Fabric dans Azure.

Un service ASP.NET Core sans état nommé `fabric:/ApiApplication/WebApiService` doit maintenant s’exécuter dans votre cluster Service Fabric dans Azure.

## <a name="download-and-understand-the-resource-manager-templates"></a>Téléchargement et découverte des modèles Resource Manager

Téléchargez et enregistrez les modèles Resource Manager et le fichier de paramètres suivants :

* [network-apim.json][network-arm]
* [network-apim.parameters.json][network-parameters-arm]
* [apim.json][apim-arm]
* [apim.parameters.json][apim-parameters-arm]

Le modèle *network-apim.json* déploie un nouveau groupe de sécurité de sous-réseau et de réseau dans le réseau virtuel dans lequel le cluster Service Fabric est déployé.

Les sections suivantes décrivent les ressources définies par le modèle *apim.json*. Pour plus d’informations, suivez les liens vers la documentation de référence du modèle dans chaque section. Les paramètres configurables définis dans le fichier de paramètres *apim.parameters.json* sont définis plus loin dans cet article.

### <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

[Microsoft.ApiManagement/service](/azure/templates/microsoft.apimanagement/service) décrit l’instance du service Gestion des API : nom, référence SKU ou niveau, emplacement du groupe de ressources, informations sur l’éditeur et réseau virtuel.

### <a name="microsoftapimanagementservicecertificates"></a>Microsoft.ApiManagement/service/certificates

[Microsoft.ApiManagement/service/certificates](/azure/templates/microsoft.apimanagement/service/certificates) configure la sécurité du service Gestion des API. Gestion des API doit s’authentifier avec votre cluster Service Fabric pour la détection de service à l’aide d’un certificat client ayant accès à votre cluster. Cet article utilise le certificat déjà spécifié lors de la création du [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md#createvaultandcert_anchor) qui peut être utilisé par défaut pour accéder à votre cluster.

Cet article utilise le même certificat pour l’authentification du client et pour la sécurité nœud à nœud du cluster. Si vous en possédez un, vous pouvez utiliser un certificat client distinct configuré pour accéder à votre cluster Service Fabric. Renseignez les champs **name**, **password** et **data** (chaîne codée en base 64) pour le fichier de clé privée (.pfx) du certificat de cluster que vous avez spécifié lors de la création de votre cluster Service Fabric.

### <a name="microsoftapimanagementservicebackends"></a>Microsoft.ApiManagement/service/backends

[Microsoft.ApiManagement/service/backends](/azure/templates/microsoft.apimanagement/service/backends) décrit le service principal vers lequel le trafic est transféré.

Pour les principaux de Service Fabric, le cluster Service Fabric est le principal plutôt qu’un service Service Fabric spécifique. Ainsi, une stratégie unique suffit pour acheminer le trafic vers plusieurs services dans le cluster. Ici, le champ **url** est le nom de service complet d’un service dans votre cluster vers lequel toutes les requêtes sont acheminées par défaut si aucun nom de service n’est spécifié dans une stratégie de principal. Vous pouvez utiliser un nom de service factice, tel que «fabric:/fake/service » si vous ne souhaitez pas disposer d’un service de secours. **resourceId** spécifie le point de terminaison de gestion du cluster.  **clientCertificateThumbprint** et **serverCertificateThumbprints** identifient les certificats utilisés pour l’authentification auprès du cluster.

### <a name="microsoftapimanagementserviceproducts"></a>Microsoft.ApiManagement/service/products

[Microsoft.ApiManagement/service/products](/azure/templates/microsoft.apimanagement/service/products) crée un produit. Dans Gestion des API Azure, un produit contient une ou plusieurs API, ainsi qu’un quota et des conditions d’utilisation. Une fois le produit publié, les développeurs peuvent s'y abonner et commencer à utiliser ses API.

Entrez un nom complet sous **displayName** et une **description** pour le produit. Un abonnement est nécessaire pour cet article, mais l’approbation de l’abonnement par un administrateur ne l’est pas.  L’état de ce produit indiqué sous **state** est « published » et est visible pour les abonnés.

### <a name="microsoftapimanagementserviceapis"></a>Microsoft.ApiManagement/service/apis

[Microsoft.ApiManagement/service/apis](/azure/templates/microsoft.apimanagement/service/apis) crée une API. Une API du service Gestion des API représente un ensemble d'opérations qui peuvent être appelées par des applications clientes. Une fois les opérations ajoutées, l'API est ajoutée à un produit et peut être publiée. Une fois l'API publiée, les développeurs peuvent s'y abonner et l'utiliser.

* Vous pouvez indiquer sous **displayName** un nom quelconque pour votre API. Dans le cadre de cet article, utilisez « Service Fabric App ».
* Le champ **name** contient un nom descriptif unique pour l’API, par exemple « service-fabric-app ». Il s’affiche dans les portails des développeurs et de publication.
* **serviceUrl** indique le service HTTP implémentant l’API. La gestion des API transmet les demandes à cette adresse. Pour les principaux de Service Fabric, cette valeur d’URL n’est pas utilisée. Vous pouvez placer n’importe quelle valeur ici. Pour cet article, par exemple « http:\//servicefabric ».
* Le chemin d’accès **path** est ajouté à l’URL de base du service Gestion des API. L'URL de base est commune à toutes les API hébergées par une instance de service Gestion des API. Gestion des API distingue les API selon leur suffixe. Celui-ci doit donc être unique pour chaque API d'un éditeur donné.
* Le champ **protocols** détermine les protocoles qui peuvent être utilisés pour accéder à l’API. Pour cet article, indiquez **http** et **https**.
* Le chemin d’accès **path** est un suffixe pour l’API. Pour cet article, utilisez « myapp ».

### <a name="microsoftapimanagementserviceapisoperations"></a>Microsoft.ApiManagement/service/apis/operations

[Microsoft.ApiManagement/service/apis/operations](/azure/templates/microsoft.apimanagement/service/apis/operations) : avant de pouvoir utiliser une API dans le service Gestion des API, vous devez ajouter des opérations à l’API.  Les clients externes utilisent une opération pour communiquer avec le service ASP.NET Core sans état en cours d’exécution dans le cluster Service Fabric.

Pour ajouter une opération d’API frontale, renseignez les valeurs suivantes :

* **displayName** et **description** décrivent l’opération. Pour cet article, utilisez « Values ».
* **method** spécifie le verbe HTTP.  Pour cet article, spécifiez **GET**.
* **urlTemplate** est ajouté à l’URL de base de l’API et identifie de façon unique l’opération HTTP.  Pour cet article, utilisez `/api/values` si vous avez ajouté le service backend .NET ou `getMessage` si vous avez ajouté le service backend Java.  Par défaut, le chemin d’accès de l’URL spécifié ici est celui envoyé au service principal Service Fabric. Si vous utilisez ici le même chemin d’URL que votre service, par exemple « /api/values », l’opération fonctionne sans modification supplémentaire. Vous pouvez également spécifier ici un chemin d’URL différent de celui utilisé par votre service Service Fabric principal, auquel cas vous devez également spécifier une réécriture de chemin dans votre stratégie d’opération ultérieurement.

### <a name="microsoftapimanagementserviceapispolicies"></a>Microsoft.ApiManagement/service/apis/policies

[Microsoft.ApiManagement/service/apis/policies](/azure/templates/microsoft.apimanagement/service/apis/policies) crée une stratégie de principal qui lie tout ensemble. Elle permet de configurer le service principal Service Fabric vers lequel les requêtes sont acheminées. Vous pouvez appliquer cette stratégie à une opération d’API.  Pour plus d’informations, voir [Vue d’ensemble des stratégies](../api-management/api-management-howto-policies.md).

La [configuration du principal de Service Fabric](../api-management/api-management-transformation-policies.md#SetBackendService) fournit les contrôles de routage de requête suivants :

* Sélection d’une instance de service en spécifiant un nom d’instance de service Service Fabric, codé en dur (par exemple, `"fabric:/myapp/myservice"`) ou généré à partir de la requête HTTP (par exemple, `"fabric:/myapp/users/" + context.Request.MatchedParameters["name"]`).
* Résolution de partition en générant une clé de partition à l’aide d’un modèle de partitionnement de Service Fabric.
* Sélection de réplica pour les services avec état.
* Conditions de nouvelle tentative de résolution vous permettant de spécifier les conditions pour résoudre à nouveau un emplacement de service et renvoyer une requête.

**policyContent** est le contenu XML de la stratégie placé dans une séquence d’échappement Json.  Pour cet article, créez une stratégie de backend pour router les demandes directement vers le service sans état .NET ou Java déployé précédemment. Ajoutez une stratégie `set-backend-service` sous les stratégies de trafic entrant.  Remplacez la valeur de *sf-service-instance-name* par `fabric:/ApiApplication/WebApiService` si vous avez déployé précédemment le service principal .NET ou par `fabric:/EchoServerApplication/EchoServerService` si vous avez déployé le service Java.  *backend-id* fait référence à une ressource principale, dans ce cas, la ressource `Microsoft.ApiManagement/service/backends` définie dans le modèle *apim.json*. *backend-id* peut également faire référence à une autre ressource principale créée à l’aide des API de gestion des API. Pour cet article, définissez *backend-id* sur la valeur du paramètre *service_fabric_backend_name*.

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@(context.LastError?.Reason == "BackendConnectionFailure")" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

Pour un ensemble complet d’attributs de stratégie de principal de Service Fabric, reportez-vous à la [documentation sur le principal de Gestion des API](../api-management/api-management-transformation-policies.md#SetBackendService)

## <a name="set-parameters-and-deploy-api-management"></a>Définir les paramètres et déployer le service Gestion des API

Renseignez les paramètres vides suivants dans le fichier *apim.parameters.json* de votre déploiement.

|Paramètre|Valeur|
|---|---|
|apimInstanceName|sf-apim|
|apimPublisherEmail|myemail@contosos.com|
|apimSku|Développeur|
|serviceFabricCertificateName|sfclustertutorialgroup320171031144217|
|certificatePassword|q6D7nN%6ck@6|
|serviceFabricCertificateThumbprint|C4C1E541AD512B8065280292A8BA6079C3F26F10 |
|serviceFabricCertificate|&lt;chaîne codée en base 64&gt;|
|url_path|/api/values|
|clusterHttpManagementEndpoint|`https://mysfcluster.southcentralus.cloudapp.azure.com:19080`|
|inbound_policy|&lt;chaîne XML&gt;|

*certificatePassword* et *serviceFabricCertificateThumbprint* doivent correspondre au certificat de cluster utilisé pour configurer le cluster.

*serviceFabricCertificate* est le certificat sous forme de chaîne codée en base 64, qui peut être généré à l’aide du script suivant :

```powershell
$bytes = [System.IO.File]::ReadAllBytes("C:\mycertificates\sfclustertutorialgroup220171109113527.pfx");
$b64 = [System.Convert]::ToBase64String($bytes);
[System.Io.File]::WriteAllText("C:\mycertificates\sfclustertutorialgroup220171109113527.txt", $b64);
```

Dans *inbound_policy*, remplacez la valeur de *sf-service-instance-name* par `fabric:/ApiApplication/WebApiService` si vous avez déployé précédemment le service principal .NET ou par `fabric:/EchoServerApplication/EchoServerService` si vous avez déployé le service Java. *backend-id* fait référence à une ressource principale, dans ce cas, la ressource `Microsoft.ApiManagement/service/backends` définie dans le modèle *apim.json*. *backend-id* peut également faire référence à une autre ressource principale créée à l’aide des API de gestion des API. Pour cet article, définissez *backend-id* sur la valeur du paramètre *service_fabric_backend_name*.

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@(context.LastError?.Reason == "BackendConnectionFailure")" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

Utilisez le script suivant pour déployer le modèle Resource Manager et les fichiers de paramètres nécessaires à la Gestion des API :

```powershell
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"
$templatepath="C:\clustertemplates"

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\network-apim.json" -TemplateParameterFile "$templatepath\network-apim.parameters.json" -Verbose

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\apim.json" -TemplateParameterFile "$templatepath\apim.parameters.json" -Verbose
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az deployment group create --name ApiMgmtNetworkDeployment --resource-group $ResourceGroupName --template-file network-apim.json --parameters @network-apim.parameters.json

az deployment group create --name ApiMgmtDeployment --resource-group $ResourceGroupName --template-file apim.json --parameters @apim.parameters.json
```

## <a name="test-it"></a>Testez-le

Vous pouvez maintenant essayer d’envoyer une requête à votre service principal dans Service Fabric via le service Gestion des API directement à partir du [Portail Azure](https://portal.azure.com).

 1. Dans le service Gestion des API, sélectionnez **API**.
 2. Dans l’API **Application Service Fabric** que vous avez créée aux étapes précédentes, sélectionnez l’onglet **Test**, puis l’opérateur **Values**.
 3. Cliquez sur le bouton **Envoyer** pour envoyer une requête test au service principal.  La réponse HTTP devrait être de ce type :

    ```http
    HTTP/1.1 200 OK

    Transfer-Encoding: chunked

    Content-Type: application/json; charset=utf-8

    Vary: Origin

    Ocp-Apim-Trace-Location: https://apimgmtstodhwklpry2xgkdj.blob.core.windows.net/apiinspectorcontainer/PWSQOq_FCDjGcaI1rdMn8w2-2?sv=2015-07-08&sr=b&sig=MhQhzk%2FEKzE5odlLXRjyVsgzltWGF8OkNzAKaf0B1P0%3D&se=2018-01-28T01%3A04%3A44Z&sp=r&traceId=9f8f1892121e445ea1ae4d2bc8449ce4

    Date: Sat, 27 Jan 2018 01:04:44 GMT


    ["value1", "value2"]
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Un cluster est composé d’autres ressources Azure en plus de la ressource de cluster elle-même. Le plus simple pour supprimer le cluster et toutes les ressources qu’il consomme consiste à supprimer le groupe de ressources.

Connectez-vous à Azure et sélectionnez l’ID d’abonnement avec lequel vous souhaitez supprimer le cluster.  Vous pouvez trouver votre ID d’abonnement en vous connectant au [portail Azure](https://portal.azure.com). Supprimez le groupe de ressources et toutes les ressources de cluster à l’aide de l’[applet de commande Remove-AzResourceGroup](/en-us/powershell/module/az.resources/remove-azresourcegroup).

```powershell
$ResourceGroupName = "sfclustertutorialgroup"
Remove-AzResourceGroup -Name $ResourceGroupName -Force
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az group delete --name $ResourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus d’informations sur [Gestion des API](../api-management/import-and-publish.md).

[azure-powershell]: /powershell/azure/

[apim-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.json
[apim-parameters-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.parameters.json

[network-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.json
[network-parameters-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.parameters.json

<!-- pics -->
[sf-apim-topology-overview]: ./media/service-fabric-tutorial-deploy-api-management/sf-apim-topology-overview.png
vice-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.parameters.jsonn

<!-- pics -->
[sf-apim-topology-overview]: ./media/service-fabric-tutorial-deploy-api-management/sf-apim-topology-overview.png
