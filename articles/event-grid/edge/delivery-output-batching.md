---
title: Traitement par lot des sorties dans Azure Event Grid sur IoT Edge | Microsoft Docs
description: Traitement par lot des sortie dans Event Grid sur IoT Edge.
author: HiteshMadan
manager: rajarv
ms.author: himad
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 0ae2261f8278c4d5e1944b01a9731afd293df20b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171633"
---
# <a name="output-batching"></a>Traitement par lot des sorties

Event Grid est capable de prendre en charge la livraison de plusieurs événements dans une même demande de livraison. Cette fonctionnalité permet d’augmenter le débit de distribution global sans payer de frais fiches par requête HTTP. Le traitement par lot est désactivé par défaut et peut être activé pour chaque abonnement.

> [!WARNING]
> La durée maximale autorisée pour traiter chaque demande de livraison ne change pas, même si le code de l’abonné doit potentiellement faire plus de travail par demande traitée par lot. Par défaut, le délai de livraison est de 60 secondes.

## <a name="batching-policy"></a>Stratégie de traitement par lot

Le comportement de traitement par lot d’Event Grid peut être personnalisé par abonné, en modifiant les deux paramètres suivants :

* Nombre maximal d’événements par lot

  Ce paramètre fixe une limite supérieure au nombre d’événements qui peuvent être ajoutés à une demande de livraison traitée par lot.

* Taille de lot préférée en kilo-octets

  Ce bouton est utilisé pour contrôler davantage le nombre maximal de kilo-octets qui peuvent être envoyés par demande de livraison

## <a name="batching-behavior"></a>Comportement du traitement par lots

* Tout ou aucun

  Le service Event Grid fonctionne selon le principe du tout ou rien. Il ne prend pas en charge la réussite partielle d’une livraison par lot. Les abonnés doivent veiller à demander un nombre d’événements par lot ne dépassant pas ce qu’il peuvent raisonnablement gérer en 60 secondes.

* Traitement par lot optimiste

  Les paramètres de stratégie de traitement par lot n’imposent pas de limites strictes au comportement du traitement par lot. Ils sont donc respectés dans la limite des possibilités. Lorsque les fréquences d’événements sont faibles, vous observerez souvent que la taille de lot est inférieure au nombre maximal d’événements demandés par lot.

* La valeur par défaut est DÉSACTIVÉ

  Par défaut, Event Grid ajoute un seul événement à chaque demande de livraison. Pour activer le traitement par lot, définissez l’un des paramètres mentionnés plus haut dans l’article, dans le JSON d’abonnement aux événements.

* Valeurs par défaut

  Lors de la création d’un abonnement aux événements, il n’est pas nécessaire de spécifier les deux paramètres (nombre maximal d’événements par lot et taille de lot approximative en kilo-octets). Si un seul paramètre est défini, Event Grid utilise des valeurs par défaut (configurables). Pour connaître les valeurs par défaut et la manière de les remplacer, consultez les sections suivantes.

## <a name="turn-on-output-batching"></a>Activer le traitement par lot des sorties

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
             {
                "endpointUrl": "<your_webhook_url>",
                "maxEventsPerBatch": 10,
                "preferredBatchSizeInKilobytes": 64
             }
        },
    }
}
```

## <a name="configuring-maximum-allowed-values"></a>Configuration des valeurs maximales autorisées

Les paramètres suivants d’heure de déploiement contrôlent la valeur maximale autorisée lors de la création d’un abonnement aux événements.

| Nom de la propriété | Description |
| ------------- | ----------- | 
| `api__deliveryPolicyLimits__maxpreferredBatchSizeInKilobytes` | Valeur maximale autorisée pour le bouton `PreferredBatchSizeInKilobytes`. Par défaut, `1033`.
| `api__deliveryPolicyLimits__maxEventsPerBatch` | Valeur maximale autorisée pour le bouton `MaxEventsPerBatch`. Par défaut, `50`.

## <a name="configuring-runtime-default-values"></a>Configuration des valeurs de runtime par défaut

Les paramètres d’heure de déploiement suivants contrôlent la valeur de runtime par défaut de chaque bouton quand elle il n’est pas spécifiée dans l’abonnement aux événements. Pour réitérer, au moins un bouton doit être défini sur l’abonnement aux événements pour activer le comportement de traitement par lot.

| Nom de la propriété | Description |
| ------------- | ----------- |
| `broker__defaultMaxBatchSizeInBytes` | Taille maximale de demande de livraison quand seule la valeur `MaxEventsPerBatch` est spécifiée. Par défaut, `1_058_576`.
| `broker__defaultMaxEventsPerBatch` | Nombre maximal d’événements à ajouter à un lot quand seule la valeur `MaxBatchSizeInBytes` est spécifiée. Par défaut, `10`.
