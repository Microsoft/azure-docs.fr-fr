---
title: Récupérer les informations sur la clé de chiffrement
titleSuffix: Azure Cognitive Search
description: Récupérez le nom et la version de la clé de chiffrement utilisés dans un mappage d’index ou de synonymes afin de pouvoir gérer la clé dans Azure Key Vault.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/01/2020
ms.openlocfilehash: 37ff94608e9756142f70a4f3c64d0a6f7eeea685
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88932897"
---
# <a name="get-customer-managed-key-information-from-indexes-and-synonym-maps"></a>Obtenir des informations de clé gérée par le client à partir d’index et de synonym maps (cartes de synonymes)

Dans Azure Cognitive Search, les clés de chiffrement gérées par le client sont créées, stockées et gérées dans Azure Key Vault. Si vous avez besoin de déterminer si un objet est chiffré ou si le nom ou la version de la clé a été utilisée, utilisez l’API REST ou un kit de développement logiciel (SDK) pour récupérer la propriété **encryptionKey** à partir d’un index ou d’une définition de synonym map. 

Nous vous recommandons d’[activer la journalisation](../key-vault/general/logging.md) sur Key Vault afin de pouvoir surveiller l’utilisation de clé.

## <a name="get-the-admin-api-key"></a>Obtient la clé de l’API administrateur

Pour obtenir des définitions d’objet à partir d’un service de recherche, vous devez vous authentifier avec des droits d’administrateur. Le moyen le plus simple d’accéder à la clé d’API d’administration se fait via le portail.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) et ouvrez la page de présentation de votre service de recherche.

1. Sur le côté gauche, cliquez sur **Clés** et copiez une API d’administration. Une clé d’administrateur est requise pour l’index et la récupération de carte des synonymes.

Pour les étapes restantes, basculez vers PowerShell et l’API REST. Le portail n’affiche pas les cartes de synonymes, ni les propriétés de clé de chiffrement des index.

## <a name="use-powershell-and-rest"></a>Utilisez PowerShell Core

Exécutez les commandes suivantes pour configurer les variables et récupérer les définitions d’objet.

```powershell
<# Connect to Azure #>
$Connect-AzAccount

<# Provide the admin API key used for search service authentication  #>
$headers = @{
'api-key' = '<YOUR-ADMIN-API-KEY>'
'Content-Type' = 'application/json'
'Accept' = 'application/json' }

<# List all existing synonym maps #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/synonyms?api-version=2020-06-30&$select=name'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# List all existing indexes #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes?api-version=2020-06-30&$select=name'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# Return a specific synonym map definition. The encryptionKey property is at the end #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/synonyms/<YOUR-SYNONYM-MAP-NAME>?api-version=2020-06-30'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# Return a specific index definition. The encryptionKey property is at the end #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/<YOUR-INDEX-NAME>?api-version=2020-06-30'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez la clé de chiffrement et la version utilisées, vous pouvez gérer la clé dans Azure Key Vault ou vérifier d’autres paramètres de configuration.

+ [Démarrage rapide : Définir et récupérer un secret depuis Azure Key Vault à l’aide de PowerShell](../key-vault/secrets/quick-create-powershell.md)

+ [Configurer des clés gérées par le client pour le chiffrement des données dans le service Recherche cognitive Azure](search-security-manage-encryption-keys.md)