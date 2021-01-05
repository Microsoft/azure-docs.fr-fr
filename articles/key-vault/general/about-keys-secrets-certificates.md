---
title: Vue d’ensemble des clés, secrets et certificats Azure Key Vault
description: Vue d’ensemble de l’interface REST Azure Key Vault et des détails de développement sur les clés, les secrets et les certificats.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: overview
ms.date: 04/17/2020
ms.author: mbaldwin
ms.openlocfilehash: e0a45bde32fed651c4b38d203b3c75a6d928e7c5
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327081"
---
# <a name="azure-key-vault-keys-secrets-and-certificates-overview"></a>Vue d’ensemble des clés, secrets et certificats Azure Key Vault

Azure Key Vault permet aux applications Microsoft Azure et à leurs utilisateurs de stocker et d’utiliser plusieurs types de données de secret/clé. Le fournisseur de ressources Key Vault prend en charge deux types de ressources : les coffres et les HSM managés.

## <a name="dns-suffixes-for-base-url"></a>Suffixes DNS pour l’URL de base
 Le tableau ci-dessous indique le suffixe DNS utilisé dans l’URL de base du point de terminaison de plan de données pour les coffres et les pools de HSM managés dans différents environnements cloud.

Environnement cloud | Suffixe DNS pour les coffres | Suffixe DNS pour les HSM managés
---|---|---
Cloud Azure | .vault.azure.net | .managedhsm.azure.net
Cloud Azure Chine | .vault.azure.cn | Non pris en charge
Azure US Government | .vault.usgovcloudapi.net | Non pris en charge
Cloud Azure – Allemagne | .vault.microsoftazure.de | Non pris en charge
|||


## <a name="object-types"></a>Types d’objets
 Le tableau ci-dessous présente les types d’objets et leurs suffixes utilisés dans l’URL de base.

Type d'objet|Suffixe d'URL|Coffres|Pools de HSM managés
--|--|--|--
**Clés de chiffrement**||
Clés protégées par HSM|/keys|Prise en charge|Prise en charge
Clés protégées par logiciel|/keys|Prise en charge|Non pris en charge
**Autres types d’objets**||
Secrets|/secrets|Prise en charge|Non pris en charge
Certificats|/certificates|Prise en charge|Non pris en charge
Clés de compte de stockage|/storageaccount|Prise en charge|Non pris en charge
|||
- **Clés de chiffrement** : Prend en charge plusieurs algorithmes et types de clés, et permet l’utilisation de clés protégées par logiciel et par HSM. Pour plus d’informations sur les clés, consultez [À propos des clés](../keys/about-keys.md).
- **Secrets** : Fournit un stockage sécurisé des secrets, comme les mots de passe et les chaînes de connexion de base de données. Pour plus d’informations, consultez [À propos des secrets](../secrets/about-secrets.md).
- **Certificats** : Prend en charge les certificats, qui sont basés sur des clés et des secrets, et ajoute une fonctionnalité de renouvellement automatique. Pour plus d’informations, consultez [À propos des certificats](../certificates/about-certificates.md).
- **Clés de compte Stockage Azure** : Peut gérer pour vous les clés d’un compte Stockage Azure. En interne, Key Vault peut lister (synchroniser) les clés avec un compte Stockage Azure et regénérer (faire tourner) régulièrement les clés. Pour plus d’informations, consultez [Gérer les clés de compte de stockage avec Key Vault](../secrets/overview-storage-keys.md).

Pour plus d’informations générales sur Key Vault, consultez [À propos d’Azure Key Vault](overview.md). Pour plus d’informations sur les pools de HSM managés, consultez [Qu’est-ce qu’Azure Key Vault Managed HSM ?](../managed-hsm/overview.md)


## <a name="data-types"></a>Types de données

Reportez-vous aux spécifications JOSE pour les types de données appropriés pour les clés, le chiffrement et la signature.  

-   **algorithm** : algorithme pris en charge pour une opération sur les clés, RSA1_5 par exemple  
-   **ciphertext-value** : octets de texte de chiffrement, codés avec Base64URL  
-   **digest-value** : sortie d’un algorithme de hachage, codée avec Base64URL  
-   **key-type** : un des types de clés pris en charge, par exemple RSA (Rivest-Shamir-Adleman).  
-   **plaintext-value** : octets de texte en clair, codés avec Base64URL  
-   **signature-value** : sortie d’un algorithme de signature, codée avec Base64URL  
-   **base64URL** : valeur binaire codée Base64URL [RFC4648]  
-   **boolean** : true (vrai) ou false (faux)  
-   **Identity** : identité d’Azure Active Directory (AAD).  
-   **IntDate** : valeur décimale JSON représentant le nombre de secondes entre 1970-01-01T0:0:0Z UTC et la date/heure UTC spécifiée. Consultez la RFC3339 pour plus d’informations sur les dates/heures en général, et sur UTC en particulier.  

## <a name="objects-identifiers-and-versioning"></a>Objets, identificateurs et gestion de versions

Les objets stockés dans Key Vault sont versionnés chaque fois qu’une nouvelle instance d’un objet est créée. Chaque version se voit assigner un identificateur unique et une URL. Quand un objet est créé, il se voit attribuer un identificateur de version unique et est marqué comme version actuelle de l’objet. La création d’une nouvelle instance portant le même nom d’objet attribue au nouvel objet un identificateur de version unique, ce qui en fait la version actuelle.  

Les objets dans Key Vault peuvent être adressés en spécifiant une version ou en omettant la version pour les opérations sur la version actuelle de l’objet. Par exemple, pour une clé nommée `MasterKey`, l’exécution d’opérations sans préciser de version amène le système à utiliser la dernière version disponible. L’exécution d’opérations avec l’identificateur spécifique à la version amène le système à utiliser cette version spécifique de l’objet.  

### <a name="vault-name-and-object-name"></a>Nom de coffre et nom d’objet
Les objets sont identifiés de façon unique dans Key Vault avec une URL. Il n’y a pas deux objets avec la même URL dans le système, quel que soit l’emplacement géographique. L’URL complète d’un objet est appelée identificateur d’objet. L’URL est constituée d’un préfixe qui identifie le coffre de clés, du type d’objet, du nom d’objet fourni par l’utilisateur et d’une version d’objet. Le nom d’objet n’est pas sensible à la casse et est non modifiable. Les identificateurs qui n’incluent pas la version d’objet sont appelés des identificateurs de base.  

Pour plus d’informations, consultez [Authentification, requêtes et réponses](authentication-requests-and-responses.md)

Un identificateur d’objet a le format général suivant (selon le type de conteneur) :  

- **Pour les coffres** : `https://{vault-name}.vault.azure.net/{object-type}/{object-name}/{object-version}`  

- **Pour les pools Managed HSM** : `https://{hsm-name}.managedhsm.azure.net/{object-type}/{object-name}/{object-version}`  

> [!NOTE]
> Pour connaître les types d’objets pris en charge par chaque type de conteneur, consultez [Types d’objets](#object-types).

Où :  

| Élément | Description |  
|-|-|  
|`vault-name` ou `hsm-name`|Nom d’un coffre ou d’un pool Managed HSM dans le service Microsoft Azure Key Vault.<br /><br />Les noms de coffre et de pool Managed HSM sont choisis par l’utilisateur et sont globalement uniques.<br /><br />Le nom d’un coffre ou d’un pool Managed HSM doit être une chaîne de 3 à 24 caractères, qui peut contenir uniquement des chiffres, des lettres et des tirets (0-9, a-z, A-Z et -).|  
|`object-type`|Type de l’objet : « keys », « secrets » ou « certificates ».|  
|`object-name`|Un `object-name` est un nom fourni par l’utilisateur et doit être unique dans un coffre de clés. Le nom doit être une chaîne comprise entre 1 et 127 caractères, commençant par une lettre et qui doit contenir uniquement des chiffres, des lettres et des tirets (0-9, a-z, A-Z et -).|  
|`object-version`|Un `object-version` est un identificateur de chaîne de 32 caractères généré par le système qui peut être utilisé pour une version unique d’un objet.|  

## <a name="next-steps"></a>Étapes suivantes

- [À propos des clés](../keys/about-keys.md)
- [À propos des secrets](../secrets/about-secrets.md)
- [À propos des certificats](../certificates/about-certificates.md)
- [Authentification, requêtes et réponses](../general/authentication-requests-and-responses.md)
- [Guide du développeur Key Vault](../general/developers-guide.md)
