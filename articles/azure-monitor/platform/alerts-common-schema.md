---
title: Schéma d’alerte commun pour les alertes Azure Monitor
description: Comprendre le schéma d’alerte commun, pourquoi vous devez l’utiliser et comment l’activer
ms.topic: conceptual
ms.subservice: alerts
ms.date: 03/14/2019
ms.openlocfilehash: aa619976c8fa03b925d66e884ad03fc4e385693e
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94565600"
---
# <a name="common-alert-schema"></a>Schéma d’alerte courant

Cet article décrit le schéma d’alerte commun, les avantages que procure son utilisation et la façon de l’activer.

## <a name="what-is-the-common-alert-schema"></a>Qu’est-ce que le schéma d’alerte commun ?

Le schéma d’alerte commun standardise l’expérience de consommation pour les notifications d’alerte dans Azure de nos jours. Les trois types d’alerte actuels dans Azure (métrique, journal et journal d’activité) ont traditionnellement eu leurs propres modèles d’e-mail, schémas de webhook, etc. Avec le schéma d’alerte commun, vous pouvez maintenant recevoir des notifications d’alerte avec un schéma cohérent.

Toute instance d’alerte décrit **la ressource qui a été affectée** et **la cause de l’alerte** ; ces instances sont décrites dans le schéma commun dans les sections suivantes :
* **Informations de base** : ensemble de **champs standardisés**, commun à tous les types d’alerte, qui décrivent **quelle ressource** est concernée par l’alerte ainsi que des métadonnées d’alerte courantes supplémentaires (par exemple, la gravité ou un description). 
* **Contexte de l’alerte** : ensemble de champs qui décrivent la **cause de l’alerte**, et qui varient **selon le type d’alerte**. Par exemple, une alerte de métrique aurait des champs tels que le nom de la métrique et la valeur de la métrique dans le contexte de l’alerte, tandis qu’une alerte de journal d’activité aurait des informations sur l’événement à l’origine de l’alerte. 

Les scénarios d’intégration classiques dont nous font part nos clients impliquent le routage de l’instance d’alerte vers l’équipe concernée en fonction d’une information pivot (par exemple, le groupe de ressources), avant son traitement par cette équipe. Avec le schéma d’alerte commun, vous pouvez avoir une logique de routage standardisée entre les types d’alerte en exploitant les champs essentiels, laissant aux équipes concernées le soin d’approfondir éventuellement les recherches à partir des champs de contexte bruts.

Cela signifie que vous pouvez avoir moins d’intégrations, qui dès lors sont _beaucoup_ plus simples à gérer et à mettre à jour. En outre, les enrichissements futurs des charges utiles d’alerte (par exemple, personnalisation, enrichissement des diagnostics, etc.) ne transparaîtront que dans le schéma commun.

## <a name="what-enhancements-does-the-common-alert-schema-bring"></a>Quelles sont les améliorations apportées par le schéma d’alerte commun ?

Le schéma d’alerte commun se manifeste principalement dans vos notifications d’alerte. Les améliorations que vous verrez sont listées ci-dessous :

| Action | Améliorations|
|:---|:---|
| sms | Modèle de SMS cohérent pour tous les types d’alerte. |
| E-mail | Un modèle d’e-mail cohérent et détaillé, ce qui vous permet de facilement diagnostiquer les problèmes en un clin d’œil. Des liens profonds incorporés vers l’instance d’alerte sur le portail et la ressource affectée vous permettent d’accéder rapidement au processus de correction. |
| Webhook/application logique/fonction Azure/runbook automation | Une structure JSON cohérente pour tous les types d’alertes, ce qui vous permet de générer facilement des intégrations entre les différents types d’alerte. |

En outre, le nouveau schéma enrichira l’expérience de consommation des alertes dans le portail Azure et Azure mobile app dans un futur proche. 

[Découvrez-en plus sur les définitions de schéma pour les webhooks/applications logiques/fonctions Azure/runbooks automation.](./alerts-common-schema-definitions.md)

> [!NOTE]
> Les actions suivantes ne prennent pas en charge le schéma d’alerte commun : connecteur ITSM.

## <a name="how-do-i-enable-the-common-alert-schema"></a>Comment faire pour activer le schéma d’alerte commun ?

Vous pouvez adhérer ou ne pas adhérer au schéma d’alerte commun par le biais des groupes d’actions, sur le portail et via l’API REST. Le bouton bascule pour basculer vers le nouveau schéma existe au niveau d’une action. Par exemple, vous devez adhérer séparément à une action d’e-mail et à une action de webhook.

> [!NOTE]
> 1. Les types d’alerte suivants prennent en charge le schéma commun par défaut (aucune adhésion requise) :
>     * Alertes de détection intelligente
> 1. Les types d’alerte suivants ne prennent pas en charge le schéma commun :
>     * Alertes générées par [Azure Monitor pour machines virtuelles](../insights/vminsights-overview.md)
>     * Alertes générées par [Azure Cost Management](../../cost-management-billing/manage/cost-management-budget-scenario.md)

### <a name="through-the-azure-portal"></a>À l’aide du portail Azure

![Adhésion au schéma d’alerte commun](media/alerts-common-schema/portal-opt-in.png)

1. Ouvrez n’importe quelle action existante ou une nouvelle action dans un groupe d’actions. 
1. Sélectionnez « Oui » pour le bouton bascule pour activer le schéma d’alerte commun, comme indiqué.

### <a name="through-the-action-groups-rest-api"></a>Par le biais de l’API REST Groupes d’actions

Vous pouvez également utiliser l’[API Groupes d’actions](/rest/api/monitor/actiongroups) pour adhérer au schéma d’alerte commun. Tout en effectuant l’appel d’API REST [Créer ou mettre à jour](/rest/api/monitor/actiongroups/createorupdate), vous pouvez définir l’indicateur « useCommonAlertSchema » sur « true » (adhérer) ou « false » (ne pas adhérer) pour les actions suivantes : e-mail/webhook/application logique/fonction Azure/runbook automation.

Par exemple, le corps de demande suivant apporté à l’API REST [Créer ou mettre à jour](/rest/api/monitor/actiongroups/createorupdate) effectue les opérations suivantes :

* Activer le schéma d’alerte commun pour l’action d’e-mail « John Doe's email »
* Désactiver le schéma d’alerte commun pour l’action d’e-mail « Jane Smith's email »
* Activer le schéma d’alerte commun pour l’action de webhook « Sample webhook »

```json
{
  "properties": {
    "groupShortName": "sample",
    "enabled": true,
    "emailReceivers": [
      {
        "name": "John Doe's email",
        "emailAddress": "johndoe@email.com",
        "useCommonAlertSchema": true
      },
      {
        "name": "Jane Smith's email",
        "emailAddress": "janesmith@email.com",
        "useCommonAlertSchema": false
      }
    ],
    "smsReceivers": [
      {
        "name": "John Doe's mobile",
        "countryCode": "1",
        "phoneNumber": "1234567890"
      },
      {
        "name": "Jane Smith's mobile",
        "countryCode": "1",
        "phoneNumber": "0987654321"
      }
    ],
    "webhookReceivers": [
      {
        "name": "Sample webhook",
        "serviceUri": "http://www.example.com/webhook",
        "useCommonAlertSchema": true
      }
    ]
  },
  "location": "Global",
  "tags": {}
}
```





## <a name="next-steps"></a>Étapes suivantes

- [Définitions de schéma d’alerte commun pour les webhooks/applications logiques/fonctions Azure/runbooks automation.](./alerts-common-schema-definitions.md)
- [Découvrez comment créer une application logique qui s’appuie sur le schéma d’alerte courant pour gérer toutes vos alertes.](./alerts-common-schema-integrations.md)
