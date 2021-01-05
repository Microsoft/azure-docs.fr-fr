---
title: Tutoriel - Optimiser le coût des instances réservées avec Cloudyn - Azure
description: Dans ce didacticiel, vous allez apprendre à optimiser les coûts associés aux instances réservées pour Azure et Amazon Web Services (AWS).
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ms.reviewer: benshy
ROBOTS: NOINDEX
ms.openlocfilehash: 8488b1f511a3b269f36f3dfe7a351765a4e312aa
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88687873"
---
<!-- Intent: As a cloud-consuming administrator, I need to ensure that my reserved instances are optimized for cost and usage
-->

# <a name="tutorial-optimize-reserved-instances"></a>Didacticiel : Optimiser les instances réservées

Dans ce tutoriel, vous allez découvrir comment Cloudyn peut vous aider à optimiser l’utilisation des instances réservées et les coûts associés pour Azure et AWS (Amazon Web Services). Une instance réservée auprès d’un fournisseur de services cloud est un engagement d’utilisation de la machine virtuelle se présentant sous la forme d’un contrat à long terme. Elle peut vous permettre de réaliser des économies considérables par rapport à un modèle de tarification à l’utilisation standard. Vous bénéficiez de ces économies potentielles uniquement lorsque vous utilisez toute la capacité de vos instances réservées.

Ce tutoriel explique comment Cloudyn prend en charge les instances réservées Azure et AWS. Il explique également comment vous pouvez optimiser les coûts associés à ces instances réservées, principalement en vous assurant que vos réservations sont entièrement utilisées. Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Comprendre les coûts associés aux instances réservées Azure
> * Découvrir les avantages des instances réservées
> * Optimiser les coûts associés aux instances réservées Azure
> * Afficher les coûts associés aux instances réservées
> * Évaluer la rentabilité des instances réservées Azure
> * Optimiser les coûts associés aux instances réservées AWS
> * Acheter les instances réservées recommandées
> * Modifier des réservations inutilisées

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="prerequisites"></a>Conditions préalables requises

- Vous devez disposer d’un compte Azure.
- Vous devez disposer d’une inscription à un essai gratuit ou d’un abonnement payant pour Cloudyn.
- Vous devez avoir acheté des instances réservées dans Azure ou AWS.

## <a name="understand-azure-ri-costs"></a>Comprendre les coûts associés aux instances réservées Azure

Lorsque vous achetez des instances de machine virtuelle réservées Azure, vous payez à l’avance pour leur utilisation future. Le paiement initial couvre le coût d’utilisation future des machines virtuelles :

- d’un type spécifique
- dans une région spécifique
- pour une durée d’un ou de trois ans
- pour un nombre défini de machines virtuelles.

Vous pouvez afficher les instances de machine virtuelle réservées Azure achetées dans le portail Azure sous [Réservations](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).

Le terme _Instance de machine virtuelle réservée Azure_ fait uniquement référence à un modèle de tarification. Vos machines virtuelles en cours d’exécution ne subissent aucune modification. Le terme est spécifique à Azure et plus généralement remplacé par le terme _instance réservée_ ou _réservation_. Les instances réservées que vous avez achetées ne font pas référence à des machines virtuelles spécifiques, elles s’appliquent à toute machine virtuelle compatible. Par exemple, une réservation pour un type de machine virtuelle qui s’exécute dans une région que vous avez choisie au moment d’acheter la réservation.

Les instances réservées achetées s’appliquent uniquement au matériel de base. Elles ne couvrent les licences logicielles d’une machine virtuelle. Par exemple, vous pouvez réserver une instance et disposer d’une machine virtuelle correspondante exécutant Windows. L’instance réservée couvre uniquement le coût de base de la machine virtuelle. Dans cet exemple, vous payez le montant total de toutes les licences Windows requises. Pour obtenir une remise sur le système d’exploitation ou d’autres logiciels s’exécutant sur vos machines virtuelles, vous pouvez par exemple utiliser [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit). Hybrid Benefit vous offre le même type de remise pour vos licences logicielles que les instances réservées vous offrent pour les machines virtuelles de base.

L’utilisation des instances réservées n’affecte pas directement les coûts. En d’autres termes, exécuter une machine virtuelle en utilisant 100 % ou 0 % de la capacité du processeur revient au même : vous payez à l’avance pour l’allocation de la machine virtuelle, pas pour son utilisation réelle.

L’image suivante compare les coûts d’utilisation de machine virtuelle à la demande standard aux coûts associés aux instances réservées :

![Coûts avec utilisation à la demande vs. coûts avec instances réservées](./media/tutorial-optimize-reserved-instances/azure01.png)



Les barres rouges indiquent le coût cumulé de l’achat d’instances réservées. Vous payez un montant unique et ponctuel. L’utilisation de la machine virtuelle n’engendre aucun frais. Les barres bleues indiquent le coût cumulé de la même machine virtuelle s’exécutant avec un modèle de tarification à l’utilisation ou à la demande. Le *seuil de rentabilité* de l’utilisation de la machine virtuelle est atteint entre le septième et le huitième mois. Dans cet exemple, vous commencez donc à économiser à partir du huitième mois.

## <a name="benefits-of-ris"></a>Avantages des instances réservées

Chaque achat d’instance réservée s’applique à une machine virtuelle d’une taille spécifique et exécutée dans une région spécifique. Par exemple, la machine D2s\_v3 qui est exécutée dans la région USA Ouest, comme indiqué dans l’image suivante :

![Informations détaillées sur les instances réservées Azure](./media/tutorial-optimize-reserved-instances/azure02.png)

L’achat d’instances réservées est avantageux lorsqu’une machine virtuelle s’exécute un nombre suffisant d’heures pour atteindre le seuil de rentabilité de la réservation. La machine virtuelle doit avoir la même taille et le même emplacement que votre instance réservée. Par exemple, dans le graphique précédent, le seuil de rentabilité est atteint au bout de sept mois et demi environ. Par conséquent, l’achat est avantageux lorsque la machine virtuelle correspondant à la réservation s’exécute au moins 7,5 mois \* 30 jours \* 24 heures = 5 400 heures. Si la machine virtuelle correspondante s’exécute moins de 5 400 heures, la réservation sera plus onéreuse qu’un paiement à l’utilisation.

Le seuil de rentabilité peut varier en fonction de la taille de la machine virtuelle et de son emplacement. Il dépend également du tarif de paiement à l’utilisation que vous avez négocié. Avant d’effectuer un achat, vous devez vérifier votre propre seuil de rentabilité.

Un autre point à prendre en compte lorsque vous achetez une réservation est la portée de l’instance réservée. La portée détermine si le bénéfice de la réservation est partagé ou s’il s’applique à un abonnement spécifique. Les instances réservées partagées sont réparties de manière aléatoire sur les premières machines virtuelles compatibles trouvées dans l’ensemble de vos abonnements.

L’achat d’instances réservées partagées offre une plus grande flexibilité et est recommandé dans la mesure du possible. Vous aurez de bien meilleures chances d’utiliser toutes vos instances réservées en choisissant la portée partagée. Toutefois, lorsque le propriétaire d’un abonnement paie pour l’instance réservée, il peut n’avoir d’autre choix que de l’acheter avec une portée de type Abonnement unique.

## <a name="optimize-azure-ri-costs"></a>Optimiser les coûts associés aux instances réservées Azure

Cloudyn prend en charge les instances réservées et Hybrid Benefit avec les fonctionnalités suivantes :

- Affichage des coûts associés aux différents modèles de tarification
- Suivi de l’utilisation des instances réservées
- Évaluation de l’impact des instances réservées
- Répartition des coûts associés aux instances réservées conformément aux stratégies de votre entreprise

La première chose à faire avant d’acheter une instance réservée est d’évaluer l’impact d’un tel achat :

- Combien cela va-t-il vous coûter ?
- Combien allez-vous économiser ?
- Quel est le seuil de rentabilité ?

Le rapport Reserved Instance Purchase Impact (Impact de l’achat d’instance réservée) peut vous aider à répondre à ces questions.

## <a name="assess-azure-ri-cost-effectiveness"></a>Évaluer la rentabilité des instances réservées Azure

Dans le portail Cloudyn, accédez à **Optimizer** (Optimiseur)  > **RI Comparison** (Comparatif Instances réservées), puis sélectionnez **Reserved Instance Purchase Impact** (Impact de l’achat d’instance réservée).

Dans le rapport sur l’impact de l’achat d’instance réservée, sélectionnez une taille de machine virtuelle (Type d’instance), un emplacement (Région), le terme de la réservation, la quantité et le runtime attendu. Vous pouvez alors déterminer si votre achat vous fera économiser de l’argent.

Par exemple, si vous achetez une réservation pour une machine virtuelle de type DS1\_v2 dans la région USA Est qui s’exécute 24h/24, 7j/7 durant une année complète, vous pourriez économiser 369,48 $ par an. Le seuil de rentabilité est atteint au bout du cinquième mois. Consultez le graphique suivant :

![Seuil de rentabilité de l’instance réservée Azure](./media/tutorial-optimize-reserved-instances/azure03.png)

Cependant, si la machine virtuelle ne s’exécute que 50 % du temps, le seuil de rentabilité sera atteint au bout du 10e mois et vous n’économiserez que 49,74 $ par an. L’achat de la réservation pour ce type d’instance ne sera peut-être pas avantageux pour vous. Consultez le graphique suivant :

![Exemple de seuil de rentabilité pour les machines virtuelles Azure](./media/tutorial-optimize-reserved-instances/azure04.png)

## <a name="view-ri-costs"></a>Afficher les coûts associés aux instances réservées

Lorsque vous achetez une réservation, vous effectuez un paiement unique. Il existe deux manières d’afficher le paiement dans Cloudyn :

- Coût réel
- Coût amorti

### <a name="actual-reserved-instance-cost"></a>Coût réel de l’instance réservée

Les rapports Actual Cost Analysis (Analyse du coût réel) et Analysis Over Time (Analyse dans le temps) affichent le montant total payé pour la réservation, à partir du mois de l’achat. Ils vous permettent de voir vos dépenses réelles sur une période donnée.

Accédez à **Coûts** > **Analyse des coûts** > dans le portail Cloudyn, puis sélectionnez **Analyse du coût réel** ou **Coûts réels dans le temps**. Définissez ensuite les filtres. Par exemple, filtrez sur le service Azure/VM et regroupez les résultats par Type de ressource et Modèle de prix. Consultez le graphique suivant :

![Exemple de coût réel des instances réservées](./media/tutorial-optimize-reserved-instances/azure05.png)

Vous pouvez filtrer sur un service, **Azure/VM** dans cet exemple, et regroupez les résultats par **Modèle de prix** et **Type de ressource** comme indiqué dans l’image suivante :

![Exemple de groupes de rapports de coût réel et de filtres regroupés par type de ressource et modèle de prix](./media/tutorial-optimize-reserved-instances/azure06.png)

Vous pouvez également analyser les types de paiements effectués tels que les frais uniques, les frais d’utilisation et les frais de licence.

### <a name="amortized-reserved-instance-cost"></a>Coût amorti de l’instance réservée

Vous payez un montant initial qui s’affiche pour le mois de l’achat lorsque vous achetez une instance réservée. Ce montant n’apparaît pas dans les factures suivantes. Par conséquent, consulter vos frais d’utilisation mensuelle peut vous induire en erreur. Le coût mensuel réel est la somme du coût d’utilisation mensuelle et de la partie proportionnelle (amortie) de tout paiement unique effectué auparavant. Le rapport Coût amorti vous offre une vision détaillée de vos coûts réels.

Pour calculer le coût amorti des instances réservées, l’outil tient compte des frais de réservation uniques qu’il amortit sur la durée de la réservation. Dans les rapports Coût réel, les frais uniques s’affichent uniquement pour le mois de l’achat de la réservation. Les dépenses journalières et mensuelles n’apparaissent pas dans le coût réel du déploiement. Les rapports Coût amorti indiquent le coût réel du déploiement dans le temps.  Le rapport sur le coût amorti est la seule façon de voir les tendances de coût réel. C’est également le seul moyen de prévoir vos dépenses futures.

Le rapport Coût réel affiche un pic correspondant à un achat d’instance réservée le 16 novembre pour un montant de 747 $. Le rapport Coût amorti (voir l’image suivante) affiche un coût journalier partiel le 16 novembre. À partir du 17 novembre, le rapport affiche un coût amorti de l’instance réservée de 747 $/365 = 2,05 $. Par ailleurs, vous remarquerez également que la réservation achetée n’est pas utilisée. Vous pouvez donc l’optimiser en l’affectant à une autre taille de machine virtuelle.

Pour l’afficher, accédez à **Coûts** > **Analyse des coûts** >, puis sélectionnez **Amortized Cost Analysis** (Analyse des coûts amortis) ou **Amortized Cost Over Time** (Coûts amortis dans le temps).

![Exemple de rapport montrant le coût amorti de l’instance réservée](./media/tutorial-optimize-reserved-instances/azure07.png)

## <a name="optimize-aws-ri-costs"></a>Optimiser les coûts associés aux instances réservées AWS

Les instances réservées constituent un engagement ouvert. Elles s’avèrent utiles lorsque vous utilisez vos machines virtuelles de manière régulière, car elles engendrent moins de frais que les instances provisionnées à la demande. Vous devez toutefois veiller à ne pas les sous-utiliser. L’engagement consiste à utiliser des ressources, généralement des machines virtuelles, sur une période définie (un ou trois ans). Lorsque vous vous engagez dans un tel achat, vous prépayez les ressources avec une réservation. Il est cependant possible que vous n’utilisiez pas toujours entièrement la capacité réservée.

Imaginons que vous avez évalué votre environnement et déterminé que vous disposiez de 20 instances D2 standard exécutées de manière constante au cours de l’année passée. Vous pourriez acheter une réservation pour ces instances et réaliser d’importantes économies. Nous pouvons également imaginer que vous vous êtes engagé pour utiliser dix instances MA4 sur l’année. Mais vous n’en avez utilisé que cinq à ce jour. Ces deux exemples illustrent une utilisation inefficace des instances réservées. Il existe deux façons d’optimiser les coûts pour les instances réservées à l’aide des rapports d’optimisation Cloudyn :

- Consulter les recommandations d’achat en fonction de votre historique d’utilisation
- Modifier des réservations inutilisées

Les rapports _EC2 RI Buying Recommendations_ (Recommandations d’achat d’instances réservées EC2) et _EC2 Currently Unused Reservations_ (Réservations EC2 actuellement inutilisées) vous aident à optimiser l’utilisation et les coûts liés aux instances réservées.

## <a name="buy-recommended-ris"></a>Acheter les instances réservées recommandées

Cloudyn compare les coûts d’utilisation d’instances à la demande aux coûts d’achat d’instances réservées. Lorsque l’outil détecte des économies potentielles, il ajoute ses recommandations dans le rapport EC2 Buying Recommendations (Recommandations d’achat EC2).

Dans le menu Rapports en haut du portail, cliquez sur **Optimizer (Optimiseur)**  > **Pricing Optimization (Optimisation des prix)**  > **EC2 RI Buying Recommendations (Recommandations d’achat d’instances réservées EC2)** .

L’image suivante montre des recommandations d’achat telles qu’elles s’affichent dans le rapport.

![Exemple montrant des recommandations d’achat dans le rapport Recommandations d’achat EC2](./media/tutorial-optimize-reserved-instances/aws01.png)

Dans cet exemple, le compte Cloudyn\_A a 32 recommandations d’achat d’instances réservées. En suivant toutes les recommandations d’achat, vous pourriez économiser 137 770 $ par an. Gardez à l’esprit que les recommandations d’achat fournies par Cloudyn se basent sur une utilisation constante de vos charges de travail en cours d’exécution.

Pour afficher les détails de chaque recommandation d’achat, cliquez sur le signe plus ( **+** ) sous **Justifications**. Voici un exemple de la première recommandation de la liste.

![Exemple montrant des détails de justification d’achat](./media/tutorial-optimize-reserved-instances/aws02.png)

L’exemple précédent indique que l’exécution à la demande de la charge de travail coûterait 90 456 $ par an. Par contre, si vous achetez la réservation à l’avance, la même charge de travail ne vous coûterait que 56 592 $, soit une économie de 33 864 $ par an.

Cliquez sur le signe plus en regard de **EC2 RI Purchase Impact** (Impact de l’achat d’instances réservées EC2) pour afficher votre seuil de rentabilité sur un an et savoir à peu près à quel moment votre investissement sera rentabilisé. Environ huit mois après l’achat, les coûts cumulés d’utilisation à la demande commencent à dépasser les coûts cumulés des instances réservées dans l’exemple suivant :

![Exemple montrant des détails sur l’impact de l’achat](./media/tutorial-optimize-reserved-instances/aws03.png)

Vous commencez à économiser de l’argent à ce moment précis.

Vous pouvez consulter le document **Instances over Time** (Instances dans le temps) pour vérifier l’exactitude de la recommandation d’achat. Dans cet exemple, six instances ont été utilisées en moyenne pour la charge de travail au cours des 30 derniers jours.

![Exemple montrant l’historique d’utilisation des instances dans le temps](./media/tutorial-optimize-reserved-instances/aws04.png)

## <a name="modify-unused-reservations"></a>Modifier des réservations inutilisées

Les réservations inutilisées sont courantes dans les environnements de ressources cloud des clients. Modifiez les réservations en fonction de vos besoins actuels afin de vous assurer que les réservations inutilisées sont utilisées dans leur intégralité et économiser ainsi de l’argent. Imaginons que vous ayez un abonnement contenant des instances D3 standard s’exécutant sous Linux. Si vous n’allez pas utiliser entièrement la réservation, vous pouvez modifier le type d’instance. Vous pouvez également transférer les ressources non utilisées vers une autre réservation ou vers un autre compte.

AWS vend des instances réservées pour des zones de disponibilité et des régions spécifiques. Si vous avez acheté des instances réservées pour une zone de disponibilité spécifique, vous ne pouvez pas transférer les réservations vers d’autres zones. Toutefois, vous pouvez facilement déplacer des instances réservées régionales dans les différentes zones en utilisant le rapport **EC2 Currently Unused Reservations** (Réservations EC2 actuellement inutilisées). Vous pouvez également les modifier pour avoir une portée régionale. Ces instances s’appliqueront ensuite aux instances correspondantes dans toutes les zones de disponibilité.

Dans le menu Rapports en haut du portail, cliquez sur **Optimizer (Optimiseur)**  > **Inefficiencies (Inefficacités)**  > **EC2 Currently Unused Reservations (Réservations EC2 actuellement inutilisées)** .

Les images suivantes montrent le rapport contenant des instances réservées inutilisées.

![Exemple montrant un résumé des informations sur les réservations inutilisées](./media/tutorial-optimize-reserved-instances/unused-ri01.png)

Cliquez sur le signe plus sous **Détails** pour afficher les informations détaillées d’une réservation spécifique.

![Exemple montrant des détails sur les réservations inutilisées](./media/tutorial-optimize-reserved-instances/unused-ri02.png)

Dans l’exemple précédent, on dénombre un total de 77 réservations inutilisées dans différentes zones de disponibilité. La première réservation a 51 instances inutilisées. Plus bas dans la liste, il est possible d’apporter des modifications aux instances réservées en utilisant le type d’instance **m3.2xlarge** dans la zone de disponibilité **us-east-1c**.

Cliquez sur **Modifier** pour la première réservation de la liste afin d’ouvrir la page **Modify RI** (Modifier les instances réservées) qui affiche les données concernant la réservation.

![Exemple montrant des réservations que vous pouvez modifier](./media/tutorial-optimize-reserved-instances/unused-ri03.png)

La page affiche les instances réservées que vous pouvez modifier. Dans l’exemple suivant, il y a 51 instances inutilisées que vous pouvez modifier, mais 54 instances sont requises pour les deux réservations. Si vous modifiez vos instances inutilisées afin de toutes les utiliser, quatre instances continueront à s’exécuter à la demande. Pour cet exemple, répartissez vos instances inutilisées pour que la première réservation en utilise 30 et la seconde réservation 21.

Cliquez sur le signe plus pour la première entrée de réservation et définissez la **quantité de réservation** sur **30**. Pour la deuxième entrée, définissez la quantité de réservation sur **21**, puis cliquez sur **Appliquer**.

![Exemple montrant des modifications apportées à la quantité de réservation](./media/tutorial-optimize-reserved-instances/unused-ri04.png)

Toutes vos instances inutilisées pour la réservation sont désormais entièrement utilisées et 51 instances ne s’exécutent plus à la demande. Dans cet exemple, votre entreprise réalise des économies en réduisant de manière considérable le taux d’utilisation à la demande et en utilisant les réservations pour lesquelles elle a déjà payé.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez effectué avec succès les tâches suivantes :

> [!div class="checklist"]
> * Comprendre les coûts associés aux instances réservées Azure
> * Découvrir les avantages des instances réservées
> * Optimiser les coûts associés aux instances réservées Azure
> * Afficher les coûts associés aux instances réservées
> * Évaluer la rentabilité des instances réservées Azure
> * Optimiser les coûts associés aux instances réservées AWS
> * Acheter les instances réservées recommandées
> * Modifier des réservations inutilisées


Passez au didacticiel suivant pour en savoir plus sur le contrôle de l’accès aux données.

> [!div class="nextstepaction"]
> [Contrôler l'accès aux données](tutorial-user-access.md)
