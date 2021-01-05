---
title: Tutoriel – Accéder au portail des développeurs et le personnaliser – Gestion des API Azure | Microsoft Docs
description: Suivez ce tutoriel pour savoir comment personnaliser le portail des développeurs Gestion des API, un site web généré automatiquement et entièrement personnalisable à l’aide de la documentation de vos API.
services: api-management
author: mikebudzynski
ms.service: api-management
ms.topic: tutorial
ms.date: 11/16/2020
ms.author: apimpm
ms.openlocfilehash: 90544fbafe7393630c3f3fbc694ae367eccb7f90
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012973"
---
# <a name="tutorial-access-and-customize-the-developer-portal"></a>Tutoriel : Accéder et personnaliser le portail des développeurs

Le *portail des développeurs* est un site web généré automatiquement et entièrement personnalisable avec la documentation de vos API. C’est là que les consommateurs d’API peuvent découvrir vos API, apprendre à les utiliser et y demander l’accès.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Accéder à la version managée du portail des développeurs
> * Parcourir son interface d’administration
> * Personnaliser le contenu
> * Publier les modifications
> * Afficher le portail publié

Pour plus d’informations sur le portail des développeurs, voir la [Vue d’ensemble du portail des développeurs Gestion des API Azure](api-management-howto-developer-portal.md).

:::image type="content" source="media/api-management-howto-developer-portal-customize/cover.png" alt-text="Portail des développeurs Gestion des API – Mode Administrateur" border="false":::

## <a name="prerequisites"></a>Prérequis

- Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md)
- Importez et publiez une instance Gestion des API Azure. Pour plus d’informations, voir [Importer et publier](import-and-publish.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="access-the-portal-as-an-administrator"></a>Accéder au portail en tant qu’administrateur

Suivez les étapes ci-dessous pour accéder à la version managée du portail.

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre instance APIM.
1. Sélectionnez le bouton **Portail des développeurs** dans la barre de navigation supérieure. Un nouvel onglet de navigateur s’ouvre, en affichant une version d’administration du portail.

## <a name="understand-the-portals-administrative-interface"></a>Comprendre l’interface d’administration du portail

### <a name="default-content"></a>Contenu par défaut 

Si vous accédez au portail pour la première fois, le contenu par défaut est automatiquement provisionné en arrière-plan. Il a été conçu pour présenter les fonctionnalités du portail et réduire les personnalisations nécessaires à la particularisation de votre portail. Pour plus d’informations sur le contenu du portail, voir la [Vue d’ensemble du portail des développeurs Gestion des API Azure](api-management-howto-developer-portal.md).

### <a name="visual-editor"></a>Éditeur visuel

Il est possible de personnaliser le contenu du portail avec l’éditeur visuel. 
* Les sections de menu de gauche permettent de créer ou de modifier des pages, des médias, des dispositions, des menus, des styles ou des paramètres de site web. 
* Les éléments de menu du bas permettent de basculer entre les fenêtres d’affichage (par exemple, mobile ou ordinateur de bureau), d’afficher les éléments du portail visibles par les utilisateurs authentifiés et anonymes, et d’enregistrer ou d’annuler des actions.
* Ajoutez des lignes à une page en cliquant sur une icône bleue dotée d’un signe plus. 
* Pour ajouter des widgets (par exemple, du texte, des images ou une liste d’API), appuyez sur une icône grise accompagnée d’un signe plus.
* Réorganisez les éléments d’une page avec l’interaction de type glisser-déplacer. 

### <a name="layouts-and-pages"></a>Dispositions et pages

:::image type="content" source="media/api-management-howto-developer-portal-customize/pages-layouts.png" alt-text="Pages et dispositions" border="false":::

Les dispositions définissent la façon dont s’affichent les pages. Par exemple, dans le contenu par défaut, il existe deux dispositions : l’une s’applique à la page d’accueil et l’autre à toutes les autres pages.

Pour appliquer une disposition à une page, faites correspondre son modèle d’URL à son URL. Par exemple, une disposition associée au modèle d’URL `/wiki/*` s’appliquera à chacune des pages dont l’URL comporte le segment `/wiki/` : `/wiki/getting-started`, `/wiki/styles`, etc.

Dans l’image précédente, le contenu appartenant à la disposition est marqué en bleu, tandis que la page est marquée en rouge. Les sections de menu sont marquées de la même manière.

### <a name="styling-guide"></a>Guide de style

:::image type="content" source="media/api-management-howto-developer-portal-customize/styling-guide.png" alt-text="Guide de style" border="false":::

Le guide de style est un panneau destiné aux concepteurs. Il permet de contrôler et de styliser tous les éléments visuels du portail. Le style est hiérarchique : de nombreux éléments héritent des propriétés d’autres éléments. Par exemple, les éléments de type bouton utilisent des couleurs du texte et de l’arrière-plan. Pour modifier la couleur d’un bouton, il faut modifier la variante de couleur d’origine.

Pour modifier une variante, sélectionnez-la, puis sélectionnez l’icône de crayon qui apparaît au-dessus. Après avoir effectué les modifications dans la fenêtre contextuelle, fermez-la.

### <a name="save-button"></a>Bouton Enregistrer

:::image type="content" source="media/api-management-howto-developer-portal-customize/save-button.png" alt-text="Bouton Enregistrer" border="false":::

Chaque fois que vous apportez une modification au portail, vous devez l’enregistrer manuellement en sélectionnant le bouton **Enregistrer** dans le menu du bas, ou en appuyant sur [Ctrl]+[S]. Le contenu modifié est alors automatiquement chargé dans votre service Gestion des API.

## <a name="customize-the-portals-content"></a>Personnaliser le contenu du portail

Avant de mettre le portail à la disposition des visiteurs, il est préférable de personnaliser le contenu généré automatiquement. Sont recommandées les modifications suivantes : dispositions, styles et contenu de la page d’accueil.

> [!NOTE]
> Pour des raisons d’intégration, les pages suivantes ne sont ni supprimables ni déplaçables sous une autre URL : `/404`, `/500`, `/captcha`, `/change-password`, `/config.json`, `/confirm/invitation`, `/confirm-v2/identities/basic/signup`, `/confirm-v2/password`, `/internal-status-0123456789abcdef`, `/publish`, `/signin`, `/signin-sso` et `/signup`.

### <a name="home-page"></a>page d'accueil

La page d’**Accueil** par défaut comporte du contenu d’espace réservé. Vous pouvez soit supprimer des sections entières comportant ce contenu, soit conserver la structure et ajuster les éléments un par un. Remplacez le texte et les images générés par les vôtres en veillant à ce que les liens pointent vers les emplacements souhaités.

### <a name="layouts"></a>Mises en forme

Remplacez le logo généré automatiquement dans la barre de navigation par votre propre image.

### <a name="styling"></a>Style

Même si vous n’avez pas besoin de modifier les styles, vous pouvez éventuellement ajuster des éléments spécifiques. Par exemple, choisissez la couleur principale en fonction de la couleur de votre marque.

### <a name="customization-example"></a>Exemple de personnalisation

Dans la vidéo suivante, nous expliquons comment modifier le contenu du portail, personnaliser l’aspect du site web et publier les modifications.

> [!VIDEO https://www.youtube.com/embed/5mMtUSmfUlw]

## <a name="publish-the-portal"></a><a name="publish"></a>Publier le portail

Pour que votre portail et ses dernières modifications soient accessibles aux visiteurs, vous devez le *publier*. Vous pouvez publier le portail dans l’interface d’administration du portail ou à partir du portail Azure.

### <a name="publish-from-the-administrative-interface"></a>Publier à partir de l’interface d’administration

1. Veillez à enregistrer vos modifications en sélectionnant l’icône **Enregistrer**.
1. Dans la section **Opérations** du menu, sélectionnez **Publier le site web**. Cette opération peut prendre quelques minutes.  

    :::image type="content" source="media/api-management-howto-developer-portal-customize/publish-portal.png" alt-text="Publier le portail" border="false":::

### <a name="publish-from-the-azure-portal"></a>Publier à partir du portail Azure

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre instance APIM.
1. Dans le menu de gauche, sous **Portail des développeurs**, sélectionnez **Vue d’ensemble du portail**.
1. Dans la fenêtre **Vue d’ensemble du portail**, sélectionnez **Publier**.

    :::image type="content" source="media/api-management-howto-developer-portal-customize/pubish-portal-azure-portal.png" alt-text="Publier le portail depuis le portail Azure":::

> [!NOTE]
> Le portail doit être publié de nouveau après des modifications de configuration du service Gestion des API. Par exemple, vous publiez de nouveau le portail après l’attribution d’un domaine personnalisé, la mise à jour des fournisseurs d’identité, la définition de la délégation ou la spécification de la connexion et des conditions d’utilisation du produit.


## <a name="visit-the-published-portal"></a>Consulter le portail publié

Une fois publié, le portail est accessible à la même URL que le panneau d’administration, par exemple `https://contoso-api.developer.azure-api.net`. Affichez-le dans une session de navigateur distincte (par le biais du mode de navigation incognito ou privée) en tant que visiteur externe.

## <a name="apply-the-cors-policy-on-apis"></a>Appliquer la stratégie CORS sur les API

Pour permettre aux visiteurs de votre portail de tester les API via la console interactive intégrée, activez CORS (cross-origin resource sharing) sur vos API. Pour plus d’informations, consultez [Vue d’ensemble du portail des développeurs Gestion des API Azure](api-management-howto-developer-portal.md#cors).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le portail des développeurs :

- [Vue d’ensemble du portail des développeurs Gestion des API Azure](api-management-howto-developer-portal.md)
- [Migrer vers le nouveau portail des développeurs](developer-portal-deprecated-migration.md) à partir du portail hérité déprécié.