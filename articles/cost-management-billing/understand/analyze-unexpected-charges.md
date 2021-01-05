---
title: Analyser les frais Azure inattendus
description: Découvrez comment analyser les frais inattendus pour votre abonnement Azure.
author: bandersmsft
ms.reviewer: adwise
ms.service: cost-management-billing
ms.subservice: cost-management
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: banders
ms.custom: contperf-fy21q1
ms.openlocfilehash: f6df8c6ff0db74798b71d57feae91e5245f2a6c2
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028716"
---
# <a name="analyze-unexpected-charges"></a>Analyser les frais inattendus

L’infrastructure de ressources cloud que vous avez créée pour votre organisation est probablement complexe. Un type de ressource Azure peut générer plusieurs types de frais. Les ressources Azure peuvent également appartenir à différentes équipes de votre organisation et avoir différents types de modèles de facturation qui s’appliquent à plusieurs ressources. Pour mieux comprendre les frais qui vous sont facturés, commencez votre analyse en adoptant une ou plusieurs des stratégies présentées dans les sections suivantes.

## <a name="review-invoice-for-resource-responsible-for-charge"></a>Examinez la facture pour connaître la ressource à l’origine de ces frais

La façon dont vous achetez vos services Azure permet de déterminer la méthodologie et les outils à votre disposition pour identifier la ressource ayant engendré des frais. Pour trouver la méthodologie adaptée à votre situation, commencez par [déterminer votre type d’offre Azure](../costs/understand-cost-mgt-data.md#determine-your-offer-type). Identifiez ensuite la catégorie de client à laquelle vous appartenez dans la liste des [offres Azure prises en charge](../costs/understand-cost-mgt-data.md#supported-microsoft-azure-offers).

Les articles suivants fournissent des étapes détaillées qui expliquent comment examiner votre facture en fonction du type de client que vous êtes. Dans chaque article, vous trouverez des instructions sur la façon de télécharger un fichier CSV contenant des détails sur l’utilisation et les coûts générés pendant une période de facturation donnée.

- [Processus d’examen d’une facture Paiement à l’utilisation](review-individual-bill.md#charges)
- [Processus d’examen d’une facture Contrat Entreprise](review-enterprise-agreement-bill.md)
- [Processus d’examen d’un Contrat client Microsoft](review-customer-agreement-bill.md#analyze-your-azure-usage-charges)
- [Processus d’examen d’un Contrat Partenaire Microsoft](review-partner-agreement-bill.md#analyze-your-azure-usage-charges)

Votre facture Azure agrège les frais mensuels par _compteur_. Les compteurs sont utilisés pour suivre l’utilisation d’une ressource au fil du temps et calculer votre facture. Quand vous créez une ressource Azure, par exemple une machine virtuelle, une ou plusieurs instances de compteur sont créées pour la ressource.

Filtrez le fichier CSV d’utilisation selon le nom du compteur (_MeterName_) qui figure sur la facture que vous souhaitez analyser. Vous pouvez ainsi voir tous les éléments de ligne qui s’appliquent au compteur. L’_InstanceID_ pour l’élément de ligne correspond à la ressource Azure ayant généré les frais.

Après avoir identifié la ressource en question, vous pouvez utiliser l’analyse des coûts dans Azure Cost Management pour analyser plus en détail les coûts liés à la ressource. Pour en savoir plus sur l’utilisation de l’analyse des coûts, consultez [Démarrer l’analyse des coûts](../costs/quick-acm-cost-analysis.md).

## <a name="review-invoiced-charges-in-cost-analysis"></a>Passer en revue les frais facturés dans l’analyse des coûts

Pour voir les détails de votre facture dans le portail Azure, accédez à l’analyse des coûts de l’étendue associée à la facture que vous analysez. Sélectionnez la vue **Détails de la facture**. Les détails de la facture vous indiquent les frais tels qu’ils apparaissent sur la facture.

[![Exemple montrant les détails de la facture](./media/analyze-unexpected-charges/invoice-details.png)](./media/analyze-unexpected-charges/invoice-details.png#lightbox)

En examinant les détails de la facture, vous pouvez identifier le service qui a généré des coûts inattendus et déterminer les ressources qui sont directement associées à la ressource dans l’analyse des coûts. Par exemple, si vous souhaitez analyser les frais liés au service Machines virtuelles, accédez à la vue **Coût cumulé**. Ensuite, choisissez **Quotidienne** comme fréquence, définissez le filtre **Nom du service : Machines virtuelles** et regroupez les frais par **Ressource**.

[![Exemple montrant les coûts cumulés pour les machines virtuelles](./media/analyze-unexpected-charges/virtual-machines.png)](./media/analyze-unexpected-charges/virtual-machines.png#lightbox)

## <a name="identify-spikes-in-cost-over-time"></a>Identifier les pics de coût dans le temps

Parfois, vous ne savez pas quels coûts récents ont entraîné des changements dans les frais qui vous sont facturés. Pour comprendre ce qui a changé, vous pouvez utiliser l’analyse des coûts qui [montre une décomposition quotidienne ou mensuelle des coûts dans le temps](../costs/cost-analysis-common-uses.md#view-costs-per-day-or-by-month). Une fois la vue créée, regroupez vos frais par **Service** ou **Ressource** pour identifier les changements. Vous pouvez également changer votre vue en **graphique en courbes** pour mieux visualiser les données.

![Exemple d’analyse des coûts montrant des coûts dans le temps](./media/analyze-unexpected-charges/costs-over-time.png)

## <a name="determine-resource-pricing-and-billing-model"></a>Déterminer le prix et le modèle de facturation d’une ressource

Une simple ressource peut générer des frais sur plusieurs produits et services Azure. Pour en savoir plus, consultez la page des [prix de chaque service Azure](https://azure.microsoft.com/pricing/#product-pricing). Par exemple, vous pouvez créer les compteurs suivants pour suivre l’utilisation d’une machine virtuelle créée dans Azure. Les prix peuvent varier d’un compteur à l’autre.

- Heures de calcul
- Heures d’adresse IP
- Transfert de données entrant
- Transfert de données sortant
- Disques managés Standard
- Opérations disque managées Standard
- E/S standard - Disque
- E/S standard - Lecture d’objet blob
- E/S standard - Écriture d’objet blob
- E/S standard - Suppression d’objet blob

Une fois la machine virtuelle créée, chaque compteur commence à émettre des enregistrements d’utilisation. L’utilisation et le prix du compteur font l’objet d’un suivi dans le système de mesure d’Azure. Vous pouvez voir les compteurs qui ont été utilisés pour calculer votre facture dans le fichier CSV d’utilisation.

## <a name="find-people-responsible-for-the-resource-and-engage"></a>Trouver les personnes responsables de la ressource et les solliciter

Souvent, l’équipe responsable d’une ressource donnée est au courant des changements qui ont été apportés à la ressource. Il est donc utile de solliciter ces personnes pour déterminer les causes de frais suspects. Par exemple, l’équipe propriétaire a peut-être récemment créé la ressource, mis à jour sa référence SKU (changeant ainsi le prix) ou augmenté la charge sur la ressource à la suite de modifications apportées au code. Poursuivez la lecture des sections suivantes pour découvrir d’autres techniques permettant de déterminer le propriétaire d’une ressource.

### <a name="analyze-the-audit-logs-for-the-resource"></a>Analyser les journaux d’audit de la ressource

Si vous disposez d’autorisations pour voir une ressource, vous devez pouvoir accéder à ses journaux d’audit. Examinez les journaux pour trouver l’utilisateur responsable des changements les plus récents apportés à une ressource. Pour plus d’informations, consultez [Afficher et récupérer les événements du journal d’activité Azure](../../azure-monitor/platform/activity-log.md#view-the-activity-log).

### <a name="analyze-user-permissions-to-the-resources-parent-scope"></a>Analyser les autorisations des utilisateurs sur l’étendue parente de la ressource

Les personnes qui ont accès en écriture à un abonnement ou à un groupe de ressources disposent généralement d’informations sur les ressources qui ont été créées. Elles doivent pouvoir vous expliquer l’objectif d’une ressource ou vous renvoyer à quelqu’un qui est au courant. Pour identifier les personnes disposant d’autorisations sur une étendue d’abonnement, consultez [Vérifier l’accès d’un utilisateur aux ressources Azure](../../role-based-access-control/check-access.md). Vous pouvez utiliser un processus similaire pour les groupes de ressources.

## <a name="get-help-to-identify-charges"></a>Obtenir de l’aide pour identifier certains frais

Si vous avez suivi les stratégies précédentes et que vous ne comprenez toujours pas pourquoi certains frais vous ont été facturés, ou si vous avez besoin d’aide concernant des problèmes de facturation, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [optimiser votre investissement dans le cloud avec Azure Cost Management](../costs/cost-mgt-best-practices.md).