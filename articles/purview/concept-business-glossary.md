---
title: Présentation des fonctionnalités de glossaire métier d’Azure Purview (préversion)
description: Cet article explique ce qu’est un glossaire métier dans Azure Purview.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/13/2020
ms.openlocfilehash: 8b391438d8d6605e7ef493a6552af634db840ad5
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550528"
---
# <a name="understand-business-glossary-features-in-azure-purview"></a>Présentation des fonctionnalités de glossaire métier d’Azure Purview

Cet article donne une vue d’ensemble de la fonctionnalité de glossaire métier d’Azure Purview. 

## <a name="business-glossary"></a>Glossaire métier

Un glossaire comporte du vocabulaire destiné aux utilisateurs professionnels.  Il se compose de termes métier, potentiellement liés les uns aux autres, et les catégorise de sorte qu’ils soient compréhensibles dans des contextes différents. Il devient alors possible de mettre ces termes en correspondance avec des ressources comme une base de données, des tables, des colonnes, etc. Cela permet d’extraire le jargon technique associé aux référentiels de données, offrant ainsi à l’utilisateur professionnel la possibilité de découvrir et d’utiliser les données dans un vocabulaire qui leur est plus familier.


Un glossaire métier est un ensemble de termes. Chaque terme représente un objet dans une organisation. Il est très courant que plusieurs termes représentent le même objet. Par exemple, un client peut également être appelé « acheteur ». Ces termes multiples sont liés, suivant différentes relations possibles, notamment :

- Synonymes : différents termes possédant la même définition
- Apparentés : différents termes possédant une définition similaire

Le même terme peut également impliquer plusieurs objets métier. Il est important que chaque terme soit clairement défini et bien compris au sein de l’organisation.

## <a name="custom-attributes"></a>Attributs personnalisés

Azure Purview prend en charge huit attributs prêts à l’emploi pour tous les termes de glossaires métier :
- Nom
- Définition
- Administrateurs des données
- Experts des données
- Acronyme
- Synonymes
- Termes associés
- Ressources

Ces attributs ne sont ni modifiables ni supprimables. Toutefois, ils ne sont pas suffisants pour définir complètement un terme dans une organisation. Pour résoudre ce problème, Purview propose une fonctionnalité permettant de créer des attributs personnalisés dans un glossaire.

## <a name="term-templates"></a>Modèles de termes

La fonctionnalité Modèles de termes fournit des attributs de glossaire personnalisés qui doivent être regroupés logiquement dans le catalogue. Elle permet de regrouper tous les attributs personnalisés pertinents dans un modèle, puis d’appliquer ce dernier lors de la création du terme de glossaire. Par exemple, tous les attributs personnalisés liés à la finance (par exemple, « centre de coûts », « centre de profit » et « code comptable ») peuvent être regroupés dans un modèle de termes Finance, qui servira à créer des termes de glossaire financiers.

Tous les attributs standard sont regroupés dans un modèle système par défaut. Ils figurent dans tous les modèles de termes que vous créez, de même que les attributs personnalisés supplémentaires éventuellement ajoutés au cours du processus de création du modèle.

## <a name="glossary-vs-classification-vs-sensitivity-labels"></a>Glossaire, classification et étiquettes de confidentialité

Les termes de glossaire, les classifications et les étiquettes constituent tous des annotations ajoutées à une ressource de données. Cependant, la signification qu’ils revêtent est différente dans le contexte du catalogue. 

### <a name="glossary"></a>Glossaire

Comme nous l’avons vu, un glossaire métier définit le vocabulaire métier d’une organisation et aide à combler le fossé entre les différents services de l’entreprise.

### <a name="classifications"></a>Classifications

Les classifications sont des annotations qui peuvent être attribuées à des entités. Du fait de leur flexibilité, elles sont utilisables dans plusieurs scénarios :

- Comprendre la nature des données stockées dans les ressources de données
- Définir des stratégies de contrôle d’accès

Purview comporte à l’heure actuelle plus de 100 classifieurs système. Vous pouvez par ailleurs définir vos propres classifieurs dans le catalogue. Au cours du processus d’analyse, ces classifications sont automatiquement détectées et appliquées aux ressources de données et aux schémas. Vous pouvez toutefois les écraser à tout moment. Les analyses automatisées ne se substituent jamais aux remplacements manuels.

### <a name="sensitivity-labels"></a>Étiquette de confidentialité

Les étiquettes de confidentialité constituent un type d’annotation permettant de classifier et de protéger les données de l’organisation, sans nuire à la productivité ni à la collaboration. Elles servent à identifier des catégories de types de classifications dans les données de l’organisation et à regrouper les stratégies à appliquer à chaque catégorie. Purview utilise les mêmes types d’informations sensibles que Microsoft 365, ce qui vous permet d’étendre votre protection et vos stratégies de sécurité existantes à l’ensemble de votre patrimoine de contenus et de données. Les mêmes étiquettes peuvent être partagées entre les produits Microsoft Office et les ressources de données de Purview.

## <a name="next-steps"></a>Étapes suivantes

- [Gestion des modèles de termes](how-to-manage-term-templates.md)
- [Navigation dans le catalogue de données dans Azure Purview](how-to-browse-catalog.md)
