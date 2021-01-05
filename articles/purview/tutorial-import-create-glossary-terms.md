---
title: 'Tutoriel : Créer et importer des termes de glossaire dans Azure Purview (préversion)'
description: Ce tutoriel explique comment créer des termes de glossaire, ajouter des termes de glossaire à une ressource et importer des termes de glossaire.
author: shsandeep123
ms.author: sandeepshah
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: 0ea6fcaff1ec699431da8b67adee68735a8611a8
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97696078"
---
# <a name="tutorial-create-and-import-glossary-terms-in-azure-purview-preview"></a>Tutoriel : Créer et importer des termes de glossaire dans Azure Purview (préversion)

> [!IMPORTANT]
> Azure Purview est actuellement disponible en PRÉVERSION. L’[Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) contient des conditions légales supplémentaires qui s’appliquent aux fonctionnalités Azure en version bêta, en préversion ou pas encore en disponibilité générale.

Un glossaire est un outil important pour la maintenance et l’organisation de votre catalogue. Vous créez votre glossaire en définissant de nouveaux termes ou en important une liste de termes, puis en appliquant ces termes à vos ressources.

Ce tutoriel est la *partie 5 d’une série de cinq tutoriels* où vous apprenez les principes fondamentaux de la gestion de la gouvernance des données dans un patrimoine de données à l’aide d’Azure Purview. Ce tutoriel s’appuie sur les exercices que vous avez effectués dans [Partie 4 : Explorer les ensembles de ressources, les détails, les schémas et les classifications dans Azure Purview (préversion)](tutorial-schemas-and-classifications.md).

Dans ce tutoriel, vous apprenez à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Créer des termes de glossaire.
> * Ajouter des termes de glossaire à une ressource.
> * Importer des termes de glossaire.

## <a name="prerequisites"></a>Prérequis

* Mener à bien le [didacticiel : Explorer les ensembles de ressources, les détails, les schémas et les classifications dans Azure Purview (préversion)](tutorial-schemas-and-classifications.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-glossary-terms"></a>Créer des termes de glossaire

Vous pouvez créer des termes professionnels ou techniques dans le glossaire. Vous pouvez également annoter vos ressources en leur appliquant des termes.

Voici un résumé de certains des champs les plus courants dans la page **Terme de glossaire** :

* **État** : attribuez un état au terme (**Brouillon**, **Approuvé**, **Expiré** ou **Alerte**).

* **Définition** : entrez une définition du terme.

* **Acronyme** : entrez une version abrégée du terme en utilisant les initiales des mots.

* **Synonymes** : sélectionnez d’autres termes avec des définitions identiques ou similaires.

* **Termes connexes** : dans le glossaire, sélectionnez d’autres termes associés à celui-ci, mais dont les définitions sont différentes. Par exemple, des termes techniques liés à un terme professionnel, un nom de code ou d’autres termes qu’il convient d’associer à ce terme.

Créez un terme de glossaire en suivant les étapes ci-dessous :

1. Accédez à votre ressource Azure Purview dans le portail Azure, puis sélectionnez **Ouvrir Purview Studio**. Vous êtes automatiquement dirigé vers la page d’accueil de Purview Studio.

1. Dans le volet gauche, sélectionnez **Glossaire** pour ouvrir la page **Termes du glossaire**.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/glossary-terms-page.png" alt-text="Capture d’écran montrant la page Termes du glossaire":::

1. Sélectionnez **Nouveau terme** > **Valeur système par défaut** > **Continuer**.

1. Sous l’onglet **Vue d’ensemble** de la page **Nouveau terme**, entrez le **nom** du nouveau terme : *Finances*.

1. Entrez la **définition** : *Ce terme représente des informations financières pour Contoso Inc.*

1. Dans la liste déroulante **État**, sélectionnez **Approuvé**.

1. Quand vous avez terminé, sélectionnez **Créer**.

    :::image type="content" source="./media/tutorial-import-create-glossary-terms/enter-new-glossary-term.png" alt-text="Capture d’écran montrant comment créer un terme de glossaire":::

## <a name="add-glossary-terms-to-an-asset"></a>Ajouter des termes de glossaire à une ressource

1. Utilisez les étapes que vous avez apprises dans la [partie 1 de cette série de tutoriels](tutorial-scan-data.md) pour rechercher une ressource. Par exemple, **Contoso_CashFlow_{N}.csv**.

1. Dans la page des détails de ressource, sélectionnez **Modifier**.

1. Dans la liste déroulante **Termes du glossaire** de l’onglet **Vue d’ensemble**, sélectionnez **Finances** puis **Enregistrer**.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/add-glossary-term-to-asset.png" alt-text="Capture d’écran montrant comment ajouter un terme de glossaire à une ressource":::

1. Accédez à la section **Termes du glossaire** de l’onglet **Vue d’ensemble** et notez que le terme *Finances* a été appliqué à la ressource.

## <a name="import-glossary-terms"></a>Importer des termes de glossaire

Pour importer des termes ou mettre à jour des termes existants en bloc, chargez un modèle prérempli dans votre glossaire.

Dans cette procédure, vous importez des termes de glossaire par le biais d’un fichier .csv :

1. Notez où vous avez stocké le fichier nommé *StarterKitTerms.csv*, qui fait partie du starter kit que vous avez téléchargé dans la [partie 1 de cette série de tutoriels](tutorial-scan-data.md).

   Ce fichier contient une liste de termes préremplis qui s’appliquent à votre patrimoine de données.

1. Pour commencer l’importation, sélectionnez **Glossaire** puis **Importer des termes**.

    :::image type="content" source="./media/tutorial-import-create-glossary-terms/import-glossary-terms-select.png" alt-text="Capture d’écran montrant comment importer des termes de glossaire":::

1. Dans la page **Importer des termes**, sélectionnez **Valeur système par défaut**, puis sélectionnez **Continuer**.

1. Sélectionnez **Parcourir**, accédez à l’emplacement où vous avez téléchargé *StarterKitTerms.csv*, puis sélectionnez **Ouvrir**.

1. Sélectionnez **OK** pour commencer l’importation des termes.

   Une fois l’importation terminée, les nouveaux termes de glossaire s’affichent dans la page **Termes du glossaire**.

1. Affichez chacun des termes nouvellement importés pour voir comment ils sont définis.

## <a name="create-custom-term-templates"></a>Créer des modèles de terme personnalisés

Dans cette section, vous allez apprendre à créer un modèle de terme personnalisé avec des attributs personnalisés et à importer des données à l’aide d’un fichier CSV de modèle.

Il existe plusieurs modèles de terme système existants, que vous pouvez afficher en sélectionnant **Glossaire** > **Gérer les modèles de terme** > **Système**.

:::image type="content" source="./media/tutorial-import-create-glossary-terms/system-term-templates.png" alt-text="Modèles de terme système":::

Pour créer un modèle de terme personnalisé, procédez comme suit :

1. Dans le menu de gauche, sélectionnez **Glossaire**.
1. Sélectionnez **Gérer les modèles de terme** > **Personnalisé** > **Nouveau modèle de terme**

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/create-new-custom-term-template.png" alt-text="Créer un modèle de terme personnalisé":::

Dans l’écran **Nnouveau modèle de terme**, procédez comme suit :

1. Entrez le **nom du modèle** : `Sensitivity level`.
1. Entrez la description de votre choix, telle que `Indicates the level of sensitivity for this data.`
1. Sélectionnez **+ Nouvel attribut** pour ouvrir une boîte de dialogue permettant d’ajouter des attributs.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/new-term-template-screen-start.png" alt-text="Écran initial de nouveau modèle de terme":::

1. Dans l’écran **Nouvel attribut**, entrez les paramètres suivants :

   |Paramètre|Valeur suggérée|
   |---------|-----------|
   |Nom de l’attribut|est sensible aux informations|
   |Type de champ|liste déroulante|Choix unique|
   |Marquer comme requis|cochez cette case.|
   |+ Ajouter un choix| Ajoutez deux choix, « Oui » et « Non ».|

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/add-new-attribute.png" alt-text="Ajouter un nouvel attribut":::

1. Répétez les étapes précédentes pour ajouter un autre attribut portant le nom `can be shared externally`. Une fois les deux attributs ajoutés, sélectionnez **Créer**.

## <a name="import-terms-from-a-template"></a>Importer des termes à partir d’un modèle

Vous allez maintenant importer un nouveau terme à l’aide du modèle **Sensitivity level** que vous avez créé. 

1. Dans l’écran **Termes du glossaire**, sélectionnez **Importer des termes**.

1. Sélectionnez **Sensitivity level** dans l’écran **Importer des termes**. Sélectionnez **Continuer**.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/select-sensitivity-level.png" alt-text="Sélectionner le modèle Sensitivity level":::

1. Le modèle de terme pour **Sensitivity level** contient les attributs système par défaut, ainsi que les nouveaux attributs que vous avez ajoutés : `can be shared externally` et `is sensitive information`. Sélectionnez **Télécharger un exemple de fichier CSV**.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/download-sample-csv-file.png" alt-text="Télécharger un exemple de fichier CSV":::

1. Un fichier de modèle est téléchargé pour que vous puissiez modifier et charger un nouveau terme de glossaire avec des attributs personnalisés. Ouvrez le fichier CSV que vous avez téléchargé. Ajoutez de nouveaux termes et leurs valeurs appropriées en tant que nouvelles lignes dans le fichier CSV.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/create-term-in-csv.png" alt-text="Créer un terme dans le fichier CSV":::

   Les termes listés dans la colonne **Termes associés** ou **Synonymes** qui n’existent pas encore seront créés lorsque le fichier sera chargé. Ils seront créés à l’aide du modèle **Valeur système par défaut**.

1. Pour charger le fichier, revenez à l’écran **Importer des termes** et sélectionnez **Parcourir** en regard de la zone **Sélectionner le fichier CSV complet à charger**. Sélectionnez votre fichier dans la boîte de dialogue qui apparaît, puis sélectionnez **OK**.

1. Les nouveaux termes sont désormais listés dans l’écran **Termes du glossaire**. Vous pouvez afficher des détails sur un nouveau terme en cliquant sur le nom du terme dans la liste.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Créer des termes de glossaire.
> * Ajouter des termes de glossaire à une ressource.
> * Importer des termes de glossaire.

Passez à l’article suivant pour en savoir plus sur les insights du catalogue.

> [!div class="nextstepaction"]
> [Présentation des insights dans Azure Purview](concept-insights.md)
