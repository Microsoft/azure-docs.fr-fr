---
title: Résoudre les problèmes de facturation Azure CSP à l’aide de tableaux croisés dynamiques créés à partir de fichiers d’utilisation
description: Cet article vous aide à résoudre les problèmes de facturation Azure CSP à l’aide de tableaux croisés dynamiques créés à partir de vos fichiers d’utilisation CSV.
author: banders
ms.reviewer: isvargas
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: troubleshooting
ms.date: 10/09/2020
ms.author: banders
ms.openlocfilehash: 6525d809805da3a19a0efe423306f18d8e67a646
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92026599"
---
# <a name="troubleshoot-csp-billing-issues-with-usage-file-pivot-tables"></a>Résoudre les problèmes de facturation CSP à l’aide de tableaux croisés dynamiques créés à partir de fichiers d’utilisation

Cet article vous aide à résoudre les problèmes de facturation CSP à l’aide de tableaux croisés dynamiques que vous utilisez dans vos fichiers de rapprochement (utilisation) sur l’Espace partenaires. Les fichiers d’utilisation Azure contiennent toutes les informations relatives à l’utilisation et à la consommation Azure. Les informations contenues dans ces fichiers peuvent vous aider à :

- Comprendre la façon dont les réservations Azure sont utilisées et appliquées
- Rapprocher les informations présentées dans Azure Cost Management avec celles de votre facture
- Gérer une flambée des coûts
- Calculer le montant d’un remboursement pour un contrat de niveau de service

Les informations de vos fichiers d’utilisation vous permettent de mieux comprendre les problèmes d’utilisation et de mieux les diagnostiquer. Les fichiers d’utilisation sont générés au format CSV. Étant donné que les fichiers d’utilisation au format CSV peuvent être volumineux, il est plus facile de les manipuler et de les afficher sous la forme de tableaux croisés dynamiques dans un tableur comme Excel. Les exemples de cet article utilisent Excel. Cependant, vous pouvez utiliser le tableur de votre choix.

Seuls les administrateurs de facturation et les administrateurs généraux peuvent télécharger les fichiers de rapprochement. Pour plus d’informations, consultez [Lire les lignes des fichiers de rapprochement de l’Espace partenaires](/partner-center/use-the-reconciliation-files).

## <a name="get-the-data-and-format-it"></a>Récupérer les données et les mettre en forme

Étant donné que les fichiers d’utilisation Azure sont au format CSV, vous devez préparer les données pour les utiliser dans Excel. Utilisez les étapes suivantes pour présenter les données sous forme de table.

1. Téléchargez le fichier d’utilisation à l’aide des instructions fournies dans [Rechercher votre facture](/partner-center/read-your-bill#find-your-bill).
1. Ouvrez le fichier dans Excel.
1. Les données non mises en forme ressemblent à l’exemple suivant.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/raw-csv-data.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/raw-csv-data.png" :::
1. Sélectionnez le premier champ de la table : **PartnerID**.
1. Appuyez sur Ctrl+Maj+Flèche bas, puis sur Ctrl+Maj+Flèche droite pour sélectionner toutes les informations de la table.
1. Dans le menu supérieur, sélectionnez **Insérer** > **Table**. Dans la zone Créer une table, sélectionnez **Ma table comporte des en-têtes**, puis sélectionnez **OK**.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/create-table-dialog.png" alt-text="Exemple montrant des données non mises en forme dans Excel" :::
1. Dans le menu supérieur, sélectionnez **Insérer** > **Tableau croisé dynamique**, puis sélectionnez **OK**. Cette action crée une nouvelle feuille dans le fichier et affiche la zone Champs de tableau croisé dynamique située sur le côté droit de la feuille.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/pivot-table-fields.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/pivot-table-fields.png" :::

La zone Champs de tableau croisé dynamique est une zone de glisser-déposer. Passez à la section suivante pour créer le tableau croisé dynamique.

## <a name="create-pivot-table-to-view-azure-costs-by-resources"></a>Créer un tableau croisé dynamique afin d’afficher les coûts Azure pour chaque ressource

Dans cette section, vous allez créer un tableau croisé dynamique dans lequel vous pourrez résoudre les problèmes d’utilisation générale d’Azure. Cet exemple de table peut vous aider à identifier le service qui consomme le plus de ressources. Vous pouvez voir quelles sont les ressources les plus coûteuses et voir la façon dont un service est facturé.

1. Dans la zone Champs de tableau croisé dynamique, faites glisser **Nom du service** et **Ressource** vers la zone **Lignes**. Placez **Ressource** sous **Nom du service**.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/rows-section.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/rows-section.png" :::
1. Ensuite, placez **Total après impôts** dans la zone **Valeurs**. Vous pouvez également utiliser la colonne Quantité consommée pour obtenir des informations sur les unités de consommation et les transactions. Par exemple, le nombre de gigaoctets et le nombre d’heures, ou les transactions au lieu des coûts dans différentes devises comme USD, EUR et INR.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/add-pivot-table-fields.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/add-pivot-table-fields.png" :::
1. Vous disposez maintenant d’un tableau de bord pour l’investigation de la consommation générale. Vous pouvez choisir d’afficher les données d’un seul service à l’aide des options de filtrage du tableau croisé dynamique.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/pivot-table-filter-option-row-label.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/pivot-table-filter-option-row-label.png" :::
    Pour filtrer à un deuxième niveau dans un tableau croisé dynamique, par exemple au niveau d’une ressource, sélectionnez un élément de deuxième niveau dans la table.
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/pivot-table-filter-option-select-field.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/pivot-table-filter-option-select-field.png" :::
1. Pour appliquer des filtres supplémentaires, vous pouvez ajouter **SubscriptionID** et **Nom de l’entreprise du client** dans la zone **Filtres**, puis sélectionner l’étendue souhaitée.

## <a name="create-a-pivot-table-to-view-azure-usage-by-date"></a>Créer un tableau croisé dynamique pour afficher l’utilisation d’Azure par date

Dans cette section, vous allez créer un tableau croisé dynamique dans lequel vous pourrez résoudre les problèmes d’utilisation générale d’Azure en fonction de la quantité consommée et de la date. Il est utile d’identifier les pics de coûts de facturation par date et par service. Vous pouvez voir quelles sont les ressources les plus coûteuses et voir la façon dont un service est facturé.

Votre fichier de rapprochement comporte deux tables. L’une se trouve en haut du document (la table principale) et l’autre se trouve en bas. Cette deuxième table présente plus ou moins les mêmes informations, mais elle ne contient pas les détails concernant les tarifs ou les coûts. Elle contient les dates d’utilisation et la quantité consommée.

:::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/reconciliation-file-two-tables.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/reconciliation-file-two-tables.png" :::

1. Utilisez les étapes de la section [Récupérer les données et les mettre en forme](#get-the-data-and-format-it) pour créer un tableau Excel avec les informations situées au bas du fichier de rapprochement.
1. Lorsque la table est prête et que vous disposez d’une feuille de tableau croisé dynamique, utilisez les étapes de la section Créer un tableau croisé dynamique afin d’afficher les coûts Azure pour chaque ressource pour préparer le tableau de bord. Au lieu d’utiliser le Total après impôts, placez **Quantité consommée** dans la zone **Valeurs**.
1. Ajoutez **Date d’utilisation** à la section Colonnes. Le tableau croisé dynamique doit ressembler à l’exemple qui suit.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/final-pivot-table-fields.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/final-pivot-table-fields.png" :::
1. Vous disposez maintenant d’un tableau de bord qui affiche l’utilisation par date. Vous pouvez développer chaque mois en sélectionnant le symbole **+** .

Le tableau de bord affiche la quantité consommée dans différentes unités telles que les gigaoctets, les heures ou les transferts.

Pour afficher le coût par jour, vous pouvez ajouter **GUID de ressource** à la zone **Lignes**. Dans la table du haut, ajoutez le prix unitaire (**ListPrice**) de la ressource. Multipliez **ListPrice** par la **Quantité consommée** afin de calculer vos coûts avant impôts. Les deux montants doivent correspondre.

Certaines ressources (ou services) ont des tarifs dégressifs qui dépendent de la quantité consommée. Par exemple, certaines ressources ont un tarif plus élevé pour les 100 premiers gigaoctets consommés, puis un tarif inférieur pour les gigaoctets utilisés par la suite. Prenez bien en compte les tarifs dégressifs lorsque vous calculez les coûts manuellement.

## <a name="create-pivot-table-to-view-cost-for-a-specific-resource"></a>Créer un tableau croisé dynamique pour afficher le coût d’une ressource

Une même ressource peut entraîner différents frais pour différents services. Par exemple, une machine virtuelle peut engendrer des frais en rapport avec le calcul, les licences de système d’exploitation, la bande passante (transferts de données), l’utilisation des instances réservées et le stockage des captures instantanées. Si vous souhaitez connaître l’utilisation globale de certaines ressources, vous pouvez effectuer les étapes suivantes qui vous aideront à créer un tableau de bord à partir de vos fichiers d’utilisation.

Les fichiers de rapprochement ne contiennent pas de détails concernant les ressources. Par conséquent, vous devez utiliser le fichier d’utilisation agrégé. Contactez le [support Facturation Azure](https://go.microsoft.com/fwlink/?linkid=2083458) pour obtenir le fichier d’utilisation agrégé de votre abonnement. Les fichiers agrégés sont générés au niveau de l’abonnement. Les données non mises en forme ressemblent à l’exemple suivant.

:::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/raw-aggregated-usage-file.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/raw-aggregated-usage-file.png" :::

Le fichier contient les colonnes suivantes.

- **UsageStart** et **UsageEnd** : date pour chaque élément de ligne (chaque unité d’utilisation). Par exemple, chaque jour.
- **MeteredResourceID** : dans Azure, cela correspond à l’ID du compteur.
- **Propriétés** : contient l’ID de l’instance (nom de la ressource) avec d’autres détails tels que l’emplacement.
- **Quantité** : quantité consommée dans le fichier de rapprochement.

1. Sélectionnez le premier champ de la table : **PartnerID**.  
1. Appuyez sur Ctrl+Maj+Flèche bas, puis sur Ctrl+Maj+Flèche droite pour sélectionner toutes les informations de la table.
1. Dans le menu supérieur, sélectionnez **Insérer** > **Table**. Dans la zone Créer une table, sélectionnez **Ma table comporte des en-têtes**, puis sélectionnez **OK**.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/create-table-dialog.png" alt-text="Exemple montrant des données non mises en forme dans Excel" :::
1. Dans le menu supérieur, sélectionnez **Insérer** > **Tableau croisé dynamique**, puis sélectionnez **OK**. Cette action crée une nouvelle feuille dans le fichier et affiche la zone Champs de tableau croisé dynamique située sur le côté droit de la feuille.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/pivot-table-fields-reconciliation.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/pivot-table-fields-reconciliation.png" :::
1. Ensuite, ajoutez **MeteredResourceID** à la zone **Lignes**, et **Quantité** à la zone **Valeurs**. Les résultats affichent les informations d’utilisation générale. Pour plus d’informations, placez **UsageEndDateTime** dans la zone **Colonnes**.  
    :::image type="content" source="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/overall-usage.png" alt-text="Exemple montrant des données non mises en forme dans Excel" lightbox="./media/troubleshoot-csp-billing-issues-usage-file-pivot-tables/overall-usage.png" :::
1. Pour afficher un rapport global, ajoutez des **Propriétés** à **Lignes** sous **MeteredResourceID**. Vous verrez ainsi un tableau de bord complet sur votre utilisation.
1. Pour afficher uniquement les données concernant une ressource, ajoutez des **Propriétés** à la zone **Filtres** et sélectionnez l’utilisation souhaitée. Vous pouvez utiliser la recherche pour rechercher un nom de ressource.
    Pour afficher le coût de la ressource, recherchez la quantité consommée totale et multipliez cette valeur par le tarif. Le tarif est propre à chaque GUID de ressource (MeteredResourceID). Si une ressource utilise plusieurs MeteredResourceID, vous devez noter la valeur totale de chaque ID.

[!INCLUDE [Transform data before using large usage files](../../../includes/cost-management-billing-transform-data-before-using-large-usage-files.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer avec Azure Cost Management pour les partenaires](../costs/get-started-partners.md)