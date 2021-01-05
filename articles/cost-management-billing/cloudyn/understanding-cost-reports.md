---
title: Compréhension des rapports de gestion des coûts Cloudyn dans Azure
description: Cet article vous permet de comprendre la structure et les fonctions de base des rapports de gestion des coûts Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 9afe34097d005daeedf46fbaf30b37e53a8e3658
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132038"
---
# <a name="understanding-cloudyn-cost-management-reports"></a>Compréhension des rapports de gestion des coûts Cloudyn

Cet article vous permet de comprendre la structure et les fonctions de base des rapports de gestion des coûts Cloudyn. La plupart des rapports Cloudyn sont intuitifs et ont une apparence uniforme. Après lecture de cet article, vous êtes prêt à utiliser tous les rapports de gestion des coûts. La plupart des fonctionnalités standard sont disponibles dans les différents rapports, vous permettant de naviguer facilement dans les rapports. Les rapports sont personnalisables et vous pouvez sélectionner plusieurs options pour calculer et afficher des résultats.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="report-fields-and-options"></a>Champs et options des rapports

Vous trouverez ci-dessous un exemple de rapport des coûts dans le temps. La plupart des rapports Cloudyn ont une mise en page similaire.

![Exemple de rapport des coûts dans le temps avec des zones numérotées correspondant aux descriptions](./media/understanding-cost-reports/sample-report.png)

Chaque zone numérotée dans l’image précédente est décrite en détail dans les informations suivantes :

1. **Plage de dates**

    Utilisez la liste des plages de dates pour définir un intervalle de temps de rapport à l’aide d’une balise prédéfinie ou personnalisée.
2. **Filtre enregistré**

    Utilisez la liste des filtres enregistrés pour enregistrer les groupes et les filtres actuellement appliqués au rapport. Les filtres enregistrés sont disponibles dans les rapports sur les coûts et les performances, notamment :

      - Analyse des coûts
      - Allocation
      - Gestion des ressources
      - Optimization

   Saisissez un nom de filtre et cliquez sur **Enregistrer**.

3. **Balises**

    Utilisez la zone Balises pour grouper par catégorie de balises. Les balises répertoriées dans le menu sont des balises du département Azure ou du centre de coûts ou encore des balises d’entité de coûts ou d’abonnement Cloudyn. Sélectionnez les balises pour filtrer les résultats. Vous pouvez également saisir un nom de balise (mot clé) pour filtrer les résultats.

    ![Exemple de liste de balises selon lesquelles filtrer les résultats](./media/understanding-cost-reports/select-options.png)

    Cliquez sur **Ajouter** pour ajouter un nouveau filtre.

    ![Ajouter une zone de filtre montrant les options et les conditions selon lesquelles effectuer le filtrage](./media/understanding-cost-reports/add-filter.png)

    Le regroupement ou le filtrage des balises ne s’applique pas aux ressources Azure ni aux balises de groupes de ressources.

    Le regroupement et le filtrage des balises d’allocation des coûts sont disponibles dans l’option du menu **Groupes**.

4. **Groupes dans les rapports**

    Utilisez les Groupes dans les rapports d’analyse des coûts pour afficher les catégories standard et détaillées à partir des données de facturation dans votre rapport.  Toutefois, les groupes dans les rapports d’allocation des coûts affichent des catégories de vue avec indicateur. Les catégories avec indicateur sont définies dans le modèle d’allocation des coûts et les catégories détaillées standard dans les données de facturation.

    ![Premier exemple de liste de balises selon lesquelles effectuer les regroupements](./media/understanding-cost-reports/groups-tags01.png)

    ![Second exemple de liste de balises selon lesquelles effectuer les regroupements](./media/understanding-cost-reports/groups-tags02.png)

    Dans les rapports d’allocation des coûts, les groupes des catégories de groupes avec indicateur doivent inclure :
      - Balises
      - Balises des groupes de ressources
      - Balises des entités de coûts Cloudyn
      - Catégories de balises d’abonnement à des fins d’allocation des coûts

   Les exemples doivent inclure :
   - Centre de coûts
   - department
   - Application
   - Environnement
   - Code de coût

     Voici une liste des groupes intégrés disponibles dans les rapports :

     - **Type de coût**
     - Sélectionnez un type de coût ou de plusieurs types de coûts, ou sélectionnez tout. Les types de coûts comprennent :
       - Frais ponctuels
       - Support
       - Coût d’utilisation
     - **Client**
       - Sélectionnez un client spécifique, plusieurs clients, ou tous les clients.
     - **Nom du compte**
       - Le nom du compte ou abonnement. Dans Azure, il s’agit du nom de l’abonnement Azure.
     - **No. de compte**
       - Sélectionnez un compte, plusieurs comptes ou tous les comptes. Dans Azure, il s’agit du GUID de l’abonnement Azure.
     - **Compte parent**
       - Sélectionnez le compte parent, plusieurs comptes ou sélectionnez tout.
     - **Service**
       - Sélectionnez un service, plusieurs services, ou tous les services.
     - **Fournisseur**
       - Le fournisseur cloud auquel les ressources et les frais sont associés.
     - **Région**
       - Région dans laquelle la ressource est hébergée.
     - **Zone de disponibilité**
       - Emplacements AWS isolés dans une région.
     - **Type de ressource**
       - Le type de ressource en cours d’utilisation.
     - **Sub-Type**
       - Sélectionnez le sous-type.
     - **opération**
       - Sélectionnez l’opération ou **Afficher tout**.
     - **Modèle de prix**
       - Frais de démarrage uniquement
       - Aucun frais de démarrage
       - Frais de démarrage partiels
       - À la demande
       - Réservation
       - Zone
     - **Type de frais**
       - Sélectionnez le type de frais, négatif, positif ou les deux.
     - **Location**
       - Indique si une machine est utilisée comme une machine dédiée.
     - **Type d'utilisation**
       - Le type d’utilisation peut être à frais uniques ou à frais d’abonnement.

5. **Filtres**

    Utilisez des filtres uniques ou à sélection multiple pour définir des plages sur des valeurs sélectionnées. Pour définir un filtre, cliquez sur **Ajouter** puis sélectionnez des catégories et des valeurs de filtres.

6. **Modèle de coût**

    Utilisez le Modèle de coût pour sélectionner un modèle de coût précédemment créé avec l’allocation des coûts 360. Vous pouvez avoir plusieurs modèles de coût Cloudyn, en fonction de vos exigences d’allocation de coûts. Certaines équipes de votre organisation peuvent avoir des exigences d’allocation de coûts différentes des autres. Chaque équipe peut disposer de son propre modèle de coût dédié.

    Pour plus d’informations sur la création d’une définition d’un modèle d’allocation de coût, consultez la rubrique [Utilisez des balises personnalisées pour allouer des coûts](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs).

7. **Amortissement**

    Utilisez les rapports Amortissement dans l’allocation des coûts pour afficher les frais de service non liés à l’utilisation ou les coûts payables une seule fois. Les coûts sont étalés dans le temps de manière régulière tout au long de leur durée de vie. Par exemple, les frais ponctuels peuvent inclure :
    - Les frais d’assistance annuels
    - les frais annuels liés aux composants de sécurité ;
    - les frais d’achat des instances réservées ;
    - certains éléments de Place de Marché Azure.

   Sous Amortissement, sélectionnez **Coût amorti** ou **Coût réel**.

8. **Résolution :**

    Utilisez Résolution pour sélectionner la résolution de temps dans la plage de dates sélectionnée. Votre résolution de temps détermine le nombre d’unités affichées dans le rapport et peut être :
    - Quotidien
    - Hebdomadaire
    - Mensuelle
    - Trimestrielle
    - Annuelle

9. **Règles d'allocation**

    Utilisez Règles d’allocation pour appliquer ou désactiver le nouveau calcul d’allocation des coûts. Vous pouvez activer ou désactiver le nouveau calcul d’allocation des coûts pour les données de facturation. Le nouveau calcul s’applique aux catégories sélectionnées dans le rapport. Ceci vous permet d’évaluer l’impact du nouveau calcul d’allocation des coûts par rapport aux données de facturation brutes.

10. **Sans catégorie**

    Utilisez Sans catégorie pour inclure ou exclure des coûts sans catégorie dans le rapport.

11. **Afficher/masquer des champs**

    L’option Afficher/masquer n’a aucun impact sur les rapports.

12. **Formats d’affichage**

    Utilisez Formats d’affichage pour sélectionner différentes vues de tableau ou de graphique.

    ![Symboles de formats d’affichage que vous pouvez sélectionner](./media/understanding-cost-reports/display-formats.png)

13. **Multicolore**

    Utilisez Multicolore pour définir la couleur des graphiques de votre rapport.

14. **Actions**

    Utilisez Action pour enregistrer, exporter ou planifier le rapport.

15. **Stratégie**

    Bien que non représentés, certains rapports incluent une stratégie de calcul des coûts prévus. Dans ces rapports, la stratégie **Consolidée** montre des recommandations pour tous les comptes et abonnements sous l’entité actuelle, comme l’inscription Microsoft ou le payeur AWS. La stratégie **Autonome** montre des recommandations pour un seul compte ou abonnement, comme s’il n’existait aucun autre abonnement. La stratégie que vous sélectionnez varie selon la stratégie d’optimisation utilisée par votre organisation. Les projections de coûts sont basées sur les 30 derniers jours d’utilisation.

## <a name="save-and-schedule-reports"></a>Enregistrer et planifier des rapports

Après avoir créé un rapport, vous pouvez l’enregistrer pour une utilisation ultérieure. Les rapports enregistrés sont disponibles dans **Mes outils** > **Mes rapports**. Si vous apportez des modifications à un rapport existant et que vous l’enregistrez, le rapport est enregistré sous une nouvelle version. Vous pouvez également l’enregistrer sous un nouveau rapport.

### <a name="save-a-report-to-the-cloudyn-portal"></a>Enregistrer un rapport dans le portail Cloudyn

Lorsque vous affichez un rapport, cliquez sur **Actions**, puis sélectionnez **Save to my reports** (Enregistrer dans Mes rapports). Nommez le rapport, puis ajoutez votre propre URL ou utilisez l’URL créée automatiquement. Vous pouvez éventuellement **Partager** le rapport publiquement avec d’autres personnes de votre organisation, ou vous pouvez le partager à votre entité. Si vous ne partagez pas le rapport, il reste un rapport personnel que vous êtes le seul à pouvoir afficher. Enregistrez le rapport.


### <a name="save-a-report-to-cloud-provider-storage"></a>Enregistrer un rapport sur un stockage de fournisseur de cloud

Pour enregistrer un rapport dans votre fournisseur de services cloud, vous devez avoir configuré un compte de stockage. Lorsque vous affichez un rapport, cliquez sur **Actions**, puis sélectionnez **Planifier le rapport**. Nommez le rapport, puis ajoutez votre propre URL ou utilisez l’URL créée automatiquement. Sélectionnez **Enregistrer sur un support de stockage**, puis sélectionnez le compte de stockage ou ajoutez-en un nouveau. Entrez un préfixe qui est ajouté au nom de fichier du rapport. Sélectionnez le format de fichier CSV ou JSON, puis enregistrez le rapport.

### <a name="schedule-a-report"></a>Planifier un rapport

Vous pouvez exécuter des rapports à des intervalles planifiés et les envoyer à une liste de destinataire ou dans un compte de stockage de fournisseur de services cloud. Lorsque vous affichez un rapport, cliquez sur **Actions**, puis sélectionnez **Planifier le rapport**. Vous pouvez envoyer le rapport par courrier électronique et l’enregistrer dans un compte de stockage. Sous **Planifier**, sélectionnez l’intervalle (tous les jours, toutes les semaines ou tous les mois). Pour les intervalles hebdomadaire et mensuel, sélectionnez le jour ou les dates de remise et sélectionnez l’heure. Enregistrez le rapport planifié. Si vous sélectionnez le format de rapport Excel, le rapport est envoyé en pièce jointe. Lorsque vous sélectionnez le format de contenu par courrier électronique, les résultats du rapport affichés au format graphique sont délivrés dans un graphique.

### <a name="export-a-report-as-a-csv-file"></a>Exporter un rapport dans un fichier CSV

Lorsque vous affichez un rapport, cliquez sur **Actions**, puis sélectionnez **Export all report data** (Exporter toutes les données du rapport). Une fenêtre contextuelle s’affiche et un fichier CSV est téléchargé.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez plus d’informations sur les rapports qui sont inclus dans Cloudyn dans [Utiliser des rapports Cloudyn](./use-reports.md).
- Découvrez plus d’informations sur l’utilisation des rapports pour créer des [tableaux de bord](./dashboards.md).