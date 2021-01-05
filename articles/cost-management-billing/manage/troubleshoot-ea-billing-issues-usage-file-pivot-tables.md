---
title: Résoudre les problèmes de facturation Azure EA à l’aide de tableaux croisés dynamiques créés à partir de fichiers d’utilisation
description: Cet article vous aide à résoudre les problèmes de facturation Contrat Entreprise (EA) à l’aide de tableaux croisés dynamiques créés à partir de vos fichiers d’utilisation CSV.
author: banders
ms.reviewer: isvargas
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: troubleshooting
ms.date: 10/09/2020
ms.author: banders
ms.openlocfilehash: b32bb979176af8a8a3751db8edf9c129caf1002e
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131987"
---
# <a name="troubleshoot-ea-billing-issues-with-usage-file-pivot-tables"></a>Résoudre les problèmes de facturation EA à l’aide de tableaux croisés dynamiques créés à partir de fichiers d’utilisation

Cet article vous aide à résoudre les problèmes de facturation Contrat Entreprise (EA) à l’aide de tableaux croisés dynamiques créés à partir de vos fichiers d’utilisation. Les fichiers d’utilisation Azure contiennent toutes les informations relatives à l’utilisation et à la consommation Azure. Les informations contenues dans ces fichiers peuvent vous aider à :

- Comprendre la façon dont les réservations Azure sont utilisées et appliquées
- Rapprocher les informations présentées dans Azure Cost Management avec celles de votre facture
- Gérer une augmentation des coûts
- Calculer le montant d’un remboursement pour un contrat de niveau de service

Les informations de vos fichiers d’utilisation vous permettent de mieux comprendre les problèmes d’utilisation et de mieux les diagnostiquer. Les fichiers d’utilisation sont générés au format CSV. Étant donné que les fichiers d’utilisation au format CSV peuvent être volumineux, il est plus facile de les manipuler et de les afficher sous la forme de tableaux croisés dynamiques, dans un tableur tel qu’Excel. Les exemples de cet article utilisent Excel. Cependant, vous pouvez utiliser le tableur de votre choix.

Seuls les administrateurs EA, les propriétaires de comptes et les administrateurs de service peuvent télécharger les fichiers d’utilisation.

## <a name="get-the-data-and-format-it"></a>Récupérer les données et les mettre en forme

Étant donné que les fichiers d’utilisation Azure sont au format CSV, vous devez préparer les données pour les utiliser dans Excel. Utilisez les étapes suivantes pour présenter les données sous forme de table.

1. Téléchargez le fichier Détails de l’utilisation version 2 avec Tous les frais (utilisation et achats) en suivant les instructions fournies dans [Télécharger l’utilisation pour les clients EA](./download-azure-invoice-daily-usage-date.md#download-usage-for-ea-customers).
1. Ouvrez le fichier dans Excel.
1. Les données non mises en forme ressemblent à l’exemple suivant.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/raw-csv-data-ea.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/raw-csv-data-ea.png" :::
1. Sélectionnez le premier champ de la table, c’est-à-dire celui qui contient le titre de la première colonne **BillingAccountID**.
1. Appuyez sur Ctrl+Maj+Flèche bas, puis sur Ctrl+Maj+Flèche droite pour sélectionner toutes les informations de la table.
1. Dans le menu supérieur, sélectionnez **Insérer** > **Table**. Dans la zone Créer une table, sélectionnez **Ma table comporte des en-têtes**, puis sélectionnez **OK**.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/create-table-dialog.png" alt-text="Exemple montrant des données non mises en forme dans Excel" :::
1. Dans le menu supérieur, sélectionnez **Insérer** > **Tableau croisé dynamique**, puis sélectionnez **OK**. Cette action crée une nouvelle feuille dans le fichier. Vous accédez à la zone Champs de tableau croisé dynamique sur le côté droit de la feuille.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-fields.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-fields.png" :::

La zone Champs de tableau croisé dynamique est une zone de glisser-déposer. Passez à la section suivante pour créer le tableau croisé dynamique.

## <a name="create-pivot-table-to-view-azure-costs-by-resources"></a>Créer un tableau croisé dynamique afin d’afficher les coûts Azure pour chaque ressource

Dans cette section, vous allez créer un tableau croisé dynamique dans lequel vous pourrez résoudre les problèmes d’utilisation générale d’Azure. Cet exemple de table peut vous aider à identifier le service qui consomme le plus de ressources. Vous pouvez voir quelles sont les ressources les plus coûteuses et voir la façon dont un service est facturé.

1. Dans la zone Champs de tableau croisé dynamique, faites glisser **Catégorie du compteur** et **Produit** vers la section **Lignes**. Placez **Produit** sous **Catégorie du compteur**.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/rows-section.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/rows-section.png" :::
1. Ensuite, ajoutez la colonne **Cost** à la section **Valeurs**. Vous pouvez également utiliser la colonne Consumed Quantity pour obtenir des informations sur les unités de consommation et les transactions. Par exemple, le nombre de gigaoctets et le nombre d’heures, ou les transactions au lieu des coûts dans différentes devises comme USD, EUR et INR.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/add-pivot-table-fields.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/add-pivot-table-fields.png" :::
1. Vous disposez maintenant d’un tableau de bord pour l’investigation de la consommation générale. Vous pouvez choisir d’afficher les données d’un seul service à l’aide des options de filtrage du tableau croisé dynamique.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-filter-option-row-label.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-filter-option-row-label.png" :::
    Pour filtrer à un deuxième niveau dans un tableau croisé dynamique, par exemple au niveau d’une ressource, sélectionnez un élément de deuxième niveau dans la table.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-filter-option-select-field.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-filter-option-select-field.png" :::
1. Faites glisser la colonne **ResourceID** vers la zone **Lignes** sous **Produit** pour voir le coût de chaque service par ressource. Pour afficher le détail des prix, affichez le prix unitaire (UnitPrice) de votre organisation et recherchez **Produit** dans la première colonne de la liste des prix.
1. Ajoutez la colonne **Date** à la zone **Colonnes** pour afficher la consommation quotidienne du produit.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-date.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-date.png" :::
1. Développez et réduisez les mois avec le symbole **+** situé dans la colonne de chaque mois.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-month-expand-collapse.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-month-expand-collapse.png" :::  
    L’ajout des colonnes **Cost** et **Quantity** dans la zone **Valeurs** est facultatif. Cela crée deux colonnes pour chaque section de données située sous chaque mois et chaque jour lorsque la colonne Date se trouve dans la section Colonnes du tableau croisé dynamique.
1. Pour appliquer des filtres supplémentaires, vous pouvez ajouter les colonnes SubscriptionID, Department, ResourceGroup, Tags ou Cost Center à la zone **Filtres** et sélectionner l’élément de votre choix.

## <a name="create-pivot-table-to-view-cost-for-a-specific-resource"></a>Créer un tableau croisé dynamique pour afficher le coût d’une ressource

Une même ressource peut entraîner différents frais pour différents services. Par exemple, une machine virtuelle peut engendrer des frais en rapport avec le calcul, les licences de système d’exploitation, la bande passante (transferts de données), l’utilisation des instances réservées et le stockage des captures instantanées. Si vous souhaitez connaître l’utilisation globale de certaines ressources, vous pouvez effectuer les étapes suivantes qui vous aideront à créer un tableau de bord à partir de vos fichiers d’utilisation.

1. Dans le menu de droite, faites glisser **ResourceID** vers la section **Filtre** dans le menu Tableau croisé dynamique.
1. Sélectionnez la ressource dont vous souhaitez voir les coûts. Pour trouver la ressource, tapez son nom dans la zone **Recherche**.
1. Ajoutez **Catégorie du compteur** et **Produit** à la section Lignes. Placez **Produit** sous **Catégorie du compteur**.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-fields-meter-category.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-fields-meter-category.png" :::
1. Ensuite, ajoutez la colonne **Cost** à la section **Valeurs**. Vous pouvez également utiliser la colonne Consumed Quantity pour obtenir des informations sur les unités de consommation et les transactions. Par exemple, le nombre de gigaoctets et le nombre d’heures, ou les transactions au lieu des coûts dans différentes devises comme USD, EUR et INR. Vous disposez maintenant d’un tableau de bord qui affiche tous les services consommés par la ressource.
1. Ajoutez la colonne **Date** à la section**Colonnes**. Il montre la consommation quotidienne.
1. Vous pouvez développer et réduire les données à l’aide du symbole **+** situé dans la colonne de chaque mois.  
    :::image type="content" source="./media/troubleshoot-ea-billing-issues-usage-file-pivot-tables/pivot-table-month-expand-collapse.png" alt-text="Exemple montrant des données non mises en forme dans Excel" :::

[!INCLUDE [Transform data before using large usage files](../../../includes/cost-management-billing-transform-data-before-using-large-usage-files.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Explorer et analyser les coûts avec une analyse des coûts](../costs/quick-acm-cost-analysis.md)