---
title: Configurer les paramètres de l’Outil de révision – Content Moderator
titleSuffix: Azure Cognitive Services
description: Utilisez l’Outil de révision pour configurer ou récupérer votre équipe, vos tags, vos connecteurs, vos flux de travail et vos informations d’identification Content Moderator.
services: cognitive-services
author: PatrickFarley
manager: mikemcca
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/15/2019
ms.author: pafarley
ms.openlocfilehash: 8812fd1e6c1efb2aa44c77573bc4b8f1c099834d
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92912019"
---
# <a name="configure-the-review-tool"></a>Configurer l’outil de révision

[L’Outil de révision](https://contentmoderator.cognitive.microsoft.com) comporte d’importantes fonctionnalités, accessibles dans le menu **Paramètres** du tableau de bord.

![Menu Paramètres de l’Outil de révision Content Moderator](images/settings-1.png)

## <a name="manage-team-and-subteams"></a>Gérer les équipes et les sous-équipes

L’onglet **Équipe** vous permet de gérer votre équipe et vos sous-équipes, c’est-à-dire des groupes d’utilisateurs qui peuvent être informés lorsque certaines [révisions humaines](../review-api.md#reviews) commencent. On ne peut avoir qu’une seule équipe (créée lors de l’inscription à l’Outil de révision), mais il est possible de créer plusieurs sous-équipes. L’administrateur d’équipe peut inviter des membres, définir leurs autorisations et les affecter à différentes sous-équipes.

![Paramètres d’équipe de l’Outil de révision](images/settings-2-team.png)

Les sous-équipes permettent de créer des équipes d’escalade ou des équipes dédiées à la révision de catégories de contenu spécifiques. Vous pouvez par exemple envoyer du contenu pour adultes à une équipe distincte en vue d’une révision approfondie.

Cette section explique comment créer des sous-équipes et leur affecter des révisions à la volée. Toutefois, vous pouvez utiliser les [flux de travail](workflows.md) pour affecter les révisions en fonction de critères spécifiques.

### <a name="create-a-subteam"></a>Créer une sous-équipe

Accédez à la section **Sous-équipes** et cliquez sur **Ajouter une sous-équipe**. Entrez le nom de votre sous-équipe dans la boîte de dialogue, puis cliquez sur **Enregistrer**.

![Nom de la sous-équipe](images/1-Teams-2.PNG)

#### <a name="invite-teammates"></a>Inviter des collègues

Il n’est pas possible d’affecter à une sous-équipe un réviseur qui ne ferait pas partie de l’équipe par défaut. Par conséquent, commencez par tous les ajouter à l’équipe par défaut. Cliquez sur **Inviter** dans l’onglet **Équipe**.

![Inviter des utilisateurs](images/invite-users.png)

#### <a name="assign-teammates-to-subteam"></a>Affecter des collègues à une sous-équipe

Cliquez sur le bouton **Ajouter un membre** pour affecter des membres de votre équipe par défaut à une ou plusieurs sous-équipes. Vous pouvez uniquement ajouter des utilisateurs existants à une sous-équipe. Pour ajouter des utilisateurs qui ne font pas encore partie de l’outil de révision, vous devez les inviter à l’aide du bouton Inviter sur la page Paramètres de l’équipe.

![Assigner des membres à la sous-équipe](images/1-Teams-3.PNG)

### <a name="assign-reviews-to-subteams"></a>Attribuer des révisions à des sous-équipes

Une fois que vous avez créé vos sous-équipes et que vous y avez affecté des membres, vous pouvez commencer à leur attribuer des [révisions](../review-api.md#reviews) de contenu, dans l’onglet **Révision** du site.
Pour attribuer du contenu à une sous-équipe, cliquez sur les points de suspension en haut à droite, sélectionnez **Déplacer vers** et choisissez une sous-équipe.

![Assigner la révision d’une image à une sous-équipe](images/3-review-image-subteam-1.png)

### <a name="switch-between-subteams"></a>Basculer entre les sous-équipes

Si vous êtes membre de plusieurs sous-équipes, vous pouvez passer de l’une à l’autre pour voir vos différentes révisions de contenu. Dans l’onglet **Révision** , sélectionnez le menu déroulant intitulé **Par défaut** et sélectionnez **Choisir une sous-équipe**. Il est possible d’afficher les révisions de contenu de différentes sous-équipes à condition d’en être membre.

![Basculer entre les sous-équipes](images/3-review-image-subteam-2.png)

## <a name="tags"></a>Balises

L’onglet **Tags** permet de définir des tags de modération personnalisés en plus des deux tags de modération par défaut, à savoir **isadult** ( **a** ) et **isracy** ( **r** ). Une fois créé, un tag personnalisé devient accessible dans les révisions au même titre que les tags par défaut. Pour choisir quels tags apparaissent, modifiez leurs paramètres de visibilité.

![Affichage des tags, avec les cases à cocher « Est visible »](images/tags-4-disable.png)

### <a name="create-custom-tags"></a>Créer des tags personnalisés

Pour créer un nouveau tag, il faut entrer un code court, un nom et une description dans leurs champs respectifs.

- **Code court** : code à deux lettres du tag. Exemple : **cb**.
- **Name**  : nom court et descriptif du tag en minuscules, sans espaces. Exemple : **isbullying**.
- **Description** : (facultatif) description du type de contenu ciblé par votre tag. Exemple : **Description de cas de cyberharcèlement**.

Cliquez sur **Ajouter** pour ajouter un tag, puis sur **Enregistrer** lorsque vous avez terminé la création de tags.

![Boîte de dialogue Créer un tag de l’Outil de révision](images/settings-3-tags.png)

### <a name="delete-tags"></a>Supprimer des tags

Pour supprimer des tags personnalisés, sélectionnez l’icône de corbeille à côté de leurs entrées dans la liste Tags. Il n’est pas possible de supprimer les tags par défaut.

## <a name="connectors"></a>Connecteurs

L’onglet **Connecteurs** permet de gérer les connecteurs, c’est-à-dire des plug-ins propres au service capables de traiter le contenu de différentes manières dans le cadre de [flux de travail](../review-api.md#workflows) de contenu.

Le connecteur par défaut lors de la création d’un flux de travail est le connecteur Content Moderator, qui peut marquer le contenu comme **adult** ou **racy** , recherchez des grossièretés, etc. Cependant, vous pouvez utiliser d’autres connecteurs (listés ci-dessous) du moment que vous disposez d’informations d’identification pour leurs services respectifs (par exemple, une clé d’abonnement [Visage](../../face/overview.md) est nécessaire pour pouvoir utiliser le connecteur Visage).

[L’Outil de révision](./human-in-the-loop.md) comprend les connecteurs suivants :

- Emotion
- Face
- Service cloud PhotoDNA
- Analyse de texte

### <a name="add-a-connector"></a>Ajouter un connecteur

Pour ajouter un connecteur (et le rendre utilisable dans les [flux de travail](../review-api.md#workflows) de contenu), sélectionnez le bouton **Se connecter** correspondant. Dans la boîte de dialogue suivante, entrez votre clé d’abonnement pour ce service. Votre nouveau connecteur devrait alors apparaître en haut de la page.

![Paramètres des connecteurs Content Moderator](images/settings-4-connectors.png)

## <a name="workflows"></a>Workflows

L’onglet **Flux de travail** permet de gérer les [flux de travail](../review-api.md#workflows). Il s’agit de filtres cloud de contenus, qui fonctionnent avec des connecteurs pour trier le contenu de différentes façons et effectuer les actions nécessaires. C’est ici que sont définis, modifiés et testés les flux de travail. Pour obtenir des conseils sur la procédure à suivre, voir [Définir et utiliser des flux de travail](Workflows.md).

![Paramètres du flux de travail Content Moderator](images/settings-5-workflows.png)

## <a name="credentials"></a>Informations d'identification

L’onglet **Informations d’identification** offre un accès rapide à la clé d’abonnement Content Moderator, nécessaire pour accéder aux services de modération à partir d’un appel REST ou d’un kit SDK client.

![Informations d’identification Content Moderator](images/settings-6-credentials.png)

### <a name="use-external-credentials-for-workflows"></a>Utiliser des informations d’identification externes pour les flux de travail

[L’Outil de révision](https://contentmoderator.cognitive.microsoft.com) génère une clé d’essai gratuit pour les services Azure Content Moderator lors de l’inscription. Cependant, vous pouvez également la configurer de façon à utiliser une clé existante de votre compte Azure, ce qui est recommandé pour les scénarios à grande échelle ; en effet, les clés d’essai gratuit ont des limites d’utilisation strictes ([Tarifs et limites](https://azure.microsoft.com/pricing/details/cognitive-services/content-moderator/)).

Si vous avez créé une [ressource Content Moderator](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator) dans Azure, accédez-y sur le Portail Azure et sélectionnez le panneau **Clés**. Copiez une de vos clés.

![Clés Content Moderator du portail Azure](images/credentials-azure-portal-keys.PNG)

Dans l’onglet **Informations d’identification** de [l’Outil de révision](https://contentmoderator.cognitive.microsoft.com), accédez au volet **Paramètres des flux de travail** , sélectionnez **Modifier** et collez votre clé dans le champ **Ocp-Apim-Subscription-Key**. Les flux de travail qui appellent les API de modération utiliseront vos informations d’identification Azure.

> [!NOTE]
> Les deux autres champs du volet **Paramètres des flux de travail** sont destinés aux listes de termes et d’images personnalisés. Pour plus d’informations à ce sujet, voir les guides [Termes personnalisés](../try-terms-list-api.md) et [Images personnalisées](../try-image-list-api.md).

### <a name="use-your-azure-account-with-the-review-apis"></a>Utiliser un compte Azure avec les API de révision

Pour utiliser votre clé Azure avec les API de révision, vous devez récupérer votre ID de ressource. Accédez à votre ressource Content Moderator sur le Portail Azure et sélectionnez le panneau **Propriétés**. Copiez la valeur ID de la ressource et collez-la dans le champ **ID de ressource(s) sur liste verte** de l’onglet **Informations d’identification** de l’Outil de révision.

![ID de ressource Content Moderator dans le portail Azure](images/credentials-azure-portal-resourceid.PNG)

Si vous avez entré votre clé d’abonnement aux deux endroits, la clé d’essai associée à votre compte Outil de révision ne sera pas utilisée, mais restera disponible.

## <a name="next-steps"></a>Étapes suivantes

Suivez le [Démarrage rapide Outil de révision](../quick-start.md) pour commencer à utiliser l’Outil de révision dans des scénarios de modération de contenu.