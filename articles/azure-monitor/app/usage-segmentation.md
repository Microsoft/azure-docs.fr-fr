---
title: Analyse des utilisateurs, des sessions et des événements dans Azure Application Insights
description: Analyse démographique des utilisateurs de votre application web.
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 01/24/2018
ms.reviewer: mbullwin
ms.openlocfilehash: d11b12ba37d543ec21985c52c4ffb3399bfc56d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87323517"
---
# <a name="users-sessions-and-events-analysis-in-application-insights"></a>Analyse des utilisateurs, des sessions et des événements dans Application Insights

Découvrez quand des personnes utilisent votre application web, les pages qui les intéressent le plus, où vos utilisateurs se trouvent, ainsi que les navigateurs et les systèmes d’exploitation qu’ils utilisent. Analysez les données de télémétrie d’utilisation et d’activité à l’aide d’[Azure Application Insights](./app-insights-overview.md).

![Capture d’écran des utilisateurs d’Application Insights](./media/usage-segmentation/0001-users.png)

## <a name="get-started"></a>Bien démarrer

Si aucune donnée n’apparaît dans les panneaux des utilisateurs, des sessions ou des événements du portail Application Insights, [découvrez comment prendre en main les outils d’utilisation](usage-overview.md).

## <a name="the-users-sessions-and-events-segmentation-tool"></a>L’outil de segmentation Utilisateurs, Sessions et Événements

Trois des panneaux d’utilisation exploitent le même outil pour segmenter et traiter les données de télémétrie fournies par votre application web selon trois perspectives. En filtrant et en segmentant les données, vous pouvez découvrir des informations sur l’utilisation relative des différentes fonctionnalités et pages.

* **Outil Utilisateurs** : nombre de personnes ayant utilisé votre application et ses fonctionnalités.  Les utilisateurs sont comptabilisés à l’aide des ID anonymes stockés dans les cookies du navigateur. Une seule personne utilisant plusieurs navigateurs ou ordinateurs est comptabilisée comme plusieurs utilisateurs.
* **Outil Sessions** : nombre de sessions d’activité utilisateur ayant inclus certaines pages et fonctionnalités de votre application. Une session est comptabilisée après une demi-heure d’inactivité de l’utilisateur ou après 24 heures d’utilisation continue.
* **Outil Événements** : fréquence à laquelle certaines pages et fonctionnalités de votre application sont utilisées. L’affichage d’une page est comptabilisé lorsqu’un navigateur charge la page à partir de votre application, à condition que vous l’ayez [instrumentée](./javascript.md). 

    Un événement personnalisé représente une occurrence de quelque chose qui se produit dans votre application, souvent une interaction utilisateur, comme un clic sur un bouton ou l’achèvement d’une tâche. Vous insérez le code dans votre application pour [générer des événements personnalisés](./api-custom-events-metrics.md#trackevent).

## <a name="querying-for-certain-users"></a>Interrogation de certains utilisateurs

Explorez les différents groupes d’utilisateurs en ajustant les options d’interrogation en haut de l’outil Utilisateurs :

* Afficher : choisissez une cohorte d’utilisateurs à analyser.
* Qui a utilisé : choisissez les événements personnalisés et les affichages de pages.
* Pendant : choisissez un intervalle de temps.
* Par : choisissez comment compartimenter les données, soit par période temporelle, soit par une autre propriété telle que le navigateur ou la ville.
* Fractionner par : choisissez une propriété selon laquelle fractionner ou segmenter les données. 
* Ajouter des filtres : limitez la requête à certains utilisateurs, certaines sessions ou certains événements en fonction de leurs propriétés, telles que le navigateur ou la ville. 
 
## <a name="saving-and-sharing-reports"></a>Enregistrement et partage de rapports 
Vous pouvez enregistrer des rapports d’utilisateurs, privés (uniquement pour vous) dans la section Mes rapports, ou partagés avec tout le monde, avec un accès à cette ressource Application Insights dans la section Rapports partagés.

Pour partager un lien vers un rapport des utilisateurs, des sessions ou des événements ; cliquez sur **Partager** dans la barre d’outils, puis copiez le lien.

Pour partager une copie des données dans un rapport des utilisateurs, des sessions ou des événements ; cliquez sur **Partager** dans la barre d’outils, puis cliquez sur l’**icône Word** pour créer un document Word avec les données. Ou cliquez sur l’**icône Word** au-dessus du graphique principal.

## <a name="meet-your-users"></a>Découvrir vos utilisateurs

La section **Meet your users** (Découvrir vos utilisateurs) fournit des informations sur cinq exemples d’utilisateurs mis en correspondance par la requête actuelle. La prise en compte et l’analyse des comportements des personnes, en plus des agrégats, fournissent des informations sur la façon dont les personnes utilisent réellement votre application.

## <a name="next-steps"></a>Étapes suivantes

- Pour activer les expériences d’utilisation, commencez à envoyer des [événements personnalisés](./api-custom-events-metrics.md#trackevent) ou des [affichages de page](./api-custom-events-metrics.md#page-views).
- Si vous envoyez déjà des événements personnalisés ou des affichages de page, explorez les outils d’utilisation pour savoir comment les utilisateurs emploient votre service.
    - [Entonnoirs](usage-funnels.md)
    - [Rétention](usage-retention.md)
    - [Flux d’utilisateurs](usage-flows.md)
    - [Classeurs](../platform/workbooks-overview.md)
    - [Ajouter du contexte utilisateur](usage-send-user-context.md)

