---
title: Mettre automatiquement à l’échelle les unités de débit - Azure Event Hubs | Microsoft Docs
description: Activez la majoration automatique sur un espace de noms pour automatiquement mettre à l’échelle les unités de débit.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 1be564472011622b71b3066495748dfdbe6cc791
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020804"
---
# <a name="automatically-scale-up-azure-event-hubs-throughput-units"></a>Mettre automatiquement à l’échelle les unités de débit Azure Event Hubs
Azure Event Hubs est une plateforme hautement évolutive de diffusion de données en continu. Ainsi, l’utilisation des unités Event Hubs augmente souvent une fois le service démarré. Ces augmentations obligent à rehausser les [unités de débit](event-hubs-scalability.md#throughput-units) prédéterminées pour mettre à l’échelle Event Hubs et gérer des taux de transfert plus conséquents. La fonctionnalité **Majoration automatique** d’Event Hubs augmente automatiquement la taille des instances en augmentant le nombre d’unités de débit pour répondre aux besoins d’utilisation. Cette opération permet d’éviter les situations de limitation, dans lesquelles :

* Les taux d’entrée de données sont supérieurs aux unités de débit définies.
* Les taux de demande de sortie de données sont supérieurs aux unités de débit définies.

Le service Event Hubs augmente le débit quand la charge dépasse le seuil minimal, sans qu’aucune demande échoue avec des erreurs ServerBusy.

## <a name="how-auto-inflate-works"></a>Fonctionnement de la majoration automatique

Le trafic Event Hubs est contrôlé par les [unités de débit](event-hubs-scalability.md#throughput-units). Une unité de débit autorise 1 Mo/s en entrée et le double en sortie. Les unités Event Hub standard peuvent être configurées avec 1 à 20 unités de débit. La majoration automatique vous permet de démarrer petit avec le nombre d’unités de débit minimal exigé que vous choisissez. Ensuite, la fonctionnalité met automatiquement à l’échelle le nombre d’unités de débit dont vous avez besoin sur la limite maximale, selon l’augmentation de votre trafic. La majoration automatique vous permet de bénéficier des avantages suivants :

- Un mécanisme de mise à l’échelle efficace pour démarrer avec la valeur minimale et monter en puissance à mesure de la croissance de votre activité.
- Mise à l’échelle automatique sur à la limite supérieure spécifiée sans problèmes de limitation.
- Contrôle amélioré de la mise à l’échelle, car vous contrôlez le moment et la quantité de la mise à l’échelle.

## <a name="enable-auto-inflate-on-a-namespace"></a>Activer la majoration automatique sur un espace de noms

Vous pouvez activer et désactiver la majoration automatique sur un espace de noms Event Hubs de niveau Standard, en appliquant l’une des méthodes suivantes :

- Le [portail Azure](https://portal.azure.com).
- [Un modèle Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-inflate).

> [!NOTE]
> Les espaces de noms Event Hubs de niveau De base ne prennent pas en charge la majoration automatique.

### <a name="enable-auto-inflate-through-the-portal"></a>Activer la majoration automatique via le portail


#### <a name="enable-at-the-time-of-creation"></a>Activer l’option au moment de la création 
Vous pouvez activer la fonctionnalité d’augmentation automatique **lors de la création d’un espace de noms Event Hub** :
 
![Activer l’augmentation automatique lors de la création d’un Event Hub](./media/event-hubs-auto-inflate/event-hubs-auto-inflate1.png)

Une fois cette option activée, vous pouvez commencer par utiliser le nombre minimal d’unités de débit, puis monter en puissance à mesure que vos besoins d’utilisation augmentent. La limite supérieure de la majoration n’affecte pas immédiatement les prix, qui dépendent du nombre d’unités de débit utilisées par heure.

#### <a name="enable-auto-inflate-for-an-existing-event-hub"></a>Activer l’augmentation automatique pour un Event Hub existant
Vous pouvez également activer la fonctionnalité d’augmentation automatique et modifier ses paramètres en suivant les instructions ci-après : 
 
1. Dans la page **Espace de noms Event Hubs**, sélectionnez **Désactivé** sous **Augmentation automatique des unités de débit**.  

    ![Sélectionner les unités de débit dans la page Espace de noms Event Hubs](./media/event-hubs-auto-inflate/select-throughput-units.png)
2. Dans la page **Paramètres de mise à l’échelle**, cochez la case **Activer** (si la fonctionnalité de mise à l’échelle automatique n’a pas été activée).

    ![Sélectionner Activer](./media/event-hubs-auto-inflate/scale-settings.png)
3. Entrez le nombre **maximal** d’unités de débit ou utilisez la barre de défilement pour définir une valeur. 
4. (facultatif) Mettez à jour nombre **minimal** d’unités de débit en haut de cette page. 


> [!NOTE]
> Quand vous appliquez la configuration avec augmentation automatique afin d’augmenter les unités de débit, le service Event Hubs émet des journaux de diagnostic qui vous indiquent quand et pourquoi le débit a augmenté. Pour activer la journalisation des diagnostics pour un hub d’événements, sélectionnez **Paramètres de diagnostic** dans le menu de gauche de la page Hub d’événements dans le portail Azure. Pour plus d’informations, consultez [Configurer les journaux de diagnostic pour un hub d’événements Azure](event-hubs-diagnostic-logs.md). 

### <a name="enable-auto-inflate-using-an-azure-resource-manager-template"></a>Activer la majoration automatique à l’aide d’un modèle Azure Resource Manager

Vous pouvez activer la majoration automatique durant le déploiement d’un modèle Azure Resource Manager. Par exemple, définissez la propriété `isAutoInflateEnabled` sur **true** et définissez `maximumThroughputUnits` sur 10. Par exemple :

```json
"resources": [
        {
            "apiVersion": "2017-04-01",
            "name": "[parameters('namespaceName')]",
            "type": "Microsoft.EventHub/Namespaces",
            "location": "[variables('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard"
            },
            "properties": {
                "isAutoInflateEnabled": true,
                "maximumThroughputUnits": 10
            },
            "resources": [
                {
                    "apiVersion": "2017-04-01",
                    "name": "[parameters('eventHubName')]",
                    "type": "EventHubs",
                    "dependsOn": [
                        "[concat('Microsoft.EventHub/namespaces/', parameters('namespaceName'))]"
                    ],
                    "properties": {},
                    "resources": [
                        {
                            "apiVersion": "2017-04-01",
                            "name": "[parameters('consumerGroupName')]",
                            "type": "ConsumerGroups",
                            "dependsOn": [
                                "[parameters('eventHubName')]"
                            ],
                            "properties": {}
                        }
                    ]
                }
            ]
        }
    ]
```

Pour accéder au modèle complet, consultez le modèle [Create Event Hubs namespace and enable inflate](https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-inflate) (Créer un espace de noms Event Hubs et activer la majoration) sur GitHub.


## <a name="next-steps"></a>Étapes suivantes

Vous pouvez en apprendre plus sur Event Hubs en consultant les liens suivants :

* [Vue d’ensemble d’Event Hubs](./event-hubs-about.md)
