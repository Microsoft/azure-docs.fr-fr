---
title: Guide pratique pour envoyer des événements d’Auth0 vers Azure à l’aide d’Azure Event Grid
description: Comme envoyer des événements d’Auth0 vers des services Azure avec Azure Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: a66a60cb926b933a6b0628a67506d0d52ab7a905
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93077861"
---
# <a name="integrate-azure-event-grid-with-auth0"></a>Intégrer Azure Event Grid à Auth0

Cet article explique comment connecter vos comptes Auth0 et Azure en créant une rubrique de partenaire Event Grid.

Pour obtenir la liste complète des événements pris en charge par Auth0, consultez le [codes de type d’événement Auth0](https://auth0.com/docs/logs/references/log-event-type-codes)

## <a name="send-events-from-auth0-to-azure-event-grid"></a>Envoyer des événements d’Auth0 vers Azure Event Grid
Pour envoyer des événements Auth0 à Azure :

1. Activer le fournisseur de ressources Event Grid
1. Configurer une rubrique de partenaire Auth0 dans le tableau de bord Auth0
1. Activer la rubrique de partenaire dans Azure
1. S’abonner aux événements à partir d’Auth0

Pour plus d’informations sur ces concepts, consultez [Concepts Event Grid](concepts.md).

### <a name="enable-event-grid-resource-provider"></a>Activer le fournisseur de ressources Event Grid
Si vous n’avez pas déjà utilisé Event Grid auparavant, vous devez inscrire le fournisseur de ressources Event Grid. Si vous avez déjà utilisé Event Grid, passez à la section suivante.

Sur le portail Azure :
1. Sélectionner Abonnements dans le menu de gauche
1. Sélectionner l’abonnement que vous utilisez pour Event Grid
1. Dans le menu de gauche, sous Paramètres, sélectionner Fournisseurs de ressources
1. Rechercher Microsoft.EventGrid
1. Sélectionner Inscrire
1. Actualiser pour vérifier que l’état passe à Inscrit

### <a name="set-up-an-auth0-partner-topic"></a>Configurer une rubrique de partenaire Auth0
Une partie du processus d’intégration consiste à configurer Auth0 pour en faire une source d’événements (cette étape se déroule dans votre [tableau de bord Auth0](https://manage.auth0.com/)).

1. Connectez-vous au [tableau de bord Auth0](https://manage.auth0.com/).
1. Accédez à Logs (Journaux) > Streams (Flux).
1. Cliquez sur + Create Stream (Créer un flux).
1. Sélectionnez Azure Event Grid et attribuez un nom unique à votre nouveau flux.
1. Créez la source d’événements en indiquant votre ID d’abonnement Azure, la région Azure et un nom de groupe de ressources. 
1. Cliquez sur Save (Enregistrer).

### <a name="activate-your-auth0-partner-topic-in-azure"></a>Activer votre rubrique de partenaire Auth0 dans Azure
L’activation de la rubrique Auth0 dans Azure permet l’afflux des événements d’Auth0 vers Azure.

1. Connectez-vous au portail Azure.
1. Recherchez `Partner Topics` dans la partie supérieure de la page et cliquez sur `Event Grid Partner Topics` sous services.
1. Cliquez sur la rubrique qui correspond au flux que vous avez créé dans votre tableau de bord Auth0.
1. Vérifiez que la valeur du champ `Source` correspond à votre compte Auth0.
1. Cliquez sur Activer.

### <a name="subscribe-to-auth0-events"></a>S’abonner aux événements Auth0

#### <a name="create-an-event-handler"></a>Créer un gestionnaire d’événements
Pour tester votre rubrique de partenaire, vous avez besoin d’un gestionnaire d’événements. Accédez à votre abonnement Azure et lancez un service pris en charge en tant que [gestionnaire d’événements](event-handlers.md), par exemple [Azure Function](custom-event-to-function.md).

#### <a name="subscribe-to-your-auth0-partner-topic"></a>S’abonner à la rubrique de partenaire Auth0
En vous abonnant à votre rubrique de partenaire Auth0, vous pouvez indiquer à Event Grid la destination de vos événements Auth0.

1. Dans le panneau de la rubrique de partenaire de votre intégration Auth0, sélectionnez + Abonnement aux événements en haut.
1. Dans la page Créer un abonnement aux événements :
    1. Entrez un nom pour l’abonnement aux événements.
    1. Sélectionnez le service Azure ou le webhook que vous avez créé pour le type Point de terminaison.
    1. Suivez les instructions pour le service en question.
    1. Cliquez sur Créer.

## <a name="testing"></a>Test
L’intégration de votre rubrique de partenaire Auth0 à Azure doit être prête à être utilisée.

### <a name="verify-the-integration"></a>Vérifier l’intégration
Pour vérifiez que l’intégration fonctionne comme prévu :

1. Connectez-vous au tableau de bord Auth0.
1. Accédez à Logs (Journaux) > Streams (Flux).
1. Cliquez sur votre flux Event Grid.
1. Une fois sur le flux, cliquez sur l’onglet Health (Intégrité). Normalement le flux est actif, et si vous ne voyez pas d’erreurs, c’est que le flux fonctionne.

Essayez d’[appeler une des actions Auth0 qui déclenchent la publication d’un événement](https://auth0.com/docs/logs/references/log-event-type-codes) pour voir le flux d’événements.

## <a name="delivery-attempts-and-retries"></a>Tentatives de remise et nouvelles tentatives
Les événements Auth0 sont remis à Azure via un mécanisme de streaming. Chaque événement est envoyé aussitôt qu’il est déclenché dans Auth0. Si Event Grid ne peut pas recevoir l’événement, Auth0 tente de remettre l’événement à trois reprises, si nécessaire. S’il n’y parvient pas, Auth0 journalise l’échec de la remise dans son système.

## <a name="next-steps"></a>Étapes suivantes

- [Rubrique de partenaire Auth0](auth0-overview.md)
- [Vue d’ensemble des rubriques de partenaire](partner-events-overview.md)
- [Devenir partenaire Event Grid](partner-onboarding-overview.md)