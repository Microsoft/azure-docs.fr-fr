---
title: 'Procédure : Exporter une application Commandes personnalisées en tant que compétence distante - Service Speech'
titleSuffix: Azure Cognitive Services
description: Dans cet article, découvrez comment exporter une application Commandes personnalisées en tant que compétence
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/30/2020
ms.author: sausin
ms.openlocfilehash: 77ade17803a35491712ec6df70aed9eb7b4883eb
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025872"
---
# <a name="export-custom-commands-application-as-a-remote-skill"></a>Exporter une application Commandes personnalisées en tant que compétence distante

Dans cet article, découvrez comment exporter une application Commandes personnalisées en tant que compétence distante.

## <a name="prerequisites"></a>Prérequis
> [!div class="checklist"]
> * [Comprendre la compétence Bot Framework](/azure/bot-service/skills-conceptual)
> * [Comprendre le manifeste de compétence](https://aka.ms/speech/cc-skill-manifest)
> * [Guide pratique pour appeler une compétence à partir d’un bot Bot Framework](/azure/bot-service/skills-about-skill-consumers)
> * Une application Commandes personnalisées existante. Si vous n’avez pas d’application Commandes personnalisées, essayez le [démarrage rapide : Créer un assistant vocal à l’aide du kit Commandes personnalisées](quickstart-custom-commands-application.md)

## <a name="custom-commands-as-remote-skills"></a>Commandes personnalisées en tant que compétences distantes
* Les compétences Bot Framework sont des composants réutilisables de compétences conversationnelles. Elles couvrent les cas d’usage conversationnels et vous permettent d’ajouter des fonctionnalités étendues à un bot en quelques minutes. Pour en savoir plus, accédez à [Compétence Bot Framework](https://microsoft.github.io/botframework-solutions/overview/skills/).
* Une application Commandes personnalisées peut être exportée en tant que compétence. Cette compétence peut ensuite être appelée via le protocole Compétences distantes à partir d’un bot Bot Framework.

## <a name="configure-an-application-to-be-exposed-as-a-remote-skill"></a>Configurer une application pour qu’elle soit exposée en tant que compétence distante

### <a name="application-level-settings"></a>Paramètres au niveau de l’application
1. Dans le volet gauche, sélectionnez **Paramètres** > **Remote skills (Compétences distantes)** .
1. Activez le bouton bascule **Remote skills enabled (Compétences distantes activées)** .

### <a name="authentication-to-skills"></a>Authentification aux compétences
1. Si vous souhaitez activer l’authentification, ajoutez les ID d’application Microsoft des bots Bot Framework que vous souhaitez configurer pour appeler l’application Commandes personnalisées.
      > [!div class="mx-imgBorder"]
      > ![Ajouter un ID MSA à une compétence](media/custom-commands/skill-add-msa-id.png)

1. Si au moins une entrée est ajoutée à la liste, l’authentification est activée sur l’application, et seuls les bots autorisés peuvent appeler l’application.
> [!TIP]
>  Pour désactiver l’authentification, supprimez tous les ID d’application Microsoft de la liste des applications autorisées. 

 ### <a name="enabledisable-commands-to-be-exposed-as-skills"></a>Activer/désactiver les commandes à exposer en tant que compétences

Vous avez la possibilité de choisir les commandes que vous souhaitez exporter vers les compétences distantes.

1. Pour exposer une commande vers des compétences, sélectionnez **Enable a new command (Activer une nouvelle commande)** sous **Enable commands for skills (Activer les commandes pour les compétences)** .
1. Dans la liste déroulante, sélectionnez la commande que vous souhaitez ajouter.
1. Sélectionnez **Enregistrer**.

### <a name="configure-triggering-utterances-for-commands"></a>Configurer les énoncés de déclenchement des commandes
Le kit Commandes personnalisées utilise les exemples de phrases qui sont configurés pour les commandes afin de générer les énoncés de déclenchement des compétences. Ces **énoncés de déclenchement** sont utilisés pour générer la section **Répartiteur** du [**manifeste de compétence**](https://microsoft.github.io/botframework-solutions/skills/handbook/manifest/).

En tant qu’auteur, vous pouvez contrôler les **exemples de phrases** qui sont utilisés pour générer les énoncés de déclenchement des compétences.
1. Par défaut, tous les **exemples de déclenchement** d’une commande sont inclus dans le fichier manifeste.
1. Si vous souhaitez éliminer explicitement un exemple, sélectionnez l’**icône Modifier** dans la commande de la section **Enabled commands for skills (Commandes activées pour les compétences)** .
    > [!div class="mx-imgBorder"]
    > ![Modifier une commande activée pour une compétence](media/custom-commands/skill-edit-enabled-command.png)

1. Ensuite, sur les exemples de phrases que vous souhaitez omettre, **cliquez avec le bouton droit sur** > **Disable Example Sentence (Désactiver l’exemple de phrase)** .
    > [!div class="mx-imgBorder"]
    > ![Désactiver les exemples](media/custom-commands/skill-disable-example-sentences.png)

1. Sélectionnez **Enregistrer**.
1. Vous remarquerez que vous ne pouvez pas ajouter de nouvel exemple dans cette fenêtre. Si vous devez le faire, passez à la sortie de la section Paramètres et sélectionnez la commande appropriée dans l’accordéon **Commandes**. À ce stade, vous pouvez ajouter la nouvelle entrée dans la section **Example sentences (Exemples de phrases)** . Cette modification est automatiquement reflétée dans la valeur des paramètres de compétences distantes pour la commande.

> [!IMPORTANT]
> Si vos exemples de phrases existants comportent des références au type de données **Chaîne > Catalogue**, ces phrases sont automatiquement omises de la liste des énoncés de déclenchement des compétences. 

## <a name="download-skill-manifest"></a>Télécharger le manifeste de compétence
1. Une fois que vous avez **publié** votre application, vous pouvez télécharger le fichier manifeste de compétence.
1. Utilisez le manifeste de compétence pour configurer votre bot Bot Framework afin qu’il appelle la compétence Commandes personnalisées.
> [!IMPORTANT]
> Vous devez **publier** votre application Commandes personnalisées afin de télécharger le manifeste de compétence. </br>
> En outre, si vous avez apporté **des modifications** à l’application, vous devez republier l’application pour que les dernières modifications soient reflétées dans le fichier manifeste.

> [!NOTE]
> Si vous rencontrez des problèmes lors de la publication de l’application et que l’erreur dirige vers les énoncés de déclenchement des compétences, revérifiez la configuration de la section **Enabled commands for skills (Commandes activées pour les compétences)** . Chacune des commandes exposées doit avoir au moins un énoncé de déclenchement valide.


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Mettre à jour une commande à partir du client](./how-to-custom-commands-update-command-from-client.md)
