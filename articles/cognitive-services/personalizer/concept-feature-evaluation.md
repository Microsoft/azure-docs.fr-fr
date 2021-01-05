---
title: Évaluation des fonctionnalités - Personalizer
titleSuffix: Azure Cognitive Services
description: Lorsque vous exécutez une évaluation dans votre ressource Personalizer à partir du portail Azure, Personalizer fournit des informations sur les fonctionnalités de contexte et les actions qui influencent le modèle.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 07/29/2019
ms.openlocfilehash: c0e47a2943cf8c934d201f76aefc41868adf0b25
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87127721"
---
# <a name="feature-evaluation"></a>Évaluation des fonctionnalités

Lorsque vous exécutez une évaluation dans votre ressource Personalizer à partir du [portail Azure](https://portal.azure.com), Personalizer fournit des informations sur les fonctionnalités de contexte et les actions qui influencent le modèle. 

Il permet notamment ce qui suit :

* Imaginer les fonctionnalités supplémentaires que vous pourriez utiliser, vous inspirer des fonctionnalités importantes du modèle.
* Identifier les fonctionnalités non importantes pour éventuellement les supprimer ou analyser plus en détail ce qui peut affecter l’utilisation.
* Fournir des conseils aux équipes éditoriales ou chargées de la collecte sur les nouveaux contenus ou produits méritant de figurer dans le catalogue.
* Résoudre les erreurs et les problèmes courants qui surviennent lors de l’envoi de fonctionnalités à Personalizer.

Les fonctionnalités les plus importantes ont plus de poids dans le modèle. Ces fonctionnalités ayant plus de poids, elles tendent à être présentes lorsque Personalizer obtient des récompenses plus élevées.

## <a name="getting-feature-importance-evaluation"></a>Obtenir une évaluation des fonctionnalités en termes d'importance

Pour afficher les résultats des fonctionnalités en termes d'importance, vous devez exécuter une évaluation. L’évaluation crée des étiquettes de fonctionnalités lisibles en fonction basées sur les noms des fonctionnalités observées lors de la période d’évaluation.

Les informations qui en résultent sur l’importance des fonctionnalités représentent le modèle en ligne Personalizer actuel. L’évaluation analyse l’importance de la fonctionnalité du modèle enregistré à la date de fin de la période d’évaluation, après l’apprentissage effectué pendant l’évaluation, avec la stratégie d’apprentissage en ligne actuelle. 

Les résultats liés à l'importance des fonctionnalités ne représentent pas les autres stratégies et modèles testés ou créés lors de l’évaluation.  L’évaluation n’inclut pas les fonctionnalités envoyées à Personalizer au terme de la période d’évaluation.

## <a name="how-to-interpret-the-feature-importance-evaluation"></a>Interpréter l'évaluation des fonctionnalités en termes d'importance

Personalizer évalue les fonctionnalités en créant des « groupes » de fonctionnalités de même importance. Un groupe peut passer pour globalement plus important que les autres, mais au sein du groupe, les fonctionnalités sont triées par ordre alphabétique.

Les informations relatives à chaque fonctionnalité sont les suivantes :

* Fonctionnalité issue du contexte ou des actions.
* Clé et valeur de la fonctionnalité.

Par exemple, une application de tri pour un glacier peut présenter une fonctionnalité « Context.Weather:Hot » comme très importante.

Personalizer affiche les corrélations des fonctionnalités qui, prises en compte conjointement, génèrent des récompenses plus élevées.

Par exemple, vous pouvez voir « Context.Weather:Hot *avec* Action.MenuItem:IceCream » ainsi que "Context.Weather:Cold *avec* Action.MenuItem:WarmTea:

## <a name="actions-you-can-take-based-on-feature-evaluation"></a>Actions que vous pouvez effectuer en fonction de l’évaluation des fonctionnalités

### <a name="imagine-additional-features-you-could-use"></a>Imaginer les fonctionnalités supplémentaires que vous pourriez utiliser

Inspirez-vous des fonctionnalités importantes du modèle. Par exemple, si vous voyez « Context.MobileBattery:Low » dans une application mobile vidéo, vous pouvez penser que le type de connexion peut également permettre aux clients de choisir d'afficher un clip vidéo sur un autre, puis d’ajouter des fonctionnalités sur le type de connectivité et la bande passante à votre application.

### <a name="see-what-features-are-not-important"></a>Identifier les fonctionnalités non importantes

Vous pouvez supprimer les fonctionnalités non importantes ou analyser plus en détail ce qui peut affecter l’utilisation. Certaines fonctionnalités peuvent être mal classées pour de nombreuses raisons. Il peut s’agir du fait que la fonctionnalité n’affecte en rien le comportement de l’utilisateur. Mais cela peut également signifier que la fonctionnalité n’est pas visible pour l’utilisateur. 

Par exemple, un site vidéo pourrait voir que « Action.VideoResolution=4k » est une fonctionnalité de faible importance, en contradiction avec la recherche de l'utilisateur. Cela peut être dû au fait que l’application n’indique ou n'affiche pas la résolution vidéo, et que dès lors, les utilisateurs ne peuvent modifier leur comportement en conséquence.

### <a name="provide-guidance-to-editorial-or-curation-teams"></a>Fournir des conseils aux équipes éditoriales ou chargées de la collecte

Fournissez des conseils sur les nouveaux contenus ou produits méritant de figurer dans le catalogue. Personalizer est conçu pour améliorer les connaissances humaines et aider les équipes. Pour ce faire, il propose aux groupes éditoriaux des informations sur les produits, les articles ou le contenu qui pilotent le comportement. Par exemple, le scénario d’application vidéo peut montrer qu’il existe une fonctionnalité importante appelée « Action.VideoEntities.Cat:true », invitant l’équipe éditoriale à intégrer plus de vidéos de chats.

### <a name="troubleshoot-common-problems-and-mistakes"></a>Résoudre les erreurs et problèmes courants

Les erreurs et problèmes courants peuvent être résolus en modifiant le code de votre application de sorte qu’il n’envoie pas de fonctionnalités inappropriées ou incorrectement mises en forme à Personalizer. 

Les erreurs courantes lors de l’envoi des fonctionnalités sont notamment les suivantes :

* Envoi d'informations d’identification personnelle (PII). Les informations d'identification personnelle spécifiques à une personne (nom, numéro de téléphone, numéros de carte de crédit, adresses IP, par exemple) ne doivent pas être utilisées avec Personalizer. Si votre application doit effectuer le suivi des utilisateurs, utilisez un UUID sans identification ou un autre numéro UserID. Dans la plupart des scénarios, cela pose également problème.
* Avec un grand nombre d’utilisateurs, il est peu probable que l’interaction de chaque utilisateur pèse plus que toute l'interaction d'analyse et ainsi, l'envoi des ID des utilisateurs (même s’il ne s'agit pas d'informations d'identification personnelle) ajoutera probablement plus de bruit que de valeur au modèle.
* Envoi de champs de date et d’heure en tant que timestamps précis plutôt que de valeurs d'heure caractérisées. Des fonctionnalités telles que Context.TimeStamp.Day=Monday ou « Context.TimeStamp.Hour"="13 » sont plus utiles. Il existe au plus 7 ou 24 valeurs de fonctionnalité pour chacune. Mais la fonctionnalité «Context. TimeStamp»: «1985-04-12T23:20:50.52 Z» est si précise qu’il ne sera pas possible d'en tirer des enseignements car elle ne se reproduira plus.

## <a name="next-steps"></a>Étapes suivantes

Comprendre l'[extensibilité et le niveau de performance](concepts-scalability-performance.md) avec Personalizer.

