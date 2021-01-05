---
title: Suppression des faux positifs ou autres alertes de sécurité indésirables dans Azure Security Center à l’aide de règles de suppression d’alerte
description: Cet article explique comment utiliser les règles de suppression d’Azure Security Center pour masquer les alertes de sécurité indésirables.
author: memildin
manager: rkarlin
services: security-center
ms.author: memildin
ms.date: 09/10/2020
ms.service: security-center
ms.topic: how-to
ms.openlocfilehash: 1ca2ded69b0279a60d8ed83cf310a58dadf1a337
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751986"
---
# <a name="suppress-alerts-from-azure-defender"></a>Supprimer les alertes d’Azure Defender

Cette page explique comment utiliser les règles de suppression d’alerte pour supprimer les faux positifs ou autres alertes de sécurité indésirables envoyées par Azure Defender.

## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :| Mise à la disposition générale |
|Prix :|Gratuit<br>(La plupart des alertes de sécurité sont uniquement disponibles avec Azure Defender)|
|Rôles et autorisations obligatoires :|**L’Administrateur de la sécurité** et le **Propriétaire** peuvent créer/supprimer des règles.<br>Le **Lecteur de sécurité** et le **Lecteur** peuvent consulter les règles.|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Oui](./media/icons/yes-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||


## <a name="what-are-suppression-rules"></a>Que sont les règles de suppression ?

Les différents plans Azure Defender détectent les menaces dans n’importe quelle zone de votre environnement et génèrent des alertes de sécurité.

Quand une alerte n’est pas intéressante ni pertinente, il est possible de la masquer manuellement. Vous pouvez également utiliser la fonctionnalité de règles de suppression pour ignorer automatiquement les alertes similaires à l’avenir. En règle générale, une règle de suppression sert à :

- supprimer des alertes identifiées comme faux positifs ;

- supprimer des alertes déclenchées trop souvent pour être utiles.

Les règles de suppression définissent les critères en fonction desquels les alertes doivent être automatiquement ignorées.

> [!CAUTION]
> La suppression des alertes de sécurité réduit la protection contre les menaces d’Azure Defender. Vérifiez soigneusement l’impact potentiel d’une règle de suppression et surveillez-la au fil du temps.

:::image type="content" source="./media/alerts-suppression-rules/create-suppression-rule.gif" alt-text="Créer une règle de suppression d’alerte":::

## <a name="create-a-suppression-rule"></a>Créer une règle de suppression

Il existe plusieurs façons de créer des règles permettant de supprimer les alertes de sécurité indésirables :

- Pour supprimer des alertes au niveau du groupe d’administration, utilisez Azure Policy.
- Pour supprimer des alertes au niveau de l’abonnement, vous pouvez utiliser le Portail Azure ou l’API REST (cf. ci-dessous).

Les règles de suppression ne peuvent masquer que les alertes qui ont déjà été déclenchées sur les abonnements sélectionnés.

Pour créer une règle directement sur le Portail Azure :

1. Sur la page des alertes de sécurité de Security Center :

    - Recherchez l’alerte que vous ne voulez plus voir, puis sélectionnez **Créer une règle de suppression** dans le menu points de suspension (…) correspondant :

        [![Option **Créer une règle de suppression**](media/alerts-suppression-rules/auto-dismiss-future-option.png)](media/alerts-suppression-rules/auto-dismiss-future-option.png#lightbox)

    - Vous pouvez également sélectionner le lien **Règles de suppression** en haut de la page, puis sélectionner **Créer une règle de suppression** sur la page des règles de suppression :

        ![Bouton Créer une règle de suppression**](media/alerts-suppression-rules/create-new-suppression-rule.png)

1. Dans le volet de nouvelle règle de suppression, entrez les détails de la nouvelle règle.
    - Votre règle peut ignorer l’alerte sur **toutes les ressources** ; vous ne recevrez alors plus aucune alerte de ce type.     
    - Votre règle peut ignorer l'alerte **selon des critères spécifiques**, c’est-à-dire lorsqu’elle est associée à une adresse IP, un nom de processus, un compte d’utilisateur, une ressource Azure ou un emplacement spécifique.

    > [!TIP]
    > Si vous avez ouvert la page de nouvelle règle à partir d’une alerte spécifique, l’alerte et l’abonnement sont automatiquement configurés dans la nouvelle règle. Si vous avez utilisé le lien **Créer une règle de suppression**, les abonnements sélectionnés correspondent au filtre actuel sur le portail.

    [![Volet de création d’une règle de suppression](media/alerts-suppression-rules/new-suppression-rule-pane.png)](media/alerts-suppression-rules/new-suppression-rule-pane.png#lightbox)
1. Entrez les détails de la règle :
    - **Nom** : nom de la règle. Les noms de règles doivent commencer par une lettre ou un chiffre, comporter entre 2 et 50 caractères, et ne contenir aucun symbole autre que des tirets (-) ou des traits de soulignement (_). 
    - **État** : activé ou désactivé.
    - **Raison** : sélectionnez l’une des raisons prédéfinies ou « autre » si elles ne répondent pas à vos besoins.
    - **Date d’expiration** : date et heure de fin de la règle. Les règles peuvent s’exécuter pendant six mois maximum.
1. Si vous le souhaitez, testez la règle à l’aide du bouton **Simuler** pour voir combien d’alertes auraient été ignorées si cette règle avait été active.
1. Enregistrez la règle. 


## <a name="edit-a-suppression-rules"></a>Modifier les règles de suppression

Pour modifier une règle que vous avez créée, accédez à la page des règles de suppression.

1. Sur la page des alertes de sécurité de Security Center, sélectionnez le lien **Règles de suppression** qui se trouve en haut.
1. La page règles de suppression s’ouvre avec toutes les règles des abonnements sélectionnés.

    [![Liste des règles de suppression](media/alerts-suppression-rules/suppression-rules-page.png)](media/alerts-suppression-rules/suppression-rules-page.png#lightbox)

1. Pour modifier une seule règle, ouvrez le menu points de suspension (…) de la règle, puis sélectionnez **Modifier**.
1. Apportez les modifications nécessaires, puis sélectionnez **Appliquer**. 

## <a name="delete-a-suppression-rule"></a>Supprimer une règle de suppression

Pour supprimer une ou plusieurs des règles que vous avez créées, accédez à la page des règles de suppression.

1. Sur la page des alertes de sécurité de Security Center, sélectionnez le lien **Règles de suppression** qui se trouve en haut.
1. La page règles de suppression s’ouvre avec toutes les règles des abonnements sélectionnés.
1. Pour supprimer une seule règle, ouvrez le menu points de suspension (…) de la règle, puis sélectionnez **Supprimer**.
1. Pour supprimer plusieurs règles, cochez les cases correspondantes, puis sélectionnez **Supprimer**.
    ![Suppression d’une ou plusieurs règles de suppression](media/alerts-suppression-rules/delete-multiple-alerts.png)

## <a name="view-suppressed-alerts"></a>Afficher les alertes supprimées

Les alertes correspondant aux règles de suppression activées sont toujours générées, mais leur état est défini sur **ignoré**. Vous pouvez voir l’état sur le Portail Azure ou en accédant à vos alertes de sécurité Security Center suivant la méthode de votre choix. 

> [!TIP]
> [Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/) ne crée pas d’incidents pour les alertes supprimées. Dans le cas des autres SIEM, vous pouvez filtrer les alertes supprimées selon l’état des alertes (« ignoré »).

Utilisez le filtre de Security Center pour afficher les alertes ignorées par vos règles.

* Sur la page des alertes de sécurité de Security Center, ouvrez les options de filtre et sélectionnez **Ignoré**.  

   [![Affichage des alertes ignorées](media/alerts-suppression-rules/view-dismissed-alerts.png)](media/alerts-suppression-rules/view-dismissed-alerts.png#lightbox)


## <a name="create-and-manage-suppression-rules-with-the-api"></a>Créer et gérer des règles de suppression avec une API

Vous pouvez utiliser l’API REST de Security Center pour créer, afficher ou supprimer des règles de suppression d’alerte. 

Les méthodes HTTP de l’API REST adaptées aux règles de suppression sont les suivantes :

- **PUT** : Pour créer ou mettre à jour une règle de suppression dans un abonnement spécifié.

- **GET** :

    - Pour lister toutes les règles configurées pour un abonnement spécifié. Cette méthode retourne le tableau des règles applicables.

    - Pour obtenir les détails d’une règle spécifique sur un abonnement spécifié. Cette méthode retourne une règle de suppression.

    - Pour simuler l’impact d’une règle de suppression encore en phase de conception. Cet appel identifie les alertes existantes qui auraient été ignorées si la règle avait été active.

- **DELETE** : Pour supprimer une règle existante (sans modifier l’état des alertes déjà ignorées par celle-ci).

Pour plus d’informations et des exemples d’utilisation, consultez la [documentation de l’API](/rest/api/securitycenter/). 


## <a name="next-steps"></a>Étapes suivantes

Cet article a décrit les règles de suppression d’Azure Security Center qui ignorent automatiquement les alertes indésirables.

Pour plus d’informations sur les alertes de sécurité d’Azure Defender, consultez les pages suivantes :

- [Alertes de sécurité et chaîne de destruction d’intention](alerts-reference.md) : Guide de référence sur les alertes de sécurité envoyées par Azure Defender.