---
title: Chiffrement des données au repos à l’aide de clés gérées par le client
titleSuffix: Azure Cognitive Search
description: Complétez le chiffrement côté serveur des index et mappages de synonymes dans Recherche cognitive Azure à l’aide de clés que vous créez et gérez dans Azure Key Vault.
manager: nitinme
author: NatiNimni
ms.author: natinimn
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/02/2020
ms.custom: references_regions
ms.openlocfilehash: 8295e619cfda0d4b83a7356d5fd21d4b80f83849
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96530882"
---
# <a name="configure-customer-managed-keys-for-data-encryption-in-azure-cognitive-search"></a>Configurer des clés gérées par le client pour le chiffrement des données dans le service Recherche cognitive Azure

Par défaut, le service Recherche cognitive Azure chiffre automatiquement le contenu indexé au repos avec des [clés gérées par le service](../security/fundamentals/encryption-atrest.md#azure-encryption-at-rest-components). Si vous avez besoin de davantage de protection, vous pouvez compléter le chiffrement par défaut avec une couche de chiffrement supplémentaire à l’aide de clés que vous créez et gérez dans Azure Key Vault. Cet article vous guide tout au long des étapes de configuration du chiffrement à l’aide de clés gérées par le client (CMK).

Le chiffrement à l’aide de clés gérées par le client dépend d’[Azure Key Vault](../key-vault/general/overview.md). Vous pouvez créer vos propres clés de chiffrement et les stocker dans un coffre de clés, ou utiliser les API d’Azure Key Vault pour générer des clés de chiffrement. Azure Key Vault vous permet également d’auditer l’utilisation des clés si vous [activez la journalisation](../key-vault/general/logging.md).  

Le chiffrement avec des clés gérées par le client est appliqué à des index individuels ou à des mappages de synonymes lors de la création de ces objets, et n’est pas spécifié au niveau du service de recherche lui-même. Seuls de nouveaux objets peuvent être chiffrés. Vous ne pouvez pas chiffrer un contenu existant.

Toutes les clés ne doivent pas nécessairement se trouver dans le même coffre de clés. Un service de recherche unique peut héberger plusieurs index chiffrés ou mappages de synonymes chiffrés avec leurs propres clés de chiffrement gérées par le client et stockées dans différents coffres de clés. Vous pouvez également avoir des index et des cartes de synonymes dans le même service qui ne sont pas chiffrés à l’aide de clés gérées par le client.

>[!Important]
> Si vous implémentez des clés gérées par le client, veillez à suivre des procédures strictes lors de la rotation régulière des clés de coffre de clés et des secrets et inscriptions d’application Active Directory. Mettez toujours à jour tout le contenu chiffré pour utiliser les nouveaux secrets et les nouvelles clés avant de supprimer les anciens et les anciennes. Si vous omettez cette étape, votre contenu ne peut pas être déchiffré.

## <a name="double-encryption"></a>Double chiffrement

Pour les services créés après le 1er août 2020 et dans certaines régions, l’étendue du chiffrement à l’aide de clés gérées par le client inclut des disques temporaires, ce qui permet un [double chiffrement complet](search-security-overview.md#double-encryption) actuellement disponible dans les régions suivantes : 

+ USA Ouest 2
+ USA Est
+ États-Unis - partie centrale méridionale
+ Gouvernement américain - Virginie
+ Gouvernement des États-Unis – Arizona

Si vous utilisez une autre région ou un service créé avant le 1er août, votre chiffrement à l’aide de clés gérées par le client est limité uniquement au disque de données, à l’exclusion des disques temporaires que le service utilise.

## <a name="prerequisites"></a>Prérequis

Les outils et services suivants sont utilisés dans ce scénario.

+ [Recherche cognitive Azure](search-create-service-portal.md) sur un [niveau facturable](search-sku-tier.md#tier-descriptions) (de base ou version ultérieure, dans n’importe quelle région).
+ [Azure Key Vault](../key-vault/general/overview.md), vous pouvez créer un coffre de clés Azure à l’aide du [portail Azure](../key-vault//general/quick-create-portal.md), d’[Azure CLI](../key-vault//general/quick-create-cli.md) ou d’[Azure PowerShell](../key-vault//general/quick-create-powershell.md). dans le même abonnement que Recherche cognitive Azure. La **suppression réversible** et la **protection contre le vidage** doivent été activées pour le coffre de clés.
+ [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md). Si vous n’en avez pas, [configurez un nouveau locataire](../active-directory/develop/quickstart-create-new-tenant.md).

Vous devez disposer d’une application de recherche capable de créer l’objet chiffré. Dans ce code, vous allez référencer une clé de coffre de clés et des informations d’inscription auprès d’Active Directory. Ce code peut être une application fonctionnelle ou un code prototype tel que l’[exemple de code C# DotNetHowToEncryptionUsingCMK](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToEncryptionUsingCMK).

> [!TIP]
> Vous pouvez utiliser [Postman ou Visual Studio Code](search-get-started-rest.md), ou encore [Azure PowerShell](./search-get-started-powershell.md), pour appeler les API REST qui créent des index et des mappages de synonymes incluant un paramètre de clé de chiffrement. Il n’existe actuellement pas de prise en charge du portail pour l’ajout de clé aux index ou aux mappages de synonymes.

## <a name="1---enable-key-recovery"></a>1 - Activer la récupération de clé

En raison de la nature du chiffrement avec des clés gérées par le client, personne ne peut récupérer vos données en cas de suppression de votre clé Azure Key Vault. Pour éviter la perte de données résultant de suppressions accidentelles d’Azure Key Vault, vous devez activer la suppression réversible et la protection contre le vidage sur le coffre de clés. La suppression réversible étant activée par défaut, vous ne rencontrerez des problèmes que si vous l’avez désactivée intentionnellement. Par défaut, la protection contre le vidage n’est pas activée, mais elle est requise pour le chiffrement à clé gérée par le client dans Recherche cognitive. Pour plus d’informations, consultez les vues d’ensemble de la [suppression réversible](../key-vault/general/soft-delete-overview.md) et de la [protection contre le vidage](../key-vault/general/soft-delete-overview.md#purge-protection) .

Vous pouvez définir les deux propriétés à l’aide du portail ou en utilisant des commandes PowerShell ou Azure CLI.

### <a name="using-azure-portal"></a>En passant par le portail Azure

1. [Connectez-vous au portail Azure](https://portal.azure.com) et ouvrez la page de présentation de votre coffre de clés.

1. Sur la page **Vue d’ensemble**, sous **Essentials**, activez les options **Suppression réversible** et **Protection contre le vidage**.

### <a name="using-powershell"></a>Utilisation de PowerShell

1. Exécutez `Connect-AzAccount` pour configurer vos informations d’identification Azure.

1. Exécutez la commande suivante pour vous connecter à votre coffre de clés, en remplaçant `<vault_name>` par un nom valide :

   ```powershell
   $resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "<vault_name>").ResourceId
   ```

1. Azure Key Vault est créé avec la suppression réversible activée. Si elle est désactivée sur votre coffre, exécutez la commande suivante :

   ```powershell
   $resource.Properties | Add-Member -MemberType NoteProperty -Name "enableSoftDelete" -Value 'true'
   ```

1. Activer la protection contre le vidage :

   ```powershell
   $resource.Properties | Add-Member -MemberType NoteProperty -Name "enablePurgeProtection" -Value 'true'
   ```

1. Enregistrez vos mises à jour :

   ```powershell
   Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
   ```

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

+ Si [Azure CLI est installé](/cli/azure/install-azure-cli), vous pouvez exécuter la commande suivante pour activer les propriétés requises.

   ```azurecli-interactive
   az keyvault update -n <vault_name> -g <resource_group> --enable-soft-delete --enable-purge-protection
   ```

## <a name="2---create-a-key-in-key-vault"></a>2 – Créer une clé dans Key Vault

Si vous avez déjà une clé dans Azure Key Vault, ignorez cette étape.

1. [Connectez-vous au portail Azure](https://portal.azure.com) et ouvrez la page de présentation de votre coffre de clés.

1. Sélectionnez **Clés** sur la gauche, puis sélectionnez **+ Générer/importer**.

1. Dans le volet **Créer une clé**, dans la liste **Options**, choisissez la méthode que vous voulez utiliser pour créer une clé. Vous pouvez **Générer** une nouvelle clé, **Charger** une clé existante ou utiliser **Restaurer la sauvegarde** pour sélectionner une sauvegarde d’une clé.

1. Entrez un **nom** pour votre clé et sélectionnez éventuellement d’autres propriétés de clé.

1. Sélectionnez **Créer** pour démarrer le déploiement.

1. Notez l’identificateur de la clé : il se compose de l’**URI de la valeur de la clé**, du **nom de la clé** et de la **version de la clé**. Vous en aurez besoin pour définir un index chiffré dans Recherche cognitive Azure.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-key-identifier.png" alt-text="Créer une clé de coffre de clés":::

## <a name="3---register-an-app-in-active-directory"></a>3 – Inscrire une application dans Active Directory

1. Dans [Portail Azure](https://portal.azure.com), recherchez la ressource Azure Active Directory pour votre abonnement.

1. Sur la gauche, sous **Gérer**, sélectionnez **Inscriptions d’applications**, puis sélectionnez **Nouvelle inscription**.

1. Donnez un nom à l’inscription, peut-être un nom similaire au nom de l’application de recherche. Sélectionnez **Inscription**.

1. Une fois l’inscription de l’application créée, copiez l’ID d’application. Vous devrez fournir cette chaîne à votre application. 

   Si vous utilisez le code [DotNetHowToEncryptionUsingCMK](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToEncryptionUsingCMK), collez cette valeur dans le fichier **appsettings.json**.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-application-id.png" alt-text="ID d’application dans la section Essentials":::

1. Ensuite, sélectionnez **Certificats et secrets** sur la gauche.

1. Sélectionnez **Nouveau secret client**. Attribuez un nom d’affichage au secret, puis sélectionnez **Ajouter**.

1. Copiez le secret de l’application. Si vous utilisez l’exemple, collez cette valeur dans le fichier **appsettings.json**.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-application-secret.png" alt-text="Secret de l’application":::

## <a name="4---grant-key-access-permissions"></a>4 - Accorder des autorisations d’accès à la clé

Dans cette étape, vous allez créer une stratégie d’accès dans Key Vault. Cette stratégie donne à l’application que vous avez inscrite auprès d’Active Directory l’autorisation d’utiliser votre clé gérée par le client.

Les autorisations d’accès peuvent être révoquées à tout moment. Une fois révoqué, tout index ou mappage de synonymes d’un service de recherche qui utilise ce coffre de clés sera inutilisable. Une restauration ultérieure des autorisations d’accès au coffre de clés restaurera l’accès à l’index ou au mappage de synonymes. Pour plus d’informations, consultez [Accès sécurisé à un coffre de clés](../key-vault/general/secure-your-key-vault.md).

1. Connectez-vous au portail Azure et ouvrez la page **Vue d’ensemble** de votre coffre de clés. 

1. Sur la gauche, sélectionnez **Stratégies d’accès**, puis **+ Ajouter une stratégie d’accès**.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-add-access-policy.png" alt-text="Ajouter une stratégie d’accès au coffre de clés":::

1. Choisissez **Sélectionner le principal** et sélectionnez l’application que vous avez inscrite auprès d’Active Directory. Vous pouvez effectuer une recherche par nom.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-access-policy-permissions.png" alt-text="Sélectionnez le principal de la stratégie d'accès au coffre de clés":::

1. Dans **Autorisations de clé**, choisissez *Obtenir*, *Désenvelopper la clé* et *Envelopper la clé*.

1. Dans **Autorisations du secret**, sélectionnez *Obtenir*.

1. Dans **Autorisations de certificat**, sélectionnez *Obtenir*.

1. Sélectionnez **Ajouter** puis **Enregistrer**.

> [!Important]
> Le contenu chiffré dans Recherche cognitive Azure est configuré pour utiliser une clé Azure Key Vault spécifique avec une **version** spécifique. Si vous modifiez la clé ou la version, l’index ou le mappage de synonymes doit être mis à jour pour utiliser la nouvelle clé\version **avant** de supprimer la clé\version précédente. Si vous ne le faites pas, l'index ou le mappage de synonymes deviendra inutilisable et vous ne pourrez pas déchiffrer le contenu une fois que l'accès aux clés sera perdu.

<a name="encrypt-content"></a>

## <a name="5---encrypt-content"></a>5 - Chiffrer le contenu

Pour ajouter une clé gérée par le client sur un index, une source de données, un ensemble de compétences ou un mappage de synonymes, vous devez utiliser l’[API REST Recherche](/rest/api/searchservice/) ou un Kit de développement logiciel (SDK). Le portail n’expose pas les mappages de synonymes ou les propriétés de chiffrement. Lorsque vous utilisez des index d’API valides, les sources de données, ensembles de compétences, indexeurs et mappages de synonymes prennent en charge une propriété **encryptionKey** de niveau supérieur.

Cet exemple utilise l’API REST, avec des valeurs pour Azure Key Vault et Azure Active Directory :

```json
{
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

> [!Note]
> Aucune de ces informations sur le coffre de clés n'est considérée comme secrète et peut être facilement récupérée en accédant à la page de la clé Azure Key Vault appropriée dans le portail Azure.

## <a name="example-index-encryption"></a>Exemple : Chiffrement d’index

Créez un index chiffré en procédant de la manière décrite dans [Créer un index (API REST du service Recherche cognitive Azure)](/rest/api/searchservice/create-index). Utilisez la propriété `encryptionKey` pour spécifier la clé de chiffrement à utiliser.
> [!Note]
> Aucune de ces informations sur le coffre de clés n'est considérée comme secrète et peut être facilement récupérée en accédant à la page de la clé Azure Key Vault appropriée dans le portail Azure.

## <a name="rest-examples"></a>Exemples REST

Cette section présente le fichier JSON complet pour un index chiffré et un mappage de synonymes.

### <a name="index-encryption"></a>Chiffrement d’index

Les détails de la création d’un nouvel index via l’API REST se trouvent dans la section [Créer un index (API REST)](/rest/api/searchservice/create-index), où la seule différence ici consiste à spécifier les détails de la clé de chiffrement dans la définition de l’index :

```json
{
 "name": "hotels",
 "fields": [
  {"name": "HotelId", "type": "Edm.String", "key": true, "filterable": true},
  {"name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": true, "facetable": false},
  {"name": "Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.lucene"},
  {"name": "Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene"},
  {"name": "Category", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
  {"name": "Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "sortable": false, "facetable": true},
  {"name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "sortable": true, "facetable": true},
  {"name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true, "facetable": true},
  {"name": "Rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true},
  {"name": "Location", "type": "Edm.GeographyPoint", "filterable": true, "sortable": true},
 ],
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Vous pouvez maintenant envoyer la demande de création d'un index, puis commencer à utiliser l'index normalement.

### <a name="synonym-map-encryption"></a>Chiffrement de mappage de synonymes

Créez une carte de synonymes chiffrée en procédant de la manière décrite dans [Créer un mappage de synonymes (API REST du service Recherche cognitive Azure)](/rest/api/searchservice/create-synonym-map). Utilisez la propriété `encryptionKey` pour spécifier la clé de chiffrement à utiliser.

```json
{
  "name" : "synonymmap1",
  "format" : "solr",
  "synonyms" : "United States, United States of America, USA\n
  Washington, Wash. => WA",
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Vous pouvez maintenant envoyer la demande de création d’un mappage de synonymes, puis commencer à utiliser l'index normalement.

## <a name="example-data-source-encryption"></a>Exemple : Chiffrement de source de données

Créez une source de données chiffrée en procédant de la manière décrite dans [Créer une source de données (API REST du service Recherche cognitive Azure)](/rest/api/searchservice/create-data-source). Utilisez la propriété `encryptionKey` pour spécifier la clé de chiffrement à utiliser.

```json
{
  "name" : "datasource1",
  "type" : "azureblob",
  "credentials" :
  { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=datasource;AccountKey=accountkey;EndpointSuffix=core.windows.net"
  },
  "container" : { "name" : "containername" },
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Vous pouvez maintenant envoyer la demande de création de la source de données, puis commencer à utiliser celle-ci normalement.

## <a name="example-skillset-encryption"></a>Exemple : Chiffrement d’ensemble de compétences

Créez un ensemble de compétences chiffré en procédant de la manière décrite dans [Créer un ensemble de compétences (API REST du service Recherche cognitive Azure)](/rest/api/searchservice/create-skillset). Utilisez la propriété `encryptionKey` pour spécifier la clé de chiffrement à utiliser.

```json
{
  "name" : "datasource1",
  "type" : "azureblob",
  "credentials" :
  { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=datasource;AccountKey=accountkey;EndpointSuffix=core.windows.net"
  },
  "container" : { "name" : "containername" },
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Vous pouvez maintenant envoyer la demande de création de l’ensemble de compétences, puis commencer à utiliser celui-ci normalement.

## <a name="example-indexer-encryption"></a>Exemple : Chiffrement d’indexeur

Créez un indexeur chiffré en procédant de la manière décrite dans [Créer un indexeur (API REST du service Recherche cognitive Azure)](/rest/api/searchservice/create-indexer). Utilisez la propriété `encryptionKey` pour spécifier la clé de chiffrement à utiliser.

```json
{
  "name": "indexer1",
  "dataSourceName": "datasource1",
  "skillsetName": "skillset1",
  "parameters": {
      "configuration": {
          "imageAction": "generateNormalizedImages"
      }
  },
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Vous pouvez maintenant envoyer la demande de création de l’indexeur, puis commencer à utiliser celui-ci normalement.

>[!Important]
> Même si la propriété `encryptionKey` peut être ajoutée aux index de recherche ou mappages de synonymes existants, elle peut être mise à jour en fournissant des valeurs différentes pour chacune des trois informations du coffre de clés (par exemple, en mettant à jour la version de la clé). Lorsque vous passez à une nouvelle clé Key Vault ou à une nouvelle version de clé, tout index de recherche ou mappage de synonymes qui utilise la clé doit d’abord être mis à jour pour utiliser la nouvelle clé/version **avant** de supprimer la clé/version précédente. Si vous ne le faites pas, l'index ou le mappage de synonymes deviendra inutilisable et il ne pourra pas déchiffrer le contenu une fois que l'accès aux clés sera perdu. Cependant, une restauration ultérieure des autorisations d’accès au coffre de clés restaurera l’accès au contenu.

## <a name="simpler-alternative-trusted-service"></a>Alternative plus simple : Service approuvé

En fonction de la configuration du locataire et des exigences d’authentification, vous pourrez peut-être implémenter une approche plus simple pour accéder à une clé de coffre de clés. Au lieu de créer et d’utiliser une application Active Directory, vous pouvez faire d’un service de recherche un service approuvé en lui donnant une identité managée par le système. Vous devez ensuite utiliser le service de recherche approuvé comme principe de sécurité, plutôt qu’une application inscrite auprès d’AD, pour accéder à la clé de coffre de clés.

Cette approche vous permet d’omettre les étapes relatives à l’inscription et aux secrets de l’application, et simplifie une définition de clé de chiffrement aux seuls composants du coffre de clés (URI, nom de coffre, version de clé).

En général, une identité managée permet à votre service de recherche de s’authentifier auprès d’Azure Key Vault, sans stocker les informations d’identification (ApplicationID ou ApplicationSecret) dans le code. Le cycle de vie de ce type d'identité managée est lié au cycle de vie de votre service de recherche, qui ne peut avoir qu'une seule identité managée. Pour plus d’informations sur le fonctionnement des identités managées, consultez [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md)

1. Faites de votre service de recherche un service approuvé.
   ![Turn on system assigned managed identity](./media/search-managed-identities/turn-on-system-assigned-identity.png "Activer l’identité managée affectée par le système")

1. Quand vous configurez une stratégie d’accès dans Azure Key Vault, choisissez le service de recherche approuvé comme principe (au lieu de l’application inscrite auprès d’AD). Attribuez les mêmes autorisations (GET multiples, WRAP, UNWRAP) comme indiqué dans l’étape d’attribution des autorisations de la clé d’accès.

1. Utilisez une construction simplifiée de `encryptionKey` qui omet les propriétés Active Directory.

    ```json
    {
      "encryptionKey": {
        "keyVaultUri": "https://demokeyvault.vault.azure.net",
        "keyVaultKeyName": "myEncryptionKey",
        "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660"
      }
    }
    ```

Les conditions qui vous empêchent d’adopter cette approche simplifiée sont les suivantes :

+ Vous ne pouvez pas accorder directement à votre service de recherche les droits d'accès au coffre de clés (par exemple, si le service de recherche se trouve dans un autre locataire Active Directory qu’Azure Key Vault).

+ Un seul service de recherche est nécessaire pour héberger plusieurs index/mappages de synonymes chiffrés, chacun utilisant une clé différente d'un coffre de clés différent, où chaque coffre de clés doit utiliser **une identité différente**  pour l'authentification. Étant donné qu’un service de recherche ne peut avoir qu’une seule identité managée, une configuration nécessitant plusieurs identités disqualifie l’approche simplifiée pour votre scénario.  

## <a name="work-with-encrypted-content"></a>Utiliser des colonnes chiffrées

Avec le chiffrement à l’aide de clés gérées par le client, vous remarquerez une latence pour l’indexation et les requêtes en raison du travail de chiffrement/déchiffrement supplémentaire. Le service Recherche cognitive Azure ne journalise pas l’activité de chiffrement, mais vous pouvez surveiller l’accès aux clés par le biais de la journalisation du coffre de clés. Nous vous recommandons d’[activer la journalisation](../key-vault/general/logging.md) dans le cadre de la configuration du coffre de clés.

Une rotation de clés est supposée se produire au fil du temps. Chaque fois que vous voulez opérer une rotation de clés, il est important de suivre cette séquence :

1. [Déterminez la clé utilisée par un index ou un mappage de synonyme](search-security-get-encryption-keys.md).
1. [Créez une clé dans le coffre de clés](../key-vault/keys/quick-create-portal.md), mais gardez la clé d’origine disponible.
1. [Mettez à jour les propriétés encryptionKey](/rest/api/searchservice/update-index) sur un index ou un mappage de synonymes pour utiliser les nouvelles valeurs. Seuls des objets créés à l’origine avec cette propriété peuvent être mis à jour pour utiliser une autre valeur.
1. Désactivez ou supprimez la clé précédente dans le coffre de clés. Surveillez l’accès à la clé pour vérifier que la nouvelle clé est utilisée.

Pour des raisons de performances, le service de recherche met en cache la clé pendant plusieurs heures. Si vous désactivez ou supprimez la clé sans en fournir de nouvelle, les requêtes continuent de fonctionner sur une base temporaire jusqu’à ce que le cache expire. Toutefois, une fois que le service de recherche ne peut pas déchiffrer le contenu, vous obtenez le message suivant : « Accès interdit. La clé de requête utilisée a peut-être été révoquée. Réessayez. » 

## <a name="next-steps"></a>Étapes suivantes

Si vous n’êtes pas familiarisé avec l’architecture de sécurité Azure, passez en revue la [documentation sur la sécurité Azure](../security/index.yml), et en particulier cet article :

> [!div class="nextstepaction"]
> [Chiffrement des données Azure au repos](../security/fundamentals/encryption-atrest.md)