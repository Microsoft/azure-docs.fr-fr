---
title: Présentation des insights dans Azure Purview
description: Cet article décrit les insights dans Azure Purview.
author: SunetraVirdi
ms.author: suvirdi
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 12/02/2020
ms.openlocfilehash: d841fa336b2702a02f3215f97a6403217986d7e0
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550274"
---
# <a name="understand-insights-in-azure-purview"></a>Présentation des insights dans Azure Purview

Cet article donne une vue d’ensemble de la fonctionnalité Insights dans Azure Purview.

Les insights sont l’un des piliers dans Purview. Cette fonctionnalité offre aux clients une vue unique de leur catalogue et vise à fournir des insights spécifiques aux administrateurs de source de données, aux utilisateurs professionnels, aux gestionnaires de données, aux responsables de données et aux administrateurs de sécurité. Actuellement, Purview contient les rapports d’insights suivants qui seront disponibles pour les clients dans la version préliminaire publique.

## <a name="asset-insights"></a>Insights relatifs aux ressources

Ce rapport donne une vue d’ensemble de votre patrimoine de données et de sa répartition par type de source, par classification et par taille de fichier comme certaines dimensions. Ce rapport s’adresse aux différents types d’utilisateurs qui peuvent gérer le compte Purview et exécuter des analyses, ou à des utilisateurs professionnels susceptibles de connaître le nombre de ressources avec une certaine classification dans le patrimoine de données de leur organisation. 

Le rapport fournit des insights généraux sur les graphiques et les indicateurs de performance clés, ainsi que des informations détaillées sur des anomalies spécifiques telles que les fichiers mal placés. Le rapport prend également en charge une expérience client de bout en bout, dans laquelle le client peut afficher le nombre de ressources avec une classification spécifique, peut décomposer les informations par types de sources et dossiers principaux, et peut également afficher la liste des ressources en vue d’une investigation plus poussée.

## <a name="scan-insights"></a>Insights relatifs aux analyses

Le rapport permet aux administrateurs de comprendre l’intégrité globale des analyses, à savoir le nombre de réussites, le nombre d’échecs et le nombre d’annulations. Ce rapport fournit une mise à jour de l’état des analyses qui ont été exécutées dans le compte Purview au cours des sept derniers jours ou des 30 derniers jours.

Le rapport permet également aux administrateurs d’aller en profondeur et d’explorer les analyses qui ont échoué et sur quels types de sources spécifiques. Pour permettre aux utilisateurs d’effectuer des recherches plus poussées, le rapport permet de naviguer dans la page historique d’analyse au sein de l’expérience « Sources ».

## <a name="glossary-insights"></a>Insights relatifs aux glossaires

Ce rapport permet aux utilisateurs professionnels et aux gestionnaires de données d’obtenir un rapport d’état sur le glossaire. Les utilisateurs peuvent afficher ce rapport pour comprendre la répartition des termes du glossaire par état, savoir combien de termes de glossaire sont attachés aux ressources et combien ne sont pas encore attachés à un élément multimédia. Les utilisateurs professionnels peuvent également en savoir plus sur l’exhaustivité de leurs termes de glossaire. 

Ce rapport résume les principaux éléments sur lesquels un utilisateur de l’entreprise ou un gestionnaire de données doit se concentrer pour créer un glossaire complet et utilisable pour son organisation. Les utilisateurs peuvent également naviguer dans l’expérience « Glossaire » à partir de l’expérience « Insights relatifs aux glossaires » pour apporter des modifications à un terme de glossaire spécifique.

## <a name="classification-insights"></a>Insights relatifs aux classifications

Ce rapport fournit des informations sur l’emplacement des données classifiées, les classifications trouvées lors d’une analyse et une exploration hiérarchique des fichiers classifiés eux-mêmes. Il permet aux administrateurs de la sécurité de comprendre les types d’informations trouvés dans le patrimoine de données de leur organisation. 

Dans Azure Purview, les classifications sont similaires aux étiquettes d’objet et permettent de marquer et identifier le contenu d’un type spécifique dans votre patrimoine de données.

Utilisez le rapport des insights relatifs aux classifications pour identifier le contenu avec des classifications spécifiques et comprendre les actions requises, telles que l’ajout d’une sécurité supplémentaire aux référentiels ou le déplacement de contenu vers un emplacement plus sécurisé.

Pour plus d’informations, consultez [Insights relatifs aux classifications de vos données à partir d’Azure Purview](classification-insights.md).

## <a name="sensitivity-labeling-insights"></a>Insights relatifs à l’étiquetage de confidentialité

Ce rapport fournit des détails sur les étiquettes de confidentialité détectées lors d’une analyse, ainsi qu’une exploration hiérarchique sur les fichiers étiquetés eux-mêmes. Il permet aux administrateurs de la sécurité de garantir la sécurité d’informations trouvés dans le patrimoine de données de leur organisation. 

Dans Azure Purview, les étiquettes de confidentialité sont utilisées pour identifier les catégories de types de classification, ainsi que les stratégies de sécurité de groupe que vous souhaitez appliquer à chaque catégorie.

Utilisez le rapport Insights relatifs à l’étiquetage pour identifier les étiquettes de confidentialité trouvées dans votre contenu et comprendre les actions requises, telles que la gestion de l’accès à des référentiels ou des fichiers spécifiques.

Pour plus d’informations, consultez [Insights relatifs aux étiquettes de confidentialité de vos données dans Azure Purview](sensitivity-insights.md).

## <a name="file-extension-insights"></a>Insights relatifs aux extensions de fichier

Ce rapport fournit des détails sur les extensions de fichiers ou les types de fichiers détectés lors d’une analyse, ainsi qu’une exploration hiérarchique sur les fichiers eux-mêmes. 

Utilisez le rapport Insights relatifs aux extensions de fichier pour comprendre le nombre de fichiers que vous avez à chaque fois, l’emplacement de ces fichiers, et s’ils peuvent être analysés pour les données confidentielles.

Pour plus d’informations, consultez [Insights relatifs aux extensions de fichiers de vos données à partir d’Azure Purview](file-extension-insights.md).

## <a name="next-steps"></a>Étapes suivantes

* [Insights relatifs aux glossaires](glossary-insights.md)
* [Insights relatifs aux analyses](scan-insights.md)
* [Insights relatifs aux classifications](./classification-insights.md)