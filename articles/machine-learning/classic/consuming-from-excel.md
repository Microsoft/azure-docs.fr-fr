---
title: 'ML Studio (classique) : Utiliser le service web dans Excel – Azure'
description: Azure Machine Learning Studio (classique) permet d’appeler facilement des services web directement à partir de Microsoft Excel sans qu’il soit nécessaire d’écrire du code.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 02/01/2018
ms.openlocfilehash: 9ec781a5110dd2e7027ca91fd195bbb796f31020
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322908"
---
# <a name="consuming-an-azure-machine-learning-studio-classic-web-service-from-excel"></a>Utilisation d’un service web Azure Machine Learning Studio (classique) depuis Excel

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Azure Machine Learning Studio (classique) permet d’appeler facilement des services web directement à partir de Microsoft Excel sans qu’il soit nécessaire d’écrire du code.

Si vous utilisez Excel 2013 (ou une version ultérieure) ou Excel Online, nous vous recommandons d’utiliser la [macro complémentaire Excel](excel-add-in-for-web-services.md).



## <a name="steps"></a>Étapes
Publiez un service web. [Tutoriel 3 : Déployer un modèle de risque de crédit](tutorial-part3-credit-risk-deploy.md) explique la procédure à suivre. Actuellement, la fonctionnalité de classeur Excel est uniquement prise en charge pour les services de requête/réponse qui produisent une seule sortie (autrement dit, une étiquette de notation unique). 

Quand vous disposez d’un service web, cliquez sur la section **WEB SERVICES** située sur la partie gauche de Microsoft Azure Machine Learning Studio, puis sélectionnez le service web à utiliser à partir de Microsoft Excel.

**Service web classique**

1. Sur l’onglet **TABLEAU DE BORD** du service web figure une ligne pour le service **REQUÊTE/RÉPONSE**. Si ce service produit une sortie unique, le lien **Télécharger un classeur Excel** doit apparaître sur cette ligne.

    ![Télécharger le classeur Excel à l’aide du portail du service web Studio (classique)](./media/consuming-from-excel/excellink.png)
2. Cliquez sur **Télécharger un classeur Excel**.

**Nouveau service web**

1. Dans le portail Service web Azure Machine Learning, sélectionnez **Consommer**.
2. Dans la page Consommer, dans les **options de consommation de service Web** cliquez sur l’icône Excel.

**Utilisation du classeur**

1. Ouvrez le classeur.
2. Un avertissement de sécurité s’affiche. Cliquez sur le bouton **Activer la modification**.

    ![Activer la modification pour supprimer l’avertissement de sécurité du mode protégé](./media/consuming-from-excel/enableeditting.png)
3. Un avertissement de sécurité apparaît. Cliquez sur le bouton **Activer le contenu** pour pouvoir exécuter des macros sur votre feuille de calcul.

    ![Activer le contenu pour ignorer l’avertissement de sécurité de la désactivation des macros](./media/consuming-from-excel/enablecontent.png)
4. Une fois les macros activées, une table est générée. Les valeurs des colonnes en bleu sont requises en tant qu’entrées dans le service web RRS (Request/Response Service), ou en tant que **PARAMÈTRES**. Notez la sortie du service RRS, appelée **VALEURS PRÉDITES** et affichée en vert. Lorsque toutes les colonnes d’une ligne donnée sont remplies, le classeur appelle automatiquement l’API de notation et affiche les notes résultantes.

    ![Tableau des entrées de paramètre et des valeurs prédites résultantes](./media/consuming-from-excel/sampletable.png)
5. Pour noter plusieurs lignes, remplissez la deuxième ligne avec les données ; des valeurs prédites sont produites. Vous pouvez même coller plusieurs lignes à la fois.

Vous pouvez utiliser toutes les fonctionnalités de Microsoft Excel (graphiques, Power Map, mise en forme conditionnelle, etc.) avec les valeurs prédites pour visualiser les données.

## <a name="sharing-your-workbook"></a>Partage de votre classeur
Pour que les macros fonctionnent, votre Clé API doit faire partie de la feuille de calcul. Cela signifie que vous devez uniquement partager le classeur avec des entités/personnes de confiance.

## <a name="automatic-updates"></a>Mises à jour automatiques
Un appel RRS est initié dans les deux cas suivants :

1. la première fois que du contenu apparaît dans l’ensemble des **PARAMÈTRES**
2. chaque fois que l’un ou l’autre des **PARAMÈTRES** change dans une ligne dont l’ensemble des **PARAMÈTRES** a été entré.