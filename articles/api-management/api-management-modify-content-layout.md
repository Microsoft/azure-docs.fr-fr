---
title: Modifier le contenu d’une page dans le portail des développeurs dans Gestion des API
titleSuffix: Azure API Management
description: Découvrez comment modifier le contenu d’une page dans le portail des développeurs dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vlvinogr
manager: vlvinogr
editor: ''
ms.assetid: 186128fe-41c0-4efb-9efe-2478ad4d103f
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/09/2017
ms.author: vlvinogr
ms.openlocfilehash: ebf2cbd430339378a09d10d91ad61327d24842e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75430629"
---
# <a name="modify-the-content-and-layout-of-pages-on-the-developer-portal-in-azure-api-management"></a>Modifier le contenu et la disposition des pages dans le portail des développeurs dans Gestion des API Azure
Il existe trois manières principales pour personnaliser le portail des développeurs dans Gestion des API Azure :

* [Modifier le contenu des pages statiques et des éléments de mise en page][modify-content-layout] (procédure expliquée dans ce guide)
* [Mettre à jour les styles utilisés pour les éléments de page dans le portail des développeurs][customize-styles]
* [Modifier les modèles utilisés pour les pages générées par le portail][portal-templates] (par exemple, documents API, produits, authentification des utilisateurs, etc.)

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="structure-of-developer-portal-pages"></a><a name="page-structure"> </a>Structure des pages du portail des développeurs

Le portail des développeurs s’appuie sur un système de gestion de contenu. La disposition de chaque page s’appuie sur un ensemble d’éléments de petite page appelés widgets :

![Structure de page du portail des développeurs][api-management-customization-widget-structure]

Tous les widgets sont modifiables.
* Le contenu de base spécifique à chaque page réside dans le widget « Contenu ». La modification d’une page entraîne la modification du contenu de ce widget.
* Tous les éléments de mise en page sont contenus dans les autres widgets. Les modifications apportées à ces widgets s’appliquent à toutes les pages. Ils sont dénommés « widgets de mise en page ».

Dans la modification de page standard, un utilisateur ne modifie en général que le widget Contenu qui contient un contenu différent pour chaque page.

## <a name="modifying-the-contents-of-a-layout-widget"></a><a name="modify-layout-widget"> </a>Modification du contenu d’un widget de mise en page

La portail des développeurs est accessible depuis le portail Azure.

1. Cliquez sur **Portail des développeurs** dans la barre d’outils de votre instance de gestion des API.
2. Pour modifier le contenu des widgets, cliquez sur l’icône constituée de deux pinceaux du menu de gauche du **portail des développeurs**.
3. Pour modifier le contenu de l’en-tête, faites défiler jusqu’à la section **En-tête** de la liste de gauche.

    Les widgets sont modifiables à l’intérieur des champs.
4. Quand vous êtes prêt à publier vos modifications, cliquez sur **Publier** dans la partie inférieure de la page.

Vous devriez à présent pouvoir voir le nouvel en-tête sur chaque page du portail des développeurs.

## <a name="next-steps"></a><a name="next-steps"> </a>Étapes suivantes
* [Mettre à jour les styles utilisés pour les éléments de page dans le portail des développeurs][customize-styles]
* [Modifier les modèles utilisés pour les pages générées par le portail][portal-templates] (par exemple, documents API, produits, authentification des utilisateurs, etc.)

[Structure of developer portal pages]: #page-structure
[Modifying the contents of a layout widget]: #modify-layout-widget
[Edit the contents of a page]: #edit-page-contents
[Next steps]: #next-steps

[modify-content-layout]: api-management-modify-content-layout.md
[customize-styles]: api-management-customize-styles.md
[portal-templates]: api-management-developer-portal-templates.md

[api-management-customization-widget-structure]: ./media/api-management-modify-content-layout/portal-widget-structure.png
[api-management-management-console]: ./media/api-management-modify-content-layout/api-management-management-console.png
[api-management-widgets-header]: ./media/api-management-modify-content-layout/api-management-widgets-header.png
[api-management-customization-manage-content]: ./media/api-management-modify-content-layout/api-management-customization-manage-content.png
