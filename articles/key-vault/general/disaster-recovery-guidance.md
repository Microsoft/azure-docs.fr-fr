---
title: Disponibilité et redondance d’Azure Key Vault - Azure Key Vault | Microsoft Docs
description: Découvrez des informations sur la disponibilité et la redondance d’Azure Key Vault.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 08/28/2020
ms.author: sudbalas
ms.openlocfilehash: d66fe736936963e601aad7cba7bdaa94f0c3ec3f
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518445"
---
# <a name="azure-key-vault-availability-and-redundancy"></a>Disponibilité et redondance d’Azure Key Vault

Azure Key Vault dispose de plusieurs couches de redondance pour garantir que vos clés et secrets restent disponibles pour votre application même en cas d’échec de composants individuels du service.

> [!NOTE]
> Ce guide s’applique aux coffres. Les pools HSM managés utilisent un autre modèle de haute disponibilité et de reprise d’activité. Pour plus d’informations, consultez [Guide de reprise d’activité pour un HSM managé](../managed-hsm/disaster-recovery-guide.md).

Le contenu de votre coffre de clés est répliqué dans la région ainsi que dans une région secondaire éloignée d’au moins 240 km, mais située au sein de la même zone géographique, afin de maintenir une durabilité élevée de vos clés et secrets. Pour plus d’informations sur les paires de régions spécifiques, consultez [Régions jumelées Azure](../../best-practices-availability-paired-regions.md). L’exception au modèle de régions jumelées est Brésil Sud, qui offre uniquement la possibilité de conserver les données résidant dans cette région. Brésil Sud utilise un stockage redondant interzone (ZRS) pour répliquer vos données trois fois dans la seule et même localisation/région.   

En cas d’échec de composants du service Key Vault, d’autres composants de la même région interviennent pour traiter votre demande afin de garantir l’intégrité des fonctionnalités. Vous n’avez besoin d’effectuer aucune action pour démarrer ce processus. Il se produit automatiquement sans que vous le perceviez.

Dans les rares cas d’indisponibilité d’une région Azure entière, les demandes d’Azure Key Vault effectuées dans cette région sont automatiquement routées (*basculées*) vers une région secondaire, sauf s’il s’agit de la région Brésil Sud. Lorsque la région principale est de nouveau disponible, les demandes sont réacheminées (*basculées*) vers la région principale. Vous n’avez pas besoin d’agir dans la mesure où tout se passe automatiquement.

Dans la région Brésil Sud, vous devez planifier la récupération de vos coffres de clés Azure dans un scénario de défaillance de région. Pour sauvegarder et restaurer votre coffre de clés Azure dans une région de votre choix, effectuez les étapes détaillées dans [Sauvegarde Azure Key Vault](backup.md). 

Dans cette conception haute disponibilité, Azure Key Vault n’exige aucun temps d’arrêt pour les activités de maintenance.

Il convient toutefois de prendre quelques précautions :

* En cas de basculement vers une autre région, quelques minutes peuvent être nécessaires pour que le service soit de nouveau opérationnel. Les demandes effectuées pendant cette période, avant le basculement, peuvent échouer.
* Si vous utilisez un lien privé pour vous connecter à votre coffre de clés, il peut falloir jusqu’à 20 minutes pour que la connexion soit rétablie en cas de basculement. 
* Au cours du basculement, votre coffre de clés est en lecture seule. Les demandes prises en charge dans ce mode sont les suivantes :
  * Afficher la liste des certificats
  * Obtenir les certificats
  * Afficher la liste des secrets
  * Get secrets (Obtenir les secrets)
  * List keys (Afficher la liste des clés)
  * Obtenir les (propriétés des) clés
  * Encrypt (Chiffrer)
  * Déchiffrer
  * Encapsuler
  * Désencapsuler
  * Vérifier
  * Signer
  * Backup

* Au cours du basculement, vous ne pouvez pas apporter de modifications aux propriétés du coffre de clés. Vous ne serez pas en mesure de modifier la configuration ni les paramètres de la stratégie d’accès et du pare-feu.

* Une fois le basculement restauré, tous les types de demandes (y compris de lecture -read- *et* d’écriture write-) sont disponibles.
