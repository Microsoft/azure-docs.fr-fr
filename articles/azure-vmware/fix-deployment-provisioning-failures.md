---
title: Support pour les échecs de provisionnement ou de déploiement d’Azure VMware Solution
description: Obtenez les informations dont vous avez besoin à partir de votre cloud privé Azure VMware Solution, et traitez une demande de service pour un échec de déploiement ou de provisionnement d’Azure VMware Solution.
ms.topic: how-to
ms.date: 10/28/2020
ms.openlocfilehash: 27b645f4ca225fdd74bca6499b6581b3803e41a4
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542403"
---
# <a name="open-a-support-request-for-an-azure-vmware-solution-deployment-or-provisioning-failure"></a>Ouvrir une demande de support pour un échec de provisionnement ou de déploiement d’Azure VMware Solution

Cet article vous montre comment ouvrir une [demande de support](https://rc.portal.azure.com/#create/Microsoft.Support) et fournir des informations essentielles pour un échec de provisionnement ou de déploiement d’Azure VMware Solution. 

Lorsqu’un échec survient sur votre cloud privé, vous devez ouvrir une demande de support dans le portail Azure. Pour ouvrir une demande de support, commencez par récupérer quelques informations indispensables dans le portail Azure :

- ID de corrélation :
- ID de circuit Azure ExpressRoute
- Messages d’erreur

## <a name="get-the-correlation-id"></a>Récupérer l’ID de corrélation
 
Lorsque vous créez un cloud privé ou une ressource dans Azure, un ID de corrélation lié à la ressource est automatiquement généré pour cette ressource. Incluez l’ID de corrélation du cloud privé dans votre demande de support pour ouvrir et résoudre plus rapidement la demande.

Dans le portail Azure, vous pouvez obtenir l’ID de corrélation d’une ressource de deux manières :

* Volet **Vue d’ensemble**
* Journaux de déploiement
 
 ### <a name="get-the-correlation-id-from-the-resource-overview"></a>Récupérer l’ID de corrélation à partir de la vue d’ensemble des ressources

Voici un exemple des détails de l’opération d’un déploiement de cloud privé ayant échoué, avec l’ID de corrélation mis en évidence :

:::image type="content" source="media/fix-deployment-provisioning-failures/failed-private-cloud-deployment.png" alt-text="Capture d’écran montrant un échec de déploiement du cloud privé avec l’ID de corrélation mis en évidence.":::

Pour accéder aux résultats du déploiement dans le volet **Vue d’ensemble** du cloud privé :

1. Dans le portail Azure, sélectionnez votre cloud privé.

1. Dans le menu de gauche, sélectionnez **Vue d’ensemble**.

Une fois le déploiement initié, les résultats du déploiement s’affichent dans le volet **Vue d’ensemble** du cloud privé.

Copiez et enregistrez l’ID de corrélation du déploiement dans le cloud privé pour l’inclure dans la demande de service.

### <a name="get-the-correlation-id-from-the-deployment-log"></a>Récupérer l’ID de corrélation à partir du journal de déploiement

Vous pouvez obtenir l’ID de corrélation d’un déploiement qui a échoué en effectuant une recherche dans le journal d’activité du déploiement sur le portail Azure.

Pour accéder au journal de déploiement :

1. Dans le portail Azure, sélectionnez votre cloud privé, puis sélectionnez l’icône des notifications.

   :::image type="content" source="media/fix-deployment-provisioning-failures/open-notifications.png" alt-text="Capture d’écran montrant l’icône des notifications dans le portail Azure.":::

1. Dans le volet **Notifications**, sélectionnez **Plus d’événements dans le journal d’activité** :

    :::image type="content" source="media/fix-deployment-provisioning-failures/more-events-in-activity-log.png" alt-text="Capture d’écran montrant dans le volet Notifications la sélection du lien « Plus d’événements dans le journal d’activité ».":::

1. Pour trouver le déploiement qui a échoué et son ID de corrélation, recherchez le nom de la ressource ou d’autres informations que vous avez utilisées pour créer cette ressource. 

    L’exemple suivant montre les résultats de la recherche d’une ressource de cloud privé nommée pc03.
 
    :::image type="content" source="media/fix-deployment-provisioning-failures/find-past-deployments.png" alt-text="Capture d’écran affichant les résultats de la recherche d’un exemple de ressource de cloud privé et le volet Créer ou mettre à jour un PrivateCloud.":::
 
1. Dans les résultats de la recherche du volet **Journal d’activité**, sélectionnez le nom d’opération du déploiement qui a échoué.

1. Dans le volet **Créer ou mettre à jour un PrivateCloud**, sélectionnez l’onglet **JSON**, puis recherchez `correlationId` dans le journal qui s’affiche. Copiez la valeur `correlationId` pour l’inclure dans votre demande de support. 
 
## <a name="copy-error-messages"></a>Copier les messages d’erreur

Pour faciliter la résolution de votre problème de déploiement, incluez tous les messages d’erreur affichés dans le portail Azure. Sélectionnez un message d’avertissement pour consulter un résumé des erreurs :
 
:::image type="content" source="media/fix-deployment-provisioning-failures/summary-of-errors.png" alt-text="Capture d’écran affichant les détails de l’erreur sous l’onglet Résumé du volet Erreurs ; l’icône de copie est sélectionnée.":::

Pour copier le message d’erreur, sélectionnez l’icône de copie. Enregistrez le message copié à inclure dans votre demande de support.
 
## <a name="get-the-expressroute-id-uri"></a>Récupérer l’ID ExpressRoute (URI)
 
Il se peut que vous tentiez de mettre à l’échelle ou d’appairer un cloud privé existant avec le circuit ExpressRoute de cloud privé, et que cela échoue. Dans ce scénario, vous avez besoin d’inclure l’ID ExpressRoute dans votre demande de support.

Pour copier l’ID ExpressRoute :

1. Dans le portail Azure, sélectionnez votre cloud privé.
1. Dans le menu de gauche, sous **Gérer**, sélectionnez **Connectivité**. 
1. Dans le volet droit, sélectionnez l’onglet **ExpressRoute**.
1. Sélectionnez l’icône de copie pour **ID ExpressRoute** et enregistrez la valeur à utiliser dans votre demande de support.
 
:::image type="content" source="media/fix-deployment-provisioning-failures/expressroute-id.png" alt-text="Copiez l’ID ExpressRoute dans le presse-papiers."::: 
 
## <a name="pre-validation-failures"></a>Échecs de prévalidation

Si le contrôle de prévalidation de votre cloud privé a échoué (avant le déploiement), aucun ID de corrélation n’aura été généré. Dans ce scénario, vous pouvez fournir les informations suivantes dans votre demande de support :

- Les messages d’échec et d’erreur. Ces messages peuvent être utiles dans la résolution de nombreuses défaillances, par exemple les problèmes liés aux quotas. Il est important de copier ces messages et de les inclure à la demande de support, comme décrit dans cet article.
- Les informations que vous avez utilisées pour créer le cloud privé Azure VMware Solution, notamment :
  - Emplacement
  - Resource group
  - Nom de la ressource

## <a name="create-your-support-request"></a>Créer votre demande de support

Pour obtenir des informations générales sur la création d’une demande de support, consultez [Comment créer une demande de support Azure](../azure-portal/supportability/how-to-create-azure-support-request.md). 

Pour créer une demande de support relatif à un échec de provisionnement ou de déploiement d’Azure VMware Solution :

1. Dans le portail Azure, sélectionnez l’icône **Aide**, puis sélectionnez **Nouvelle demande de support**.

    :::image type="content" source="media/fix-deployment-provisioning-failures/open-sr-on-avs.png" alt-text="Capture d’écran du volet Nouvelle demande de support dans le portail Azure.":::

1. Entrez ou sélectionnez les informations nécessaires :

   1. Sous l’onglet **Général** :

      1. Pour **Type de problème**, sélectionnez **Problèmes de configuration et d’installation**.

      1. Pour **Sous-type de problème**, sélectionnez **Configurer un cloud privé**.

   1. Sous l’onglet **Détails** :

      1. Entrez ou sélectionnez les informations nécessaires.

      1. Collez votre ID de corrélation ou votre ID ExpressRoute à l’endroit où cette information est demandée. Si vous ne voyez pas de zone de texte spécifique pour ces valeurs, collez-les dans la zone de texte **Fournissez des détails sur le problème.**

    1. Collez tous les détails de l’erreur, notamment les messages d’échec ou d’erreur que vous avez copiés, dans la zone de texte **Fournissez des détails sur le problème**.

1. Passez en revue vos entrées, puis sélectionnez **Créer** pour générer votre demande de support.
