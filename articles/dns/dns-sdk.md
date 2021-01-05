---
title: Créer des zones et des jeux d’enregistrements DNS à l’aide du Kit de développement logiciel (SDK) .NET
titleSuffix: Azure DNS
description: Dans ce parcours d’apprentissage, apprenez à créer des zones DNS et des jeux d’enregistrements dans Azure DNS à l’aide du Kit de développement logiciel (SDK) .NET.
services: dns
documentationcenter: na
author: rohinkoul
manager: kumudD
ms.assetid: eed99b87-f4d4-4fbf-a926-263f7e30b884
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2016
ms.author: rohink
ms.custom: devx-track-csharp
ms.openlocfilehash: 8e116096afbd01af4914be49d5675881724d5069
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015058"
---
# <a name="create-dns-zones-and-record-sets-using-the-net-sdk"></a>Créer des zones et des jeux d’enregistrements DNS à l’aide du Kit de développement logiciel (SDK) .NET

Vous pouvez automatiser les opérations de création, de suppression ou de mise à jour des zones , des jeux d’enregistrements et des enregistrements DNS à l’aide du Kit de développement logiciel (SDK) DNS avec la bibliothèque de gestion DNS de .NET. Un projet Visual Studio complet est disponible [ici.](https://www.microsoft.com/en-us/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True)

## <a name="create-a-service-principal-account"></a>Créer un compte de principal du service

En règle générale, l’accès par programme aux ressources Azure est accordé via un compte dédié au lieu de vos propres informations d’identification de l’utilisateur. Ces comptes dédiés sont appelés comptes de « principal du service ». Pour utiliser l’exemple de projet de kit de développement logiciel Azure DNS, vous devez d’abord créer un compte de principal du service et lui attribuer les autorisations appropriées.

1. Suivez [ces instructions](../active-directory/develop/howto-authenticate-service-principal-powershell.md) pour créer un compte de principal du service (l’exemple de projet de kit de développement logiciel Azure DNS part du principe qu’il s’agit d’une authentification par mot de passe).
2. Créez un groupe de ressources ([procédure](../azure-resource-manager/templates/deploy-portal.md)).
3. Utilisez Azure RBAC pour accorder une autorisation « Collaborateur de zone DNS » au groupe de ressources ([procédure](../role-based-access-control/role-assignments-portal.md)).
4. Si vous utilisez l’exemple de projet de kit de développement logiciel Azure DNS, modifiez le fichier program.cs comme suit :

   * Insérez les valeurs correctes pour `tenantId`, `clientId` (également appelé ID de compte), `secret` (mot de passe du compte du principal du service) et `subscriptionId` tels qu’utilisés à l’étape 1.
   * Entrez le nom du groupe de ressources choisi à l’étape 2.
   * Entrez le nom de zone DNS de votre choix.

## <a name="nuget-packages-and-namespace-declarations"></a>Packages NuGet et déclaration des espaces de noms

Pour utiliser le Kit de développement logiciel .NET d’Azure DNS, vous devez installer le package NuGet de la **bibliothèque de gestion Azure DNS** et les autres packages Azure requis.

1. Dans **Visual Studio**, ouvrez un projet existant ou un nouveau projet.
2. Accédez à **Outils** **>** **Gestionnaire de package NuGet** **>** **Gérer les packages NuGet pour la solution...** .
3. Cliquez sur **Parcourir**, cochez la case **Inclure la version préliminaire** et tapez **Microsoft.Azure.Management.Dns** dans la zone de recherche.
4. Sélectionnez le package et cliquez sur **Installer** pour l’ajouter à votre projet Visual Studio.
5. Répétez le processus ci-dessus pour installer les packages suivants : **Microsoft.Rest.ClientRuntime.Azure.Authentication** et **Microsoft.Azure.Management.ResourceManager**.

## <a name="add-namespace-declarations"></a>Ajout de déclarations d'espaces de noms

Ajoutez les déclarations d’espace de noms suivantes :

```cs
using Microsoft.Rest.Azure.Authentication;
using Microsoft.Azure.Management.Dns;
using Microsoft.Azure.Management.Dns.Models;
```

## <a name="initialize-the-dns-management-client"></a>Initialiser le client de gestion DNS

Le `DnsManagementClient` contient les méthodes et les propriétés nécessaires à la gestion des zones et des jeux d’enregistrements DNS.  Le code suivant ouvre une session du compte de principal du service et crée un objet `DnsManagementClient`.

```cs
// Build the service credentials and DNS management client
var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, secret);
var dnsClient = new DnsManagementClient(serviceCreds);
dnsClient.SubscriptionId = subscriptionId;
```

## <a name="create-or-update-a-dns-zone"></a>Créer ou mettre à jour une zone DNS

Pour créer une zone DNS, un objet « Zone » est d’abord créé pour contenir les paramètres de la zone DNS. Comme les zones DNS ne sont pas liées à une région spécifique, l’emplacement est défini sur « global ». Dans cet exemple, une [« balise » Azure Resource Manager](https://azure.microsoft.com/updates/organize-your-azure-resources-with-tags/) est également ajoutée à la zone.

Pour créer ou mettre à jour la zone dans Azure DNS, l’objet de zone qui contient les paramètres de zone est transmis à la méthode `DnsManagementClient.Zones.CreateOrUpdateAsyc`.

> [!NOTE]
> DnsManagementClient prend en charge trois modes de fonctionnement : synchrone (CreateOrUpdate), asynchrone (CreateOrUpdateAsync) ou asynchrone avec accès à la réponse HTTP (CreateOrUpdateWithHttpMessagesAsync).  Vous pouvez choisir l’un de ces modes, selon les besoins de votre application.

Azure DNS prend en charge l’accès simultané optimiste, appelé [ETags](./dns-getstarted-powershell.md). Dans cet exemple, le fait de spécifier « * » pour l’en-tête « If-None-Match » indique à Azure DNS de créer une zone DNS si celle-ci n’existe pas déjà.  L’appel échoue si une zone portant le nom spécifié existe déjà dans le groupe de ressources donné.

```cs
// Create zone parameters
var dnsZoneParams = new Zone("global"); // All DNS zones must have location = "global"

// Create an Azure Resource Manager 'tag'.  This is optional.  You can add multiple tags
dnsZoneParams.Tags = new Dictionary<string, string>();
dnsZoneParams.Tags.Add("dept", "finance");

// Create the actual zone.
// Note: Uses 'If-None-Match *' ETAG check, so will fail if the zone exists already.
// Note: For non-async usage, call dnsClient.Zones.CreateOrUpdate(resourceGroupName, zoneName, dnsZoneParams, null, "*")
// Note: For getting the http response, call dnsClient.Zones.CreateOrUpdateWithHttpMessagesAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*")
var dnsZone = await dnsClient.Zones.CreateOrUpdateAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*");
```

## <a name="create-dns-record-sets-and-records"></a>Créer des jeux d’enregistrements et d’enregistrements DNS

Les enregistrements DNS sont gérés en tant que jeu d'enregistrements. Un jeu d’enregistrements est un ensemble d’enregistrements ayant le même nom et le même type d’enregistrement au sein d’une zone.  Le nom du jeu d’enregistrements est relatif au nom de zone, et pas au nom DNS.

Pour créer ou mettre à jour un jeu d’enregistrements, un objet « RecordSet » est créé et transmis à la méthode `DnsManagementClient.RecordSets.CreateOrUpdateAsync`. Comme avec les zones DNS, il existe trois modes de fonctionnement : synchrone (CreateOrUpdate), asynchrone (CreateOrUpdateAsync) ou asynchrone avec accès à la réponse HTTP (CreateOrUpdateWithHttpMessagesAsync).

Comme avec les zones DNS, les opérations sur les jeux d’enregistrements prennent en charge l’accès concurrentiel optimiste.  Dans cet exemple, étant donné que ni « If-Match » ni « If-None-Match » n’est spécifié, le jeu d’enregistrements est toujours créé.  Cet appel remplace tout jeu d’enregistrements existant portant le même nom et le même type d’enregistrement dans cette zone DNS.

```cs
// Create record set parameters
var recordSetParams = new RecordSet();
recordSetParams.TTL = 3600;

// Add records to the record set parameter object.  In this case, we'll add a record of type 'A'
recordSetParams.ARecords = new List<ARecord>();
recordSetParams.ARecords.Add(new ARecord("1.2.3.4"));

// Add metadata to the record set.  Similar to Azure Resource Manager tags, this is optional and you can add multiple metadata name/value pairs
recordSetParams.Metadata = new Dictionary<string, string>();
recordSetParams.Metadata.Add("user", "Mary");

// Create the actual record set in Azure DNS
// Note: no ETAG checks specified, will overwrite existing record set if one exists
var recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSetParams);
```

## <a name="get-zones-and-record-sets"></a>Obtenir des zones et des jeux d’enregistrements

Les méthodes `DnsManagementClient.Zones.Get` et `DnsManagementClient.RecordSets.Get` récupèrent des zones et jeux d’enregistrements, respectivement. Les jeux d’enregistrements sont identifiés par leur type et leur nom, ainsi que la zone et le groupe de ressources où ils sont présents. Les zones sont identifiées par leur nom et le groupe de ressources dans lequel elles sont présentes.

```cs
var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);
```

## <a name="update-an-existing-record-set"></a>Mettre à jour un jeu d’enregistrements

Pour mettre à jour un jeu d’enregistrements DNS, récupérez d’abord le jeu d’enregistrements, puis mettez à jour son contenu et soumettez la modification.  Dans cet exemple, nous spécifions l’Etag du jeu d’enregistrements récupéré dans le paramètre If-Match. L’appel échoue si une opération simultanée a modifié le jeu d’enregistrements entre-temps.

```cs
var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);

// Add a new record to the local object.  Note that records in a record set must be unique/distinct
recordSet.ARecords.Add(new ARecord("5.6.7.8"));

// Update the record set in Azure DNS
// Note: ETAG check specified, update will be rejected if the record set has changed in the meantime
recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSet, recordSet.Etag);
```

## <a name="list-zones-and-record-sets"></a>Répertorier les zones et les jeux d’enregistrements

Pour répertorier les zones, utilisez les méthodes *DnsManagementClient.Zones.List...* , qui prennent en charge l’énumération de toutes les zones d’un groupe de ressources donné ou toutes les zones d’un abonnement Azure donné (parmi les groupes de ressources). Pour répertorier les jeux d’enregistrements, utilisez les méthodes *DnsManagementClient.RecordSets.List...* , qui prennent en charge l’énumération de tous les jeux d’enregistrements d’une zone donnée ou uniquement les jeux d’enregistrements d’un type spécifique.

Notez que les résultats peuvent être paginés lors du répertoriage des zones et des jeux d’enregistrements.  L’exemple suivant montre comment effectuer une itération dans les pages de résultats. (Une taille de page artificiellement petite de « 2 » est utilisée pour forcer la pagination. Dans la pratique, ce paramètre doit être omis et la taille de page par défaut doit être utilisée.)

```cs
// Note: in this demo, we'll use a very small page size (2 record sets) to demonstrate paging
// In practice, to improve performance you would use a large page size or just use the system default
int recordSets = 0;
var page = await dnsClient.RecordSets.ListAllInResourceGroupAsync(resourceGroupName, zoneName, "2");
recordSets += page.Count();

while (page.NextPageLink != null)
{
    page = await dnsClient.RecordSets.ListAllInResourceGroupNextAsync(page.NextPageLink);
    recordSets += page.Count();
}
```

## <a name="next-steps"></a>Étapes suivantes

Téléchargez l’[exemple de projet de SDK Azure DNS .NET](https://www.microsoft.com/en-us/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True), qui inclut d’autres exemples d’utilisation du Kit de développement logiciel (SDK) Azure DNS .NET, notamment des exemples d’autres types d’enregistrements DNS.