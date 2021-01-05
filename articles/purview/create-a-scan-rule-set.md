---
title: Créer un ensemble de règles d’analyse
description: Créez un ensemble de règles d’analyse dans Azure Purview pour analyser rapidement les sources de données au sein de votre organisation.
author: chandrakavya
ms.author: kchandra
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 12/02/2020
ms.openlocfilehash: 9662652a6a40285ad382857975ec0dd04b8ba8be
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550271"
---
# <a name="create-a-scan-rule-set"></a>Créer un ensemble de règles d’analyse

Dans un catalogue Azure Purview, vous pouvez créer des ensembles de règles d’analyse pour vous permettre d’analyser rapidement les sources de données au sein de votre organisation.

Un ensemble de règles d’analyse est un conteneur permettant de regrouper un ensemble de règles d’analyse afin de pouvoir les associer facilement à une analyse. Par exemple, vous pouvez créer un ensemble de règles d’analyse par défaut pour chacun de vos types de source de données, puis utiliser ces ensembles de règles d’analyse par défaut pour toutes les analyses au sein de votre entreprise. Vous pouvez également accorder aux utilisateurs les autorisations appropriées pour créer d’autres ensembles de règles d’analyse avec différentes configurations en fonction des besoins métiers.

## <a name="steps-to-create-a-scan-rule-set"></a>Étapes pour créer un ensemble de règles d’analyse

Pour créer un ensemble de règles d’analyse :

1. Dans votre catalogue Azure Purview, sélectionnez **Centre de gestion**.

1. Dans le volet gauche, sélectionnez **Analyser les ensembles de règles**, puis **Nouveau**.

1. Dans la page **Nouvel ensemble de règles d’analyse**, dans la liste déroulante **Type de source**, sélectionnez les sources de données que le scanneur de catalogue prend en charge. Vous pouvez créer un ensemble de règles d’analyse pour chaque type de source de données que vous souhaitez analyser.

1. Attribuez un **Nom** à votre ensemble de règles d’analyse. La longueur maximale du nom est de 63 caractères, sans espace. Entrez éventuellement une **Description**. La longueur maximale est de 256 caractères.

   :::image type="content" source="./media/create-a-scan-rule-set/purview-home-page.png" alt-text="Capture d’écran montrant la page Ensemble de règles d’analyse." border="true":::

1. Sélectionnez **Continuer**.

   La page **Sélectionner les types de fichiers** s’affiche. Notez que les options de type de fichier de cette page varient en fonction du type de source de données que vous avez choisi dans la page précédente. Tous les types de fichiers sont activés par défaut.

      :::image type="content" source="./media/create-a-scan-rule-set/select-file-types-page.png" alt-text="Capture d’écran montrant la page Sélectionner les types de fichiers.":::

   La sélection **Types de fichiers de document** sur cette page vous permet d’inclure ou d’exclure les types de fichiers Office suivants : .doc, .docm, .docx, .dot, .ODP, .ods, .odt, .pdf, .pot, .PPS, .ppsx, .ppt, .pptm, .pptx, .xlc, .xls, .xlsb, .xlsm, .xlsx et . xlt.

1. Activez ou désactivez une vignette de type de fichier en activant ou désactivant sa case à cocher. Si vous choisissez une source de données de type Data Lake (par exemple, Azure Data Lake Storage Gen2 ou Blob Azure), activez les types de fichiers pour lesquels vous souhaitez que le schéma soit extrait et classifié.

1. Pour certains types de sources de données, vous pouvez également [Créer un type de fichier personnalisé](#create-a-custom-file-type).

1. Sélectionnez **Continuer**.

   La page **Sélectionner les règles de classification** s’affiche. Cette page affiche les **Règles système** et les **Règles personnalisées** sélectionnées, ainsi que le nombre total de règles de classification sélectionnées. Par défaut, toutes les cases **Règles système** sont activées

1. Pour les règles que vous souhaitez inclure ou exclure, vous pouvez activer ou désactiver les cases à cocher de règle de classification **Règles systèmes** de manière globale par catégorie.

   :::image type="content" source="./media/create-a-scan-rule-set/select-classification-rules.png" alt-text="Capture d’écran montrant la page Sélectionner les règles de classification.":::

1. Vous pouvez développer le nœud de catégorie et activer ou désactiver des cases à cocher individuelles. Par exemple, si la règle pour **Argentina.DNI Number** a un nombre élevé de faux positifs, vous pouvez désactiver cette case à cocher spécifique.

   :::image type="content" source="./media/create-a-scan-rule-set/select-system-rules.png" alt-text="Capture d’écran montrant comment sélectionner des règles système":::.

1. Sélectionnez **Créer** pour terminer l’ensemble de règles d’analyse.

### <a name="create-a-custom-file-type"></a>Créer un type de fichier personnalisé

Azure Purview prend en charge l’ajout d’une extension personnalisée et la définition d’un délimiteur de colonne personnalisé dans un ensemble de règles d’analyse.

Pour créer un type de fichier personnalisé :

1. Suivez les étapes 1 à 5 de la section [Étapes pour créer un ensemble de règles d’analyse](#steps-to-create-a-scan-rule-set) ou modifiez un ensemble de règles d’analyse existant.

1. Dans la page **Sélectionner les types de fichiers**, sélectionnez **Nouveau type de fichier** pour créer un nouveau type de fichier personnalisé.

   :::image type="content" source="./media/create-a-scan-rule-set/select-new-file-type.png" alt-text="Capture d’écran montrant comment sélectionner un nouveau type de fichier dans la page Sélectionner les types de fichiers.":::

1. Entrez une **Extension de fichier** et une **Description** facultative.

   :::image type="content" source="./media/create-a-scan-rule-set/new-custom-file-type-page.png" alt-text="Capture d’écran montrant la page Nouveau type de fichier personnalisé." border="true":::

1. Opérez l’une des sélections suivantes pour **Contenu de fichier dans** afin de spécifier le type de contenu de votre fichier :

   - Sélectionnez **Délimiteur personnalisé**, puis entrez votre propre **Délimiteur personnalisé** (un seul caractère).

   - Sélectionnez **Type de fichier système**, puis choisissez un type de fichier système (par exemple XML) dans la liste déroulante **Type de fichier système**.

1. Sélectionnez **Créer** pour enregistrer le fichier personnalisé.

   Le système revient à la page **Sélectionner les types de fichiers**, puis insère le nouveau type de fichier personnalisé en tant que nouvelle vignette.

   :::image type="content" source="./media/create-a-scan-rule-set/new-custom-file-type-tile.png" alt-text="Capture d’écran montrant la vignette de nouveau type de fichier personnalisé dans la page Sélectionner les types de fichiers.":::

1. Sélectionnez **Modifier** dans la vignette de nouveau type de fichier si vous souhaitez la modifier ou la supprimer.

1. Sélectionnez **Continuer** pour terminer la configuration de l’ensemble de règles d’analyse.

## <a name="system-scan-rule-sets"></a>Ensembles de règles d’analyse système

Les ensembles de règles d’analyse système sont des ensembles de règles d’analyse définis par Microsoft qui sont automatiquement créés pour chaque catalogue Azure Purview. Chaque ensemble de règles d’analyse système est associé à un type de source de données spécifique. Lorsque vous créez une analyse, vous pouvez l’associer à un ensemble de règles d’analyse système. Chaque fois que Microsoft effectue une mise à jour de ces ensembles de règles système, vous pouvez les mettre à jour dans votre catalogue et appliquer la mise à jour à toutes les analyses associées.

1. Pour afficher la liste des ensembles de règles d’analyse système, dans le **Centre de gestion**, sélectionnez **Ensembles de règles d’analyse**, puis choisissez l’onglet **Système**.

   :::image type="content" source="./media/create-a-scan-rule-set/system-scan-rule-sets.jpg" alt-text="Capture d’écran montrant la liste des ensembles de règles d’analyse système." lightbox="./media/create-a-scan-rule-set/system-scan-rule-sets.jpg":::

1. Chaque ensemble de règles d’analyse système a un **Nom**, un **Type de source** et une **Version**. Si vous sélectionnez le numéro de version d’un ensemble de règles d’analyse dans la colonne **Version**, vous voyez les règles associées à la version actuelle et aux versions précédentes (le cas échéant).

   :::image type="content" source="./media/create-a-scan-rule-set/system-scan-rule-set-page.jpg" alt-text="Capture d’écran montrant une page d’ensemble de règles d’analyse système." lightbox="./media/create-a-scan-rule-set/system-scan-rule-set-page.jpg":::

1. Si une mise à jour est disponible pour un ensemble de règles d’analyse système, dans la colonne **Version**, vous pouvez sélectionner **Mise à jour**. Dans la page de règle d’analyse du système, dans la liste déroulante **Sélectionner une nouvelle version à mettre à jour**, choisissez une version. La page affiche une liste des règles de classification système associées à la nouvelle version et à la version actuelle.

   :::image type="content" source="./media/create-a-scan-rule-set/system-scan-rule-set-version.jpg" alt-text="Capture d’écran montrant comment modifier la version d’un ensemble de règles d’analyse système." lightbox="./media/create-a-scan-rule-set/system-scan-rule-set-version.jpg":::

### <a name="associate-a-scan-with-a-system-scan-rule-set"></a>Associer une analyse à un ensemble de règles d’analyse système

Lorsque vous [créez une analyse](tutorial-scan-data.md#scan-data-into-the-catalog), vous pouvez choisir de l’associer à un ensemble de règles d’analyse système comme suit :

1. Dans la page **Sélectionner un ensemble de règles d’analyse**, sélectionnez l’ensemble de règles d’analyse système.

   :::image type="content" source="./media/create-a-scan-rule-set/set-system-scan-rule-set.jpg" alt-text="Capture d’écran montrant comment sélectionner un ensemble de règles d’analyse système pour une analyse." lightbox="./media/create-a-scan-rule-set/set-system-scan-rule-set.jpg":::

1. Sélectionnez **Continuer**, puis **Enregistrer et exécuter**.
