---
title: Types de clés, algorithmes et opérations – Azure Key Vault
description: Types de clés, algorithmes et opérations pris en charge (détails).
services: key-vault
author: amitbapat
manager: msmbaldwin
ms.service: key-vault
ms.subservice: keys
ms.topic: conceptual
ms.date: 10/22/2020
ms.author: ambapat
ms.openlocfilehash: b483ffc480f9ad750f8d9901d6bec382db2378c2
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368917"
---
# <a name="key-types-algorithms-and-operations"></a>Types de clés, algorithmes et opérations

Key Vault prend en charge deux types de ressources : les coffres et les HSM managés. Les deux types de ressources prennent en charge différentes clés de chiffrement. Pour afficher un résumé des types de clé pris en charge, ainsi que des types de protections par type de ressource, consultez [À propos des clés](about-keys.md).

Le tableau suivant présente un résumé des types de clés et des algorithmes pris en charge.

|Types de clés/tailles/courbes| Chiffrer/Déchiffrer<br>(Inclure/Ne pas inclure dans un wrapper) | Signer/Vérifier | 
| --- | --- | --- |
|EC-P256, EC-P256K, EC-P384, EC-521|N/D|ES256<br>ES256K<br>ES384<br>ES512|
|RSA 2K, 3K, 4K| RSA1_5<br>RSA-OAEP<br>RSA-OAEP-256|PS256<br>PS384<br>PS512<br>RS256<br>RS384<br>RS512<br>RSNULL| 
|AES 128 bits, 256 bits| AES-KW<br>AES-GCM<br>AES-CBC| N/D| 
|||

##  <a name="ec-algorithms"></a>Algorithmes EC
 Les identificateurs d’algorithme suivants sont pris en charge avec les clés EC-HSM

### <a name="curve-types"></a>Types de courbe

-   **P-256** - La courbe NIST P-256, définie sur [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).
-   **P-256_K** : courbe SEC SECP256K1, définie dans [SEC 2: Recommended Elliptic Curve Domain Parameters](https://www.secg.org/sec2-v2.pdf).
-   **P-384** - La courbe NIST P-384, définie sur [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).
-   **P-521** - La courbe NIST P-521, définie sur [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).

### <a name="signverify"></a>SIGN/VERIFY

-   **ES256** : ECDSA pour codes de hachage SHA-256 et clés créées avec la courbe P-256. Cet algorithme est décrit dans [RFC7518](https://tools.ietf.org/html/rfc7518).
-   **ES256K** : ECDSA pour codes de hachage SHA-256 et clés créées avec la courbe P-256K. Cet algorithme est en attente de normalisation.
-   **ES384** : ECDSA pour codes de hachage SHA-384 et clés créées avec la courbe P-384. Cet algorithme est décrit dans [RFC7518](https://tools.ietf.org/html/rfc7518).
-   **ES512** : ECDSA pour codes de hachage SHA-512 et clés créées avec la courbe P-521. Cet algorithme est décrit dans [RFC7518](https://tools.ietf.org/html/rfc7518).

##  <a name="rsa-algorithms"></a>Algorithmes RSA  
 Les identificateurs d’algorithme suivants sont pris en charge avec les clés RSA et RSA-HSM  

### <a name="wrapkeyunwrapkey-encryptdecrypt"></a>WRAPKEY/UNWRAPKEY, ENCRYPT/DECRYPT

-   **RSA1_5** : chiffrement à clé RSAES-PKCS1-V1_5 [RFC3447]  
-   **RSA-OAEP** : RSAES utilisant OAEP (Optimal Asymmetric Encryption Padding) [RFC3447], avec les paramètres par défaut spécifiés par RFC 3447, section A.2.1. Ces paramètres par défaut utilisent une fonction de hachage de SHA-1 et une fonction de génération de masque de MGF1 avec SHA-1.  
-  **RSA-OAEP-256** : RSAES utilisant OAEP (Optimal Asymmetric Encryption Padding) avec une fonction de hachage SHA-256 et une fonction de génération de masque MGF1 avec SHA-256

### <a name="signverify"></a>SIGN/VERIFY

-   **PS256** : RSASSA-PSS utilisant SHA-256 et MGF1 avec SHA-256, comme décrit dans le document [RFC7518](https://tools.ietf.org/html/rfc7518).
-   **PS384** : RSASSA-PSS utilisant SHA-384 et MGF1 avec SHA-384, comme décrit dans le document [RFC7518](https://tools.ietf.org/html/rfc7518).
-   **PS512** : RSASSA-PSS utilisant SHA-512 et MGF1 avec SHA-512, comme décrit dans le document [RFC7518](https://tools.ietf.org/html/rfc7518).
-   **RS256** : RSASSA-PKCS-v1_5 utilisant SHA-256. La valeur de synthèse fournie par l’application doit être calculée à l’aide de SHA-256 et doit être d’une longueur de 32 octets.  
-   **RS384** : RSASSA-PKCS-v1_5 utilisant SHA-384. La valeur de synthèse fournie par l’application doit être calculée à l’aide de SHA-384 et doit être d’une longueur de 48 octets.  
-   **RS512** : RSASSA-PKCS-v1_5 utilisant SHA-512. La valeur de synthèse fournie par l’application doit être calculée à l’aide de SHA-512 et doit être d’une longueur de 64 octets.  
-   **RSNULL** : consultez [RFC2437](https://tools.ietf.org/html/rfc2437), un cas d’usage spécial permettant certains scénarios TLS.  

##  <a name="symmetric-key-algorithms"></a>Algorithmes de clé symétrique
- **AES-KW** : AES Key Wrap ([RFC3394](https://tools.ietf.org/html/rfc3394)).
- **AES-GCM** : chiffrement AES en mode Galois/Compteur ([NIST SP 800-38d](https://csrc.nist.gov/publications/sp800)).
- **AES-CBC** : chiffrement AES en mode d’opération Enchaînement des blocs ([NIST SP 800-38a](https://csrc.nist.gov/publications/sp800)).

> [!NOTE] 
> L’implémentation AES-GCM actuelle et les API correspondantes sont en phase d’expérimentation. L’implémentation et les API pourront être substantiellement différentes dans les itérations futures. 

##  <a name="key-operations"></a>Opérations sur les clés

Le HSM managé prend en charge les opérations suivantes sur les clés :  

-   **Créer** : permet à un client de créer une clé dans Key Vault. La valeur de la clé est générée par le coffre de clés et stockée, mais n’est pas communiquée au client. Les clés asymétriques peuvent être créées dans un coffre de clés.  
-   **Importer** : permet à un client d’importer une clé existante dans Key Vault. Des clés asymétriques peuvent être importées dans un coffre de clés selon diverses méthodes d’empaquetage dans un construct JWK. 
-   **Mettre à jour** : permet à un client disposant des autorisations suffisantes de modifier les métadonnées (attributs de clé) associées à une clé précédemment stockée dans Key Vault.  
-   **Supprimer** : permet à un client disposant des autorisations suffisantes de supprimer une clé dans Key Vault.  
-   **Lister** : permet à un client de lister toutes les clés d’un coffre de clés donné.  
-   **Lister les versions** : permet à un client de lister toutes les versions d’une clé donnée dans un coffre de clés donné.  
-   **Obtenir** : permet à un client de récupérer les parties publiques d’une clé donnée dans un coffre de clés.  
-   **Sauvegarder** : permet d’exporter une clé sous une forme protégée.  
-   **Restaurer** : permet d’importer une clé précédemment sauvegardée.  

Pour plus d’informations, voir [Informations de référence sur les opérations liées aux clés dans l’API REST Key Vault](/rest/api/keyvault).  

Lorsqu’une clé a été créée dans un coffre de clés, les opérations de chiffrement suivantes peuvent être exécutées à l’aide de la clé :  

-   **Signer et vérifier** : cette opération vise à « signer le hachage » ou à « vérifier le hachage », car Key Vault ne prend pas en charge le hachage du contenu lors la création de la signature. Les applications doivent hacher les données à signer localement puis demander à Key Vault de signer le hachage. La vérification des hachages signés est prise en charge par souci pratique pour les applications qui n’ont peut-être pas accès au matériel de clé [publique]. Pour optimiser les performances des applications, les opérations VERIFY doivent être effectuées localement.  
-   **Chiffrement / encapsulation de clé** : une clé stockée dans Key Vault peut être utilisée pour protéger une autre clé, généralement une clé de chiffrement symétrique de contenu (CEK). Lorsque la clé dans Key Vault est asymétrique, le chiffrement de clé est utilisé. Par exemple, les opérations WRAPKEY/UNWRAPKEY et RSA-OAEP sont équivalentes à ENCRYPT/DECRYPT. Lorsque la clé dans Key Vault est symétrique, le wrapping de clé est utilisé. Par exemple, AES-KW. L’opération WRAPKEY est prise en charge par souci pratique pour les applications qui n’ont peut-être pas accès au matériel de clé [publique]. Pour optimiser les performances des applications, les opérations WRAPKEY doivent être effectuées localement.  
-   **Chiffrer et déchiffrer** : une clé stockée dans Key Vault peut être utilisée pour chiffrer ou déchiffrer un bloc de données. La taille du bloc est déterminée par le type de clé et l’algorithme de chiffrement sélectionné. L’opération Encrypt est fournie par souci pratique pour les applications qui n’ont peut-être pas accès au matériel de clé [publique]. Pour optimiser les performances des applications, les opérations ENCRYPT doivent être effectuées localement.  

Alors que les opérations WRAPKEY/UNWRAPKEY utilisant des clés asymétriques peuvent sembler superflues (car elles sont équivalentes à ENCRYPT/DECRYPT), l’utilisation d’opérations distinctes est importante. La distinction fournit une séparation de la sémantique et des autorisations de ces opérations, ainsi qu’une cohérence quand d’autres types de clés sont pris en charge par le service.  

Key Vault ne prend pas en charge les opérations EXPORT. Lorsqu’une clé est provisionnée dans le système, elle ne peut pas être extraite et son matériel de clé ne peut pas être modifié. Toutefois, les utilisateurs de Key Vault peuvent avoir besoin de leur clé pour d’autres utilisations, par exemple après sa suppression. Dans ce cas, ils peuvent utiliser les opérations BACKUP et RESTORE pour exporter/importer la clé dans un formulaire protégé. Les clés créées par l’opération BACKUP ne peuvent pas être utilisées en dehors de Key Vault. L’opération IMPORT peut également être utilisée sur plusieurs instances de Key Vault.  

Les utilisateurs peuvent limiter les opérations de chiffrement prises en charge par Key Vault, par clé, à l’aide de la propriété key_ops de l’objet JWK.  

Pour plus d’informations sur les objets JWK, consultez [Clé web JSON (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41).  

## <a name="key-attributes"></a>Attributs de clé

Les attributs suivants peuvent être spécifiés en plus du matériel de clé. Dans une requête JSON, le mot clé attributes et les accolades, « { » « } », sont requis même si aucun attribut n’est spécifié.  

- *enabled* : booléen, facultatif, **true** par défaut. Spécifie si la clé est activée et peut être utilisée pour des opérations de chiffrement. L’attribut *enabled* est utilisé avec *nbf* et *exp*. Lorsqu’une opération se produit entre *nbf* et *exp*, elle n’est autorisée que si *enabled* est défini sur **true**. Les opérations en dehors de la fenêtre *nbf* / *exp* sont automatiquement interdites, à l’exception de certains types d’opérations dans des [conditions particulières](#date-time-controlled-operations).
- *nbf* : IntDate, facultatif, « now » par défaut. L’attribut *nbf* (not before ou pas avant) identifie l’heure avant laquelle la clé NE DOIT PAS être utilisée pour des opérations de chiffrement, à l’exception de certains types d’opérations dans des [conditions particulières](#date-time-controlled-operations). Le traitement de l’attribut *nbf* requiert que la date/heure actuelle SOIT postérieure ou égale à la date/heure « not-before » (pas avant) indiquée dans l’attribut *nbf*. Key Vault PEUT prévoir une légère marge, normalement pas plus de quelques minutes, pour prendre en compte la variation d’horloge. Sa valeur DOIT être un nombre contenant une valeur IntDate.  
- *exp* : IntDate, facultatif, « forever » par défaut. L’attribut *exp* (heure d’expiration) identifie l’heure à ou après laquelle la clé NE DOIT PAS être utilisée pour une opération de chiffrement, à l’exception de certains types d’opérations dans des [conditions particulières](#date-time-controlled-operations). Le traitement de l’attribut *exp* requiert que la date/heure actuelle SOIT antérieure à la date/heure d’expiration indiquée dans l’attribut *exp*. Key Vault PEUT prévoir une légère marge, normalement pas plus de quelques minutes, pour prendre en compte la variation d’horloge. Sa valeur DOIT être un nombre contenant une valeur IntDate.  

Des attributs supplémentaires en lecture seule sont inclus dans toute réponse contenant des attributs de clé :  

- *created* : IntDate, facultatif. L’attribut *created* indique quand cette version de la clé a été créée. La valeur est null pour les clés créées avant l’ajout de cet attribut. Sa valeur DOIT être un nombre contenant une valeur IntDate.  
- *updated* : IntDate, facultatif. L’attribut *updated* indique quand cette version de la clé a été mise à jour. La valeur est null pour les clés qui ont été mises à jour pour la dernière fois avant l’ajout de cet attribut. Sa valeur DOIT être un nombre contenant une valeur IntDate.  

Pour plus d’informations sur IntDate et d’autres types de données, consultez [À propos des clés, des secrets et des certificats : [Types de données système](../general/about-keys-secrets-certificates.md#data-types).

### <a name="date-time-controlled-operations"></a>Opérations contrôlées par date/heure

Les clés pas encore valides et ayant expiré, en dehors de la fenêtre *nbf* / *exp*, s’appliqueront pour les opérations **decrypt**, **unwrap** et **verify** (elles ne retourneront pas d’erreur 403, Interdit). La logique d’attribution d’utilisation de l’état « not-yet-valid » (pas encore valide) consiste à autoriser le test de la clé avant son utilisation en production. La logique d’attribution d’utilisation de l’état « expired » (expiré) consiste à autoriser des opérations de récupération sur des données qui ont été créées alors que la clé était valide. Vous pouvez également désactiver l’accès à une clé à l’aide de stratégies de Key Vault, ou en définissant l’attribut de clé *enabled* sur **false**.

Pour plus d’informations sur les types de données, consultez [Types de données](../general/about-keys-secrets-certificates.md#data-types).

Pour plus d’informations sur les autres attributs possibles, consultez [Clé web JSON (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41).

## <a name="key-tags"></a>Balises de clé

Vous pouvez spécifier des métadonnées spécifiques à l’application supplémentaires sous la forme de balises. Key Vault prend en charge jusqu’à 15 balises, chacune d’entre elles pouvant avoir un nom de 256 caractères et une valeur de 256 caractères.  

> [!NOTE] 
> Les étiquettes peuvent être lues par un appelant qui a l’autorisation *list* ou *get* sur cette clé.

##  <a name="key-access-control"></a>Contrôle d’accès aux clés

Le contrôle d’accès pour les clés gérées par Key Vault est fourni au niveau d’un coffre de clés qui fait office de conteneur de clés. La stratégie de contrôle d’accès pour les clés est différente de la stratégie de contrôle d’accès pour les secrets dans un même coffre de clés. Les utilisateurs peuvent créer un ou plusieurs coffres pour stocker les clés et doivent maintenir une segmentation et une gestion des clés appropriées au scénario. Le contrôle d’accès pour les clés est indépendant du contrôle d’accès pour les secrets.  

Les autorisations suivantes peuvent être accordées, par utilisateur/principal du service, dans l’entrée du contrôle d’accès aux clés sur un coffre. Ces autorisations reflètent précisément les opérations autorisées sur un objet clé.  L’octroi de l’accès à un principal de service dans le coffre de clés est une opération unique et demeure identique pour tous les abonnements Azure. Vous pouvez l’utiliser pour déployer autant de certificats que vous le souhaitez. 

- Autorisations pour les opérations de gestion de clés
  - *get* : lire la partie publique d’une clé, ainsi que ses attributs
  - *list* : lister les clés ou les versions d’une clé stockée dans un coffre de clés
  - *update* : mettre à jour les attributs d’une clé
  - *create* : créer des clés
  - *import* : importer une clé dans un coffre de clés
  - *delete* : supprimer l’objet clé
  - *recover* : récupérer une clé supprimée
  - *backup* : sauvegarder une clé dans un coffre de clés
  - *restore* : restaurer une clé sauvegardée dans un coffre de clés

- Autorisations pour les opérations de chiffrement
  - *decrypt* : utiliser la clé pour déprotéger une séquence d’octets
  - *encrypt* : utiliser la clé pour protéger une séquence arbitraire d’octets
  - *unwrapKey* : utiliser la clé pour déprotéger des clés symétriques wrappées
  - *wrapKey* : utiliser la clé pour protéger une clé symétrique
  - *verify* : utiliser la clé pour vérifier des synthèses  
  - *sign* : utiliser la clé pour signer des synthèses
    
- Autorisations pour les opérations privilégiées
  - *purge* : effacer (supprimer définitivement) une clé supprimée

Pour plus d’informations sur l’utilisation des clés, consultez [Informations de référence sur les opérations de clé dans l’API REST Key Vault](/rest/api/keyvault). Pour plus d’informations sur l’établissement d’autorisations, consultez [Coffres : créer ou mettre à jour](/rest/api/keyvault/vaults/createorupdate) et [Coffres : mettre à jour la stratégie d’accès](/rest/api/keyvault/vaults/updateaccesspolicy). 

## <a name="next-steps"></a>Étapes suivantes
- [À propos de Key Vault](../general/overview.md)
- [À propos de HSM managé](../managed-hsm/overview.md)
- [À propos des secrets](../secrets/about-secrets.md)
- [À propos des certificats](../certificates/about-certificates.md)
- [Vue d’ensemble de l’API REST Azure Key Vault](../general/about-keys-secrets-certificates.md)
- [Authentification, requêtes et réponses](../general/authentication-requests-and-responses.md)
- [Guide du développeur Key Vault](../general/developers-guide.md)
