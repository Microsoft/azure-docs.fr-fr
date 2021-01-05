---
title: Utilisation des groupes dans des classeurs Azure Monitor
description: Guide pratique pour utiliser des groupes dans des classeurs Azure Monitor
services: azure-monitor
author: lgayhardt
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: lagayhar
ms.openlocfilehash: b6377cdcdb5816426eba62fdbef79eeb42659dcc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82891786"
---
# <a name="how-to-use-groups-in-workbooks"></a>Comment utiliser des groupes dans des classeurs

En utilisant un élément de groupe dans un classeur, vous pouvez regrouper de façon logique un ensemble d’étapes dans ce classeur.

L’utilisation de groupes dans des classeurs apporte les avantages suivants :

- Disposition
  - Si vous souhaitez que des éléments soient organisés verticalement, vous pouvez créer un groupe d’éléments qui seront tous empilés et définir le pourcentage de largeur pour l’ensemble du groupe, en tant qu’élément de style, au lieu de devoir définir le pourcentage de largeur de chaque élément.
- Visibilité
  - Si vous souhaitez masquer ou afficher de nombreux éléments, vous pouvez définir la visibilité pour l’ensemble du groupe, au lieu de devoir la définir pour chaque élément. Cela peut être utile dans les modèles qui utilisent des onglets, car vous pouvez utiliser un groupe en tant que contenu de l’onglet, puis afficher ou masquer tous les éléments de ce groupe en fonction d’un paramètre défini par l’onglet sélectionné.
- Performances
  - Si votre modèle est très volumineux et contient beaucoup de sections ou d’onglets, vous pouvez convertir chaque section en un sous-modèle, puis utiliser des groupes pour charger tous ces sous-modèles dans le modèle de niveau supérieur. Le contenu des sous-modèles ne peut pas être chargé ou exécuté tant qu’un utilisateur n’a pas affiché ces groupes. Découvrez [comment diviser un modèle volumineux en plusieurs modèles](#how-to-split-a-large-template-into-many-templates).

## <a name="using-groups"></a>Utilisation des groupes

Pour ajouter un groupe à votre classeur, sélectionnez *Add* (Ajouter) puis *Add group* (Ajouter un groupe).

![Capture d’écran montrant l’option Add group (Ajouter un groupe).](./media/workbooks-groups/add-group.png)

L’image ci-dessous est une capture d’écran montrant un groupe en mode de lecture. Il contient deux éléments : un élément de texte et un élément de requête.  

![Capture d’écran d’un groupe en mode de lecture.](./media/workbooks-groups/groups-view.png)

Lorsque vous modifiez le classeur, vous pouvez voir que ces deux éléments sont en fait à l’intérieur d’un élément de groupe :

![Capture d’écran montrant un groupe en mode d’édition. ](./media/workbooks-groups/groups-edit.png)

Dans la capture d’écran ci-dessus, le groupe est en mode d’édition, et l’on peut voir qu’il contient deux éléments (à l’intérieur de la zone en pointillés). Chaque étape peut être en mode d’édition ou de lecture, indépendamment l’une de l’autre. Par exemple, l’étape de texte est en mode d’édition alors que l’étape de requête est en mode de lecture.

## <a name="scoping"></a>Étendue

Actuellement, un groupe est traité comme une nouvelle étendue dans le classeur. Les paramètres créés dans le groupe sont visibles uniquement à l’intérieur du groupe. Cela est également vrai pour la fusion. Vous pouvez uniquement voir les données à l’intérieur de leur groupe ou au niveau parent.

## <a name="group-types"></a>Types de groupes

L’élément de groupe de classeurs vous permet d’ajouter un groupe d’éléments à un classeur. En tant qu’auteur d’un classeur, vous spécifiez de quel type de groupe il s’agit. Il existe deux types de groupes :

- Modifiable
  - Ce groupe de classeur vous permet d’ajouter, de supprimer ou de modifier le contenu des éléments du groupe. Il est le plus souvent utilisé pour améliorer la disposition et la visibilité.
- À partir d'un modèle
  - Ce groupe de classeur se charge à partir du contenu d’un autre modèle avec son ID. Le contenu de ce modèle est chargé et fusionné dans le classeur au moment de l’exécution. En mode d’édition, vous ne pouvez pas modifier le contenu du groupe, car il sera chargé à nouveau à partir du modèle lors du prochain chargement de l’élément.  

## <a name="load-types"></a>Types de chargements

Les contenus d’un groupe peut être chargé de différentes façons. En tant qu’auteur d’un classeur, vous pouvez spécifier quand et comment le contenu d’un groupe doit être chargé.

### <a name="lazy-the-default"></a>Lazy (Différé) (valeur par défaut)

Le groupe se charge uniquement lorsque l’élément est visible. Cela permet à un groupe d’être utilisé par des éléments d’onglet. Si l’onglet n’est jamais sélectionné, le groupe n’est jamais visible, par conséquent le contenu n’est pas chargé.

Pour les groupes créés à partir d’un modèle, le contenu du modèle n’est pas récupéré et les éléments du groupe ne sont pas créés tant que le groupe n’est pas visible. L’utilisateur verra les boucles de progression de l’ensemble du groupe pendant la récupération du contenu.

### <a name="explicit"></a>Explicit (Explicite)

Dans ce mode, un bouton s’affiche à l’emplacement où se trouve le groupe et aucun contenu n’est récupéré ou créé tant que l’utilisateur n’a pas explicitement cliqué sur le bouton pour charger le contenu. Cela est utile dans les scénarios où le calcul du contenu peut être coûteux ou rarement utilisé. L’auteur peut spécifier le texte à afficher sur le bouton.

Vous trouverez ci-dessous une capture d’écran des paramètres de chargement explicite montrant un bouton « Load more » (« Charger plus ») configuré.

![Paramètres de chargement explicite](./media/workbooks-groups/groups-explicitly-loaded.png)

Le groupe avant son chargement dans le classeur :

![Groupe explicite avant son chargement dans le classeur](./media/workbooks-groups/groups-explicitly-loaded-before.png)

Le groupe après avoir été chargé dans le classeur :

![Groupe explicite après avoir été chargé dans un classeur](./media/workbooks-groups/groups-explicitly-loaded-after.png)

### <a name="always"></a>Always (Toujours)

Dans ce mode, le contenu du groupe est toujours chargé et créé dès le chargement du classeur. Il s’agit du mode le plus souvent utilisé lorsqu’un groupe est conçu uniquement à des fins de mise en page, où le contenu reste visible.

## <a name="using-templates-inside-a-group"></a>Utilisation de modèles à l’intérieur d’un groupe

Lorsqu’un groupe est configuré pour être chargé à partir d’un modèle, ce contenu est chargé en différé par défaut. Il se charge uniquement lorsque le groupe est visible.

Lorsqu’un modèle est chargé dans un groupe, le classeur tente de fusionner tous les paramètres déclarés dans le modèle en cours de chargement avec les paramètres déjà existants dans le groupe. Tous les paramètres qui existent déjà dans le classeur et ayant des noms identiques seront fusionnés en dehors du modèle en cours de chargement. Si tous les paramètres d’une étape de paramètre sont fusionnés, l’intégralité de l’étape des paramètres disparaît.

### <a name="example-1-all-parameters-have-identical-names"></a>Exemple 1 : tous les paramètres ont des noms identiques

Prenons l’exemple d’un modèle qui commence par deux paramètres.

- `TimeRange` : paramètre d’intervalle de temps.
- `Filter` : paramètre de texte.

![Capture d’écran montrant la fenêtre Editing parameters item: "top-level parameters" (Modification d’un élément de paramètres : « Paramètres de premier niveau »)](./media/workbooks-groups/groups-top-level-params.png)

Un élément de groupe charge ensuite un deuxième modèle qui possède ses deux paramètres et une étape de texte, où les paramètres sont nommés de la même façon :

![Capture d’écran montrant la fenêtre permettant de modifier un élément de paramètres pour le deuxième modèle](./media/workbooks-groups/groups-merged-away.png)

Lorsque le deuxième modèle est chargé dans le groupe, les paramètres dupliqués sont fusionnés. Étant donné que tous les paramètres sont fusionnés, l’étape des paramètres internes est également fusionnée, ce qui donne un groupe contenant uniquement l’étape texte.

### <a name="example-2-one-parameter-has-an-identical-name"></a>Exemple 2 : un paramètre a un nom identique

Imaginez un modèle de groupe commençant par deux paramètres.

- `TimeRange` : paramètre d’intervalle de temps.
- `FilterB` : paramètre de texte. Remarquez qu’il n’est pas `Filter` comme le modèle précédent.

![Capture d’écran montrant la modification d’un élément de groupe dont le résultat est une fusion de paramètres](./media/workbooks-groups/groups-wont-merge-away.png)

Lorsque le modèle d’élément du groupe est chargé, le paramètre `TimeRange` est fusionné en dehors du groupe. Le classeur passera alors à l’étape des paramètres initiaux avec `TimeRange` et `Filter`, puis l’étape de paramètre du groupe inclura uniquement `FilterB`

![Capture d’écran montrant des paramètres qui ne seront pas fusionnés](./media/workbooks-groups/groups-wont-merge-away-result.png)

Si le modèle chargé avait contenu `TimeRange` et `Filter` (au lieu de `FilterB`), le classeur résultant aurait eu une étape de paramètres et un groupe dans lequel il ne resterait que l’étape de texte.

## <a name="how-to-split-a-large-template-into-many-templates"></a>Comment diviser un modèle volumineux en plusieurs modèles

Pour améliorer les performances, il est préférable de diviser un modèle volumineux en plusieurs modèles plus petits qui chargent du contenu en différé ou à la demande de l’utilisateur. Cela accélère le chargement initial, en réduisant la taille du modèle de niveau supérieur.

Lors de la division d’un modèle en parties, vous devez diviser le modèle en plusieurs sous-modèles qui fonctionnent tous individuellement. Par conséquent, si le modèle de niveau supérieur a un paramètre `TimeRange` que d’autres étapes utilisent, le sous-modèle doit également avoir une étape de paramètres définissant un paramètre avec le même nom. Cela permet aux sous-modèles de fonctionner de manière indépendante et de les charger dans des groupes plus volumineux.

Pour transformer un modèle plus volumineux en plusieurs sous-modèles :

1.  Créez un nouveau groupe vide près de la partie supérieure du classeur, après les paramètres partagés. Ce nouveau groupe finira par devenir un sous-modèle.
2. Créez une copie de l’étape des paramètres partagés, puis utilisez l’option *move into group* (déplacer dans le groupe) pour déplacer la copie dans le groupe créé à l’étape 1. Cette étape de paramètres permet au sous-modèle de fonctionner indépendamment du modèle externe, et d’être fusionné lorsqu’il est chargé dans le modèle externe.
    > [!NOTE]
    > Il n’est pas nécessaire que les paramètres du sous-modèle soient fusionnés si vous n’envisagez pas que les sous-modèles soient visibles indépendamment. Toutefois, cela compliquera énormément leur modification ou leur débogage si vous devez le faire plus tard.

3. Déplacez dans le groupe créé lors de l’étape 1 chaque élément du classeur qui doit être dans le sous-modèle.
4. Si les étapes individuelles déplacées à l’étape 3 comportaient des visibilités conditionnelles, ces dernières deviendront les visibilités du groupe externe (comme pour les onglets). Supprimez-les des éléments à l’intérieur du groupe et ajoutez ce paramètre de visibilité au groupe lui-même. Après cela, enregistrez votre travail pour éviter de perdre des modifications et/ou exportez et enregistrez une copie du contenu JSON.
5. Si vous souhaitez que ce groupe soit chargé à partir d’un modèle, vous pouvez utiliser le bouton *Edit* (Modifier) dans la barre d’outils du groupe. Cela ouvre uniquement le contenu de ce groupe sous la forme d’un classeur dans une nouvelle fenêtre. Vous pouvez ensuite l’enregistrer comme il convient et fermer ce mode de classeur (ne fermez pas le navigateur, mais uniquement cette vue pour revenir au classeur précédent que vous avez modifié).
6. Vous pouvez ensuite modifier l’étape du groupe à charger à partir du modèle et définir le champ ID du modèle sur le classeur/modèle que vous avez créé lors de l’étape 5. Pour travailler avec des ID de classeurs, la source doit être un ID de ressource de classeur partagé. Appuyez sur *Load* (Charger) et le contenu de ce groupe sera maintenant chargé à partir de ce sous-modèle au lieu d’être enregistré dans ce classeur externe.

## <a name="next-steps"></a>Étapes suivantes
- [Présentation des classeurs](workbooks-overview.md)
- [Utilisation de JSONPath avec des classeurs](workbooks-jsonpath.md)