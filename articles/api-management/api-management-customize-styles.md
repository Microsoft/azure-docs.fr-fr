---
title: Personnaliser le style des pages du portail des développeurs hérité Gestion des API
titleSuffix: Azure API Management
description: Suivez les étapes de ce démarrage rapide pour personnaliser le style des éléments du portail des développeurs Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.custom: mvc
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 0cdad245981ad02d3cdaefba447d131c775e242f
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93145767"
---
# <a name="customize-the-style-of-the-developer-portal-pages"></a>Personnaliser le style des pages du portail des développeurs

Les trois façons les plus courantes de personnaliser le portail des développeurs dans la Gestion des API Azure sont les suivantes :
 
* [Modifier le contenu des pages statiques et des éléments de mise en page](api-management-modify-content-layout.md)
* Mettre à jour les styles utilisés pour les éléments de page dans le portail des développeurs (procédure expliquée dans ce guide)
* [Modifier les modèles utilisés pour les pages générées par le portail](api-management-developer-portal-templates.md) (par exemple, documents API, produits, authentification des utilisateurs.)

Dans cet article, vous allez apprendre à personnaliser le style des éléments des pages du portail des **développeurs** hérité et à afficher vos modifications.

![Capture d’écran indiquant l’endroit où modifier vos paramètres dans le portail des développeurs hérité.](./media/modify-developer-portal-style/developer_portal.png)

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="prerequisites"></a>Conditions préalables requises

+ Apprenez la [terminologie relative à Gestion des API Azure](api-management-terminology.md).
+ Suivez le guide de démarrage rapide suivant : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).
+ Suivez également le didacticiel suivant : [Importer et publier votre première API](import-and-publish.md).

## <a name="customize-the-developer-portal"></a>Personnaliser le portail des développeurs

1. Sélectionnez **Vue d’ensemble**.
2. Cliquez sur le bouton **Portail des développeurs (hérité)** en haut de la fenêtre **Vue d’ensemble**.
3. Sur le côté supérieur gauche de l’écran, une icône composée de deux pinceaux apparaît. Placez le curseur de la souris au-dessus de cette icône pour ouvrir le menu de personnalisation du portail.

    ![Capture d’écran mettant en évidence l’icône avec deux pinceaux.](./media/modify-developer-portal-style/modify-developer-portal-style01.png)
4. Sélectionnez **Styles** dans le menu pour ouvrir le volet de personnalisation du style.

    Tous les éléments que vous pouvez personnaliser à l’aide de **Styles** apparaissent dans la page.
5. Entrez « headings-color » dans le champ **Changez les valeurs des variables pour personnaliser l’apparence du portail des développeurs**.

    L’élément **\@headings-color** apparaît dans la page. Cette variable contrôle la couleur du texte.

    ![Personnaliser le style](./media/modify-developer-portal-style/modify-developer-portal-style02.png)
    
6. Cliquez sur le champ associé à la variable **\@headings-color**. 
    
    Une liste déroulante de sélecteur de couleurs s’ouvre.
7. Dans cette liste déroulante, sélectionnez une nouvelle couleur.

    > [!TIP]
    > Un aperçu en temps réel est disponible pour toutes les modifications. Un indicateur de progression s’affiche en haut du volet de personnalisation. Après quelques secondes, la couleur nouvellement sélectionnée est appliquée au texte d’en-tête.

8. Sélectionnez **Publier** dans la partie inférieure gauche du menu du volet de personnalisation.
9. Sélectionnez **Publier les personnalisations** pour publier les modifications.

## <a name="view-your-change"></a>Afficher votre modification

1. Accédez au portail des développeurs.
2. Vous pouvez voir la modification que vous avez apportée.

## <a name="next-steps"></a>Étapes suivantes

Nous vous recommandons également d’apprendre à [personnaliser le portail des développeurs Gestion des API Azure à l’aide de modèles](api-management-developer-portal-templates.md).