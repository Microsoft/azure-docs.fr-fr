---
title: 'Tutoriel : Examiner l’utilisation et les coûts avec Cloudyn dans Azure'
description: Dans ce didacticiel, vous vérifiez l’utilisation et les coûts pour effectuer le suivi des tendances, détecter le manque d’efficacité et créer des alertes.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ms.reviewer: benshy
ROBOTS: NOINDEX
ms.openlocfilehash: afdcd458dbda282a23b7b8f7f1cd8459bc8ec5c5
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131273"
---
<!-- Intent: As a cloud-consuming user, I need to view usage and costs for my cloud resources and services.
-->

# <a name="tutorial-review-usage-and-costs"></a>Tutoriel : Réviser l’utilisation et les coûts

Cloudyn vous montre l’utilisation et les coûts pour que vous puissiez effectuer le suivi des tendances, détecter le manque d’efficacité et créer des alertes. Toutes les données d’utilisation et de coût sont affichées dans les rapports et tableaux de bord Cloudyn. Les exemples de ce didacticiel vous montrent comment réviser l’utilisation et les coûts à l’aide de tableaux de bord et rapports.

Azure Cost Management offre des fonctionnalités similaires à Cloudyn. Azure Cost Management est une solution native de gestion des coûts Azure. Il vous permet d’analyser les coûts, de créer et de gérer des budgets, d’exporter des données, et de consulter des recommandations d’optimisation et d’agir en conséquence pour dépenser moins. Pour plus d’informations, consultez [Azure Cost Management](../cost-management-billing-overview.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Suivre les tendances d’utilisation et de coût
> * Détecter le manque d’efficacité au niveau de l’utilisation
> * Créer des alertes pour dépenses inhabituelles ou excessives
> * Exporter des données

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’un compte Azure.
- Vous devez disposer d’une inscription à un essai gratuit ou d’un abonnement payant pour Cloudyn.

## <a name="open-the-cloudyn-portal"></a>Ouvrir le portail Cloudyn

Vous révisez toutes les données d’utilisation et de coût dans le portail Cloudyn. Ouvrez le portail Cloudyn à partir du portail Azure ou accédez à https://azure.cloudyn.com et connectez-vous.

## <a name="track-usage-and-cost-trends"></a>Suivre les tendances d’utilisation et de coût

Vous suivez l’argent dépensé au niveau de l’utilisation et des coûts à travers des rapports dans le temps pour identifier les tendances. Pour commencer à analyser les tendances, utilisez le rapport des coûts réels dans le temps. En haut à gauche du portail, cliquez sur **Coûts** > **Analyse des coûts** > **Coûts réels dans le temps**. Lorsque vous ouvrez le rapport pour la première fois, aucun groupe ou filtre n’est appliqué à ce dernier.

Voici un exemple de rapport :

![Exemple de rapport des coûts réels dans le temps](./media/tutorial-review-usage/actual-cost01.png)

Le rapport affiche toutes les dépenses au cours des 30 derniers jours. Pour afficher uniquement les dépenses pour les services Azure, appliquez le groupe Service et filtrez tous les services Azure. L’illustration suivante montre les services filtrés.

![Exemple montrant les services Azure filtrés](./media/tutorial-review-usage/actual-cost02.png)

Dans l’exemple précédent, la quantité d’argent dépensée a baissé à partir du 29/10/2018. Notez toutefois qu’un trop grand nombre de colonnes peut masquer une tendance évidente. Vous pouvez modifier l’affichage du rapport sous forme d’un graphique en courbes ou en aires pour visualiser les données affichées dans d’autres vues. L’illustration suivante montre plus clairement la tendance.

![Exemple illustrant une tendance à la baisse des coûts de machines virtuelles Azure](./media/tutorial-review-usage/actual-cost03.png)

Toujours avec cet exemple, vous pouvez voir que le coût de la machine virtuelle Azure a baissé. Les coûts des autres services Azure a également commencé à baisser ce jour-là. Pourquoi cette baisse des dépenses ? Dans cet exemple, un projet de travail important a été effectué. La consommation de nombreux services Azure a également baissé.

Pour visionner un tutoriel vidéo sur le suivi de l’utilisation et des tendances des coûts, consultez [Analyse des données de facturation cloud en fonction du temps avec Cloudyn](https://youtu.be/7LsVPHglM0g).

## <a name="detect-usage-inefficiencies"></a>Détecter le manque d’efficacité au niveau de l’utilisation

Les rapports de l’optimiseur améliorent l’efficacité, optimisent l’utilisation et identifient les moyens d’économiser de l’argent au niveau des ressources de cloud. Ils sont particulièrement utiles avec des recommandations de dimensionnement économique destinées à faciliter la réduction du nombre de machines virtuelles inactives ou coûteuses.

La stratégie de virtualisation des organisations est un problème qui affecte souvent ces dernières quand elles déplacent au départ des ressources vers le cloud. Elles utilisent souvent une approche similaire à celle qu’elles utilisaient pour créer des machines virtuelles pour l’environnement de virtualisation local. Elles partent également du principe que les coûts sont réduits en déplaçant leurs machines virtuelles locales vers le cloud, telles quelles. Mais il est peu probable que cette approche permette de réduire les coûts.

Le problème est que leur infrastructure existante a été déjà payée. Les utilisateurs pouvaient créer et faire fonctionner des machines virtuelles de taille importante s’ils le souhaitaient, qu’elles soient inactives ou pas, sans grandes conséquences. Le déplacement de machines virtuelles inactives ou de taille importante vers le cloud risque *d’augmenter* les coûts. L’affection des coûts des ressources est importante quand vous signez des contrats avec des fournisseurs de services cloud. Vous devez payer en fonction de votre engagement, que vous utilisiez entièrement la ressource ou non.

Le rapport de recommandations de dimensionnement économique identifie les économies annuelles potentielles en comparant la capacité des types d’instance de machine virtuelle à leurs données d’historique d’utilisation de la mémoire et de l’UC.  

Dans le menu en haut du portail, cliquez sur **Optimizer**(Optimiseur) > **Sizing Optimization**(Optimisation du dimensionnement) > **Cost Effective Sizing Recommendations**(Recommandations de dimensionnement économique). Si cela est utile, appliquez un filtre pour réduire les résultats. Cette image en fournit un exemple.

![Rapport de recommandations de dimensionnement rentable pour machines virtuelles Azure](./media/tutorial-review-usage/sizing01.png)

Dans cet exemple, l’organisation pourrait économiser 2 382 $ en suivant les recommandations pour modifier les types d’instance de machine virtuelle. Cliquez sur le signe plus (+) sous **Details (Détails)** pour la première recommandation. Voici les détails de la première recommandation.

![Exemple montrant des détails de recommandation](./media/tutorial-review-usage/sizing02.png)

Pour afficher les ID d’instance de machine virtuelle, cliquez sur le signe plus situé en regard de **List of Candidates (Liste de candidats)** .

![Exemple montrant une liste de candidats de machines virtuelles à redimensionner](./media/tutorial-review-usage/sizing03.png)

Pour visionner un tutoriel vidéo sur la détection des inefficacités d’utilisation, consultez [Optimisation de la taille de machine virtuelle dans Cloudyn](https://youtu.be/1xaZBNmV704).

Azure Cost Management fournit également des recommandations économiques pour les services Azure. Pour plus d’informations, consultez [Didacticiel : Optimiser les coûts à partir de recommandations](../costs/tutorial-acm-opt-recommendations.md).

## <a name="create-alerts-for-unusual-spending"></a>Créer des alertes pour dépenses inhabituelles

Les alertes vous permettent d’avertir automatiquement des parties prenantes en cas d’anomalies de dépenses et de risques de dépenses excessives. Vous pouvez créer des alertes à l’aide de rapports prenant en charge les alertes basées sur des seuils budgétaires et des seuils de coût.

Cet exemple utilise le rapport des **coûts réels dans le temps** pour envoyer une notification quand vos dépenses sur une machine virtuelle Azure s’approchent de votre budget total. Dans ce scénario, vous disposez d’un budget total de 20 000 $, et souhaitez recevoir une notification quand les coûts sont proches de la moitié de votre budget, 9 000 $, et une alerte supplémentaire quand ils atteignent 10 000 $.

1. Dans le menu en haut du portail Cloudyn, sélectionnez **Coûts** > **Analyse des coûts** > **Rapport des coûts réels dans le temps**.
2. Définissez **Groups (Groupes)** sur **Service** et **filtrez le service** sur **Azure/VM**.
3. Dans le coin supérieur droit du rapport, sélectionnez **Actions**, puis sélectionnez **Planifier le rapport**.
4. Pour recevoir un e-mail du rapport à intervalle planifié, sélectionnez l’onglet **Planification** dans la boîte de dialogue **Enregistrer ou planifier ce rapport**. Veillez à sélectionner **Envoyer par e-mail**. Tous les regroupements, filtrages et balises que vous utilisez sont inclus dans le rapport envoyé par e-mail.
5. Sélectionnez l’onglet **Seuil**, puis **Actual Cost vs. Threshold** (Coût réel vs seuil).
   1. Dans le seuil **Alerte rouge**, entrez 10000.
   2. Dans le seuil **Alerte jaune**, entrez 9000.
   3. Dans la zone **Nombre d’alertes consécutives**, entrez le nombre d’alertes consécutives à recevoir. Quand vous aurez reçu le nombre total d’alertes que vous avez spécifié, vous ne recevrez plus d’autres alertes.
6. Sélectionnez **Enregistrer**.

![Exemple illustrant les alertes rouges et jaunes basées sur les seuils de dépenses](./media/tutorial-review-usage/schedule-alert01.png)

Vous pouvez également sélectionner la métrique de seuil **Pourcentage du coût vs. budget** pour créer des alertes. Vous pouvez ainsi spécifier les seuils sous forme de pourcentages de votre budget, plutôt que sous forme de valeurs de devise.

## <a name="export-data"></a>Exporter des données

De la même façon que vous créez des alertes pour des rapports, vous pouvez aussi exporter des données depuis n’importe quel rapport. Par exemple, vous pouvez exporter une liste des comptes Cloudyn ou autres données utilisateur. Pour exporter un rapport, ouvrez-le puis, dans le coin supérieur droit du rapport, cliquez sur **Actions**. Vous pouvez par exemple **Exporter toutes les données d’un rapport**, de façon à télécharger ou imprimer les informations. Vous pouvez aussi sélectionner **Planifier le rapport** pour planifier l’envoi du rapport sous forme d’e-mail.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Suivre les tendances d’utilisation et de coût
> * Détecter le manque d’efficacité au niveau de l’utilisation
> * Créer des alertes pour dépenses inhabituelles ou excessives
> * Exporter des données


Passez au didacticiel suivant pour apprendre à prévoir les dépenses à partir des données historiques.

> [!div class="nextstepaction"]
> [Prévoir les dépenses futures](./tutorial-forecast-spending.md)