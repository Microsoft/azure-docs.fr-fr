---
title: Passer en revue les demandes de tirage (pull request) dans les environnements de préproduction dans Azure Static Web Apps
description: Découvrez comment utiliser les environnements de pré-production pour passer en revue les modifications apportées aux requêtes de tirage (pull request) dans Azure static Web Apps.
services: static-web-apps
author: sinedied
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: yolasors
ms.openlocfilehash: f8f2e352ae458e3e2825c9701437ea652ba07375
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91825666"
---
# <a name="review-pull-requests-in-pre-production-environments-in-azure-static-web-apps-preview"></a>Passer en revue les demandes de tirage (pull request) dans les environnements de préproduction dans Azure Static Web Apps (préversion)

Cet article explique comment utiliser les environnements de pré-production pour examiner les modifications apportées aux applications déployées avec [Azure Static Web Apps](overview.md).

Un environnement de préproduction (intermédiaire) est une version indexée entièrement fonctionnelle de votre application, qui comprend des modifications non disponibles en production.

Azure Static Web Apps génère un workflow GitHub Actions dans le référentiel. Quand une demande de tirage (pull request) est créée sur une branche surveillée par le workflow, l’environnement de préproduction est généré. L’environnement de préproduction prépare l’application et vous permet d’effectuer des révisions avant de passer en production.

Plusieurs environnements de préproduction peuvent coexister lors de l’utilisation d’Azure Static Web Apps. Chaque fois que vous créez une demande de tirage (pull request) sur la branche surveillée, une version indexée avec vos modifications est déployée dans un environnement de préproduction distinct.

L’utilisation d’environnements de préproduction présente de nombreux avantages. Vous pouvez par exemple :

- Passez en revue les modifications visuelles entre la phase de production et la phase intermédiaire. Par exemple, en consultant les mises à jour du contenu et de la mise en page.
- Montrez les modifications apportées à votre équipe.
- Comparez les différentes versions de votre application.
- Validez les modifications à l’aide des tests d’acceptation.
- Effectuez des vérifications d’intégrité avant le déploiement en production.

> [!NOTE]
> Pendant la préversion, seul un [environnement intermédiaire](quotas.md) est autorisé à la fois.

## <a name="prerequisites"></a>Prérequis

- Un référentiel GitHub existant configuré avec Azure Static Web Apps. Consultez le guide de démarrage rapide [Générer votre première application web statique](getting-started.md) si vous ne l’avez pas déjà fait.

## <a name="make-a-change"></a>Apporter une modification

Commencez par modifier votre référentiel. Vous pouvez le faire directement sur GitHub comme indiqué dans les étapes suivantes.

1. Accédez au référentiel de votre projet sur GitHub, puis cliquez sur le bouton **Branch** (Branche) pour créer une branche.

    :::image type="content" source="./media/review-publish-pull-requests/create-branch.png" alt-text="Créer une branche à l’aide de l’interface GitHub":::]

    Tapez un nom de branche, puis cliquez sur **Create branch**.

1. Accédez à votre dossier _app_ et modifiez du texte. Par exemple, vous pouvez modifier un titre ou un paragraphe. Lorsque vous avez trouvé le fichier que vous souhaitez modifier, cliquez sur **Edit** pour effectuer la modification.

    :::image type="content" source="./media/review-publish-pull-requests/edit-file.png" alt-text="Créer une branche à l’aide de l’interface GitHub":::

1. Une fois votre modification effectuée, cliquez sur **Commit changes** pour valider les modifications apportées à la branche.

    :::image type="content" source="./media/review-publish-pull-requests/commit-changes.png" alt-text="Créer une branche à l’aide de l’interface GitHub":::

## <a name="create-a-pull-request"></a>Créer une demande de tirage (pull request)

Ensuite, créez une requête de tirage (pull request) à partir de cette modification.

1. Dans votre projet GitHub, ouvrez l’onglet **Pull request** (Demande de tirage).

    :::image type="content" source="./media/review-publish-pull-requests/tab.png" alt-text="Créer une branche à l’aide de l’interface GitHub":::

1. Cliquez sur le bouton **Compare & pull request** (Comparer et demander un tirage) de votre branche.

1. Vous pouvez éventuellement entrer des informations sur vos modifications, puis cliquer sur **Create pull request** (Créer une demande de tirage).

    :::image type="content" source="./media/review-publish-pull-requests/open.png" alt-text="Créer une branche à l’aide de l’interface GitHub":::

Vous pouvez affecter des réviseurs et ajouter des commentaires pour discuter de vos modifications si nécessaire.

> [!NOTE]
> Vous pouvez apporter plusieurs modifications en transmettant de nouvelles validations à votre branche. La demande de tirage est ensuite automatiquement mise à jour pour refléter toutes les modifications.

## <a name="review-changes"></a>Examiner les modifications

Une fois la demande de tirage créée, le workflow de déploiement [GitHub Actions](https://github.com/features/actions) s’exécute et déploie vos modifications dans un environnement de préproduction.

Lorsque le workflow a terminé la génération et le déploiement de votre application, le bot GitHub ajoute un commentaire à votre demande de tirage contenant l’URL de l’environnement de préproduction. Vous pouvez cliquer sur ce lien pour afficher vos modifications indexées.

:::image type="content" source="./media/review-publish-pull-requests/bot-comment.png" alt-text="Créer une branche à l’aide de l’interface GitHub":::

Cliquez sur l’URL générée pour voir les modifications.

Si vous examinez de plus près l’URL, vous pouvez voir qu’elle se compose comme ceci : `https://<SUBDOMAIN-PULL_REQUEST_ID>.<AZURE_REGION>.azurestaticapps.net`.

L’URL d’une demande de tirage ne change pas, même si vous envoyez de nouvelles mises à jour. De même, l’environnement de préproduction est réutilisé pendant toute la durée de vie de la demande de tirage.

## <a name="publish-changes"></a>Publier les modifications

Une fois les modifications approuvées, vous pouvez publier vos modifications en production en fusionnant la demande de tirage.

Cliquez sur **Merge pull request** (Fusionner la demande de tirage) :

:::image type="content" source="./media/review-publish-pull-requests/merge.png" alt-text="Créer une branche à l’aide de l’interface GitHub":::

La fusion copie vos modifications dans la branche suivie (la branche de « production »). Ensuite, le workflow de déploiement démarre sur la branche suivie et les modifications sont publiées après la nouvelle génération de votre application.

Pour vérifier les modifications dans votre environnement de production, ouvrez votre URL de production afin de charger la version dynamique du site web.

## <a name="limitations"></a>Limites

Les versions indexées de votre application sont actuellement accessibles publiquement via leur URL, même si votre référentiel GitHub est privé.

> [!WARNING]
> Soyez prudent lors de la publication de contenu sensible dans des versions indexées, car l’accès à des environnements de préproduction n’est pas restreint.

Le nombre d’environnements de préproduction disponibles pour chaque application déployée avec Static Web Apps dépend du niveau de référence SKU que vous utilisez. Par exemple, avec le niveau Gratuit, vous pouvez avoir 1 environnement de préproduction en plus de l’environnement de production.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configurer un domaine personnalisé](custom-domain.md)
