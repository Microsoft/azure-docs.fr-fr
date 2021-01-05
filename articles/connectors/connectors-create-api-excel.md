---
title: Gérer les données, feuilles de calcul et tables dans Excel Online
description: Gérer les données de feuilles de calcul et de tables dans Excel Online pour les entreprises ou dans Excel Online pour OneDrive à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 08/23/2018
tags: connectors
ms.openlocfilehash: 097db6683127b410e713be53e6de838cf7734ddc
ms.sourcegitcommit: 4313e0d13714559d67d51770b2b9b92e4b0cc629
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2020
ms.locfileid: "91400721"
---
# <a name="manage-excel-online-data-with-azure-logic-apps"></a>Gérer les données Excel Online avec Azure Logic Apps

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le connecteur [Excel Online pour les entreprises](/connectors/excelonlinebusiness/) ou le connecteur [Excel Online pour OneDrive](/connectors/excelonline/), vous pouvez créer des tâches et des flux de travail automatisés basés sur vos données dans Excel Online pour les entreprises ou OneDrive. Ce connecteur fournit des actions qui vous aident à travailler avec vos données et à gérer des feuilles de calcul, par exemple :

* Créer des tables et des feuilles de calcul.
* Obtenir et gérer des feuilles de calcul, des tables et des lignes.
* Ajouter des lignes uniques et des colonnes clés.

Vous pouvez ensuite utiliser les sorties de ces actions avec des actions pour d’autres services. Par exemple, si vous utilisez une action qui crée des feuilles de calcul chaque semaine, vous pouvez utiliser une autre action qui envoie un e-mail de confirmation à l’aide du connecteur Outlook d’Office 365.

Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

> [!NOTE]
> Les connecteurs [Excel Online pour les entreprises](/connectors/excelonlinebusiness/) et [Excel en ligne pour OneDrive](/connectors/excelonline/) fonctionnent avec Azure Logic Apps et diffère du [connecteur Excel pour PowerApps](/connectors/excel/).

## <a name="prerequisites"></a>Conditions préalables requises

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un [compte professionnel ou scolaire](https://www.office.com/) pour votre compte Microsoft professionnel ou personnel

  Vos données Excel peuvent exister dans un fichier de valeurs séparées par des virgules (CSV) dans un dossier de stockage, OneDrive par exemple. 
  Vous pouvez également utiliser le même fichier CSV avec le [connecteur de fichier plat](../logic-apps/logic-apps-enterprise-integration-flatfile.md).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* L’application logique à partir de laquelle vous souhaitez accéder à vos données Excel Online. Ce connecteur ne fournit que des actions ; ainsi, pour démarrer votre application logique, sélectionnez un déclencheur distinct, par exemple, le déclencheur **Périodicité**.

## <a name="add-excel-action"></a>Ajouter une action Excel

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques, si elle n’est pas encore ouverte.

1. Sous le déclencheur, choisissez **Nouvelle étape**.

1. Dans la zone de recherche, entrez « excel » comme filtre. Dans la liste des actions, sélectionnez l’action souhaitée.

   > [!NOTE]
   > Le concepteur d’application logique ne peut pas charger les tables qui ont 100 colonnes ou plus. Si possible, réduisez le nombre de colonnes dans la table sélectionnée, afin que le concepteur puisse charger la table.

1. Si vous y êtes invité, connectez-vous à votre compte professionnel ou scolaire.

   Vos informations d’identification autorisent votre application logique à créer une connexion à Excel Online et à accéder à votre données.

1. Continuez de fournir les informations nécessaires pour l’action sélectionnée et générez le flux de travail de votre application logique.

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d'informations techniques, notamment sur les déclencheurs, les actions et les limites, comme décrit dans le fichier OpenAPI (anciennement Swagger) du connecteur, consultez ses pages de référence du connecteur :

* [Excel Online pour les entreprises](/connectors/excelonlinebusiness/)
* [Excel Online pour OneDrive](/connectors/excelonline/)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
