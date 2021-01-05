---
title: À propos des secrets Azure Key Vault - Azure Key Vault
description: Vue d’ensemble de l’interface REST Azure Key Vault et des détails de développement sur les secrets.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: overview
ms.date: 09/04/2019
ms.author: mbaldwin
ms.openlocfilehash: 7aa2feba5a2b2fa47bbb0c055a2f556b8997ab34
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "82930469"
---
# <a name="about-azure-key-vault-secrets"></a>À propos des secrets Azure Key Vault

Key Vault offre un stockage sécurisé des secrets, tels que les mots de passe et les chaînes de connexion de base de données.

Pour les développeurs, les API Key Vault acceptent et retournent des valeurs de secret sous forme de chaînes. En interne, Key Vault stocke et gère les secrets par séquence d’octets (8 bits), avec une taille maximale de 25 Ko chacune. Le service Key Vault ne fournit pas la sémantique pour les secrets. Il accepte simplement les données, les chiffre, les stocke et retourne un identificateur de secret (« id »). L’identificateur peut être utilisé pour récupérer le secret ultérieurement.  

Pour les données extrêmement sensibles, les clients doivent envisager des couches supplémentaires de protection des données. Un exemple serait de chiffrer les données à l’aide d’une clé de protection distincte avant le stockage dans Key Vault.  

Key Vault prend également en charge un champ contentType pour les secrets. Les clients peuvent spécifier le type de contenu d’un secret pour simplifier l’interprétation des données du secret lors de leur récupération. La longueur maximale de ce champ est de 255 caractères. Il n’existe aucune valeur prédéfinie. L’utilisation suggérée est donnée à titre de conseil pour interpréter les données du secret. Par exemple, une implémentation peut stocker des mots de passe et des certificats en tant que secret. Ensuite, utilisez ce champ pour les différencier. Il n’existe aucune valeur prédéfinie.  

## <a name="encryption"></a>Chiffrement

Tous les secrets de votre Key Vault sont stockés sous forme chiffrée. Ce chiffrement est transparent et ne nécessite aucune action de la part de l’utilisateur. Le service Azure Key Vault chiffre vos secrets lorsque vous les ajoutez et les déchiffre automatiquement quand vous les lisez. La clé de chiffrement est propre à chaque coffre de clés.

## <a name="secret-attributes"></a>Attributs de secret

Les attributs suivants peuvent être spécifiés en plus des données du secret :  

- *exp* : IntDate, facultatif, la valeur par défaut est **forever**. L’attribut *exp* (heure d’expiration) identifie l’heure d’expiration à ou après laquelle les données du secret NE DOIVENT PAS être récupérées, sauf dans des [conditions particulières](#date-time-controlled-operations). Ce champ n’est fourni qu’à titre d’**information**, il informe uniquement les utilisateurs du service de coffre de clés qu’un secret particulier ne peut pas être utilisé. Sa valeur DOIT être un nombre contenant une valeur IntDate.   
- *nbf* : IntDate, facultatif, la valeur par défaut est **now**. L’attribut *nbf* (pas avant) identifie l’heure avant laquelle les données du secret NE DOIVENT PAS être récupérées, sauf dans des [conditions particulières](#date-time-controlled-operations). Ce champ est fourni à titre d’**information** uniquement. Sa valeur DOIT être un nombre contenant une valeur IntDate. 
- *enabled* : booléen, facultatif, **true** par défaut. Cet attribut spécifie si les données du secret peuvent être récupérées. L’attribut enabled est utilisé avec *nbf* et *nbf* lorsqu’une opération se produit entre *nbf* et *exp*, elle ne sera autorisée que si enabled est défini sur **true**. Les opérations en dehors de la fenêtre *nbf* et *exp* sont automatiquement interdites, sauf dans des [conditions particulières](#date-time-controlled-operations).  

Des attributs supplémentaires en lecture seule sont inclus dans toute réponse contenant des attributs de secret :  

- *created* : IntDate, facultatif. L’attribut created indique quand cette version du secret a été créée. Cette valeur est null pour les secrets créés avant l’ajout de cet attribut. Sa valeur doit être un nombre contenant une valeur IntDate.  
- *updated* : IntDate, facultatif. L’attribut updated indique quand cette version du secret a été mise à jour. Cette valeur est null pour les secrets qui ont été mis à jour pour la dernière fois avant l’ajout de cet attribut. Sa valeur doit être un nombre contenant une valeur IntDate.

### <a name="date-time-controlled-operations"></a>Opérations contrôlées par date/heure

L’opération **get** d’un secret s’appliquera aux secrets pas encore valides ou ayant expiré, en dehors de la fenêtre *nbf* / *exp*. L’appel de l’opération **get** d’un secret, pour un secret pas encore valide, peut être utilisé à des fins de test. La récupération (opération **get**ting) d’un secret expiré peut être utilisée pour des opérations de récupération.

## <a name="secret-access-control"></a>Contrôle d’accès aux secrets

Le contrôle d’accès pour les secrets géré dans Key Vault est fourni au niveau du coffre de clés qui contient ces secrets. La stratégie de contrôle d’accès pour les secrets est différente de la stratégie de contrôle d’accès pour les clés dans un même coffre de clés. Les utilisateurs peuvent créer un ou plusieurs coffres pour stocker les secrets et doivent maintenir une segmentation et une gestion des secrets appropriées au scénario.   

Les autorisations suivantes peuvent être utilisées, par principal, dans l’entrée de contrôle d’accès aux secrets sur un coffre, et reflètent précisément les opérations autorisées sur un objet secret :  

- Autorisations pour les opérations de gestion de secrets
  - *get* : lire un secret  
  - *list* : lister les secrets ou les versions d’un secret stockés dans un coffre de clés  
  - *set* : Créer un secret  
  - *delete* : supprimer un secret  
  - *recover* : récupérer un secret supprimé
  - *backup* : sauvegarder un secret dans un coffre de clés
  - *restore* : restaurer un secret sauvegardé sur un coffre de clés

- Autorisations pour les opérations privilégiées
  - *purge* : effacer (supprimer définitivement) un secret supprimé

Pour plus d’informations sur l’utilisation des secrets, voir [Informations de référence sur les opérations liées aux secrets dans l’API REST Key Vault](/rest/api/keyvault). Pour plus d’informations sur l’établissement d’autorisations, consultez [Coffres : créer ou mettre à jour](/rest/api/keyvault/vaults/createorupdate) et [Coffres : mettre à jour la stratégie d’accès](/rest/api/keyvault/vaults/updateaccesspolicy). 

## <a name="secret-tags"></a>Balises de secret  
Vous pouvez spécifier des métadonnées spécifiques à l’application supplémentaires sous la forme de balises. Key Vault prend en charge jusqu’à 15 balises, chacune d’entre elles pouvant avoir un nom de 256 caractères et une valeur de 256 caractères.  

>[!Note]
>Les étiquettes peuvent être lues par un appelant s’il dispose de l’autorisation *list* ou *get*.

## <a name="azure-storage-account-key-management"></a>Gestion des clés de compte de stockage Azure

Key Vault peut gérer les clés de compte de stockage Azure :

- En interne, Key Vault peut lister (synchroniser) les clés avec un compte de stockage Azure. 
- Key Vault regénère (fait tourner) les clés régulièrement.
- Les valeurs de clés ne sont jamais retournées en réponse à l’appelant.
- Key Vault gère les clés des comptes de stockage et des comptes de stockage classiques.

Pour plus d’informations, consultez [Clés de compte de stockage Azure Key Vault](../secrets/overview-storage-keys.md).

## <a name="storage-account-access-control"></a>Contrôle d’accès aux comptes de stockage

Vous pouvez utiliser les autorisations suivantes quand vous autorisez un utilisateur ou un principal d’application à effectuer des opérations sur un compte de stockage géré :  

- Autorisations pour les opérations de définition SAS et de compte de stockage géré
  - *get* : obtenir des informations sur un compte de stockage 
  - *list* : lister les comptes de stockage gérés par un coffre de clés
  - *update* : mettre à jour un compte de stockage
  - *delete* : Suppression d'un compte de stockage  
  - *recover* : récupérer un compte de stockage supprimé
  - *backup* : sauvegarder un compte de stockage
  - *restore* : restaurer un compte de stockage sauvegardé sur un coffre de clés
  - *set* : créer ou mettre à jour un compte de stockage
  - *regeneratekey* : regénérer une valeur de clé spécifiée pour un compte de stockage
  - *getsas* : obtenir des informations sur une définition SAS pour un compte de stockage
  - *listsas* : lister les définitions SAS de stockage d’un compte de stockage
  - *deletesas* : supprimer une définition SAS d’un compte de stockage
  - *setsas* : créer ou mettre à jour une définition/des attributs SAS pour un compte de stockage

- Autorisations pour les opérations privilégiées
  - *purge* : effacer (supprimer définitivement) un compte de stockage géré

Pour plus d’informations, consultez [Informations de référence sur les opérations de compte de stockage dans l’API REST Key Vault](/rest/api/keyvault). Pour plus d’informations sur l’établissement d’autorisations, consultez [Coffres : créer ou mettre à jour](/rest/api/keyvault/vaults/createorupdate) et [Coffres : mettre à jour la stratégie d’accès](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="next-steps"></a>Étapes suivantes

- [À propos de Key Vault](../general/overview.md)
- [Présentation des clés, des secrets et des certificats](../general/about-keys-secrets-certificates.md)
- [À propos des clés](../keys/about-keys.md)
- [À propos des certificats](../certificates/about-certificates.md)
- [Authentification, requêtes et réponses](../general/authentication-requests-and-responses.md)
- [Guide du développeur Key Vault](../general/developers-guide.md)
