---
title: Usages courants de l’analyse des coûts dans Azure Cost Management
description: Cet article explique comment obtenir des résultats des tâches courantes d’analyse des coûts dans Azure Cost Management.
author: bandersmsft
ms.author: banders
ms.date: 07/24/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.openlocfilehash: c03bf5e52ae7f6b259c7b744e6033d760af23dd0
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683672"
---
# <a name="common-cost-analysis-uses"></a>Usages courants de l’analyse des coûts

Souvent, les utilisateurs d’Azure Cost Management souhaitent obtenir des réponses aux questions que beaucoup d’autres se posent. Cet article vous explique, pas à pas, comment obtenir des résultats des tâches courantes d’analyse des coûts dans Cost Management.

## <a name="view-forecasted-costs"></a>Afficher les coûts prévus

Les coûts prévus sont affichés dans les zones d’analyse des coûts pour les vues en zones et en colonnes empilées. La prévision est basée sur votre historique d’utilisation des ressources. Les modifications apportées à votre utilisation des ressources affectent les coûts prévus.

Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.

Dans la vue par défaut, le graphique du haut affiche les sections Coût réel/amorti et Coût prévu. La couleur unie du graphique affiche votre coût réel/amorti. La couleur ombrée indique le coût prévu.

[![Coût prévu](./media/cost-analysis-common-uses/enrollment-forecast.png)](./media/cost-analysis-common-uses/enrollment-forecast.png#lightbox)

## <a name="view-forecasted-costs-grouped-by-service"></a>Afficher les coûts prévus regroupés par service

La vue par défaut n’affiche pas les coûts prévus regroupés par service. Vous devez donc ajouter un groupe par sélection.

Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.

Sélectionnez **Regrouper par** > **Nom du service**.

La vue affiche vos coûts regroupés pour chaque service. Le coût prévu n’est pas calculé pour chaque service. Il est projeté pour le **total** de tous vos services.

[![Coût prévu groupé](./media/cost-analysis-common-uses/forecast-group-by-service.png)](./media/cost-analysis-common-uses/forecast-group-by-service.png#lightbox)

## <a name="view-forecasted-costs-for-a-service"></a>Afficher les coûts prévus pour un service

Vous pouvez afficher les coûts prévus limités à un seul service. Par exemple, vous souhaiterez peut-être consulter les coûts prévus pour les machines virtuelles uniquement.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Sélectionnez **Ajouter un filtre**, puis **Nom du service**.
1. Dans la liste **choisir**, sélectionnez un service. Par exemple, sélectionnez **Machines virtuelles**.

Examinez le coût réel de la sélection et le coût prévu.

Vous pouvez ajouter d’autres personnalisations à la vue.

1. Ajoutez un second filtre pour **Compteur** et sélectionnez une valeur pour filtrer sur un type spécifique de compteur sous le nom du service sélectionné.
1. Regrouper par **Ressource** pour afficher les ressources spécifiques qui accumulent les coûts. Le coût prévu n’est pas calculé pour chaque service. Il est projeté pour le **total** de toutes vos ressources.

[![Coût prévu pour un service](./media/cost-analysis-common-uses/forecast-by-service.png)](./media/cost-analysis-common-uses/forecast-by-service.png#lightbox)

## <a name="view-your-azure-and-aws-costs-together"></a>Afficher les coûts Azure et AWS ensemble  

Pour afficher les coûts Azure et AWS ensemble, utilisez des étendues de groupe d’administration dans Azure.

1. Créez un groupe d’administration ou sélectionnez-en un.
1. Attribuez les abonnements Azure existants nécessaires au groupe d’administration.
1. Attribuez le *même* groupe d’administration au compte lié du connecteur.
1. Accédez à Analyse des coûts et sélectionnez **Coûts cumulés**.
1. Sélectionnez **Grouper par** - **fournisseur**.

## <a name="view-cost-breakdown-by-azure-service"></a>Voir la décomposition des coûts par service Azure

La visualisation des coûts par service Azure peut vous aider à mieux comprendre les parties les plus coûteuses de votre infrastructure. Par exemple, si les coûts de calcul des machines virtuelles peuvent être faibles, vous pouvez cumuler des coûts de réseau importants en raison de la quantité d’informations qu’elles émettent. Il est essentiel de comprendre les principaux facteurs de coût de vos services Azure. Vous pourrez ainsi ajuster l’utilisation des services selon vos besoins.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Sélectionnez **Coût par service**, puis, groupez les coûts selon le critère **Niveau de service**.
1. Choisissez la vue **Tableau**.

[![Décomposition des coûts par service Azure](./media/cost-analysis-common-uses/breakdown-by-service.png)](./media/cost-analysis-common-uses/breakdown-by-service.png#lightbox)

## <a name="review-invoiced-charges-in-cost-analysis"></a>Passer en revue les frais facturés dans l’analyse des coûts

Pour voir les détails de votre facture dans le portail Azure, accédez à l’analyse des coûts de l’étendue associée à la facture que vous analysez. Sélectionnez la vue **Détails de la facture**. Les détails de la facture vous indiquent les frais tels qu’ils apparaissent sur la facture.

[![Exemple montrant les détails de la facture](./media/cost-analysis-common-uses/invoice-details.png)](./media/cost-analysis-common-uses/invoice-details.png#lightbox)

En examinant les détails de la facture, vous pouvez identifier le service qui a généré des coûts inattendus et déterminer les ressources qui sont directement associées à la ressource dans l’analyse des coûts. Par exemple, si vous souhaitez analyser les frais liés au service Machines virtuelles, accédez à la vue **Coût cumulé**. Ensuite, choisissez **Quotidienne** comme fréquence, définissez le filtre **Nom du service : Machines virtuelles** et regroupez les frais par **Ressource**.

[![Exemple montrant les coûts cumulés pour les machines virtuelles](./media/cost-analysis-common-uses/virtual-machines.png)](./media/cost-analysis-common-uses/virtual-machines.png#lightbox)

## <a name="view-cost-breakdown-by-azure-resource"></a>Voir la décomposition des coûts par ressource Azure

Vos services s’articulent autour de ressources Azure. L’examen des coûts en fonction des ressources vous permet d’identifier rapidement les principaux facteurs de coût. Si un service comprend des ressources trop coûteuses, envisagez des modifications pour réduire vos coûts.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Sélectionnez **Coût par ressource**.
1. Choisissez la vue **Tableau**.

[![Voir la décomposition des coûts par ressource Azure](./media/cost-analysis-common-uses/cost-by-resource.png)](./media/cost-analysis-common-uses/cost-by-resource.png#lightbox)

## <a name="view-cost-breakdown-by-selected-dimensions"></a>Voir la décomposition des coûts selon certaines dimensions

Les dimensions vous permettent d’organiser les coûts en fonction de différentes valeurs de métadonnées apparaissant dans vos frais. Par exemple, vous pouvez grouper vos coûts par emplacement.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Sélectionnez le filtre **Grouper par**.  
    [![Sélectionner un critère de groupement](./media/cost-analysis-common-uses/group-by.png)](./media/cost-analysis-common-uses/group-by.png#lightbox)
1. Vous pouvez enregistrer la vue pour une utilisation ultérieure si vous le souhaitez.
1. Cliquez sur un graphique à secteurs sous le graphe pour afficher des données plus détaillées.  
    [![Voir la décomposition des coûts selon certaines dimensions](./media/cost-analysis-common-uses/drill-down.png)](./media/cost-analysis-common-uses/drill-down.png#lightbox)

## <a name="view-costs-per-day-or-by-month"></a>Voir les coûts par jour ou par mois

La consultation des coûts par jour et par mois peut vous aider à mieux comprendre si vos coûts sont plus élevés à certaines périodes de la semaine ou de l’année. Un trafic client plus élevé sur une période de vacances entraîne-t-il une augmentation de vos coûts Azure ? Les coûts sont-ils plus élevés le vendredi que le lundi ?

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Sous **Granularité**, sélectionnez **Mensuelle** ou **Quotidienne**.

[![Voir les coûts par jour](./media/cost-analysis-common-uses/daily-granularity.png)](./media/cost-analysis-common-uses/daily-granularity.png#lightbox)


## <a name="view-your-spot-vm-charges"></a>Voir les frais associés aux machines virtuelles Spot

Les machines virtuelles Spot peuvent vous permettre de réaliser de grandes économies pour les charges de travail capables de gérer les interruptions. Les charges de travail sont exécutées sur une capacité Azure inutilisée. Dans la mesure où elles peuvent être supprimées à tout moment, les machines virtuelles Spot bénéficient de remises importantes. Effectuez les étapes suivantes pour voir les frais de vos machines virtuelles Spot.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple, **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
2. Ajoutez un filtre **Modèle tarifaire : Spot**.

![Exemple montrant un filtre de machine virtuelle Spot](./media/cost-analysis-common-uses/spot-vm-filter.png)

La dimension Modèle tarifaire permet également de voir les frais de réservation et d’utilisation à la demande.

## <a name="view-your-reservation-charges"></a>Voir vos frais de réservation

Les instances réservées vous permettent de faire des économies avec Azure. En optant pour des réservations, vous dépensez de l’argent au départ pour un nombre donné de ressources au long cours. L’analyse des coûts montre les frais tels qu’ils apparaissent sur votre facture. Les frais sont présentés comme des coûts réels ou amortis au cours de votre période de réservation.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple, **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Ajoutez un filtre **Modèle tarifaire : Réservation**.
1. Sous **Étendue** et à côté du coût indiqué, cliquez sur le symbole de flèche vers le bas, sélectionnez la métrique **Coût réel** ou **Coût amorti**.

![Sélectionner une métrique de coût](./media/cost-analysis-common-uses/metric-cost.png)

Chaque métrique affecte la façon dont les données sont affichées pour vos frais de réservation.

**Coût réel** : Montre l’achat tel qu’il apparaît sur votre facture. Par exemple, si vous avez acheté une réservation d’un an pour 1 200 dollars US en janvier, l’analyse des coûts montre un coût de 1 200 dollars US en janvier pour la réservation. Elle ne montre aucun coût de réservation pour les autres mois de l’année. Si vous regroupez vos coûts réels par machine virtuelle, une machine virtuelle recevant l’avantage de réservation pour un mois donné n’engendre aucun coût pour le mois.

**Coût amorti** : Montre un achat de réservation réparti sur la durée de la période de réservation. Dans l’exemple ci-dessus, l’analyse du coût montre un coût de 100 dollars US pour chaque mois de l’année si vous avez acheté une réservation d’un an pour 1 200 dollars US en janvier. Si vous regroupez les coûts par machine virtuelle dans cet exemple, vous pouvez voir le coût attribué à chaque machine virtuelle ayant bénéficié de l’avantage de réservation.

## <a name="view-your-reservation-utilization"></a>Voir l’utilisation des réservations

Après l’achat d’une réservation, il est important de suivre son utilisation pour voir si son achat est justifié. Par exemple, si vous achetez 10 machines virtuelles pour un an et que vous n’en utilisez que cinq, la moitié de votre achat ne sert à rien. Vous pouvez évaluer votre utilisation de deux façons :

### <a name="view-unused-ri-costs-in-cost-analysis"></a>Voir les coûts des instances réservées inutilisées dans l’analyse des coûts

Pour identifier le montant de votre achat de réservation qui est gaspillé chaque mois, effectuez les étapes ci-dessous.

1. Dans le portail Azure, accédez à l’analyse des coûts pour l’étendue où votre réservation est appliquée. Par exemple, **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Ajoutez un filtre **Modèle tarifaire : Réservation**.
1. Sélectionnez la vue **Coût amorti**.
1. Sélectionnez **Mensuelle** comme granularité.
1. Sélectionnez l’année en cours ou le terme de votre réservation comme période de temps.
1. Sélectionnez **Colonnes (empilées)** comme type de graphique.
1. Regroupez les frais par **Type de frais**.
1. Examinez les résultats pour les valeurs `unusedreservation`.

[![Exemple montrant l’utilisation d’une réservation](./media/cost-analysis-common-uses/view-reservation-cost.png)](./media/cost-analysis-common-uses/view-reservation-cost.png#lightbox)

### <a name="view-utilization-in-reservations"></a>Voir l’utilisation dans Réservations

Pour obtenir des instructions détaillées, consultez [Optimiser l’utilisation de la réservation](../reservations/manage-reserved-vm-instance.md#optimize-reservation-use).

## <a name="view-costs-for-a-specific-tag"></a>Voir les coûts pour une étiquette spécifique

De nombreux utilisateurs d’Azure appliquent des étiquettes à leurs ressources pour mieux catégoriser les frais. Il peut s’agir, par exemple, d’un centre de coûts ou d’un environnement de développement (de production ou de test). Les étiquettes apparaissent en tant que dimension dans l’analyse des coûts. Vous pouvez utiliser la dimension pour obtenir des insights sur vos catégories personnalisées de ressources étiquetées.

La prise en charge des étiquettes s’applique à l’utilisation signalée *après* l’application de l’étiquette à la ressource. Les étiquettes ne sont pas appliquées rétroactivement pour les cumuls de coûts.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Sélectionnez l’option d’étiquette sous **Grouper par**.

[![Voir les coûts pour une étiquette spécifique](./media/cost-analysis-common-uses/tag.png)](./media/cost-analysis-common-uses/tag.png#lightbox)

## <a name="download-your-usage-details"></a>Télécharger vos détails d’utilisation

Le fichier de rapport des détails d’utilisation, au format CSV, fournit une décomposition de l’ensemble des frais cumulés sur une facture. Vous pouvez comparer la facture avec le rapport et ainsi mieux la comprendre. Tous les frais facturés correspondent à des frais décomposés dans le rapport d’utilisation.

1. Dans le portail Azure, accédez à l’onglet **Utilisation et frais** pour un abonnement ou un compte de facturation. Par exemple : **Gestion des coûts + facturation** > **Facturation** > **Utilisation + frais**.
1. Sélectionnez la ligne pour laquelle vous souhaitez télécharger les informations, puis cliquez sur le symbole de téléchargement.  
    [![Télécharger les données sur l’utilisation et les frais](./media/cost-analysis-common-uses/download1.png)](./media/cost-analysis-common-uses/download1.png#lightbox)
1.  Sélectionnez le fichier d’utilisation à télécharger.  
    ![Choisir un fichier d’utilisation à télécharger](./media/cost-analysis-common-uses/download2.png)

## <a name="view-monthly-ea-cost-breakdown"></a>Voir la décomposition des coûts d’EA

Votre inscription EA entraîne des coûts pour l’ensemble de votre organisation. Comprendre la façon dont ces coûts s’accumulent et sont facturés au fil du temps vous aide à impliquer les parties prenantes concernées. Vous pouvez ainsi vous assurer que les coûts sont gérés de manière responsable.

Les coûts s’affichent uniquement pour votre inscription active. Si vous avez transféré une inscription (inactive) vers une nouvelle (active), les coûts de l’inscription précédente ne sont pas indiqués dans Cost Management.


1. Dans le portail Azure, accédez à **Gestion des coûts + facturation** > **Vue d’ensemble**.
1. Cliquez sur **Décomposition** pour le mois en cours afin de visualiser la ventilation de votre engagement financier.  
    [![Vue d’ensemble des coûts d’EA - décomposition récapitulative](./media/cost-analysis-common-uses/breakdown1.png)](./media/cost-analysis-common-uses/breakdown1.png#lightbox)
1.  Cliquez sur l’onglet **Utilisation et frais** pour voir la décomposition du mois précédent sur la période choisie.  
    [![Onglet Utilisation et frais](./media/cost-analysis-common-uses/breakdown2.png)](./media/cost-analysis-common-uses/breakdown2.png#lightbox)

## <a name="view-enrollment-monthly-cost-by-term"></a>Voir le coût mensuel de l’inscription par période

Utilisez une vue graphique des coûts mensuels de votre inscription pour comprendre les tendances de coût et les montants facturés sur une période donnée.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Sélectionnez votre inscription et définissez la période d’inscription.
1. Choisissez une granularité mensuelle, puis définissez la vue sur **Histogramme (empilé)** .

Vous pouvez grouper et filtrer vos données pour obtenir une analyse plus détaillée.

[![Coût mensuel de l’inscription par période](./media/cost-analysis-common-uses/enrollment-term1.png)](./media/cost-analysis-common-uses/enrollment-term1.png#lightbox)

## <a name="view-ea-enrollment-accumulated-costs"></a>Voir les coûts cumulés de l’inscription EA

Pour comprendre les dépenses globales de votre organisation sur une période donnée, vous pouvez visualiser les frais nets cumulés au fil du temps.

1. Dans le portail Azure, accédez à l’analyse des coûts pour votre étendue. Par exemple : **Gestion des coûts + facturation** > **Gestion des coûts** > **Analyse du coût**.
1. Sélectionnez votre inscription pour voir les coûts cumulés à ce jour.

[![Coûts cumulés de l’inscription](./media/cost-analysis-common-uses/cost-analysis-enrollment.png)](./media/cost-analysis-common-uses/cost-analysis-enrollment.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes
- Si vous n’avez pas encore effectué le premier guide de démarrage rapide relatif à Cost Management, lisez-le à partir de [Démarrer l’analyse des coûts](quick-acm-cost-analysis.md).
- Lisez la [documentation sur Azure Cost Management](../index.yml).
