---
title: Personnaliser le portail des développeurs Gestion des API à l’aide de modèles
titleSuffix: Azure API Management
description: Apprenez à personnaliser le portail des développeurs Gestion des API Azure à l’aide de modèles.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: a195675b-f7d0-4fc9-90bf-860e6f17ccf7
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 16c0b3e6dc4bd9097312e3a8c43618e66781ef85
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92092883"
---
# <a name="how-to-customize-the-azure-api-management-developer-portal-using-templates"></a>Comment personnaliser le portail des développeurs Gestion des API Azure à l’aide de modèles

Il existe trois manières principales pour personnaliser le portail des développeurs dans Gestion des API Azure :

* [Modifier le contenu des pages statiques et des éléments de mise en page][modify-content-layout]
* [Mettre à jour les styles utilisés pour les éléments de page dans le portail des développeurs][customize-styles]
* [Modifier les modèles utilisés pour les pages générées par le portail][portal-templates] (expliqué dans ce guide)

Les modèles permettent de personnaliser le contenu des pages du portail des développeurs générées par le système (par exemple, documents sur les API, produits, authentification utilisateur, etc.). En utilisant la syntaxe [DotLiquid](http://dotliquidmarkup.org/) et un ensemble de ressources de chaîne localisées, d’icônes et de contrôles de page, vous disposez d’un large choix pour configurer le contenu des pages selon vos besoins.

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="developer-portal-templates-overview"></a>Vue d’ensemble des modèles du portail des développeurs

La modification des modèles s’effectue à partir du **portail des développeurs** lorsque vous êtes connecté en tant qu’administrateur. Pour y accéder, ouvrez tout d’abord le Portail Azure et cliquez sur **Portail des développeurs** dans la barre d’outils de services de votre instance Gestion des API.

Pour accéder aux modèles du portail des développeurs, cliquez sur l’icône de personnalisation à gauche pour afficher le menu de personnalisation, puis cliquez sur **Templates** (Modèles).

![Capture d’écran qui met en surbrillance l’icône Personnaliser pour afficher le menu de personnalisation.][api-management-customize-menu]

La liste des modèles affiche plusieurs catégories de modèles applicables aux différentes pages du portail des développeurs. Chaque modèle est différent, mais les étapes pour les modifier et publier les modifications sont identiques. Pour modifier un modèle, cliquez sur le nom du modèle.

![Modèles du portail des développeurs][api-management-templates-menu]

Cliquez sur un modèle pour accéder à la page du portail des développeurs que vous pouvez personnaliser avec ce modèle. Le modèle utilisé dans cet exemple s’intitule **Product list** (Liste de produits). Le modèle **Product list** contrôle la zone de l’écran indiquée par le rectangle rouge.

![Modèle Products list (Liste de produits)][api-management-developer-portal-templates-overview]

Certains modèles, comme les modèles **User Profile** (Profil utilisateur), permettent de personnaliser différentes parties d’une même page.

![Modèles User profile (Profil utilisateur)][api-management-user-profile-templates]

L’éditeur de chaque modèle du portail des développeurs comporte deux sections affichées au bas de la page. Dans la partie gauche figure le volet d’édition du modèle, tandis que la partie droite présente le modèle de données du modèle.

Le volet d’édition du modèle contient le balisage qui contrôle l’apparence et le comportement de la page correspondante dans le portail des développeurs. Le balisage figurant dans le modèle utilise la syntaxe [DotLiquid](http://dotliquidmarkup.org/) . Pour DotLiquid, il existe un éditeur assez répandu : [DotLiquid for Designers](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers). Les modifications apportées au modèle en phase d’édition s’affichent en temps réel dans le navigateur, mais vos clients ne peuvent pas les voir tant que vous [n’enregistrez](#to-save-a-template) et ne [publiez](#to-publish-a-template) pas le modèle.

![Balisage de modèle][api-management-template]

Le volet **Template data** (Données du modèle) propose un guide du modèle de données pour les entités à utiliser dans un modèle déterminé. Ce guide apparaît en affichant les données actives qui figurent actuellement dans le portail des développeurs. Vous pouvez développer les volets de modèle en cliquant sur le rectangle situé en haut à droite du volet **Template data** (Données du modèle).

![Modèle de données du modèle][api-management-template-data]

Dans l’exemple précédent, deux produits sont affichés dans le portail des développeurs. Ils ont été récupérés à partir des données affichées dans le volet **Template data** (Données du modèle), comme l’illustre l’exemple suivant :

```json
{
    "Paging": {
        "Page": 1,
        "PageSize": 10,
        "TotalItemCount": 2,
        "ShowAll": false,
        "PageCount": 1
    },
    "Filtering": {
        "Pattern": null,
        "Placeholder": "Search products"
    },
    "Products": [
        {
            "Id": "56ec64c380ed850042060001",
            "Title": "Starter",
            "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",
            "Terms": "",
            "ProductState": 1,
            "AllowMultipleSubscriptions": false,
            "MultipleSubscriptionsCount": 1
        },
        {
            "Id": "56ec64c380ed850042060002",
            "Title": "Unlimited",
            "Description": "Subscribers have completely unlimited access to the API. Administrator approval is required.",
            "Terms": null,
            "ProductState": 1,
            "AllowMultipleSubscriptions": false,
            "MultipleSubscriptionsCount": 1
        }
    ]
}
```

Le balisage figurant dans le modèle **Product list** (Liste de produits) traite les données pour fournir le résultat souhaité en itérant au sein de la collection de produits pour afficher des informations et un lien vers chaque produit. Notez la présence des éléments `<search-control>` et `<page-control>` dans le balisage. Ceux-ci contrôlent l’affichage des contrôles de recherche et de pagination dans la page. `ProductsStrings|PageTitleProducts` est une référence de chaîne localisée qui contient le texte d’en-tête `h2` de la page. Pour obtenir la liste des ressources de chaîne, des contrôles de page et des icônes à utiliser dans les modèles du portail des développeurs, consultez les [informations de référence sur les modèles du portail des développeurs Gestion des API](api-management-developer-portal-templates-reference.md).

```html
<search-control></search-control>
<div class="row">
    <div class="col-md-9">
        <h2>{% localized "ProductsStrings|PageTitleProducts" %}</h2>
    </div>
</div>
<div class="row">
    <div class="col-md-12">
    {% if products.size > 0 %}
    <ul class="list-unstyled">
    {% for product in products %}
        <li>
            <h3><a href="/products/{{product.id}}">{{product.title}}</a></h3>
            {{product.description}}
        </li>
    {% endfor %}
    </ul>
    <paging-control></paging-control>
    {% else %}
    {% localized "CommonResources|NoItemsToDisplay" %}
    {% endif %}
    </div>
</div>
```

## <a name="to-save-a-template"></a>Pour enregistrer un modèle
Pour enregistrer un modèle, cliquez sur save (enregistrer) dans l’éditeur de modèle.

![Enregistrer un modèle][api-management-save-template]

Les modifications enregistrées ne sont pas actives dans le portail des développeurs tant qu’elles ne sont pas publiées.

## <a name="to-publish-a-template"></a>Pour publier un modèle
Les modèles enregistrés peuvent être publiés individuellement ou en bloc. Pour publier un modèle individuel, cliquez sur publish (publier) dans l’éditeur de modèle.

![Publier un modèle][api-management-publish-template]

Cliquez sur **Yes** (Oui) pour valider et rendre le modèle actif sur le portail des développeurs.

![Capture d’écran montrant où vous sélectionnez Oui pour rendre le modèle actif.][api-management-publish-template-confirm]

Pour publier toutes les versions des modèles non publiés, cliquez sur **Publish** (Publier) dans la liste des modèles. Les modèles non publiés sont reconnaissables à la présence d’un astérisque après leur nom. Dans cet exemple, les modèles **Product list** et **Product** sont sur le point d’être publiés.

![Publier des modèles][api-management-publish-templates]

Cliquez sur **Publish customizations** (Publier les personnalisations) pour confirmer.

![Confirmer une publication][api-management-publish-customizations]

Les modèles nouvellement publiés prennent immédiatement effet dans le portail des développeurs.

## <a name="to-revert-a-template-to-the-previous-version"></a>Pour rétablir la version précédente d’un modèle
Pour rétablir la version précédente d’un modèle publié, cliquez sur revert (rétablir) dans l’éditeur de modèle.

![Capture d’écran qui met en surbrillance l’icône que vous utilisez pour rétablir un modèle.][api-management-revert-template]

Cliquez sur **Oui** pour confirmer.

![Capture d’écran montrant où vous sélectionnez Oui pour confirmer les modifications.][api-management-revert-template-confirm]

La version précédente d’un modèle publié est actif dans le portail des développeurs dès que l’opération de rétablissement est terminée.

## <a name="to-restore-a-template-to-the-default-version"></a>Pour restaurer la version par défaut d’un modèle
La restauration de la version par défaut d’un modèle est une procédure en deux étapes qui consiste d’abord à restaurer le modèle, puis à publier sa version restaurée.

Pour restaurer la version par défaut d’un modèle, cliquez sur restore (restaurer) dans l’éditeur de modèle.

![Rétablir un modèle][api-management-reset-template]

Cliquez sur **Oui** pour confirmer.

![Confirmer][api-management-reset-template-confirm]

Pour restaurer la version par défaut de tous les modèles, cliquez sur **Restore default templates** (Restaurer les modèles par défaut) dans la liste des modèles.

![Restaurer des modèles][api-management-restore-templates]

Les modèles restaurés doivent ensuite être publiés un par un ou tous en même temps en suivant les étapes décrites dans la section [Pour publier un modèle](#to-publish-a-template).

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des informations de référence sur les modèles du portail des développeurs, les ressources de chaîne, les icônes et les contrôles de page, consultez les [informations de référence sur les modèles du portail des développeurs Gestion des API](api-management-developer-portal-templates-reference.md).

[modify-content-layout]: api-management-modify-content-layout.md
[customize-styles]: api-management-customize-styles.md
[portal-templates]: api-management-developer-portal-templates.md

[api-management-customize-menu]: ./media/api-management-developer-portal-templates/api-management-customize-menu.png
[api-management-templates-menu]: ./media/api-management-developer-portal-templates/api-management-templates-menu.png
[api-management-developer-portal-templates-overview]: ./media/api-management-developer-portal-templates/api-management-developer-portal-templates-overview.png
[api-management-template]: ./media/api-management-developer-portal-templates/api-management-template.png
[api-management-template-data]: ./media/api-management-developer-portal-templates/api-management-template-data.png
[api-management-developer-portal-menu]: ./media/api-management-developer-portal-templates/api-management-developer-portal-menu.png
[api-management-management-console]: ./media/api-management-developer-portal-templates/api-management-management-console.png
[api-management-browse]: ./media/api-management-developer-portal-templates/api-management-browse.png
[api-management-user-profile-templates]: ./media/api-management-developer-portal-templates/api-management-user-profile-templates.png
[api-management-save-template]: ./media/api-management-developer-portal-templates/api-management-save-template.png
[api-management-publish-template]: ./media/api-management-developer-portal-templates/api-management-publish-template.png
[api-management-publish-template-confirm]: ./media/api-management-developer-portal-templates/api-management-publish-template-confirm.png
[api-management-publish-templates]: ./media/api-management-developer-portal-templates/api-management-publish-templates.png
[api-management-publish-customizations]: ./media/api-management-developer-portal-templates/api-management-publish-customizations.png
[api-management-revert-template]: ./media/api-management-developer-portal-templates/api-management-revert-template.png
[api-management-revert-template-confirm]: ./media/api-management-developer-portal-templates/api-management-revert-template-confirm.png
[api-management-reset-template]: ./media/api-management-developer-portal-templates/api-management-reset-template.png
[api-management-reset-template-confirm]: ./media/api-management-developer-portal-templates/api-management-reset-template-confirm.png
[api-management-restore-templates]: ./media/api-management-developer-portal-templates/api-management-restore-templates.png
