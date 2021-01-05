---
title: Affichage des tendances de coût de laboratoire mensuelles estimées dans Azure DevTest Labs
description: Cet article fournit des informations sur le suivi du coût de votre laboratoire (graphique de tendance des coûts mensuels estimés) dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 220cbd1bc793e4bcfdf03a184d7463114506e6e0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87283824"
---
# <a name="track-costs-associated-with-a-lab-in-azure-devtest-labs"></a>Suivre les coûts associés à un laboratoire dans Azure DevTest Labs
Cet article fournit des informations sur la façon de suivre le coût de votre laboratoire. Il explique la procédure d’affichaeg de la tendance des coûts estimés du labo pour le mois en cours. L’article vous montre également comment afficher le coût du mois en cours par ressource du laboratoire.

## <a name="view-the-monthly-estimated-lab-cost-trend"></a>Afficher les tendances de coût mensuelles estimées du laboratoire 
Dans cette section, découvrez comment utiliser le graphique **Tendance des coûts mensuels estimés** pour afficher le coût estimé jusqu’à la date actuelle ainsi que le coût projeté pour la fin du mois en cours. Vous apprenez également comment gérer les coûts de laboratoire en définissant des seuils et des dépenses cibles qui, une fois atteints, déclenchent la génération de résultats par DevTest Labs.

Pour afficher le graphique Tendance des coûts mensuels estimés, procédez comme suit : 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
3. Dans la liste de laboratoires, sélectionnez votre laboratoire.  
4. Sélectionnez **Configuration et stratégies** dans le menu de gauche.  
4. Sélectionnez **Tendance des coûts** dans la section **Suivi des coûts** du menu de gauche. La capture d'écran suivante montre un exemple de graphique de coûts. 
   
    ![Graphique de coûts](./media/devtest-lab-configure-cost-management/graph.png)

    La valeur **Coût estimé** est le coût estimé à ce jour pour le mois calendaire en cours. Le **Coût projeté** est le coût estimé pour le mois calendaire entier, calculé à l’aide du coût du laboratoire lors des cinq derniers jours.

    Les montants des coûts sont arrondis à l’entier supérieur. Par exemple : 

   * 5,01 est arrondi à 6 
   * 5,50 est arrondi à 6
   * 5,99 est arrondi à 6

     Comme indiqué au-dessus du graphique, les coûts que vous voyez par défaut dans le graphique sont les coûts *estimés* avec les tarifs de l’offre [Paiement à l'utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/). Vous pouvez également définir vos propres dépenses cibles à afficher dans les graphiques en [gérant les coûts cibles de votre laboratoire](#managing-cost-targets-for-your-lab).

     Les coûts suivants ne sont *pas* inclus dans le calcul des coûts :

   * Les abonnements CSP et Dreamspark ne sont pas pris en charge. En effet, Azure DevTest Labs utilise les [API de facturation Azure](../cost-management-billing/manage/usage-rate-card-overview.md) pour calculer les coûts de laboratoire, et celles-ci ne prennent pas en charge les abonnements CSP et Dreamspark.
   * Les tarifs de votre offre. Actuellement, vous ne pouvez pas utiliser les tarifs de l’offre (affichés sous votre abonnement) que vous avez négociés avec Microsoft ou les partenaires Microsoft. Seuls les tarifs du paiement à l'utilisation sont disponibles.
   * Vos taxes
   * Vos remises
   * Votre devise de facturation. Actuellement, le coût du labo s'affiche uniquement dans la devise USD.

### <a name="managing-cost-targets-for-your-lab"></a>Gestion des coûts cibles de votre laboratoire
DevTest Labs vous aide à mieux gérer les coûts de votre laboratoire en définissant des dépenses cibles que vous pouvez ensuite afficher dans le graphique de tendance du coût estimé par mois. DevTest Labs peut également vous envoyer une notification quand le seuil ou les dépenses cibles spécifiés sont atteints. 

1. À la page **Tendance des coûts**, sélectionnez **Gérer la cible**.

    ![Bouton Gérer la cible](./media/devtest-lab-configure-cost-management/cost-trend-manage-target.png)
2. À la page **Gérer la cible**, spécifiez les dépenses cibles et des seuils. Vous pouvez également indiquer si chaque seuil sélectionné doit figurer sur le graphique de tendance du coût ou être communiqué via une notification Webhook.

    ![Volet Gérer la cible](./media/devtest-lab-configure-cost-management/cost-trend-manage-target-pane.png)

   - Sélectionnez la période de temps durant laquelle vos coûts cibles doivent être suivis.
      - **Tous les mois** : le suivi des coûts cibles est effectué mensuellement.
      - **Fixe** : le suivi des coûts cibles est effectué dans la plage de dates que vous spécifiez dans les champs des dates de début et de fin. Généralement, ces valeurs représentent la durée d’exécution prévue de votre projet.
   - Spécifiez un **coût cible**. Par exemple, le montant que vous prévoyez de dépenser pour ce laboratoire dans la période de temps que vous avez définie.
   - Sélectionnez l’option appropriée pour activer ou désactiver un seuil à rapporter (par incréments de 25 %), jusqu’à 125 % du **coût cible** spécifié.
      - **Informer** : quand ce seuil est atteint, vous êtes informé par une URL webhook que vous avez spécifiée.
      - **Tracer sur le graphique** : quand ce seuil est atteint, les résultats sont tracés sur le graphique de tendance du coût que vous pouvez afficher, comme décrit dans Affichage du graphique de tendance du coût estimé mensuel.
   - Si vous choisissez l’option **Notifier** pour l’envoi d’une notification quand le seuil est atteint, vous devez spécifier une URL Webhook. Dans la zone Intégrations des coûts, sélectionnez **Cliquez ici pour ajouter une intégration**. Entrez une **URL Webhook** dans le volet Configurer la notification, puis sélectionnez **OK**.

       ![Volet Configurer la notification](./media/devtest-lab-configure-cost-management/configure-notification-new.png)

     - Si vous sélectionnez l’option **Notifier**, vous devez définir une URL Webhook.
     - De même, si vous définissez une URL Webhook, vous devez définir le paramètre **Notification** sur **Activé** dans le volet Seuil de coût.
     - Vous devez créer un Webhook avant de l’indiquer ici.  

       Pour plus d’informations sur les webhooks, consultez [Créer une fonction Azure d’API ou de webhook](../azure-functions/functions-bindings-http-webhook.md). 

## <a name="view-cost-by-resource"></a>Afficher le coût par ressource 
La fonctionnalité de tendance de coût mensuel des laboratoires permet de voir combien vous avez dépensé pendant le mois en cours. Elle montre également la projection des dépenses jusqu'à la fin du mois sur la base de vos dépenses au cours des sept derniers jours. Pour vous aider à comprendre dès le départ pourquoi les dépenses du laboratoire atteignent des seuils, vous pouvez utiliser la fonctionnalité **coût par ressource**, qui vous montre le coût mensuel jusqu’à la date actuelle **par ressource** dans une table.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
3. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  
4. Sélectionnez **Configuration et stratégies** dans le menu de gauche.
5. Sélectionnez **Coût par ressource** dans la section **Suivi des coûts** du menu de gauche. Vous voyez les coûts associés à chaque ressource liée à un laboratoire. 

    ![Coût par ressource](./media/devtest-lab-configure-cost-management/cost-by-resource.png)

Cette fonctionnalité aide à identifier facilement les ressources les plus onéreuses, donc à prendre des mesures pour réduire les dépenses de laboratoire. Par exemple, le coût d’une machine virtuelle est basé sur sa taille. Plus la machine virtuelle est grande, plus elle coûte cher. Vous pouvez facilement trouver la taille d’une machine virtuelle et son propriétaire, puis vous adresser à lui pour comprendre pourquoi il a besoin d’une machine virtuelle de cette taille et s’il est possible de la réduire.

Une [stratégie d’arrêt automatique](devtest-lab-set-lab-policy.md?#set-auto-shutdown-policy) aide à réduire le coût en arrêtant les machines virtuelles du laboratoire à une heure précise de la journée. Toutefois, un utilisateur de laboratoire peut refuser de la stratégie d’arrêt, ce qui augmente le coût d’exécution de la machine virtuelle. Vous pouvez sélectionner une machine virtuelle dans le tableau pour voir si elle a été exclue de la stratégie d’arrêt automatique. Le cas échéant, vous pouvez communiquer avec son propriétaire afin de savoir pourquoi la machine virtuelle a été exclue de la stratégie.
 
## <a name="next-steps"></a>Étapes suivantes
Voici quelques possibilités d’opérations pour la suite :

* [Définir des stratégies de laboratoire](devtest-lab-set-lab-policy.md) : apprenez à définir les différentes stratégies utilisées pour gérer l’utilisation de votre laboratoire et de ses machines virtuelles. 
* [Créer une image personnalisée](devtest-lab-create-template.md) : quand vous créez une machine virtuelle, vous spécifiez une base, qui peut être soit une image personnalisée, soit une image Marketplace. Cet article explique comment créer une image personnalisée à partir d’un fichier VHD.
* [Configurer des images Marketplace](devtest-lab-configure-marketplace-images.md) : DevTest Labs prend en charge la création de machines virtuelles basées sur des images Azure Marketplace. Cet article explique comment spécifier, le cas échéant, les images Azure Marketplace pouvant être utilisées lors de la création de machines virtuelles dans un laboratoire.
* [Créer une machine virtuelle dans un laboratoire](devtest-lab-add-vm.md) : montre comment créer une machine virtuelle à partir d’une image de base (personnalisée ou Place de marché) et comment utiliser des artefacts dans votre machine virtuelle.
