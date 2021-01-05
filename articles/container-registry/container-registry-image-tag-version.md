---
title: Bonnes pratiques relatives aux étiquettes d’image
description: Bonnes pratiques pour l’étiquetage et le versioning des images conteneur Docker lors de l’envoi (push) et du tirage (pull) d’images vers et depuis un registre de conteneurs Azure
author: stevelasker
ms.topic: article
ms.date: 07/10/2019
ms.author: stevelas
ms.openlocfilehash: b483317960409fe1fbea181706f12375606fe659
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75445750"
---
# <a name="recommendations-for-tagging-and-versioning-container-images"></a>Suggestions pour la création de balises et de versions pour les images de conteneurs

Lorsque vous déplacez des images de conteneurs vers un registre et que vous les déployez ensuite, vous avez besoin d’une stratégie pour la création de balises et de versions des images. Cet article traite de deux approches et de la place de chacune d’elles dans le cycle de vie des conteneurs :

* **Balises stables :** balises que vous réutilisez, par exemple, pour indiquer une version majeure ou mineure telle que *mycontainerimage:1.0*.
* **Balises uniques :** autre balise pour chaque image que vous envoyez vers un registre, tel que *mycontainerimage:abc123*.

## <a name="stable-tags"></a>Balises stables

**Recommandation** : Utilisez des balises stables pour maintenir des **images de base** pour la génération de vos conteneurs. Évitez les déploiements avec des balises stables, car ces balises continuent de recevoir des mises à jour et peuvent introduire des incohérences dans les environnements de production.

*Les balises stables* signifient qu’un développeur, ou un système de génération, peut continuer à extraire une balise spécifique, qui continue à recevoir des mises à jour. Stable ne signifie pas que le contenu est figé. Au contraire, le terme stable implique que l’image doit être stabilisée pour les besoins de cette version. Pour rester « stable », il est possible d’appliquer des correctifs de sécurité ou des mises à jour de l’infrastructure.

### <a name="example"></a>Exemple

Une équipe d’infrastructure distribue la version 1.0. Ils savent qu’ils enverront des mises à jour, même celles qui sont mineures. Pour prendre en charge les balises stables pour une version majeure et mineure, ils ont deux ensembles de balises stables.

* `:1` – une balise stable pour la version majeure. `1` représente la version 1.* « la plus récente » ou « la dernière ».
* `:1.0` – une balise stable pour la version 1.0, permettant à un développeur de se lier aux mises à jour de la 1.0, et de ne pas la restaurer par progression en 1.1 quand cette version sera disponible.

L’équipe utilise également la balise `:latest`, qui pointe vers la dernière balise stable ajoutée, quel que soit la version majeure actuelle.

Lorsque des mises à jour de l’image de base sont disponibles, ou tout type de version de maintenance de l’infrastructure, les images avec les balises stables sont mises à jour vers la version la plus récente qui représente la version stable la plus récente de cette version.

Dans ce cas, les balises majeures et mineures sont continuellement en cours de maintenance. A partir d’un scénario d’image de base, cela permet au propriétaire de l’image de fournir des images dont la maintenance a été effectuée.

### <a name="delete-untagged-manifests"></a>Supprimer les manifestes sans balise

Si une image avec une balise stable est mise à jour, l’image balisée précédemment n’est pas marquée, ce qui entraîne une image orpheline. Le manifeste de l’image précédente et les données de couche uniques restent dans le registre. Pour maintenir la taille de votre registre, vous pouvez régulièrement supprimer les manifestes sans balise résultant de mises à jour d’image stables. Par exemple, [purgez automatiquement](container-registry-auto-purge.md) les manifestes non balisés antérieurs à une durée spécifiée, ou définissez une [stratégie de rétention](container-registry-retention-policy.md) pour les manifestes sans balise.

## <a name="unique-tags"></a>Balises uniques

**Recommandation** : Utilisez des balises uniques pour les **déploiements**, en particulier dans un environnement pouvant évoluer sur plusieurs nœuds. Vous préférerez certainement des déploiements délibérées d’une version cohérente de composants. Si votre conteneur redémarre ou si un orchestrateur fait monter en charge plus d’instances, vos hôtes n’extraireront pas accidentellement une version plus récente, incompatible avec les autres nœuds.

Le balisage unique signifie simplement que chaque image poussée vers un registre possède une balise unique. Les balises ne sont pas réutilisés. Il existe plusieurs modèles que vous pouvez suivre pour générer des balises uniques, y compris :

* **Horodatage :** cette approche est assez courante, dans la mesure où vous pouvez indiquer clairement quand l’image a été créée. Mais, comment le mettre en corrélation avec votre système de génération ? Est-il nécessaire de trouver la génération qui a été achevée en même temps ? Dans quel fuseau horaire êtes-vous ? Est-ce que tous vos systèmes de génération sont étalonnés au temps universel coordonné ?
* **Validation Git :** cette approche fonctionne jusqu’à ce que vous commenciez à prendre en charge les mises à jour de l’image de base. Si une mise à jour de l’image de base se produit, votre système de compilation démarre avec la même validation Git que la version précédente. Toutefois, l’image de base a un nouveau contenu. En règle générale, une validation Git fournit une balise *semi* stable.
* **Manifeste de code de hachage :** chaque image de conteneur envoyée à un registre est associée à un manifeste, identifié par un hachage ou un code de hachage SHA-256 unique. Bien qu’unique, le code de hachage est long et difficile à lire sans corrélation avec votre environnement de génération.
* **ID de génération :** cette option peut être la meilleure puisqu’elle est probablement incrémentale et qu’elle vous permet d’établir une corrélation avec la compilation spécifique pour trouver tous les artefacts et les journaux. Toutefois, comme un manifeste de code de hachage, il peut être difficile pour un humain de le lire.

  Si votre organisation possède plusieurs systèmes de génération, préfixer l’étiquette avec le nom du système est une variante de cette option :`<build-system>-<build-id>`. Par exemple, vous pourrez différencier les générations de système de génération Jenkins de l’équipe API et de celui d’Azure Pipelines de l’équipe Web.

### <a name="lock-deployed-image-tags"></a>Verrouiller les balises d’image déployées

Il est recommandé de [verrouiller](container-registry-image-lock.md) toute balise d’image déployée en définissant son attribut `write-enabled` sur `false`. Cette pratique vous empêche de supprimer par inadvertance une image du registre et d’interrompre potentiellement vos déploiements. Vous pouvez inclure l’étape de verrouillage dans votre pipeline de mise en production.

Le verrouillage d’une image déployée vous permet de supprimer les autres images, non déployées, de votre registre à l’aide des fonctionnalités d’Azure Container Registry pour gérer votre registre. Par exemple, [purgez automatiquement](container-registry-auto-purge.md) les images non verrouillées ou les manifestes non balisés antérieurs à une durée spécifiée, ou définissez une [stratégie de rétention](container-registry-retention-policy.md) pour les manifestes sans balise.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une présentation plus détaillée des concepts décrits dans cet article, consultez le blog [Balisage Docker : Meilleures pratiques pour la création de balises et de versions des images docker](https://stevelasker.blog/2018/03/01/docker-tagging-best-practices-for-tagging-and-versioning-docker-images/).

Pour aider à optimiser les performances et la rentabilité d’utilisation de votre registre de conteneurs Azure, consultez les [Meilleures pratiques pour Azure Container Registry](container-registry-best-practices.md).

<!-- IMAGES -->


<!-- LINKS - Internal -->

