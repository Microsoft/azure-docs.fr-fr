---
title: Créer, gérer et sécuriser des clés API d’administration et de requête
titleSuffix: Azure Cognitive Search
description: Une clé API contrôle l’accès au point de terminaison de service. Les clés d’administration accordent un accès en écriture. Les clés de requête peuvent être créées pour l’accès en lecture seule.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: 29a314553584843ed6241b9311e9d72b42ec8705
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516416"
---
# <a name="create-and-manage-api-keys-for-an-azure-cognitive-search-service"></a>Créer et gérer des clés API pour un service Recherche cognitive Azure

Toutes les demandes adressées à un service de recherche ont besoin d’une `api-key` en lecture seule générée spécialement pour votre service. Cette `api-key` constitue le seul mécanisme d’authentification de l’accès au point de terminaison de votre service de recherche et doit être incluse dans chaque requête. 

+ Dans les [solutions REST](search-get-started-rest.md), la clé API est généralement spécifiée dans un en-tête de requête

+ Dans les [solutions .NET](search-howto-dotnet-sdk.md), une clé est souvent spécifiée comme paramètre de configuration, puis transmise en tant que [AzureKeyCredential](/dotnet/api/azure.azurekeycredential)

Lors du provisionnement du service, les clés sont créées avec votre service de recherche. Vous pouvez afficher et obtenir des valeurs de clés dans le [portail Azure](https://portal.azure.com).

:::image type="content" source="media/search-manage/azure-search-view-keys.png" alt-text="page du portail, récupérer les paramètres, section clés" border="false":::

## <a name="what-is-an-api-key"></a>Qu’est-ce qu’une clé API ?

Une clé API est une chaîne composée de nombres et de lettres générée de manière aléatoire. Par le biais des [autorisations basées sur le rôle](search-security-rbac.md), vous pouvez supprimer ou lire les clés, mais vous ne pouvez pas remplacer une clé avec un mot de passe défini par l’utilisateur ou utiliser Active Directory en tant que méthode d’authentification principale pour accéder aux opérations de recherche. 

Deux types de clés sont utilisés pour accéder à votre service de recherche : administration (lecture-écriture) et requête (lecture seule).

|Clé|Description|limites|  
|---------|-----------------|------------|  
|Admin|Accorde des droits d’accès complets à toutes les opérations, avec notamment la possibilité de gérer le service ou de créer et supprimer des index, des indexeurs et des sources de données.<br /><br /> Deux clés d’administration, appelées clés *principale* et *secondaire* dans le portail, sont générées quand le service est créé et peuvent être régénérées individuellement à la demande. La possession de deux clés permet de substituer une clé quand l’autre est utilisée pour un accès continu au service.<br /><br /> Les clés d’administration sont spécifiées uniquement dans les en-têtes de requête HTTP. Vous ne pouvez pas insérer de clé API d’administration dans une URL.|2 max. par service|  
|Requête|Accorde un accès en lecture seule aux index et aux documents. Ces clés sont généralement distribuées aux applications clientes qui émettent des demandes de recherche.<br /><br /> Les clés de requête sont créées à la demande. Vous pouvez les créer manuellement dans le portail ou par programme via l’[API REST de gestion](/rest/api/searchmanagement/).<br /><br /> Les clés de requête peuvent être spécifiées dans un en-tête de requête HTTP pour les opérations de recherche, de suggestion ou de consultation. Vous pouvez également transmettre une clé de requête en tant que paramètre pour une URL. Selon la façon dont votre application cliente formule la demande, il peut être plus facile de transmettre la clé en tant que paramètre de requête :<br /><br /> `GET /indexes/hotels/docs?search=*&$orderby=lastRenovationDate desc&api-version=2020-06-30&api-key=[query key]`|50 par service|  

 Visuellement, il n’existe aucune distinction entre une clé d’administration et une clé de requête. Les deux clés sont des chaînes composées de 32 caractères alphanumériques générés de façon aléatoire. Si vous n’êtes pas sûr du type de clé spécifié dans votre application, vous pouvez [vérifier les valeurs de clé dans le portail](https://portal.azure.com) ou utiliser l’[API REST](/rest/api/searchmanagement/) pour retourner la valeur et le type de clé.  

> [!NOTE]  
>  L’insertion de données sensibles comme une `api-key` dans l’URI de requête est considérée comme une pratique peu sécurisée. C’est pourquoi Recherche cognitive Azure accepte uniquement une clé de requête sous forme de `api-key` dans la chaîne de requête, et il est conseillé de procéder autrement, sauf si le contenu de l'index doit être accessible au public. En règle générale, nous vous recommandons de transmettre votre `api-key` en tant qu'en-tête de demande.  

## <a name="find-existing-keys"></a>Rechercher des clés existantes

Vous pouvez obtenir les clés d’accès dans le portail ou via l’[API REST de gestion](/rest/api/searchmanagement/). Pour plus d’informations, consultez la page [Gérer les clés API d’administration et de requête](search-security-api-keys.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Répertoriez les [services de recherche](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) pour votre abonnement.
3. Sélectionnez le service, puis sur la page de présentation, cliquez sur **Paramètres** >**Clés** pour afficher les clés d'administration et de requête.

   :::image type="content" source="media/search-security-overview/settings-keys.png" alt-text="page du portail, afficher les paramètres, section clés" border="false":::

## <a name="create-query-keys"></a>Créer des clés de requête

Les clés de requête sont utilisées pour l'accès en lecture seule aux documents au sein d'un index pour les opérations ciblant une collection de documents. Les requêtes de recherche, de filtrage et de suggestion sont toutes des opérations qui utilisent une clé de requête. Toute opération en lecture seule qui renvoie des données système ou des définitions d'objet, comme une définition d'index ou un statut d'indexation, nécessite une clé d'administration.

Il est essentiel de restreindre l'accès et les opérations dans les applications clientes afin de protéger les ressources de recherche de votre service. Utilisez toujours une clé de requête plutôt qu'une clé d'administration pour toutes les requêtes provenant d'une application cliente.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Répertoriez les [services de recherche](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) pour votre abonnement.
3. Sélectionnez le service, puis sur la page de présentation, cliquez sur **Paramètres** >**Clés**.
4. Cliquez sur **Gérer les clés de requête**.
5. Utilisez la clé de requête déjà générée pour votre service, ou créez jusqu'à 50 nouvelles clés de requête. La clé de requête par défaut n'est pas nommée, mais des clés de requête supplémentaires peuvent être nommées pour faciliter la gestion.

   :::image type="content" source="media/search-security-overview/create-query-key.png" alt-text="Créer ou utiliser une clé de requête" border="false":::

> [!Note]
> Vous trouverez un exemple de code illustrant l'utilisation de la clé de requête dans [Interroger un index Recherche cognitive Azure en C#](./search-get-started-dotnet.md).

<a name="regenerate-admin-keys"></a>

## <a name="regenerate-admin-keys"></a>Régénération des clés d’administration

Deux clés d'administration sont créées pour chaque service. Vous pouvez ainsi remplacer la clé primaire par la clé secondaire pour assurer la continuité de vos activités.

1. Dans la page **Paramètres** >**Clés**, copiez la clé secondaire.
2. Pour toutes les applications, mettez à jour les paramètres de la clé API afin d’utiliser la clé secondaire.
3. Régénérez la clé principale.
4. Mettez à jour toutes les applications pour qu’elles utilisent la nouvelle clé principale.

Si, par inadvertance, vous régénérez les deux clés en même temps, toutes les requêtes de client utilisant ces clés échoueront (HTTP 403 Refusé). Toutefois, le contenu ne sera pas supprimé et vous ne subirez pas de verrouillage permanent. 

Vous pouvez toujours accéder au service via le portail ou la couche de gestion ([REST API](/rest/api/searchmanagement/), [PowerShell](./search-manage-powershell.md) ou Azure Resource Manager). Les fonctions de gestion reposent sur un ID d'abonnement et non sur une clé API de service. Elles restent donc disponibles même si vos clés API ne le sont pas. 

Après avoir créé de nouvelles clés via le portail ou la couche de gestion, l'accès à votre contenu (index, indexeurs, sources de données, cartes de synonymes) est restauré dès que vous disposez des nouvelles clés et que vous les fournissez sur les requêtes.

## <a name="secure-api-keys"></a>Sécuriser les clés API

La sécurité des clés est assurée en limitant l’accès via le portail ou des interfaces Resource Manager (PowerShell ou interface de ligne de commande). Comme indiqué, les administrateurs des abonnements peuvent afficher et régénérer toutes les clés API. Par précaution, passez en revue les affectations de rôle pour comprendre qui a accès aux clés Admin.

+ Dans le tableau de bord de service, cliquez sur **Contrôle d’accès (IAM)** , puis sur l’onglet **Attributions de rôles** pour afficher les affectations de rôle pour votre service.

Les membres des rôles suivants peuvent afficher et régénérer les clés : Propriétaire, Collaborateur, [Collaborateurs Search Service](../role-based-access-control/built-in-roles.md#search-service-contributor)

> [!Note]
> Pour un accès en fonction de l’identité sur les résultats de recherche, vous pouvez créer des filtres de sécurité pour ajuster les résultats par identité, en supprimant les documents auxquels le demandeur ne doit pas avoir accès. Pour plus d’informations, consultez [Filtres de sécurité](search-security-trimming-for-azure-search.md) et [Sécuriser avec Active Directory](search-security-trimming-for-azure-search-with-aad.md).

## <a name="see-also"></a>Voir aussi

+ [Contrôle d’accès en fonction du rôle dans Recherche cognitive Azure](search-security-rbac.md)
+ [Gestion à l’aide de PowerShell](search-manage-powershell.md) 
+ [Article sur les performances et l’optimisation](search-performance-optimization.md)