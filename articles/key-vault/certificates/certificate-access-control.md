---
title: À propos du contrôle d’accès aux certificats Azure Key Vault
description: Vue d’ensemble du contrôle d’accès aux certificats Azure Key Vault
services: key-vault
author: sebansal
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: overview
ms.date: 10/12/2020
ms.author: sebansal
ms.openlocfilehash: 1308debb34d724f93526b776f19e0cbf1914d945
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128544"
---
# <a name="certificate-access-control"></a>Contrôle d’accès aux certificats

 Le contrôle d’accès pour les certificats est géré par Key Vault et fourni par le coffre de clés qui contient ces certificats. La stratégie de contrôle d’accès pour les certificats est différente des stratégies de contrôle d’accès pour les clés et les secrets dans un même coffre de clés. Les utilisateurs peuvent créer un ou plusieurs coffres pour stocker les certificats afin de maintenir une segmentation et une gestion des certificats appropriées au scénario.  

 Les autorisations suivantes peuvent être utilisées, par principal, dans l’entrée de contrôle d’accès aux secrets sur un coffre de clés, et reflètent précisément les opérations autorisées sur un objet secret :  

- Autorisations pour les opérations de gestion des certificats
  - **get**  : obtenir la version actuelle ou n’importe quelle version d’un certificat
  - **list**  : lister les certificats actuels ou les versions d’un certificat  
  - **update**  : mettre à jour un certificat
  - **create**  : créer un certificat Key Vault
  - **import**  : importer les éléments d’un certificat dans un certificat Key Vault
  - **delete**  : supprimer un certificat, sa stratégie et toutes ses versions  
  - **recover**  : récupérer un certificat supprimé
  - **backup**  : sauvegarder un certificat dans un coffre de clés
  - **restore**  : restaurer un certificat sauvegardé sur un coffre de clés
  - **managecontacts**  : gérer les contacts du certificat Key Vault  
  - **manageissuers**  : gérer les autorités/émetteurs du certificat Key Vault
  - **getissuers**  : obtenir les autorités/émetteurs d’un certificat
  - **listissuers**  : lister les autorités/émetteurs d’un certificat  
  - **setissuers**  : créer ou mettre à jour les autorités/émetteurs d’un certificat Key Vault  
  - **deleteissuers**  : supprimer les autorités/émetteurs d’un certificat Key Vault  
 
- Autorisations pour les opérations privilégiées
  - **purge**  : effacer (supprimer définitivement) un certificat supprimé

Pour plus d’informations, voir [Informations de référence sur les opérations liées aux certificats dans l’API REST Key Vault](/rest/api/keyvault). Pour obtenir des informations sur l’établissement d’autorisations, consultez [Coffres – Mettre à jour la stratégie d’accès](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="troubleshoot"></a>Dépanner
Une erreur peut s’afficher en raison d’une stratégie d’accès manquante. Par exemple, ```Error type : Access denied or user is unauthorized to create certificate```. Pour résoudre cette erreur, vous devez ajouter des certificats/créer une autorisation.

## <a name="next-steps"></a>Étapes suivantes

- [À propos de Key Vault](../general/overview.md)
- [Présentation des clés, des secrets et des certificats](../general/about-keys-secrets-certificates.md)
- [Authentification, requêtes et réponses](../general/authentication-requests-and-responses.md)
- [Guide du développeur Key Vault](../general/developers-guide.md)
