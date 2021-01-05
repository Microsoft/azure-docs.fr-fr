---
title: Azure Event Hubs – Exceptions Resource Manager | Microsoft Docs
description: Liste des exceptions Azure Event Hubs signalées par Azure Resource Manager et actions suggérées.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: cec24696d0d49ba408860f6562c34dd14876c311
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91334206"
---
# <a name="azure-event-hubs---resource-manager-exceptions"></a>Azure Event Hubs – Exceptions Resource Manager
Cet article répertorie les exceptions générées lors de l’interaction avec Azure Event Hubs à l’aide d’Azure Resource Manager par le biais de modèles ou d’appels directs.

> [!IMPORTANT]
> Ce document est fréquemment mis à jour. Revenez plus tard pour suivre l’évolution de la situation.

Les sections suivantes indiquent différentes exceptions ou erreurs signalées par Azure Resource Manager.

## <a name="error-code-conflict"></a>Code d’erreur : Conflit

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| Conflit | 40300 | Le nombre maximal de ressources de type EventHub a été atteint ou dépassé. Réel : #, maximum autorisé : # | L’espace de noms a atteint son [quota](event-hubs-quotas.md) pour le nombre d’Event Hubs qu’il peut contenir. | Supprimez les Event Hubs inutilisés ou superflus de l’espace de noms ou envisagez la mise à niveau vers un [cluster dédié](event-hubs-dedicated-overview.md). |
| Conflit | Aucun | La configuration de la récupération d’urgence (DR) ne peut pas être supprimée, car la réplication est en cours. Faites basculer ou arrêtez l’association avant de tenter de supprimer la configuration DR. | La [réplication de la géorécupération d’urgence (GéoDR)](event-hubs-geo-dr.md) est en cours, de sorte que la configuration ne peut pas être supprimée pour l’instant. | Pour débloquer la suppression de la configuration, attendez que la réplication soit terminée, déclenchez un basculement ou arrêtez l’association GéoDR. |
| Conflit | Aucun | Échec de la mise à jour de l’espace de noms avec conflit dans le serveur principal. | Une autre opération est en cours sur cet espace de noms. | Attendez la fin de l’opération actuelle, puis réessayez. |

## <a name="error-code-429"></a>Code d’erreur : 429

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| 429 | Aucun | Approvisionnement de l’espace de noms en transition. | Une autre opération est en cours sur cet espace de noms. | Attendez la fin de l’opération actuelle, puis réessayez. |
| 429 | Aucun | Opération de récupération d’urgence en cours. | Une opération [GéoDR](event-hubs-geo-dr.md) est en cours sur cet espace de noms. | Attendez la fin de l’opération GéoDR actuelle, puis réessayez. |

## <a name="error-code-badrequest"></a>Code d’erreur : BadRequest

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| BadRequest | 40000 | Impossible de modifier PartitionCount pour un Event Hub. | Le niveau de service De base ou Standard d’Azure Event Hubs ne prend pas en charge la modification des partitions. | Créez un Event Hub doté du nombre de partitions souhaité dans votre espace de noms de niveau De base ou Standard. La montée en charge des partitions est prise en charge pour les [clusters dédiés](event-hubs-dedicated-overview.md). |
| BadRequest | 40000 | La valeur « # » pour MessageRetentionInDays n’est pas valide pour le niveau De base. La valeur ne peut pas dépasser « 1 » jour(s). | Les espaces de noms Event Hubs de niveau De base prennent uniquement en charge la rétention des messages pendant 1 jour. | Si vous souhaitez conserver les messages pendant plus d’un jour, [créez un espace de noms Event Hubs Standard](event-hubs-create.md). | 
| BadRequest | Aucun | Le nom spécifié n’est pas disponible. | Les noms d’espaces de noms doivent être uniques, et le nom spécifié est déjà utilisé. | Si vous êtes le propriétaire de l’espace de noms portant le nom spécifié, vous pouvez le supprimer, ce qui entraînera une perte des données. Ensuite, réessayez avec le même nom. Si la suppression de l’espace de noms n’est pas sûre (ou si vous n’êtes pas le propriétaire), choisissez un autre nom d’espace de noms. |
| BadRequest | Aucun | L’abonnement spécifié a atteint son quota d’espaces de noms. | Votre abonnement a atteint le [quota](event-hubs-quotas.md) de nombre d’espaces de noms qu’il peut contenir. | Envisagez de supprimer les espaces de noms inutilisés de cet abonnement, de créer un autre abonnement ou de procéder à une mise à niveau vers un [cluster dédié](event-hubs-dedicated-overview.md). |
| BadRequest | Aucun | Impossible de mettre à jour un espace de noms secondaire. | L’espace de noms ne peut pas être mis à jour, car il s’agit de l’espace de noms secondaire dans une [association GéoDR](event-hubs-geo-dr.md). | Le cas échéant, modifiez l’espace de noms principal dans cette association à la place. Sinon, vous pouvez arrêter l’association GéoDR pour effectuer la modification. |
| BadRequest | Aucun | Impossible de définir l’augmentation automatique de la référence SKU de base. | L’augmentation automatique ne peut pas être activée sur les espaces de noms Event Hubs de niveau De base. | Pour [activer l’augmentation automatique](event-hubs-auto-inflate.md) sur un espace de noms, assurez-vous qu’il s’agit d’un niveau Standard. |
| BadRequest | Aucun | La capacité est insuffisante pour créer l’espace de noms. Contactez votre administrateur Event Hubs. | La région sélectionnée est au maximum de sa capacité et d’autres espaces de noms ne peuvent pas être créés. | Sélectionnez une autre région pour héberger votre espace de noms. |
| BadRequest | Aucun | L’opération ne peut pas être effectuée sur le type d’entité « ConsumerGroup », car l’espace de noms « nom de l’espace de noms » utilise le niveau « De base ».  | Les espaces de noms Event Hubs de niveau De base ont un quota(event-hubs-quotas.md) de groupe de consommateurs (par défaut). La création de groupes de consommateurs supplémentaires n’est pas prise en charge. | Continuez d’utiliser le groupe de consommateurs par défaut ($Default) ou, si vous avez besoin de plus de groupes, envisagez plutôt d’utiliser un espace de noms Event Hubs de niveau Standard. | 
| BadRequest | Aucun | L’espace de noms « nom de l’espace de noms » n’existe pas. | L’espace de noms fourni est introuvable. | Vérifiez que le nom de l’espace de noms est correct et qu’il se trouve dans votre abonnement. Si ce n’est pas le cas, [créez un espace de noms Event Hubs](event-hubs-create.md). | 
| BadRequest | Aucun | La propriété d’emplacement de la ressource ne correspond pas à son Espace de noms conteneur. | La création d’un Event Hub dans une région spécifique a échoué, car celle-ci ne correspondait pas à la région de l’espace de noms. | Essayez de créer l’Event Hub dans la même région que l’espace de noms. | 

## <a name="error-code-internal-server-error"></a>Code d’erreur : Erreur interne du serveur

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| Erreur interne du serveur | Aucun | Erreur interne du serveur. | Le service Event Hubs a rencontré une erreur interne. | Réessayez l’opération qui a échoué. Si l’opération continue d’échouer, contactez le support. |