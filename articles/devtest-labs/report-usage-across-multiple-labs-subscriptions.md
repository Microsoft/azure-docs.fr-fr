---
title: Utilisation de Azure DevTest Labs dans plusieurs laboratoires et abonnements
description: Découvrez comment rendre compte de l’utilisation d’Azure DevTest Labs dans plusieurs labos et abonnements.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 9640dc3fd74095a737f382994201dc5610af921a
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92152204"
---
# <a name="report-azure-devtest-labs-usage-across-multiple-labs-and-subscriptions"></a>Rendre compte de l’utilisation d’Azure DevTest Labs dans plusieurs labos et abonnements

La plupart des grandes organisations veulent suivre l’usage des ressources afin d’exploiter celles-i plus efficacement en visualisant les tendances et les anomalies d’utilisation. Selon l’utilisation des ressources, les propriétaires ou administrateurs de labos peuvent personnaliser ceux-ci pour [améliorer l’utilisation et les coûts des ressources](../cost-management-billing/cost-management-billing-overview.md). Avec Azure DevTest Labs, vous pouvez télécharger l’utilisation des ressources par labo qui vous permet d’avoir une vision historique plus approfondie des modèles d’utilisation. Ces modèles d’utilisation peuvent vous aider à épingler des modifications pour améliorer l’efficacité. La plupart des organisations veulent pouvoir choisir entre une utilisation en labo individuel et une utilisation globale dans [plusieurs labos et abonnements](/azure/architecture/cloud-adoption/decision-guides/subscriptions/). 

Cet article explique comment gérer les informations sur l’utilisation des ressources de plusieurs labos et abonnements.

![Utilisation des rapports](./media/report-usage-across-multiple-labs-subscriptions/report-usage.png)

## <a name="individual-lab-usage"></a>Utilisation en labo individuel

Cette section explique comment exporter l’utilisation des ressources pour un labo.

Avant de pouvoir exporter l’utilisation des ressources de DevTest Labs, vous devez configurer un compte Stockage Azure pour stocker les différents fichiers contenant les données d’utilisation. Les deux méthodes couramment utilisées pour exporter des données sont les suivantes :

* [API REST DevTest Labs](/rest/api/dtl/labs/exportresourceusage) 
* Le module PowerShell AZ.Resource [Invoke-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction?view=azps-2.5.0&viewFallbackFrom=azps-2.3.2) avec l’action de `exportResourceUsage`, l’ID de ressource de labo et les paramètres nécessaires. 

    L’article [Exporter ou supprimer des données personnelles](personal-data-delete-export.md) contient un exemple de script PowerShell avec des informations détaillées sur les données exportées. 

    > [!NOTE]
    > Le paramètre date n’incluant pas d’horodatage, les données incluent toutes les données collectées depuis minuit dans le fuseau horaire où se trouve le labo.

Une fois l’exportation terminée, le stockage blob comprend des fichiers CSV contenant les différentes informations sur les ressources.
  
Ces fichiers CSV sont actuellement au nombre de deux :

* *virtualmachines.csv* : contient des informations sur les machines virtuelles du labo
* *disks.csv* : contient des informations sur les différents disques utilisés dans le cadre du labo 

Ces fichiers sont stockés dans le conteneur d’objets blob *labresourceusage* sous le nom du labo, l’ID unique du labo, la date d’exécution et soit la date de début complète, soit la date de début définie dans la demande d’exportation. Voici un exemple de structure blob :

* `labresourceusage/labname/1111aaaa-bbbb-cccc-dddd-2222eeee/<End>DD26-MM6-2019YYYY/full/virtualmachines.csv`
* `labresourceusage/labname/1111aaaa-bbbb-cccc-dddd-2222eeee/<End>DD-MM-YYYY/26-6-2019/20-6-2019<Start>DD-MM-YYYY/virtualmachines.csv`

## <a name="exporting-usage-for-all-labs"></a>Exportation de l’utilisation pour tous les labos

Pour exporter les informations d’utilisation de plusieurs labos, vous pouvez utiliser : 

* [Azure Functions](../azure-functions/index.yml), disponible dans de nombreux langages, dont PowerShell, ou 
* [Runbook Azure Automation](../automation/index.yml), PowerShell, Python ou un concepteur graphique personnalisé pour écrire le code d’exportation.

Ces technologies vous permettent d’exécuter les exportations de labo individuelles de tous les labos à une date et une heure spécifiques. 

Votre fonction Azure doit envoyer les données au stockage à long terme. L’exportation de données pour plusieurs labos peut prendre un certain temps. Pour améliorer les performances et réduire la probabilité d’une duplication d’informations, nous vous recommandons d’exécuter les labos en parallèle. Pour obtenir ce parallélisme, exécutez Azure Functions de façon asynchrone. Tirez également parti du déclencheur de minuteur intégré dans Azure Functions.

## <a name="using-a-long-term-storage"></a>Utilisation d’un stockage à long terme

Un stockage à long terme consolide les informations d’exportation de différents labos dans une source de données unique. Un autre avantage de l’utilisation d’un stockage à long terme est la possibilité d’y supprimer des fichiers afin de réduire la duplication et les coûts. 

Vous pouvez utiliser un stockage à long terme pour effectuer toute manipulation de texte, par exemple : 

* ajout de noms conviviaux ;
* création de regroupements complexes ;
* agrégation des données.

Voici des solutions de stockage courantes : [SQL Server](https://azure.microsoft.com/services/sql-database/), [Azure Data Lake](https://azure.microsoft.com/services/storage/data-lake-storage/) et [CosmosDB](https://azure.microsoft.com/services/cosmos-db/). Le choix de la solution de stockage à long terme dépend de vos préférences. Vous pouvez envisager de choisir l’outil en fonction de ce qu’il offre en termes de possibilité d’interaction lors de la visualisation des données.

## <a name="visualizing-data-and-gathering-insights"></a>Visualisation des données et collecte d’informations

Utilisez l’outil de visualisation de données de votre choix pour vous connecter à votre stockage à long terme afin d’afficher les données d’utilisation et de collecter des informations pour vérifier l’efficacité d’utilisation. Par exemple, [Power BI](/power-bi/power-bi-overview) permet d’organiser et d’afficher les données d’utilisation. 

[Azure Data Factory](https://azure.microsoft.com/services/data-factory/) permet de créer, de lier et de gérer des ressources via une interface unique. Si un contrôle accru est nécessaire, vous pouvez créer la ressource individuelle dans un groupe de ressources unique et la gérer indépendamment du service Data Factory.  

## <a name="next-steps"></a>Étapes suivantes

Une fois le système configuré et les données déplacées vers le stockage à long terme, l’étape suivante consiste à trouver les questions auxquelles les données doivent répondre. Par exemple : 

-   Quelle est la taille de machine virtuelle utilisée ?

    Les utilisateurs sélectionnent-ils des tailles de machines virtuelles hautes performances (plus coûteuses) ?
-   Quelles sont les images de la Place de marché utilisées ?

    Les images personnalisées constituent-elles la base la plus commune des machines virtuelles si un magasin d’images commun doit être construit comme une [Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md) ou une [Fabrique d’images](image-factory-create.md).
-   Quelles images personnalisées sont utilisées ou non utilisées ?