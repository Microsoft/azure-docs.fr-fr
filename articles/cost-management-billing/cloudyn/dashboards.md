---
title: Visualiser les métriques clés dans des tableaux de bord Cloudyn dans Azure
description: Cet article explique comment afficher les mesures clés avec les tableaux de bord de Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: vitavor
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 03d5e3959b85a2b0bd9ae30744051895ff026b77
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88691222"
---
# <a name="view-key-cost-metrics-with-dashboards"></a>Afficher les métriques de coût clés dans des tableaux de bord

Les tableaux de bord dans Cloudyn donnent une vue d’ensemble des rapports. Il vous permettent d’afficher les métriques de coût clés dans une seule vue. Ils fournissent également des informations sur des tendances pour vous aider à prendre des décisions importantes.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

Des tableaux de bord sont également utilisés pour créer des vues pour les personnes exerçant différentes responsabilités dans votre organisation, à savoir :

- Contrôleur financier
- Propriétaire d’application ou de projet
- Ingénieur DevOps
- Cadres

Les tableaux de bord sont composés de widgets qui sont essentiellement des miniatures de rapport. Vous pouvez cliquer sur un widget pour ouvrir le rapport correspondant. Lorsque vous personnalisez des rapports, vous les enregistrez dans Mes rapports, et ils sont ajoutés au tableau de bord.

Les versions de tableau de bord diffèrent pour les utilisateurs de Cloudyn Management (MSP), Enterprise et Premium. Les différences sont déterminées par les niveaux d’accès d’entité. Pour plus d’informations sur les niveaux d’accès, voir [Niveaux d’accès d’entité](tutorial-user-access.md#entity-access-levels).

La disponibilité des tableaux de bord varie selon le type de compte de fournisseur de service cloud utilisé lors de leur affichage. Le type d’informations disponibles et collectées par Cloudyn affecte les rapports présentés dans les tableaux de bord. Par exemple, si vous n’avez pas de compte AWS, vous ne verrez pas le tableau de bord S3 Tracker (Traqueur S3). De même, si vous n’activez pas l’accès d’Azure Resource Manager à Cloudyn, vous ne verrez pas d’informations spécifiques d’Azure dans les widgets du tableau de bord Optimizer (Optimiseur).

Vous pouvez utiliser l’un des tableaux de bord prédéfinis, ou créer votre propre tableau de bord avec des rapports personnalisés. Si vous n’êtes pas familiarisé avec les rapports Cloudyn, consultez [Utiliser des rapports Cloudyn](use-reports.md).

## <a name="create-a-custom-dashboard"></a>Créer un tableau de bord personnalisé

Pour commencer rapidement avec un tableau de bord personnalisé, vous pouvez dupliquer un tableau de bord existant afin d’utiliser ses propriétés. Vous pouvez ensuite l’adapter à vos besoins. Sur le tableau de bord que vous souhaitez copier, cliquez sur **Enregistrer sous**. Vous ne pouvez dupliquer que des tableaux de bord personnalisés, pas les tableaux de bord prédéfinis inclus dans Cloudyn.

Pour créer un tableau de bord personnalisé :

1. Dans la page d’accueil, cliquez sur **Ajouter nouveau +**. La page My Dashboard (Mon tableau de bord) s’affiche.  
    ![Ma page de tableau de bord pour ajouter de nouveaux rapports](./media/dashboards/my-dashboard.png)
2. Cliquez sur **Add New Report (Ajouter un nouveau rapport)**. La zone Add Report (Ajouter un rapport) s’affiche.
3. Sélectionnez le rapport que vous souhaitez ajouter au widget de tableau de bord. Le widget est ajouté au tableau de bord.
4. Répétez les étapes précédentes jusqu’à ce que le tableau de bord soit complet.
5. Pour modifier le nom du tableau de bord, cliquez dessus dans la page d’accueil Dashboard (Tableau de bord), puis tapez le nouveau nom.

## <a name="modify-a-custom-dashboard"></a>Modifier un tableau de bord personnalisé

Tout comme pour la création de tableaux de bord personnalisés, vous ne pouvez pas modifier les tableaux de bord prédéfinis inclus dans Cloudyn. Pour modifier un rapport de tableau de bord personnalisé :

1. Dans le tableau de bord, recherchez le rapport que vous souhaitez modifier, puis cliquez sur **Edit (Modifier)**. Le rapport s’affiche.
2. Apportez les modifications que vous souhaitez au rapport, puis cliquez sur **Save (Enregistrer)**. Le rapport est mis à jour et reflète vos modifications.

## <a name="share-a-custom-dashboard"></a>Partager un tableau de bord personnalisé

Vous pouvez partager un tableau de bord personnalisé avec _Public_ ou _My Entity (Mon entité)_. Lorsque vous partagez le tableau de bord avec Public, tous les utilisateurs peuvent l’afficher. Lorsque vous partagez le tableau de bord avec My Entity (Mon entité), seuls les utilisateurs ayant accès à l’entité actuelle peuvent l’afficher. Les étapes permettant de partager un tableau de bord personnalisé avec Public et My Entity (Mon entité) sont similaires.

Pour partager un tableau de bord personnalisé avec Public :

1. Dans un tableau de bord, cliquez sur **Dashboard Settings (Paramètres du tableau de bord)**. La zone Dashboard Settings (Paramètres du tableau de bord) s’affiche.  
    ![Paramètres d’un tableau de bord personnalisé](./media/dashboards/dashboard-options.png)
2. Dans la zone Dashboard Settings (Paramètres du tableau de bord), cliquez sur le symbole de flèche, puis sur **Public**. La boîte de dialogue de confirmation Public Dashboard (Tableau de bord public) s’affiche.
3. Cliquez sur **Yes (Oui)** . Le tableau de bord est désormais disponible pour d’autres utilisateurs.

## <a name="delete-a-custom-dashboard-report"></a>Supprimer un rapport de tableau de bord personnalisé

Vous pouvez supprimer un rapport personnalisé du tableau de bord. La suppression du rapport du tableau de bord n’a pas pour effet de le supprimer de la liste des rapports. Elle ne supprime le rapport que du tableau de bord.

Pour supprimer un composant de tableau de bord, sur le composant, cliquez sur **Delete (Supprimer)**. Cliquer sur **Delete (Supprimer)** a pour effet de supprimer immédiatement le composant du tableau de bord.

## <a name="share-a-dashboard-enterprise"></a>Partager un tableau de bord (Enterprise)

Vous pouvez partager des tableaux de bord personnalisés avec tous les utilisateurs au sein de votre organisation ou avec les utilisateurs de l’entité actuelle. Partager un tableau de bord peut donner aux autres utilisateurs une vue d’ensemble rapide de votre indicateur de performance clé. Lorsque vous partagez un tableau de bord, celui-ci est répliqué automatiquement vers l’ensemble de vos entités/clients Cloudyn. Les modifications apportées au tableau de bord partagé sont mises à jour automatiquement.

Pour partager un tableau de bord avec tous les utilisateurs, y compris des sous-entités :

1. Dans la page d’accueil Dashboard (Tableau de bord), cliquez sur **Edit (Modifier)**.
2. Cliquez sur **Share (Partager)**, puis sélectionnez **Public**.
3. La boîte de dialogue de confirmation Global Public Dashboard (Tableau de bord public global) s’affiche.
4. Cliquez sur **Yes (Oui)** pour définir le tableau de bord en tant que tableau de bord public global.

Pour partager un tableau de bord avec tous les utilisateurs d’une entité actuelle :

1. Dans la page d’accueil Dashboard (Tableau de bord), cliquez sur **Edit (Modifier)**.
2. Cliquez sur **Share (Partager)**, puis sélectionnez **My Entity (Mon entité)**.
3. Cliquez sur **Yes (Oui)** pour définir le tableau de bord en tant que tableau de bord public.

## <a name="duplicate-a-custom-dashboard"></a>Dupliquer un tableau de bord personnalisé

Lorsque vous créez un tableau de bord, il se peut que vous souhaitiez utiliser des propriétés similaires d’un tableau de bord existant. Vous pouvez dupliquer un tableau de bord pour en créer un autre.

Vous pouvez dupliquer uniquement des tableaux de bord personnalisés. Vous ne pouvez pas dupliquer des tableaux de bord standard.

Pour dupliquer (cloner) un tableau de bord personnalisé :

1. Dans le tableau de bord que vous souhaitez dupliquer, cliquez sur **Save As (Enregistrer sous)**. Un nouveau tableau de bord s’ouvre, portant les mêmes nom et numéro.
2. Renommez le tableau de bord dupliqué et modifiez-le à votre guise.

-Ou-

1. Dans Dashboard Settings (Paramètres du tableau de bord), cliquez sur **Save As (Enregistrer sous)** dans la ligne du tableau de bord que vous souhaitez dupliquer.
2. Le tableau de bord dupliqué s’ouvre.
3. Renommez le tableau de bord et modifiez-le à votre guise.

## <a name="set-a-default-dashboard"></a>Définir un tableau de bord par défaut

Vous pouvez définir tout tableau de bord en tant que tableau de bord par défaut. Le tableau de bord par défaut apparaît dans l’onglet le plus à gauche de la liste d’onglets de tableau de bord. Le tableau de bord par défaut s’affiche lors de l’ouverture du portail Cloudyn.

- Cliquez sur l’onglet de tableau de bord que vous souhaitez définir en tant que tableau de bord par défaut, puis cliquez sur **Default (Par défaut)** sur la droite.

-Ou-

1. Cliquez sur **Dashboard Settings (Paramètres du tableau de bord)** pour afficher la liste des tableaux de bord disponibles, puis sélectionnez le tableau de bord que vous souhaitez définir en tant tableau de bord par défaut.  
    ![Options d’un tableau de bord par défaut](./media/dashboards/dashboard-options.png)
2. Cliquez sur **Default (Par défaut)** dans la ligne du tableau de bord. La boîte de dialogue de confirmation Default Dashboard (Tableau de bord par défaut) s’affiche.
3. Cliquez sur **Oui**. Le tableau de bord est défini en tant que tableau de bord par défaut.

## <a name="management-dashboard"></a>Tableau de bord de gestion
Le tableau de bord Management (Gestion), ou le tableau de bord MSP pour les utilisateurs de MSP, inclut les principales informations des types de rapports principaux.  
![Tableau de bord Management montrant différents rapports](./media/dashboards/management-dash.png)

### <a name="cost-entity-summary-enterprise-only"></a>Cost Entity Summary (Résumé de l’entité de coût) (version Enterprise uniquement)
Ce widget résume les entités de coût gérées, dont le nombre d’entités et le nombre de comptes.
- Cliquez dessus pour ouvrir le rapport Enterprise Details (Détails de l’entreprise).

### <a name="cost-over-time"></a>Cost Over Time (Coût au fil du temps)
Ce widget peut vous aider à repérer des tendances de coût. Il met en évidence le coût du dernier jour, en fonction de la tendance des 30 derniers jours.
- Cliquez sur le widget pour ouvrir le rapport Actual Cost Over Time (Coût réel au fil du temps) afin d’afficher et de filtrer des détails supplémentaires.

### <a name="asset-controller"></a>Asset Controller (Contrôleur de ressource)
Ce widget met en évidence le nombre d’instances en cours d’exécution depuis le jour précédent, au-dessus de la tendance d’utilisation au cours des 30 derniers jours.
- Cliquez dessus pour ouvrir le tableau de bord Asset Controller (Contrôleur de ressource).

### <a name="unused-ri-detector"></a>Unused RI Detector (Détecteur d’instance réservée inutilisée)
Ce widget met en évidence le nombre de réservations Amazon EC2 inutilisées.
- Cliquez dessus pour ouvrir le rapport Currently Unused Reservations (Réservations actuellement inutilisées) qui présente les réservations inutilisées que vous pouvez modifier.

### <a name="cost-by-service"></a>Cost by Service (Coût par Service)
Ce widget met en évidence les coûts amortis par service pour les 30 derniers jours. Pointez sur le graphique à secteurs pour afficher les coûts par service.
- Cliquez sur le widget pour ouvrir le rapport Actual Cost Analysis (Analyse du coût réel).

### <a name="potential-savings"></a>Potential Savings (Économies potentielles)
Ce widget affiche des suggestions de tarification de type d’instance pour Amazon EC2 et Amazon RDS.
- Cliquez dessus pour ouvrir le rapport Savings Analysis (Analyse d’économies). Il répertorie vos coûts par types d’instances avec les économies potentielles.

### <a name="compute-instances---daily-trend"></a>Compute Instances - Daily Trend (Instances de calcul - Tendance quotidienne)
Ce widget affiche les instances actives par type pour les 30 derniers jours.
- Cliquez dessus pour ouvrir le rapport Instances Over Time (Instances au fil du temps) qui présente une ventilation de toutes les instances actives au cours des 30 derniers jours.

### <a name="storage-by-department"></a>Storage by department (Stockage par département)
Ce widget affiche les services de stockage utilisés par les départements. Pointez sur le graphique à secteurs pour afficher la consommation de stockage par département.
- Cliquez dessus pour ouvrir le tableau de bord S3 Tracker (Traqueur S3).

## <a name="cost-controller-dashboard"></a>Tableau de bord Cost Controller (Contrôleur de coût)
Le tableau de bord Cost Controller (Contrôleur de coût) affiche les informations d’allocation de coût prédéfinies.  
![Tableau de bord Cost Controller montrant différents rapports](./media/dashboards/cost-controller-dashboard.png)

### <a name="cost-over-time"></a>Cost Over Time (Coût au fil du temps)
Ce widget vous aide à repérer des tendances de coût. Il met en évidence le coût du dernier jour, en fonction de la tendance des 30 derniers jours.
- Cliquez sur le widget pour ouvrir le rapport Actual Cost Over Time (Coût réel au fil du temps) afin d’afficher et de filtrer des détails supplémentaires.

### <a name="monthly-cost-trends"></a>Monthly Cost Trends (Tendances de coût mensuel)
Ce widget met en évidence les dépenses amorties projetées et vos dépenses réelles depuis le début du mois.
- Cliquez dessus pour ouvrir le rapport Current Month Projected Cost (Coût projeté pour le mois en cours), qui fournit un résumé du coût du mois à ce jour.

Ce rapport présente le coût depuis le début du mois, le coût du mois précédent et le coût projeté du mois en cours. Le coût projeté du mois en cours est calculé en additionnant le coût mensuel à ce jour et une projection. La projection est basée sur le coût analysé au cours des 30 derniers jours.

### <a name="12-month-planner"></a>12 Month Planner (Planificateur sur 12 mois)
Ce widget met en évidence les coûts projetés sur les 12 prochains mois et les économies potentielles.
- Cliquez dessus pour ouvrir le rapport annuel Annual Projected Cost (Coût annuel projeté).

### <a name="cost-by-service"></a>Cost by Service (Coût par Service)
Ce widget met en évidence les coûts amortis par service pour les 30 derniers jours.
- Pointez sur le graphique à secteurs pour afficher les coûts par service.
- Cliquez sur le widget pour ouvrir le rapport Actual Cost Analysis (Analyse du coût réel).

### <a name="cost-by-account"></a>Cost by Account (Coût par compte)
Ce widget met en évidence les coûts amortis par compte pour les 30 derniers jours.
- Pointez sur le graphique à secteurs pour afficher les coûts par compte.
- Cliquez sur le widget pour ouvrir le rapport Actual Cost Analysis (Analyse du coût réel).

### <a name="cost-trend-by-day"></a>Cost Trend by Day (Tendance du coût par jour)
Ce widget met en évidence les dépenses au cours des 30 derniers jours.
- Pointez sur le graphique à barres pour afficher les coûts par jour.
- Cliquez sur le widget pour ouvrir le rapport Actual Cost Over Time (Coût réel au fil du temps).

### <a name="cost-trend-by-month---last-6-months"></a>Cost Trend by Month - Last 6 months (Tendance du coût par mois - 6 derniers mois)

Ce widget met en évidence les dépenses au cours des six derniers mois.
- Pointez sur le graphique à barres pour afficher les coûts par mois.
- Cliquez sur le widget pour ouvrir le rapport Actual Cost Over Time (Coût réel au fil du temps).

## <a name="asset-controller-dashboard"></a>Tableau de bord Asset Controller (Contrôleur de ressource)

Ce tableau de bord affiche le nombre d’instances en cours d’exécution, les disques disponibles et utilisés, la distribution des types d’instances et des informations sur le stockage.  
![Tableau de bord Asset Controller montrant différents rapports](./media/dashboards/asset-controller-dashboard.png)

### <a name="compute-instances"></a>Compute Instances (Instances de calcul)
Ce widget affiche le nombre d’instances en cours d’exécution en fonction de la tendance d’utilisation au cours des 30 derniers jours.
- Cliquez dessus pour ouvrir le rapport Instances Over Time (Instances au fil du temps).

### <a name="disks"></a>Disques
Ce widget met en évidence le nombre et le volume totaux des disques disponibles et utilisés.
- Cliquez dessus pour ouvrir le rapport Active Disks (Disques actifs).

### <a name="instance-type-distribution"></a>Instance Type Distribution (Distribution des types d’instances)
Ce widget met en évidence les types d’instances dans un graphique à secteurs.
- Cliquez dessus pour ouvrir le rapport Instance Distribution (Distribution des instances) qui présente une ventilation de vos instances actives par l’agrégation sélectionnée.

### <a name="compute-instances---daily-trend"></a>Compute Instances - Daily Trend (Instances de calcul - Tendance quotidienne)
Ce widget met en évidence les instances de calcul (actives, réservées et à la demande) par jour pour les 30 derniers jours.
- Pointez sur le graphique pour afficher le nombre d’instances de calcul, par type et par jour.
- Cliquez dessus pour ouvrir le rapport Instances Over Time (Instances au fil du temps).

### <a name="all-buckets-s3"></a>All Buckets (S3) (Tous les compartiments (S3))
Ce widget met en évidence le stockage S3 total et le nombre d’objets stockés.
- Cliquez dessus pour ouvrir le tableau de bord S3 Tracker (Traqueur S3). Le tableau de bord vous aide à trouver, à analyser et à afficher votre utilisation du stockage et les tendances actuelles.

### <a name="sql-db-instances-rds"></a>SQL DB Instances (RDS) (Instances SQL DB (RDS))
Ce widget met en évidence le nombre d’instances Amazon RDS en cours d’exécution en fonction de la tendance des 30 derniers jours.
- Cliquez dessus pour ouvrir le rapport RDS Instance Over Time (Instances RDS au fil du temps).

## <a name="optimizer-dashboard"></a>Tableau de bord Optimizer (Optimiseur)
Ce tableau de bord affiche les suggestions de réduction de taille, les ressources inutilisées et les économies potentielles.  
![Tableau de bord Optimizer montrant différents rapports](./media/dashboards/optimizer-dashboard.png)

### <a name="ri-calculator"></a>RI Calculator (Calculatrice d’instances réservées)
Ce widget affiche des suggestions d’achat en lien avec le nombre d’instances réservées, et met en évidence les économies annuelles potentielles.
- Cliquez dessus pour ouvrir Reserved Instance Calculator (Calculatrice d’instances réservées) qui vous permet de déterminer quand utiliser des plans de tarification d’instances à la demande ou réservées.

### <a name="sizing"></a>Dimensionnement
Ce widget met en évidence le dimensionnement recommandé et les économies potentielles en cas d’implémentation.
- Cliquez dessus pour ouvrir le rapport EC2 Cost Effective Sizing Recommendations (Suggestions de dimensionnement EC2 rentable).

### <a name="unused-ri-detector"></a>Unused RI Detector (Détecteur d’instance réservée inutilisée)
Ce widget met en évidence le nombre de réservations Amazon EC2 inutilisées.
- Cliquez dessus pour ouvrir le rapport Currently Unused Reservations (Réservations actuellement inutilisées) qui présente les réservations inutilisées que vous pouvez modifier.

###  <a name="available-disks"></a>Available Disks (Disques disponibles)
Ce widget met en évidence le nombre de disques non attachés dans votre déploiement.
- Cliquez dessus pour ouvrir le rapport Unattached Disks (Disques non attachés).

### <a name="rds-ri-calculator"></a>RDS RI Calculator (Calculatrice d’instances réservées RDS)
Ce widget met en évidence le nombre de suggestions de réservation pour vos instances Amazon RDS et les économies potentielles.
- Cliquez dessus pour ouvrir le rapport RDS RI Buying Recommendations (Suggestions d’achat d’instance réservée RDS) qui présente les suggestions de Cloudyn pour l’utilisation d’instances réservées au lieu d’instances à la demande.

### <a name="rds-sizing"></a>Dimensionnement RDS
Ce widget affiche le nombre de suggestions de dimensionnement et les économies potentielles.
- Cliquez dessus pour ouvrir le rapport RDS Sizing Recommendations (Suggestions de dimensionnement RDS) qui affiche des suggestions détaillées en matière de dimensionnement d’Amazon RDS.

Les suggestions d’optimisation sont basées sur les données d’utilisation et de performances analysées au cours du dernier mois.

## <a name="s3-tracker-dashboard"></a>Tableau de bord S3 Tracker (Traqueur S3)
Le tableau de bord S3 Tracker (Traqueur S3) vous aide à trouver, à analyser et à afficher votre utilisation du stockage et les tendances actuelles.  
![Tableau de bord S3 Tracker montrant différents rapports](./media/dashboards/s3-tracker-dashboard.png)

### <a name="all-buckets"></a>All Buckets (Tous les compartiments)
Ce widget met en évidence la taille totale de tous vos compartiments, exprimée en Go, et le nombre total d’objets dans ceux-ci.
- Cliquez dessus pour ouvrir le rapport Distribution of S3 Size (Répartition de taille S3). Ce rapport vous permet d’analyser votre taille S3 par compartiment, le dossier de niveau supérieur, la classe de stockage et l’état du contrôle de version.

### <a name="bucket-properties"></a>Bucket Properties (Propriétés des compartiments)
Ce widget met en évidence le nombre total de compartiments de stockage.
- Cliquez dessus pour afficher le rapport S3 Bucket Properties (Propriétés des compartiments S3).

### <a name="scan-status"></a>Scan Status (État de l’analyse)
Ce widget met en évidence le moment où la dernière analyse S3 a été effectuée et celui du prochain démarrage.
- Cliquez dessus pour ouvrir le rapport S3 Scan Status (État de l’analyse S3).

### <a name="storage-by-bucket"></a>Storage by Bucket (Stockage par compartiment)
Ce widget met en évidence le pourcentage que chaque classe de stockage de compartiment utilise.
- Cliquez dessus pour ouvrir le rapport Distribution of S3 Size (Répartition de taille S3). Ce rapport vous permet d’analyser votre taille S3 par compartiment, le dossier de niveau supérieur, la classe de stockage et l’état du contrôle de version.

### <a name="number-of-objects-by-bucket"></a>Number of Objects by Bucket (Nombre d’objets par compartiment)
Ce widget met en évidence le nombre d’objets par compartiment en chiffres réels et en pourcentage. Pointez sur le compartiment pour voir le nombre total d’objets.
- Cliquez sur le widget pour ouvrir le rapport Distribution of S3 Size (Répartition de taille S3) (basé sur une analyse).

## <a name="cloud-comparison-dashboard"></a>Tableau de bord Cloud Comparison (Comparaison cloud)
Le tableau de bord Cloud Comparison (Comparaison cloud) vous permet de comparer les coûts de différents fournisseurs de cloud en fonction de la tarification, du type de processeur et de la taille de la RAM.  
![Tableau de bord Cloud Comparison montrant différents rapports](./media/dashboards/cloud-comparison-dashboard.png)

### <a name="ec2-cost-in-azure-by-instance-type"></a>EC2 Cost in Azure by Instance Type (Coût EC2 dans Azure par type d’instance)
Ce widget met en évidence les 30 derniers jours d’utilisation aux taux d’utilisation à la demande. Il compare le coût actuel d’Amazon EC2 au coût potentiel dans Azure.
- Pointez sur les barres pour comparer les coûts par type d’instance.
- Cliquez sur le widget pour ouvrir le rapport Porting Your Deployment – Cost Analysis (Portage de votre déploiement – Rapport d’analyse des coûts).

### <a name="ec2-cost-in-azure"></a>EC2 Cost in Azure (Coût d’EC2 dans Azure)
Ce widget affiche vos coûts Amazon EC2 actuels et les compare à Azure. La comparaison est basée sur les 30 derniers jours d’utilisation aux taux d’utilisation à la demande.
- Cliquez sur le widget pour ouvrir le rapport Porting Your Deployment – Cost Analysis (Portage de votre déploiement – Rapport d’analyse des coûts).

### <a name="ec2azure-instance-type-mapping"></a>EC2/Azure Instance Type Mapping (Mappage de type d’instance EC2/Azure)
Ce widget met en évidence le meilleur mappage d’unités de calcul élastiques entre Amazon EC2 et Azure.
- Cliquez dessus pour ouvrir le rapport Instance Type Mapping (Mappage de type d’instance).

## <a name="next-steps"></a>Étapes suivantes
- Lisez l’article [Utiliser des rapports Cloudyn](use-reports.md) pour en savoir plus sur les rapports.
