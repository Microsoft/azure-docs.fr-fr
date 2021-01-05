---
title: À propos des clés - Azure Key Vault
description: Vue d’ensemble de l’interface REST Azure Key Vault et des détails de développement sur les clés.
services: key-vault
author: amitbapat
manager: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: overview
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 2ae7b28d5e9e7a520ee8cbd090b6681d5ad7015a
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422754"
---
# <a name="about-keys"></a>À propos des clés

Azure Key Vault fournit deux types de ressources pour stocker et gérer les clés de chiffrement :

|Type de ressource|Méthodes de protection des clés|URL de base du point de terminaison de plan de données|
|--|--|--|
| **Coffres** | Protégés par logiciel<br/><br/>et<br/><br/>Protégés par HSM (avec la référence SKU Premium)</li></ul> | https://{nom-coffre}.vault.azure.net |
| **Pools HSM managés** | Protégés par HSM | https://{nom-hsm}.managedhsm.azure.net |
||||

- **Coffres** : ils fournissent une solution de gestion des clés adaptée aux scénarios d’applications cloud les plus courants. Cette solution est peu coûteuse, facile à déployer, multilocataire, résiliente aux zones (si disponible) et à haute disponibilité.
- **HSM managés** : ils offrent une solution monolocataire, résiliente aux zones (si disponible) et à haute disponibilité pour le stockage et la gestion de vos clés de chiffrement. Ils sont particulièrement adaptés aux scénarios d’usage et d’applications qui utilisent des clés de grande valeur. Ils aident également à remplir les exigences les plus strictes en matière de sécurité, de conformité et de réglementation. 

> [!NOTE]
> Les coffres vous permettent également de stocker et de gérer plusieurs types d’objets tels que les secrets, les certificats et les clés de compte de stockage, en plus des clés de chiffrement.

Les clés de chiffrement dans Key Vault sont représentées en tant qu’objets de clé web JSON [JWK]. Les spécifications JSON (JavaScript Object Notation) et JOSE (JavaScript Object Signing and Encryption) sont :

-   [Clé web JSON (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key)  
-   [Chiffrement web JSON (JWE)](http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption)  
-   [Algorithmes web JSON (JWA)](http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms)  
-   [Signature web JSON (JWS)](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature) 

Les spécifications JWK/JWA de base sont également étendues pour rendre les types de clés uniques dans les implémentations de coffres Azure Key Vault et de HSM managés. 

Les clés protégées par HSM (aussi appelées clés HSM) sont traitées dans un HSM (module de sécurité matériel) et restent toujours dans les limites de la protection par HSM. 

- Les coffres utilisent des HSM conformes à la norme **FIPS 140-2 niveau 2** pour protéger les clés HSM dans une infrastructure back-end HSM partagée. 
- Les pools HSM managés utilisent des HSM conformes à la norme **FIPS 140-2 niveau 3** pour protéger vos clés. Chaque pool HSM est une instance monolocataire isolée qui a son propre [domaine de sécurité](../managed-hsm/security-domain.md), offrant ainsi une isolation de chiffrement complète de tous les autres pools HSM partageant la même infrastructure matérielle.

Ces clés sont protégées dans des pools HSM monolocataires. Vous pouvez importer une clé RSA, EC et symétrique sous forme logicielle ou en l’exportant à partir d’un matériel HSM pris en charge. Vous pouvez aussi générer des clés dans les pools HSM. Lorsque vous importez des clés HSM en utilisant la méthode décrite dans la [spécification BYOK (Bring Your Own Key)](../keys/byok-specification.md), vous sécurisez les éléments de clé de transport dans des pools HSM managés. 

Pour plus d’informations sur les frontières géographiques, consultez [Centre de gestion de la confidentialité Microsoft Azure](https://azure.microsoft.com/support/trust-center/privacy/)

## <a name="key-types-and-protection-methods"></a>Types de clés et méthodes de protection

Key Vault prend en charge les clés RSA, EC et symétriques. 

### <a name="hsm-protected-keys"></a>Clés protégées par HSM

|Type de clé|Coffres (SKU Premium uniquement)|Pools de HSM managés|
|--|--|--|--|
**EC-HSM** : clé Elliptic Curve|HSM FIPS 140-2 niveau 2|HSM FIPS 140-2 niveau 3
**RSA-HSM** : clé RSA|HSM FIPS 140-2 niveau 2|HSM FIPS 140-2 niveau 3
**oct-HSM** : Symétrique|Non pris en charge|HSM FIPS 140-2 niveau 3
||||

### <a name="software-protected-keys"></a>Clés protégées par logiciel

|Type de clé|Coffres|Pools de HSM managés|
|--|--|--|--|
**RSA** : clé RSA « protégée par logiciel »|FIPS 140-2 niveau 1|Non pris en charge
**EC** : clé Elliptic Curve « protégée par logiciel »|FIPS 140-2 niveau 1|Non pris en charge
||||

Pour plus d’informations sur chaque type de clé, mais aussi sur les algorithmes, les opérations, les attributs et les étiquettes, consultez [Types de clés, algorithmes et opérations](about-keys-details.md).

## <a name="next-steps"></a>Étapes suivantes
- [À propos de Key Vault](../general/overview.md)
- [À propos de HSM managé](../managed-hsm/overview.md)
- [À propos des secrets](../secrets/about-secrets.md)
- [À propos des certificats](../certificates/about-certificates.md)
- [Vue d’ensemble de l’API REST Azure Key Vault](../general/about-keys-secrets-certificates.md)
- [Authentification, requêtes et réponses](../general/authentication-requests-and-responses.md)
- [Guide du développeur Key Vault](../general/developers-guide.md)