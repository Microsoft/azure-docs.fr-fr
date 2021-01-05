---
title: Log Analytics dans Azure Monitor propose des ensembles d’exemples de requêtes que vous pouvez exécuter de manière autonome ou utiliser comme point de départ pour vos propres requêtes.
description: Requêtes que vous pouvez démarrer et modifier en cas de besoin
ms.subservice: logs
ms.topic: article
author: rboucher
ms.author: robb
ms.date: 06/16/2020
ms.openlocfilehash: e4b264d980eac50525e9d9013bc2841fe68065a3
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94496299"
---
# <a name="example-queries-in-azure-monitor-log-analytics"></a>Exemples de requêtes dans Azure Monitor Log Analytics
Log Analytics propose des ensembles d’exemples de requêtes que vous pouvez exécuter de manière autonome ou utiliser comme point de départ pour vos propres requêtes. Cet article décrit des exemples de requêtes et la manière de les utiliser.

Si vous n’êtes pas familiarisé avec Log Analytics ou le langage de requête KQL, les exemples de requêtes sont un excellent moyen de commencer. Ils peuvent fournir des informations instantanées sur une ressource et offrent un excellent moyen de commencer à apprendre le KQL et à l’utiliser, réduisant ainsi le temps nécessaire pour commencer à utiliser Log Analytics. Nous avons collecté et organisé plus de 250 exemples de requêtes conçus pour vous fournir une valeur instantanée, et ce nombre d’exemples de requêtes croît continuellement.

## <a name="in-context-queries"></a>Requêtes contextuelles

La nouvelle expérience filtre et suggère des requêtes en contexte. En d’autres termes, le système affiche automatiquement uniquement les requêtes pertinentes pour l’étendue que vous avez sélectionnée.

- Pour une **ressource unique** : les requêtes sont filtrées en fonction du type de ressource.
- Pour un **groupe de ressources** : les requêtes sont filtrées en fonction des ressources présentes dans le groupe de ressources spécifique.
- Pour un **espace de travail** : les requêtes sont filtrées en fonction des solutions installées sur l’espace de travail.

Ce comportement est cohérent pour toutes les étendues de Log Analytics. Si vous ne voyez pas d’exemple de requête pour le type de ressource que vous souhaitez, cela peut être dû à des filtres induits par le contexte. Une section ultérieure explique comment supprimer l’étendue contextuelle afin de pouvoir visualiser toutes les requêtes possibles.

> [!TIP]
> Plus vous avez de ressources dans votre étendue, plus le portail a besoin de temps pour filtrer et afficher la boîte de dialogue des exemples de requêtes.

## <a name="example-query-user-interface"></a>Exemple d’interface utilisateur de requête

Vous pouvez accéder à des exemples de requêtes à partir de deux emplacements différents.

### <a name="example-query-dialog"></a>Boîte de dialogue des exemples de requêtes

Lorsque vous entrez pour la première fois dans l’expérience Log Analytics, la *boîte de dialogue Exemples de requêtes* s’affiche automatiquement.  Vous pouvez également y accéder en cliquant en haut à droite de l’écran sur **Exemples de requêtes**.

![Barre latérale : exemples de requêtes](media/saved-queries/sidebar-2.png)

La boîte de dialogue des exemples de requêtes s’affiche alors comme indiqué ci-dessous :  

![Écran d’exemples de requêtes](media/saved-queries/example-query-start.png)

La capture d’écran précédente affiche l’écran des journaux pour une instance Azure Key Vault. Comme mentionné précédemment dans cet article, les requêtes sont affichées en contexte.  Par conséquent, la capture d’écran ne montre que des exemples relatifs à Key Vault. Si vous sélectionnez un abonnement entier, les requêtes pour tous les types de ressources de cet abonnement s’affichent.  

Chaque exemple de requête est représenté par une carte. Vous pouvez rapidement parcourir les requêtes pour trouver ce dont vous avez besoin. Vous pouvez exécuter la requête directement à partir de la boîte de dialogue ou choisir de la charger dans l’éditeur de requêtes pour des réglages et des ajustements supplémentaires.

### <a name="sidebar-query-experience"></a>Barre latérale de l’expérience des requêtes

Toutes les fonctionnalités de l’expérience de la boîte de dialogue sont accessibles à partir du volet des requêtes dans la barre latérale gauche de Log Analytics. Vous pouvez pointer sur un nom de requête pour obtenir la description de la requête et des fonctionnalités supplémentaires.

![Capture d’écran montrant le volet Requêtes.](media/saved-queries/sidebar-3.png)

## <a name="finding-and-filtering-queries"></a>Recherche et filtrage des requêtes

Les options de cette section sont disponibles à la fois dans la boîte de dialogue et dans la barre latérale, mais avec une interface utilisateur légèrement différente.  

### <a name="use-favorites"></a>Utiliser les favoris

Vous pouvez mettre en favoris les requêtes les plus fréquemment utilisées pour obtenir un accès plus rapide.

> [!TIP]
> La collecte et l’affichage des requêtes par la suite sont un bon moyen de commencer. Recherchez les requêtes dont vous avez besoin, puis cliquez sur l’étoile à côté de la requête pour l’ajouter aux favoris. Si, par la suite, vous trouvez que la requête n’est pas utile, vous pouvez la supprimer des favoris.  

### <a name="filtering-and-group-by"></a>Filtrage et regroupement

Alors que la boîte de dialogue de requête utilise des filtres pour n’afficher que les requêtes ayant le contexte approprié, vous pouvez choisir de supprimer le filtre et voir toutes les requêtes.

### <a name="group-by"></a>Regrouper par

Modifiez le regroupement des requêtes en cliquant sur la liste déroulante *Regrouper par* :

![Écran d’exemples de requêtes GroupBy](media/saved-queries/example-query-groupby.png)

La boîte de dialogue prend en charge le regroupement par :

- **Type de ressource** : ressource telle que définie dans Azure, comme une machine virtuelle. Pour une cartographie complète des tables Azure Monitor Logs/Log Analytics par type de ressource, consultez la [référence des tables Azure Monitor](/azure/azure-monitor/reference/tables/tables-resourcetype).  
- **Catégorie** : type d’informations telles que *Sécurité* ou *Audit*. Les catégories sont identiques à celles définies dans le volet latéral Tables. Pour obtenir la liste complète des catégories, consultez la [référence des tables Azure Monitor](/azure/azure-monitor/reference/tables/tables-category).  
- **Solution** : solution Azure Monitor associée aux requêtes.
- **Rubrique** : rubrique de l’exemple de requête, telle que *Journaux d’activité* ou *Journaux d’application*. La propriété Rubrique est unique aux exemples de requêtes et peut différer selon le type de ressource spécifique.

Les valeurs de regroupement agissent également comme une table des matières active. Cliquez sur l’une des valeurs à gauche de l’écran pour faire défiler l’affichage des requêtes jusqu’à l’élément sur lequel vous avez cliqué.

### <a name="filter"></a>Filtrer

Vous pouvez également filtrer les requêtes en fonction des valeurs **group by** mentionnées précédemment. Dans la boîte de dialogue des exemples de requêtes, les filtres se trouvent en haut.

![Écran des filtres d’exemples de requêtes](media/saved-queries/example-query-filter.png)

### <a name="combining-group-by-and-filter"></a>Combinaison du regroupement et du filtrage

Les fonctionnalités de filtrage et de regroupement sont conçues pour fonctionner en tandem. Elles offrent une certaine flexibilité quant à la façon dont les requêtes sont organisées. Par exemple, si vous utilisez un groupe de ressources composé de plusieurs ressources, vous pouvez filtrer sur un type de ressource spécifique et organiser les requêtes résultantes par rubrique.

## <a name="sample-query-dialog-appearance-behavior"></a>Comportement de l’apparence de la boîte de dialogue des exemples de requêtes

Si vous êtes un maître du langage KQL et que vous préférez accéder directement à l’éditeur de requêtes, vous pouvez désactiver la boîte de dialogue de requêtes. Avec l’option désactivée, la boîte de dialogue des exemples de requêtes ne se charge pas lorsque l’écran Log Analytics se charge.

![Activation/désactivation des exemples](media/saved-queries/examples-on-off.png)

Vous pouvez toujours accéder à la fenêtre contextuelle des exemples de requêtes à partir du bouton *Exemples de requêtes* dans la barre supérieure de Log Analytics.

## <a name="query-explorer"></a>Explorateur de requêtes

L’explorateur de requêtes pour l’enregistrement et le partage de requêtes générées par l’utilisateur reste inchangé pour le moment.

## <a name="next-steps"></a>Étapes suivantes

[Prise en main des requêtes KQL](get-started-queries.md)

