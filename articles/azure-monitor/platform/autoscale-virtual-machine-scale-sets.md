---
title: Mise à l’échelle automatique avancée à l’aide de machines virtuelles Azure
description: Utilisez Resource Manager et les grupes identiques de machines virtuelles avec plusieurs règles et profils qui envoient un e-mail et appellent les URL webhook avec des actions de mise à l’échelle.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 06/25/2020
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: 96d8c32791bd968dd8740a2d4633496b42e9f3f4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91360971"
---
# <a name="advanced-autoscale-configuration-using-resource-manager-templates-for-vm-scale-sets"></a>Configuration avancée de la mise à l’échelle automatique à l’aide des modèles Resource Manager pour VM Scale Sets
Vous pouvez diminuer ou augmenter la taille des instances dans VM Scale Sets en fonction de certains seuils de métriques de performances, selon une planification périodique ou à une date donnée. Vous pouvez également configurer des e-mails et des webhooks de notification pour les actions de mise à l’échelle. Cette procédure pas à pas présente un exemple de configuration de ces objets à l’aide d’un modèle Resource Manager sur un groupe de machines virtuelles identiques (VM Scale Set).

> [!NOTE]
> Bien que cette procédure pas à pas explique les étapes pour les Virtual Machine Scale Sets, les mêmes informations s’appliquent pour la mise à l’échelle des [services cloud](https://azure.microsoft.com/services/cloud-services/), [d’App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) et des [services Gestion des API](../../api-management/api-management-key-concepts.md) Pour paramétrer de façon simple un scale-in/out d’un élément VM Scale Set basé sur une métrique de performances simple, reportez-vous aux documents [Linux](../../virtual-machine-scale-sets/tutorial-autoscale-cli.md) et [Windows](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md)
>
>

## <a name="walkthrough"></a>Procédure pas à pas
Dans cette procédure pas à pas, nous utilisons [Azure Resource Explorer](https://resources.azure.com/) pour configurer et mettre à jour le paramètre de mise à l’échelle automatique pour un groupe de machines virtuelles identiques. L’Explorateur de ressources Azure est un moyen simple de gérer des ressources Azure via des modèles Resource Manager. Si vous débutez avec l’Explorateur de ressources Azure, lisez [cette introduction](https://azure.microsoft.com/blog/azure-resource-explorer-a-new-tool-to-discover-the-azure-api/).

1. Déployez un nouveau groupe de machines virtuelles identiques avec un paramètre de mise à l’échelle automatique de base. Cet article utilise celui de la galerie de démarrage rapide Azure, qui comporte un groupe de machines virtuelles identiques Windows avec un modèle de mise à l’échelle automatique de base. Les groupes de machines virtuelles identiques Linux fonctionnent de la même façon.
2. Une fois le groupe créé, accédez à la ressource du groupe à partir de l’Explorateur de ressources Azure. Le code suivant apparaît sous le nœud Microsoft.Insights.

    ![Azure Explorer](media/autoscale-virtual-machine-scale-sets/azure_explorer_navigate.png)

    L’exécution du modèle a créé un paramètre de mise à l’échelle automatique par défaut portant le nom **'autoscalewad'** . Dans la partie droite, vous pouvez afficher la définition complète de ce paramètre de mise à l’échelle. Dans ce cas, le paramètre de mise à l’échelle par défaut est associé à une règle de diminution et d’augmentation de la taille des instances basé sur le % de processeur.  

3. Vous pouvez maintenant ajouter d’autres profils et règles à partir d’un calendrier ou d’exigences spécifiques. Nous allons créer un paramètre de mise à l’échelle automatique avec trois profils. Pour comprendre les profils et les règles de mise à l’échelle automatique, consultez [Meilleures pratiques pour la mise à l’échelle automatique d’Azure Insights](autoscale-best-practices.md).  

    | Profils et règles | Description |
    |--- | --- |
    | **Profil** |**En fonction des métriques/performances** |
    | Règle |Nombre de messages de la file d’attente Service Bus > x |
    | Règle |Nombre de messages de la file d’attente Service Bus < y |
    | Règle |%UC > n |
    | Règle |% UC, < p |
    | **Profil** |**Heures dans la matinée, en semaine (aucune règle)** |
    | **Profil** |**Jour de lancement de produit (aucune règle)** |

4. Voici un scénario de mise à l’échelle hypothétique que nous utiliserons pour cette procédure pas à pas.

   * **Selon la charge** : je souhaite effectuer un scale-out ou un scale-in en fonction de la charge de mon application hébergée sur mon groupe de machines virtuelles identiques.*
   * **Taille de la file d’attente** : j’utilise une file d’attente Service Bus pour les messages entrants dans mon application. J’utilise le nombre de messages de la file d’attente et le % processeur, et je configure un profil par défaut pour déclencher une action de mise à l’échelle si le nombre de messages ou le % processeur atteint le seuil défini.\*
   * **Heure de la journée** : je souhaite définir un profil hebdomadaire basé sur « l’heure de la journée », appelé « Heures dans la matinée, en semaine ». Au vu des données historiques, je sais qu’il est préférable d’avoir un certain nombre d’instances de machine virtuelle pour gérer la charge de mon application pendant cette période.\*
   * **Dates particulières** : j’ai ajouté un profil « Jour de lancement de produit ». Je planifie à l’avance des dates spécifiques afin que mon application soit prête à gérer la charge, en fonction des annonces marketing et des dates d’introduction de nouveaux produits dans l’application.\*
   * *Les deux derniers profils peuvent également contenir d’autres règles basées sur les métriques de performances. Dans mon cas, j’ai décidé de ne pas en utiliser et de me concentrer uniquement sur les règles basées sur les métriques de performances par défaut. Les règles sont facultatives pour les profils récurrents et les profils basés sur la date.*

     La définition des priorités des profils et des règles du moteur de mise à l’échelle automatique est également décrite dans l’article [Meilleures pratiques pour la mise à l’échelle automatique d’Azure Insights](autoscale-best-practices.md).
     Pour obtenir une liste des métriques communément utilisées pour la mise à l’échelle, consultez [Métriques courantes pour la mise à l’échelle automatique d’Azure Insights](autoscale-common-metrics.md)

5. Assurez-vous que vous utilisez l’Explorateur de ressources en mode **lecture/écriture**

    ![Autoscalewad, paramètre de mise à l’échelle automatique par défaut](media/autoscale-virtual-machine-scale-sets/autoscalewad.png)

6. Cliquez sur Modifier. **Remplacez** l’élément « profiles » dans le paramètre de mise à l’échelle automatique par la configuration suivante :

    ![Capture d’écran montrant l’élément Profils.](media/autoscale-virtual-machine-scale-sets/profiles.png)

    ```
    {
            "name": "Perf_Based_Scale",
            "capacity": {
              "minimum": "2",
              "maximum": "12",
              "default": "2"
            },
            "rules": [
              {
                "metricTrigger": {
                  "metricName": "MessageCount",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT5M",
                  "timeAggregation": "Average",
                  "operator": "GreaterThan",
                  "threshold": 10
                },
                "scaleAction": {
                  "direction": "Increase",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "MessageCount",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT5M",
                  "timeAggregation": "Average",
                  "operator": "LessThan",
                  "threshold": 3
                },
                "scaleAction": {
                  "direction": "Decrease",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "Percentage CPU",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/<this_vmss_name>",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT30M",
                  "timeAggregation": "Average",
                  "operator": "GreaterThan",
                  "threshold": 85
                },
                "scaleAction": {
                  "direction": "Increase",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "Percentage CPU",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/<this_vmss_name>",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT30M",
                  "timeAggregation": "Average",
                  "operator": "LessThan",
                  "threshold": 60
                },
                "scaleAction": {
                  "direction": "Decrease",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              }
            ]
          },
          {
            "name": "Weekday_Morning_Hours_Scale",
            "capacity": {
              "minimum": "4",
              "maximum": "12",
              "default": "4"
            },
            "rules": [],
            "recurrence": {
              "frequency": "Week",
              "schedule": {
                "timeZone": "Pacific Standard Time",
                "days": [
                  "Monday",
                  "Tuesday",
                  "Wednesday",
                  "Thursday",
                  "Friday"
                ],
                "hours": [
                  6
                ],
                "minutes": [
                  0
                ]
              }
            }
          },
          {
            "name": "Product_Launch_Day",
            "capacity": {
              "minimum": "6",
              "maximum": "20",
              "default": "6"
            },
            "rules": [],
            "fixedDate": {
              "timeZone": "Pacific Standard Time",
              "start": "2016-06-20T00:06:00Z",
              "end": "2016-06-21T23:59:00Z"
            }
          }
    ```
    Pour les champs pris en charge et leurs valeurs, consultez la [documentation API REST de la mise à l’échelle automatique](/rest/api/monitor/autoscalesettings). Votre paramètre de mise à l’échelle automatique contient à présent les trois profils expliqués précédemment.

7. Examinons pour finir la section **notification** de la mise à l’échelle automatique. Les notifications de mise à l’échelle automatique permettent de faire trois choses au moment du déclenchement d’une action de diminution ou d’augmentation de la taille des instances.
   - Avertir l’administrateur et les co-administrateurs de votre abonnement
   - Envoyer un e-mail à un ensemble d’utilisateurs
   - Déclencher un appel webhook. Quand il est déclenché, ce webhook envoie les métadonnées relatives à la condition de mise à l’échelle automatique et à la ressource du groupe de machines virtuelles identiques. Pour plus d’informations sur la charge utile du webhook de mise à l’échelle automatique, consultez la page [Utilisation d’actions de mise à l’échelle automatique pour envoyer des notifications d’alerte webhook et par courrier électronique dans Azure Insights](autoscale-webhook-email.md).

   Ajoutez le code suivant au paramètre de mise à l’échelle automatique en remplaçant votre élément **notification** dont la valeur est null

   ```
   "notifications": [
      {
        "operation": "Scale",
        "email": {
          "sendToSubscriptionAdministrator": true,
          "sendToSubscriptionCoAdministrators": false,
          "customEmails": [
              "user1@mycompany.com",
              "user2@mycompany.com"
              ]
        },
        "webhooks": [
          {
            "serviceUri": "https://foo.webhook.example.com?token=abcd1234",
            "properties": {
              "optional_key1": "optional_value1",
              "optional_key2": "optional_value2"
            }
          }
        ]
      }
    ]

   ```

   Cliquez sur le bouton **Put** dans l’Explorateur de ressources pour mettre à jour le paramètre de mise à l’échelle automatique.

Vous avez mis à jour un paramètre de mise à l’échelle automatique sur un groupe de machines virtuelles identiques pour inclure plusieurs profils de mise à l’échelle ainsi que des notifications de mise à l’échelle.

## <a name="next-steps"></a>Étapes suivantes
Suivez ces liens pour en savoir plus sur la mise à l’échelle automatique.

[Dépanner la mise à l’échelle automatique avec des jeux de mise à l’échelle de machine virtuelle](../../virtual-machine-scale-sets/virtual-machine-scale-sets-troubleshoot.md)

[Métriques courantes pour la mise à l’échelle automatique d’Azure Insights](autoscale-common-metrics.md)

[Meilleures pratiques pour la mise à l’échelle automatique d’Azure Insights](autoscale-best-practices.md)

[Gérer les paramètres de mise à l’échelle automatique à l’aide de PowerShell](../samples/powershell-samples.md#create-and-manage-autoscale-settings)

[Gérer les paramètres de mise à l’échelle automatique à l’aide de la CLI](../samples/cli-samples.md#autoscale)

[Utilisation d’actions de mise à l’échelle automatique pour envoyer des notifications d’alerte webhook et par courrier électronique dans Azure Insights](autoscale-webhook-email.md)

Référence de modèle [Microsoft.Insights/autoscalesettings](/azure/templates/microsoft.insights/autoscalesettings)
