---
title: Gérer les ressources de données dans Azure Data Catalog
description: Cet article explique comment contrôler la visibilité et la propriété des ressources de données inscrites dans Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 9d035417b613c81487c493bde90cdcc461f04412
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017285"
---
# <a name="manage-data-assets-in-azure-data-catalog"></a>Gérer les ressources de données dans Azure Data Catalog
## <a name="introduction"></a>Introduction
Azure Data Catalog est conçu pour la détection des sources de données. Il permet de détecter facilement les sources de données dont vous avez besoin pour effectuer des analyses et prendre des décisions. Ces fonctionnalités de détection se révèlent très utiles, car elles vous permettent d’effectuer des recherches sur les nombreuses sources de données disponibles et de les comprendre. Par conséquent, le comportement par défaut de Data Catalog consiste à rendre visibles et détectables l’ensemble des sources de données inscrites pour tous les utilisateurs du catalogue.

Data Catalog ne permet pas d’accéder aux données proprement dites. L’accès aux données est contrôlé par le propriétaire de la source de données. Data Catalog vous permet de détecter les sources de données et d’afficher les métadonnées relatives aux sources inscrites dans le catalogue.

Il peut arriver, cependant, que seuls certains utilisateurs ou groupes soient autorisés à accéder à une source de données. Dans ce cas, les utilisateurs peuvent prendre possession des ressources de données inscrites dans le catalogue et contrôler ensuite la visibilité des ressources dont ils sont propriétaires.

> [!NOTE]
> La fonctionnalité décrite dans cet article est uniquement disponible dans l’édition Standard d’Azure Data Catalog. L’édition gratuite ne fournit aucune fonctionnalité relative à la propriété ou à la restriction de la visibilité des ressources de données.
>
>

## <a name="manage-ownership-of-data-assets"></a>Gérer la propriété des ressources de données
Par défaut, les ressources de données qui sont inscrites dans le catalogue de données n’ont pas de propriétaire. Tout utilisateur autorisé à accéder au catalogue peut détecter et annoter ces ressources. Les utilisateurs peuvent prendre possession des ressources de données sans propriétaire et limiter ensuite la visibilité des ressources dont ils sont propriétaires.

Lorsqu’une ressource de données de Data Catalog est affectée à un propriétaire, seuls les utilisateurs autorisés par le propriétaire peuvent détecter la ressource et afficher ses métadonnées, et seul le propriétaire est habilité à supprimer la ressource du catalogue.

> [!NOTE]
> Dans Data Catalog, la propriété affecte uniquement les métadonnées qui sont stockées dans le catalogue. Elle ne confère aucune autorisation sur la source de données sous-jacente.
>
>

### <a name="take-ownership"></a>Prendre possession
Les utilisateurs peuvent prendre possession des ressources de données en sélectionnant l’option **Prendre possession** dans le portail Data Catalog. Aucune autorisation spéciale n’est nécessaire pour prendre possession d’une ressource de données sans propriétaire. Tout utilisateur peut prendre possession d’une ressource de données sans propriétaire.

### <a name="add-owners-and-co-owners"></a>Ajouter des propriétaires et des copropriétaires
Si une ressource de données est déjà la propriété d’un utilisateur, les autres utilisateurs ne peuvent pas en prendre possession. Toutefois, ils peuvent être ajoutés en tant que copropriétaires par le propriétaire existant. N’importe quel propriétaire peut ajouter des utilisateurs ou groupes de sécurité supplémentaires en tant que copropriétaires.

> [!NOTE]
> Il est recommandé de désigner au moins deux propriétaires pour n’importe quelle ressource de données détenue par un propriétaire.
>
>

### <a name="remove-owners"></a>Supprimer des propriétaires
De la même manière que n’importe quel propriétaire de ressources peut ajouter des copropriétaires, n’importe quel propriétaire de ressource a la possibilité de supprimer des copropriétaires.

Si le propriétaire d’une ressource se supprime lui-même en tant que propriétaire, il n’aura plus aucun contrôle sur la ressource. Si le propriétaire d’une ressource se supprime lui-même en tant que propriétaire alors qu’il n’a désigné aucun copropriétaire, la ressource de données redevient sans propriétaire.

## <a name="control-visibility"></a>Contrôler la visibilité
Les propriétaires de ressources de données peuvent contrôler la visibilité des ressources qu’ils détiennent. Pour définir une visibilité plus restreinte que celle du paramètre par défaut (où tous les utilisateurs de Data Catalog ont la possibilité de détecter et d’afficher la ressource de données), le propriétaire de la ressource peut modifier le paramètre de visibilité dans les propriétés de la ressource en remplaçant l’option **Tout le monde** par l’option **Propriétaires et ces utilisateurs**. Les propriétaires peuvent ensuite ajouter des utilisateurs et groupes de sécurité spécifiques.

> [!NOTE]
> Dans la mesure du possible, les autorisations de possession et de visibilité des ressources doivent être attribuées à des groupes de sécurité, et non à des utilisateurs individuels.
>
>

## <a name="catalog-administrators"></a>Administrateurs de Data Catalog
Les administrateurs de Data Catalog sont implicitement copropriétaires de toutes les ressources de données du catalogue. Les propriétaires des ressources ne peuvent pas supprimer la visibilité des ressources pour les administrateurs. En outre, les administrateurs peuvent gérer la propriété et la visibilité de toutes les ressources de données contenues dans le catalogue.

## <a name="summary"></a>Résumé
Data Catalog repose sur un modèle de détection des données et des métadonnées basé sur la notion de crowdsourcing, qui permet à tous les utilisateurs du catalogue de collaborer et détecter des ressources. L’édition Standard de Data Catalog comprend des fonctionnalités de propriété et de gestion permettant de restreindre la visibilité et l’utilisation des ressources de données.
