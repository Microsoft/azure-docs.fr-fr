---
title: Application automatique d’étiquettes de confidentialité à des données
description: Découvrez comment créer des étiquettes de confidentialité et les appliquer automatiquement à vos données lors d’une analyse.
author: batamig
ms.author: bagol
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 12/03/2020
ms.openlocfilehash: c824e0abea7320a20441e51caa2a05d534ff61b3
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092684"
---
# <a name="automatically-label-your-data-in-azure-purview"></a>Étiquetage automatique de données dans Azure Purview

Cet article explique comment créer des étiquettes de confidentialité Microsoft Information Protection (MIP) et les appliquer automatiquement à des ressources Azure dans Azure Purview.

## <a name="what-are-sensitivity-labels"></a>Définition des étiquettes de confidentialité 

Dans le cadre de leur travail, les membres de votre organisation collaborent avec d’autres personnes en interne ou non. Les données ne restent pas toujours dans votre cloud : elles sont souvent en itinérance sur les différents appareils, applications et services. 

L’itinérance des données doit être sécurisée, protégée et conforme aux stratégies de votre organisation.

Les étiquettes de confidentialité permettent de préciser le niveau de confidentialité de certaines données de l’organisation. Par exemple, il se peut qu’un certain nom de projet soit hautement confidentiel dans votre organisation, alors que ce même terme n’est pas confidentiel pour d’autres organisations. 

### <a name="sensitivity-labels-in-azure-purview"></a>Étiquettes de confidentialité dans Azure Purview

Dans Purview, les classifications sont similaires aux étiquettes d’objet. Elles permettent de marquer et d’identifier les données d’un type spécifique qui se trouvent dans votre patrimoine de données pendant l’analyse.

Purview utilise les mêmes classifications, également appelées « types d’informations sensibles », que Microsoft 365.  Les étiquettes de confidentialité MIP sont créées dans le Centre de sécurité et conformité Microsoft 365. Vous pouvez ainsi étendre vos étiquettes de confidentialité existantes à vos différentes ressources Azure Purview.

> [!NOTE]
> Si les classifications font l’objet d’une correspondance directe (un numéro de sécurité sociale porte la classification **Numéro de sécurité sociale**), les étiquettes de confidentialité sont appliquées lorsqu’un ou plusieurs scénarios et classifications sont détectés ensemble. 
> 

Les étiquettes de confidentialité proposées dans Azure Purview peuvent être appliquées automatiquement aux fichiers et aux colonnes de base de données.

Pour plus d’informations, consultez :

- [En savoir plus sur les étiquettes de confidentialité](/microsoft-365/compliance/sensitivity-labels) dans la documentation de Microsoft 365
- [Présentation des règles d’étiquetage automatique](#what-are-autolabeling-rules)
- [Types de données pris en charge pour les étiquettes de confidentialité dans Azure Purview](#supported-data-types-for-sensitivity-labels-in-azure-purview)

#### <a name="what-are-autolabeling-rules"></a>Présentation des règles d’étiquetage automatique

Les données s’enrichissent et évoluent constamment. S’il est fastidieux de suivre des données non étiquetées et d’y appliquer manuellement des étiquettes, ce n’est pas non plus nécessaire. 

Les règles d’étiquetage automatique sont des conditions indiquant quand une étiquette particulière doit être appliquée. Lorsque ces conditions sont remplies, l’étiquette est automatiquement affectée aux données. Ce système permet de maintenir la cohérence des étiquettes de confidentialité sur les données, à grande échelle.

Lorsque vous créez vos étiquettes, veillez à définir des règles d’étiquetage automatique pour les [fichiers](#define-autolabeling-rules-for-files) et pour les [colonnes de base de données](#define-autolabeling-rules-for-database-columns) pour appliquer automatiquement les étiquettes à chaque analyse des données. 

Après avoir analysé vos données dans Purview, vous pouvez consulter les étiquettes appliquées automatiquement dans le catalogue Purview et les rapports Insight.

#### <a name="supported-data-types-for-sensitivity-labels-in-azure-purview"></a>Types de données pris en charge pour les étiquettes de confidentialité dans Azure Purview

Les étiquettes de confidentialité sont prises en charge dans Azure Purview pour les types de données suivants :

|Type de données  |Sources  |
|---------|---------|
|Étiquetage automatique des fichiers     |      - Stockage Blob Azure  </br>- Azure Data Lake Storage Gen1 et Gen2  |
|Étiquetage automatique des colonnes de base de données     |  - SQL Server </br>- Azure SQL Database </br>- Azure SQL Database Managed Instance   <br> - Azure Synapse  <br> - Azure Cosmos DB   |
| | |

## <a name="how-to-create-sensitivity-labels-in-microsoft-365"></a>Guide pratique pour créer des étiquettes de confidentialité dans Microsoft 365

Si vous ne disposez pas encore d’étiquettes de confidentialité, vous devez les créer et les rendre accessibles à Azure Purview. Il est également possible de modifier les étiquettes de confidentialité existantes pour les rendre accessibles à Azure Purview.

Pour plus d’informations, consultez :

- [Exigences en termes de licence](#licensing-requirements)
- [Extension des étiquettes de confidentialité à Azure Purview](#extending-sensitivity-labels-to-azure-purview)
- [Création d’étiquettes de confidentialité ou modification d’étiquettes existantes](#creating-new-sensitivity-labels-or-modifying-existing-labels)
### <a name="licensing-requirements"></a>Exigences en termes de licence

Les étiquettes de confidentialité MIP sont créées et gérées dans le Centre de sécurité et conformité Microsoft 365. Pour créer des étiquettes de confidentialité utilisables dans Azure Purview, vous devez disposer d’une licence Microsoft 365 E5 active.

Si vous ne disposez pas encore de la licence requise, vous pouvez vous inscrire pour obtenir une version d’évaluation de [Microsoft 365 E5](https://www.microsoft.com/microsoft-365/business/compliance-solutions#midpagectaregion).

### <a name="extending-sensitivity-labels-to-azure-purview"></a>Extension des étiquettes de confidentialité à Azure Purview

Par défaut, les étiquettes de confidentialité MIP ne sont disponibles que pour les ressources de Microsoft 365, où elles sont applicables aux fichiers et aux e-mails.

Pour appliquer des étiquettes de confidentialité MIP à des ressources Azure dans Azure Purview, vous devez accepter explicitement d’étendre les étiquettes et sélectionner celles que vous souhaitez utiliser dans Purview.

En étendant les étiquettes de confidentialité de MIP avec Azure Purview, les organisations peuvent découvrir, classer et analyser le niveau de confidentialité à travers un éventail plus large de sources de données. Le risque de conformité s’en trouve ainsi réduit.

> [!NOTE]
> Étant donné que Microsoft 365 et Azure Purview sont des services distincts, il est possible qu’ils soient déployés dans des régions différentes. Le nom des étiquettes et celui des types d’informations sensibles personnalisés sont considérés comme des données client. Ils sont conservés dans le même emplacement géographique par défaut pour préserver la confidentialité des données et éviter l’application des lois RGPD.
>
> C’est la raison pour laquelle les étiquettes et les types d’informations sensibles personnalisés ne sont pas partagés par défaut avec Azure Purview. Votre consentement est nécessaire pour les utiliser dans Azure Purview.

> [!IMPORTANT]
> Votre consentement permet à Microsoft de partager le nom de l’étiquette et celui du type d’informations sensibles personnalisé pour Azure Purview *et* Azure Security Center (ASC). Microsoft utilise les informations d’étiquette d’Azure Purview pour enrichir vos recommandations et vos alertes dans ASC. 
>
> Le consentement donné dans le Centre de conformité Microsoft 365 s’applique au partage de ces données avec les deux services. Il n’existe actuellement aucun choix permettant de partager des informations d’étiquetage avec Azure Purview uniquement.

**Extension des étiquettes de confidentialité à Purview :**

Dans Microsoft 365, accédez à la page **Information Protection**. Dans **Étendre l’étiquetage aux ressources dans Azure Purview**, sélectionnez le bouton **Activer**, puis sélectionnez **Oui** dans la boîte de dialogue de confirmation qui s’affiche.

Par exemple :

:::image type="content" source="media/create-sensitivity-label/extend-sensitivity-labels-to-purview-small.png" alt-text="Sélection de **Activer** pour étendre les étiquettes de confidentialité à Purview" lightbox="media/create-sensitivity-label/extend-sensitivity-labels-to-purview.png":::
 
Une fois que vous avez étendu l’étiquetage aux ressources dans Azure Purview, vous pouvez sélectionner les étiquettes à rendre disponibles dans Purview. Pour plus d’informations, consultez [Création d’étiquettes de confidentialité ou modification d’étiquettes existantes](#creating-new-sensitivity-labels-or-modifying-existing-labels).
### <a name="creating-new-sensitivity-labels-or-modifying-existing-labels"></a>Création d’étiquettes de confidentialité ou modification d’étiquettes existantes

1. Ouvrez le [Centre de sécurité et conformité Microsoft 365](https://protection.office.com/homepage). 

1. Sous **Solutions**, sélectionnez **Information Protection**, puis **Créer une étiquette**. 

    :::image type="content" source="media/create-sensitivity-label/create-sensitivity-label-full-small.png" alt-text="Création d’étiquettes de confidentialité dans le Centre de sécurité et conformité Microsoft 365" lightbox="media/create-sensitivity-label/create-sensitivity-label-full.png":::

1. Donnez un nom à l’étiquette. Ensuite, sous **Définir l’étendue de cette étiquette**, sélectionnez **Fichiers et e-mails**, puis **Ressources Azure Purview**.
    
    :::image type="content" source="media/create-sensitivity-label/create-label-scope-small.png" alt-text="Création de l’étiquette dans le Centre de sécurité et conformité Microsoft 365" lightbox="media/create-sensitivity-label/create-label-scope.png":::

1. Suivez les autres invites de l’Assistant pour définir les paramètres de votre étiquette. 

    Plus précisément, définissez des règles d’étiquetage automatique pour les fichiers et les colonnes de base de données :

    - [Définition de règles d’étiquetage automatique pour les fichiers](#define-autolabeling-rules-for-files)
    - [Définition de règles d’étiquetage automatique pour les colonnes de base de données](#define-autolabeling-rules-for-database-columns)

    Pour plus d’informations sur les options de l’Assistant, consultez [Fonctions des étiquettes de confidentialité](/microsoft-365/compliance/sensitivity-labels#what-sensitivity-labels-can-do) dans la documentation de Microsoft 365.

1. Répétez les étapes indiquées ci-dessus pour créer des étiquettes supplémentaires. 

    Pour créer une sous-étiquette, sélectionnez l’étiquette parent > **…**  > **Autres actions** > **Ajouter une sous-étiquette**.

1. Pour modifier des étiquettes existantes, accédez à **Information Protection** > **Étiquettes**, puis sélectionnez votre étiquette. 

    Sélectionnez ensuite **Modifier l’étiquette** pour ouvrir à nouveau l’Assistant **Modifier l’étiquette de confidentialité**, avec tous les paramètres que vous avez définis lors de la création de l’étiquette.

    :::image type="content" source="media/create-sensitivity-label/edit-sensitivity-label-full-small.png" alt-text="Modification d’une étiquette de confidentialité existante" lightbox="media/create-sensitivity-label/edit-sensitivity-label-full.png":::

1. Lorsque vous avez terminé de créer toutes vos étiquettes, veillez à vérifier leur ordre et à les réorganiser en fonction des besoins. 

    Pour modifier l’ordre d’une étiquette, sélectionnez **…** **> Autres actions** > **Déplacer vers le haut** ou **Déplacer vers le bas**. 

    Pour plus d’informations, consultez [Priorité des étiquettes (ordre significatif)](/microsoft-365/compliance/sensitivity-labels#label-priority-order-matters) dans la documentation de Microsoft 365.

> [!IMPORTANT]
> Ne supprimez pas une étiquette à moins d’en connaître l’impact sur vos utilisateurs. 
>
> Pour plus d’informations, consultez [Suppression d’étiquettes](/microsoft-365/compliance/create-sensitivity-labels#removing-and-deleting-labels) dans la documentation de Microsoft 365.

Ensuite, [analysez vos données pour appliquer automatiquement des étiquettes](#scan-your-data-to-apply-labels-automatically), puis passez aux opérations suivantes :

- [Affichage des étiquettes sur les ressources](#view-labels-on-assets)
- [Affichage des rapports d’analyse pour les classifications et les étiquettes de confidentialité](#view-insight-reports-for-the-classifications-and-sensitivity-labels)

#### <a name="define-autolabeling-rules-for-files"></a>Définition de règles d’étiquetage automatique pour les fichiers

Définissez des règles d’étiquetage automatique pour les fichiers dans l’Assistant lorsque vous créez ou modifiez votre étiquette. 

Sur la page **Étiquetage automatique pour les applications Office**, activez **Étiquetage automatique pour les applications Office**, puis définissez les conditions dans lesquelles vous souhaitez que votre étiquette soit appliquée automatiquement à vos données.

Par exemple :

:::image type="content" source="media/create-sensitivity-label/create-auto-labeling-rules-files-small.png" alt-text="Définition de règles d’étiquetage automatique pour les fichiers dans le Centre de sécurité et conformité Microsoft 365" lightbox="media/create-sensitivity-label/create-auto-labeling-rules-files.png":::
 
Pour plus d’informations, consultez [Application automatique d’une étiquette de confidentialité aux données](/microsoft-365/compliance/apply-sensitivity-label-automatically#how-to-configure-auto-labeling-for-office-apps) dans la documentation de Microsoft 365. 

#### <a name="define-autolabeling-rules-for-database-columns"></a>Définition de règles d’étiquetage automatique pour les colonnes de base de données

Définissez des règles d’étiquetage automatique pour les colonnes de base de données dans l’Assistant lorsque vous créez ou modifiez votre étiquette. 

Sous l’option **Ressources Azure Purview (préversion)**  :

1. Sélectionnez le curseur **Étiquetage automatique pour les colonnes de base de données**.

1. Sélectionnez **Vérifier les types d’informations sensibles** pour choisir les types d’informations sensibles à appliquer à votre étiquette.

Par exemple :
        
:::image type="content" source="media/create-sensitivity-label/create-auto-labeling-rules-db-columns-small.png" alt-text="Définition de règles d’étiquetage automatique pour les colonnes SQL dans le Centre de sécurité et conformité Microsoft 365" lightbox="media/create-sensitivity-label/create-auto-labeling-rules-db-columns.png":::

## <a name="scan-your-data-to-apply-labels-automatically"></a>Analyse des données pour appliquer automatiquement des étiquettes

Analysez vos données dans Azure Purview pour appliquer automatiquement les étiquettes créées, en fonction des règles d’étiquetage automatique définies. 

Pour savoir comment configurer des analyses sur différentes ressources dans Azure Purview, consultez :

|Source  |Référence  |
|---------|---------|
|**Stockage Blob Azure**     |[Inscription et analyse du Stockage Blob Azure](register-scan-azure-blob-storage-source.md)         |
|**Azure Data Lake Storage**     |[Inscription et analyse d’Azure Data Lake Storage Gen1](register-scan-adls-gen1.md) </br>[Inscription et analyse d’Azure Data Lake Storage Gen2](register-scan-adls-gen2.md)         |
|**Bases de données SQL Azure**|[Inscrire et analyser une base de données Azure SQL](register-scan-azure-sql-database.md) </br>[Inscrire et analyser une instance Azure SQL Database Managed Instance](register-scan-azure-sql-database-managed-instance.md)|
| | |

## <a name="view-labels-on-assets"></a>Affichage des étiquettes sur les ressources

Une fois que vous avez défini les règles d’étiquetage automatique de vos étiquettes dans Microsoft 365 et analysé vos données dans Azure Purview, les étiquettes sont automatiquement appliquées à vos ressources. 

**Pour afficher les étiquettes appliquées à vos ressources dans le catalogue Azure Purview :**

Dans le catalogue Azure Purview, utilisez les options de filtrage **Étiquette** pour afficher uniquement les fichiers comportant des étiquettes spécifiques. Par exemple : 

:::image type="content" source="media/create-sensitivity-label/filter-search-results-small.png" alt-text="Recherche de ressources par étiquette" lightbox="media/create-sensitivity-label/filter-search-results.png":::

Par exemple :

:::image type="content" source="media/create-sensitivity-label/view-labeled-files-blob-storage-small.png" alt-text="Affichage d’une étiquette de confidentialité sur un fichier dans le Stockage Blob Azure" lightbox="media/create-sensitivity-label/view-labeled-files-blob-storage.png":::

## <a name="view-insight-reports-for-the-classifications-and-sensitivity-labels"></a>Affichage des rapports d’analyse pour les classifications et les étiquettes de confidentialité

Retrouvez des insights sur vos données classées et étiquetées dans Azure Purview à l’aide des rapports **Classification** et **Étiquettes de confidentialité**.

> [!div class="nextstepaction"]
> [Insights relatifs aux classifications](./classification-insights.md)

> [!div class="nextstepaction"]
> [Insights relatifs aux étiquettes de confidentialité](sensitivity-insights.md)


