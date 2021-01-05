---
title: Notes de publication de l’API Key Vault .NET 2.x | Microsoft Docs
description: Découvrez comment mettre à jour des applications écrites pour des versions antérieures de Azure Key Vault pour utiliser la version 2.0 de la bibliothèque Azure Key Vault pour C# et .NET.
services: key-vault
author: msmbaldwin
manager: rkarlin
editor: bryanla
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 05/02/2017
ms.author: mbaldwin
ms.openlocfilehash: 018570019b306dced76760fefa4441ee7d86ad2a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013953"
---
# <a name="azure-key-vault-net-20---release-notes-and-migration-guide"></a>Azure Key Vault .NET 2.0 - Notes de publication et guide de migration
Les informations suivantes vous permettent d’effectuer la migration vers la version 2.0 de la bibliothèque Azure Key Vault pour C# et .NET.  Les applications écrites pour les versions antérieures doivent être mises à jour pour prendre en charge la version la plus récente.  Ces modifications sont nécessaires pour prendre en charge des fonctionnalités nouvelles et améliorées, telles que les **certificats Key Vault**.

## <a name="key-vault-certificates"></a>Certificats Key Vault

Les certificats Key Vault gèrent les certificats x509 et prennent en charge les comportements suivants :  

* Création de certificats par le biais d’un processus de création Key Vault ou importation de certificat existant. Cela concerne à la fois les certificats auto-signés et ceux générés par une autorité de certification.
* Stockage en toute sécurité et gestion du stockage de certificats x509 sans interaction à l’aide d’éléments de clé privée.  
* Définition de stratégies qui font en sorte que Key Vault gère le cycle de vie de certificat.  
* Spécification d’informations de contact pour les événements de cycle de vie, tels que les avertissements d’expiration et les notifications de renouvellement.  
* Renouvellement automatique des certificats avec des émetteurs sélectionnés (autorités de certification et fournisseurs de certificats X509 partenaires de Key Vault). * Prise en charge des certificats émis par d’autres fournisseurs et autorités de certification (non partenaires) (renouvellement automatique non pris en charge).  

## <a name="net-support"></a>Prise en charge de .NET

* **.NET 4.0** n’est pas pris en charge par la version 2.0 de la bibliothèque .NET d’Azure Key Vault
* **.NET Framework 4.5.2** n’est pas pris en charge par la version 2.0 de la bibliothèque .NET d’Azure Key Vault
* **.NET Standard 1.4** n’est pas pris en charge par la version 2.0 de la bibliothèque .NET d’Azure Key Vault

## <a name="namespaces"></a>Espaces de noms

* L’espace de noms des **modèles** **Microsoft.Azure.KeyVault** est remplacé par **Microsoft.Azure.KeyVault.Models**.
* L’espace de noms **Microsoft.Azure.KeyVault.Internal** est supprimé.
* Les espaces de noms de dépendances Azure SDK suivants ont changé : 

    - **Hyak.Common** est désormais **Microsoft.Rest**.
    - **Hyak.Common.Internals** est désormais **Microsoft.Rest.Serialization**.

## <a name="type-changes"></a>Modifications des types

* *Secret* est remplacé par *SecretBundle*
* *Dictionary* est remplacé par *IDictionary*
* *List\<T>, chaîne []* est remplacé par *IList\<T>*
* *NextList* est remplacé par *NextPageLink*

## <a name="return-types"></a>Types de retour

* **KeyList** et **SecretList** retournent désormais *IPage\<T>* au lieu de *ListKeysResponseMessage*
* Le résultat généré **BackupKeyAsync** retourne désormais *BackupKeyResult*, qui contient *Value* (objet blob de sauvegarde). Auparavant, la méthode était encapsulée et ne retournait que la valeur.

## <a name="exceptions"></a>Exceptions

* *KeyVaultClientException* est remplacé par *KeyVaultErrorException*
* L’erreur de service *exception.Error* est remplacée par *exception.Body.Error.Message*.
* Les informations supplémentaires du message d’erreur pour **[JsonExtensionData]** ont été supprimées.

## <a name="constructors"></a>Constructeurs

* Au lieu d’accepter un *HttpClient* comme un argument de constructeur, le constructeur accepte seulement *HttpClientHandler* ou *DelegatingHandler[]* .

## <a name="downloaded-packages"></a>Packages téléchargés

Quand un client traite une dépendance Key Vault, les packages suivants sont téléchargés :

### <a name="previous-package-list"></a>Liste des packages précédents

* `package id="Hyak.Common" version="1.0.2" targetFramework="net45"`
* `package id="Microsoft.Azure.Common" version="2.0.4" targetFramework="net45"`
* `package id="Microsoft.Azure.Common.Dependencies" version="1.0.0" targetFramework="net45"`
* `package id="Microsoft.Azure.KeyVault" version="1.0.0" targetFramework="net45"`
* `package id="Microsoft.Bcl" version="1.1.9" targetFramework="net45"`
* `package id="Microsoft.Bcl.Async" version="1.0.168" targetFramework="net45"`
* `package id="Microsoft.Bcl.Build" version="1.0.14" targetFramework="net45"`
* `package id="Microsoft.Net.Http" version="2.2.22" targetFramework="net45"`

### <a name="current-package-list"></a>Liste des packages actuels

* `package id="Microsoft.Azure.KeyVault" version="2.0.0-preview" targetFramework="net45"`
* `package id="Microsoft.Rest.ClientRuntime" version="2.2.0" targetFramework="net45"`
* `package id="Microsoft.Rest.ClientRuntime.Azure" version="3.2.0" targetFramework="net45"`

## <a name="class-changes"></a>Modifications de classe

* La classe **UnixEpoch** a été supprimée.
* La classe **Base64UrlConverter** a été renommée **Base64UrlJsonConverter**.

## <a name="other-changes"></a>Autres modifications

* La prise en charge de la configuration de la stratégie de nouvelle tentative d’opération KV sur les défaillances passagères a été ajoutée à cette version de l’API.

## <a name="microsoftazuremanagementkeyvault-nuget"></a>Microsoft.Azure.Management.KeyVault NuGet

* Pour les opérations qui retournaient *vault*, le type de retour était une classe contenant une propriété **Vault**. Le type de retour est maintenant *Vault*.
* *PermissionsToKeys* et *PermissionsToSecrets* sont à présent *Permissions.Keys* et *Permissions.Secrets*
* Certaines modifications des types de retour s’appliquent également à control-plane.

## <a name="microsoftazurekeyvaultextensions-nuget"></a>Microsoft.Azure.KeyVault.Extensions NuGet

* Le package est interrompu jusqu’à **Microsoft.Azure.KeyVault.Extensions** et **Microsoft.Azure.KeyVault.Cryptography** pour les opérations de chiffrement.

