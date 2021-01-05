---
title: Configurer votre expérience de Recherche personnalisée Bing | Microsoft Docs
titleSuffix: Azure Cognitive Services
description: Le portail vous permet de créer une instance de recherche qui spécifie les tranches du web, domaines, sous-pages et pages web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: conceptual
ms.date: 02/12/2019
ms.author: aahi
ms.openlocfilehash: b2d4eb75002474bbec57753bbf0255cc4c2b4a16
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96338637"
---
# <a name="configure-your-bing-custom-search-experience"></a>Configurer votre expérience de Recherche personnalisée Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Une instance de Recherche personnalisée vous permet d’adapter l’expérience de recherche pour inclure uniquement du contenu provenant de sites Web qui intéressent vos utilisateurs. Au lieu d’effectuer une recherche à l’échelle du web, Bing ne recherche que dans les sections du web qui vous intéressent. Pour créer votre vue personnalisée du web, utilisez le [portail](https://www.customsearch.ai) de la Recherche personnalisée Bing.

Le portail vous permet de créer une instance de recherche qui spécifie les sections du web : domaines, sous-pages et pages web que Bing doit ou ne doit pas utiliser pour ses recherches. Le portail peut également suggérer du contenu que vous pouvez inclure.

Utilisez les éléments suivants pour définir vos sections du web :

| Nom de la section | Description                                                                                                                                                                                                                                                                                                |
|------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Domain     | Une section de domaine inclut tout le contenu trouvé dans un domaine internet. Par exemple : `www.microsoft.com`. L’omission de `www.` oblige Bing à rechercher également dans les sous-domaines du domaine. Par exemple, si vous spécifiez `microsoft.com`, Bing retourne également les résultats de `support.microsoft.com` ou `technet.microsoft.com`. |
| Sous-page    | Une section de sous-page comprend tout le contenu trouvé dans la sous-page, et les chemins situés en dessous de celle-ci. Vous pouvez indiquer un maximum de deux sous-pages dans le chemin. Par exemple : `www.microsoft.com/en-us/windows/`                                                                                                                       |
| Page web    | Une section de page Web ne peut inclure que cette page Web dans une recherche personnalisée. Si vous le souhaitez, vous pouvez préciser s’il faut inclure ou non les sous-pages.                                                                                                                                                                                  |

> [!IMPORTANT]
> Tous les domaines, sous-pages et pages Web que vous spécifiez doivent être publics et indexés par Bing. Si vous possédez un site public à inclure dans la recherche, et que Bing ne l’a pas l’indexé, consultez la [documentation du webmaster](https://www.bing.com/webmaster/help/webmaster-guidelines-30fba23a) Bing pour savoir comment obliger Bing à l’indexer. Consultez également la documentation du webmaster pour savoir comment forcer Bing à mettre à jour votre site analysé si l’index est obsolète.

## <a name="add-slices-of-the-web-to-your-custom-search-instance"></a>Ajouter des sections du web à votre instance de recherche personnalisée

Lorsque vous créez votre instance de recherche personnalisée, vous pouvez spécifier les sections du web, domaines, sous-pages et des pages web que vous souhaitez voir inclus ou bloqués dans les résultats de votre recherche. 

Si vous savez les sections que vous souhaitez inclure dans votre instance de recherche personnalisée, ajoutez-les à la liste des éléments actifs (**Active**) de votre instance. 

Si vous ignorez les sections à inclure, vous pouvez adresser des requêtes de recherche à Bing via le volet **Preview** (Visualisation) en sélectionnant les sections souhaitées. Pour ce faire : 

1. Sélectionnez « Bing » dans la liste déroulante du volet Preview, puis entrez une requête de recherche.

2. Cliquez sur **Add site** (Ajouter le site) en regard du résultat que vous souhaitez inclure. Puis cliquez sur OK.

>[!NOTE]
> [!INCLUDE[publish or revert](./includes/publish-revert.md)]

<a name="active-and-blocked-lists"></a>

### <a name="customize-your-search-experience-with-active-and-blocked-lists"></a>Personnaliser votre expérience de recherche avec les listes des éléments actifs et des éléments bloqués 

Vous pouvez accéder à la liste des sections actives et bloquées en cliquant respectivement sur les onglets **Active** et **Blocked** dans votre instance de recherche personnalisée. Les sections ajoutées à la liste des sections actives seront incluses dans votre recherche personnalisée. Les sections bloquées ne feront l’objet d’aucune recherche et n’apparaîtront pas dans les résultats de votre recherche.

Pour spécifier les sections du web dans lesquelles Bing doit rechercher, cliquez sur l’onglet **Active**, puis ajoutez une ou plusieurs URL. Pour modifier ou supprimer des sections, utilisez les options de la colonne **Controls** (Contrôles). 

Lors de l’ajout d’URL à la liste **Active**, vous pouvez ajouter des URL uniques ou plusieurs URL à la fois en chargeant un fichier texte à l’aide de l’icône de chargement.

![Onglet Active de Recherche personnalisée Bing](media/file-upload-icon.png)

Pour charger un fichier, créez un fichier texte en spécifiant un domaine, une sous-page ou une page web par ligne. Votre fichier sera rejeté s’il n’est pas correctement mis en forme.

> [!NOTE]
> * Vous ne pouvez charger qu’un seul fichier dans la liste **Active**. Vous ne pouvez pas l’utiliser pour ajouter des sections à la liste **Blocked**.  
> * Si la liste **Blocked** contient un domaine, une sous-page ou une page web spécifiés dans le fichier chargé, ces éléments sont supprimés de la liste **Blocked** et ajoutés à la liste **Active**.
> * Recherche personnalisée Bing ignore les entrées en double dans votre fichier chargé. 

### <a name="get-website-suggestions-for-your-search-experience"></a>Obtenir des suggestions de site web pour votre expérience de recherche

Après l’ajout de sections web à la liste **Active**, le portail Recherche personnalisée Bing génère des suggestions de sites web et de sous-pages en bas de l’onglet. Il s’agit de sections dont Recherche personnalisée Bing estime que vous pourriez vouloir les inclure. Cliquez sur **Refresh** (Actualiser) pour obtenir des suggestions actualisées suite à la mise à jour des paramètres de votre instance de recherche personnalisée. Cette section n’est visible que si des suggestions sont disponibles.

## <a name="search-for-images-and-videos"></a>Rechercher des images et vidéos

Vous pouvez rechercher des images et vidéos de la même façon que du contenu web à l’aide de l’[API Recherche d’images personnalisée Bing](/rest/api/cognitiveservices-bingsearch/bing-custom-images-api-v7-reference) ou de l’[API Recherche de vidéos personnalisée Bing](/rest/api/cognitiveservices-bingsearch/bing-custom-videos-api-v7-reference). Vous pouvez afficher ces résultats avec l’[interface utilisateur hébergée](hosted-ui.md) ou les API. 

Ces API sont similaires aux API [Recherche d’images Bing](../Bing-Image-Search/overview.md) et [Recherche de vidéos Bing](../bing-video-search/overview.md) non personnalisées, mais recherchent dans l’intégralité du web et ne nécessitent pas le paramètre de requête `customConfig`. Pour plus d’informations sur l’utilisation d’images et de vidéos, voir ces ensembles de documentation. 

## <a name="test-your-search-instance-with-the-preview-pane"></a>Tester votre instance de recherche avec le volet de visualisation

Vous pouvez tester votre instance de recherche en utilisant le volet de visualisation situé du côté droit du portail pour soumettre des requêtes de recherche et afficher les résultats. 

1. Sous la zone de recherche, sélectionnez **My instance** (Mon instance). Vous pouvez comparer les résultats de votre expérience de recherche à Bing en sélectionnant **Bing**. 
2. Sélectionnez un filtre de recherche sécurisée et le marché dans lequel effectuer la recherche (voir [Paramètres de requête](/rest/api/cognitiveservices-bingsearch/bing-custom-search-api-v7-reference#query-parameters)).
3. Entrez une requête et appuyez sur Entrée ou cliquez sur l’icône de recherche pour afficher les résultats issus de la configuration active. Vous pouvez modifier le type de recherche que vous effectuez en cliquant sur **Web**, **Image** ou **Vidéo** pour obtenir les résultats correspondants. 

<a name="adjustrank"></a>

## <a name="adjust-the-rank-of-specific-search-results"></a>Ajuster le classement des résultats de recherche spécifique

Le portail vous permet d’ajuster le classement de contenu de la recherche à partir de domaines, sous-pages et pages web spécifiques. Après l’envoi d’une requête de recherche dans le volet de visualisation, chaque résultat de recherche contient une liste d’ajustements que vous pouvez faire :  

| Ajustement | Description |
|------------|-------------|
| Block      | Déplace le domaine, la sous-page ou la page Web dans la liste des éléments bloqués. Bing exclut le contenu du site sélectionné de l’affichage dans les résultats de recherche.                    |
| Boost (Promouvoir)      | Promeut le contenu du domaine ou de la sous-page à un niveau supérieur dans les résultats de recherche.                                                                                        |
| Demote (Rétrograder)     | Rétrograde le contenu du domaine ou de la page secondaire à un niveau inférieur dans les résultats de recherche. Vous choisissez s’il faut rétrograder le contenu issu de la sous-page ou du domaine auquel la page Web appartient. |
| Épingler en haut | Déplace le domaine, la sous-page ou la page web vers la liste des éléments **épinglés**. Cela force l’affichage de la page web en tant que premier résultat de recherche pour une requête de recherche donnée.                   |

L’ajustement du classement n’est pas disponible pour les recherches d’images ou de vidéos.

### <a name="boosting-and-demoting-search-results"></a>Promotion et rétrogradation de résultats de recherche

Vous pouvez promouvoir, un peu ou beaucoup, ou rétrograder tout domaine ou sous-page figurant dans la liste **Active**. Par défaut, toutes les sections sont ajoutées sans ajustement de classement. Les sections du web qui sont promues (Boosted) ou très promues (Super boosted) dans le classement bénéficient d’une meilleure position dans les résultats de recherche (une position très promue étant supérieure à une position promue). Les éléments qui sont rétrogradés sont situés plus bas dans les résultats de recherche.

Vous pouvez promouvoir des éléments, un peu ou beaucoup, ou les rétrograder en utilisant les contrôles **Ranking Adjust** (Ajustement du classement) dans la liste **Active** (Éléments actifs), ou les contrôles Boost (Promouvoir) et Demote (Rétrograder) dans le volet de visualisation. Le service ajoute la section à votre liste d’éléments actifs et ajuste le classement en conséquence.

> [!NOTE] 
> La promotion et la rétrogradation de domaines et sous-pages figurent parmi les nombreuses méthodes que Recherche personnalisée Bing utilise pour déterminer l’ordre des résultats de recherche. En raison d’autres facteurs influençant le classement des différents contenus web, les effets de l’ajustement du classement peuvent varier. Utilisez le volet de visualisation pour tester les effets de l’ajustement du classement de vos résultats de recherche. 

Il n’est pas possible de promouvoir ou rétrograder des résultats de recherches d’images ou de vidéos.

## <a name="pin-slices-to-the-top-of-search-results"></a>Épingler des sections en tête des résultats de recherche

Le portail vous permet également d’épingler des URL en tête des résultats de recherche pour des termes spécifiques à l’aide de l’onglet **Pinned** (Épinglé). Entrez une URL et une requête pour spécifier la page web qui s’affiche en tête des résultats. Notez que vous ne pouvez épingler qu’une seule page web par requête de recherche, et que seules les pages web indexées apparaissent dans les recherches. Il n’est pas possible d’épingler des résultats de recherches d’images ou de vidéos.

Vous pouvez épingler une page web en tête des résultats de deux manières :

* Sous l’onglet **Pinned** (Épinglé), entrez l’URL de la page web à épingler en tête des résultats et la requête correspondante.

* Dans le volet **Preview** (Visualisation), entrez une requête de recherche, puis cliquez sur Search (Rechercher). Recherchez la page web à épingler pour votre requête, puis cliquez sur **Pin to top** (Épingler en haut). La page web et la requête seront ajoutées à la liste **Pinned** (Épinglé).

### <a name="specify-the-pins-match-condition"></a>Spécifier la condition de correspondance de l’épingle

Par défaut, des pages web ne sont épinglées en tête des résultats de recherche que quand la chaîne de requête d’un utilisateur correspond exactement à une chaîne répertoriée dans la liste **Pinned** (Épinglé). Vous pouvez modifier ce comportement en spécifiant l’une des conditions de correspondance suivantes :

> [!NOTE]
> Les comparaisons entre la requête de recherche de l’utilisateur et la requête de recherche de l’épingle ne respectent pas la casse.

| Valeur | Description                                                                          |
|---------------|----------------------------------------------------------------------------------|
| Starts With (Commence par) | Une correspondance d’épingle est trouvée si la chaîne de requête de l’utilisateur commence par la chaîne de requête de l’épingle. |
| Se termine par   | Une correspondance d’épingle est trouvée si la chaîne de requête de l’utilisateur se termine par la chaîne de requête de l’épingle.  |
| Contient    | Une correspondance d’épingle est trouvée si la chaîne de requête de l’utilisateur contient la chaîne de requête de l’épingle.   |


Pour modifier la condition de correspondance de l’épingle, cliquez sur l’icône de modification de l’épingle. Dans la colonne **Query match condition** (Condition de correspondance de la requête), cliquez sur la liste déroulante, puis sélectionnez la nouvelle condition à utiliser. Ensuite, cliquez sur l’icône d’enregistrement pour enregistrer les modifications.

### <a name="change-the-order-of-your-pinned-sites"></a>Modifier l’ordre de vos sites épinglés

Pour modifier l’ordre de vos épingles, vous pouvez les glisser-déplacer ou modifier leur numéro d’ordre en cliquant sur l’icône « Modifier » dans la colonne **Controls** (contrôles) de la liste **Pinned** (Épinglé).

Si plusieurs épingles remplissent une condition de correspondance, Recherche personnalisée Bing utilise la plus élevée dans la liste.

## <a name="view-statistics"></a>Consulter les statistiques

Si vous êtes abonné au niveau approprié de la Recherche personnalisée (voir les [Tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/)), un onglet **Statistics** (Statistiques) est ajouté à vos instances de production. Cet onglet de statistiques affiche des informations détaillées sur l’utilisation de vos points de terminaison de Recherche personnalisée, notamment sur le volume d’appels, les requêtes principales, la répartition géographique, les codes de réponse et la recherche sécurisée. Vous pouvez filtrer ces informations à l’aide des contrôles fournis.

## <a name="usage-guidelines"></a>Instructions d’utilisation

- Pour chaque instance de recherche personnalisée, le nombre maximal d’ajustements de classement que vous pouvez apporter aux sections **Active** (Éléments actifs) et **Blocked** (Éléments bloqués) est limité à 400.
- L’ajout d’une section à l’onglet Active (Éléments actifs) ou Blocked (Éléments bloqués) compte pour un ajustement de classement.
- L’élévation et la rétrogradation comptent pour deux ajustements de classement.
- Pour chaque instance de recherche personnalisée, le nombre maximal d’épingles à utiliser est limité à 200.

## <a name="next-steps"></a>Étapes suivantes

- [Appeler votre recherche personnalisée](./search-your-custom-view.md)
- [Configurer votre expérience d’interface utilisateur hébergée](./hosted-ui.md)
- [Utiliser des marqueurs d’ornement pour mettre en surbrillance du texte](../bing-web-search/hit-highlighting.md)
- [Paginer des pages web](../bing-web-search/paging-search-results.md)