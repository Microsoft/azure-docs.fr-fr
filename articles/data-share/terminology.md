---
title: Terminologie Azure Data Share
description: Découvrez les termes couramment employés pour décrire les ressources utilisées dans Azure Data Share (fournisseur de données, consommateur de données, partage de données, abonnement de partage, instantané, invitation, destinataire).
ms.service: data-share
author: joannapea
ms.author: joanpo
ms.topic: conceptual
ms.date: 07/10/2019
ms.openlocfilehash: 33532380d8f98df44029eeea998130d1da5fdafd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73468558"
---
# <a name="azure-data-share-concepts"></a>Concepts d’Azure Data Share 

Azure Data Share introduit quelques nouveaux termes liés au partage de données. Cet article explique quelques-uns des termes pouvant être utilisés à travers le service. 

## <a name="data-provider"></a>Fournisseur de données

Un fournisseur de données est l’organisation qui partage des données avec ses consommateurs. En règle générale, le fournisseur de données peut être un propriétaire ou un conservateur de données. Les fournisseurs de données souhaitent partager des données de divers types. Les données que vous souhaiterez peut-être partager avec un fournisseur de données comprennent les données brutes, comme les données de point de vente ou de série chronologique. Un fournisseur de données peut également partager des données prétraitées, organisées, qui contiennent déjà des insights et des analyses. 

## <a name="data-consumer"></a>Consommateur de données 

Un consommateur de données est l’organisation qui reçoit des données venant d’un fournisseur de données. Le consommateur de données souhaite joindre les données partagées avec ses propres données pour en extraire des insights. Dans certains cas, le consommateur de données peut recevoir des données qui ont déjà été traitées. 

## <a name="data-share"></a>Partage de données

Un partage de données est un groupe de jeux de données qui est partagé en tant qu’entité unique. Les jeux de données peuvent provenir de différentes sources de données Azure prises en charge par Azure Data Share. Actuellement, Azure Data Share prend en charge le stockage Blob Azure et Azure Data Lake Store. 

## <a name="share-subscription"></a>Abonnement de partage 

Un abonnement de partage est créé lorsqu’un consommateur de données accepte une invitation de partage de données d’un fournisseur de données. Les fournisseurs de données peuvent voir les abonnements de partage actifs en accédant à **Partages envoyé** dans leur compte Azure Data Share, puis en sélectionnant **Abonnements de partage**.

Un consommateur de données peut vérifier s’il a un abonnement de partage actif en accédant à **Partages reçu** et en affichant l’état de ses partages reçus. 

## <a name="snapshot"></a>Instantané

Un instantané peut être créé par un consommateur de données lorsqu’il accepte une invitation de partage de données. Lorsqu’il accepte une invitation, il peut déclencher un instantané complet des données partagées avec lui. L’instantané est une copie des données au point dans le temps auquel le consommateur de données a généré l’instantané. 

Il existe deux types d’instantanés : complets et incrémentiels. Un instantané complet contient toutes les données au sein du partage de données. Un instantané incrémentiel contient toutes les données qui ont été mises à jour/ajoutées depuis la dernière capture instantanée. 

## <a name="snapshot-settings-in-azure-data-share"></a>Paramètres d’instantané dans Azure Data Share
 
Un fournisseur de données peut activer un paramètre d’instantané pour un partage de données. Ce paramètre permet aux consommateurs de données de recevoir des mises à jour incrémentielles quand elles se produisent. Ce paramètre doit être activé si le fournisseur de données souhaite que ses consommateurs de données reçoivent des mises à jour pour les données qui ont été partagées. 

Si un fournisseur de données active ce paramètre, un intervalle de récurrence peut être sélectionné. L’intervalle de récurrence peut être horaire ou quotidien. 

Un consommateur de données a la possibilité de participer à cette planification de capture instantanée pour recevoir des mises à jour incrémentielles, qui incluent toutes les données qui ont changé depuis le premier instantané généré. 

## <a name="invitation"></a>Invitation

Un fournisseur de données peut inviter plusieurs destinataires à son partage de données. Pour ce faire, il peut ajouter des destinataires au partage de données. Les invitations peuvent également être ajoutées après la création d’un partage de données. 

Un fournisseur de données peut supprimer une invitation après son envoi si elle n’a pas été acceptée. Si le fournisseur de données supprime une invitation qui n’a pas encore été acceptée, le consommateur de données ne sera pas en mesure de l’accepter. 

Les invitations peuvent être renvoyées jusqu’à cinq fois par jour. 

## <a name="recipient"></a>Recipient

Un destinataire est une personne qui reçoit une invitation à un partage de données. En règle générale, un fournisseur de données ajoute des destinataires au partage de données qu’il crée. Une fois que le destinataire d’une invitation accepte l’invitation, il devient un consommateur de données.  

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment commencer à partager des données, passez au tutoriel [Partager vos données](share-your-data.md).
