---
title: Demander l’accès juste-à-temps
description: Décrit comment les éditeurs d’applications managées Azure demander l'accès juste à temps à une application managée.
author: MSEvanhi
ms.topic: conceptual
ms.date: 09/25/2020
ms.author: evanhi
ms.openlocfilehash: f839eb04e0c036b5059e35fac16d9c702646cbd7
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96435686"
---
# <a name="enable-and-request-just-in-time-access-for-azure-managed-applications"></a>Activer et demander l’accès juste à temps pour les applications managées Azure

Les consommateurs de votre application managée être réticents à vous accorder un accès permanent au groupe de ressources managé. En tant qu'éditeur d'une application managée, vous préférerez peut-être que les consommateurs sachent précisément quand vous avez besoin d'accéder aux ressources managées. Pour donner aux consommateurs un plus grand contrôle sur l'octroi de l'accès aux ressources managées, le service Applications managées Azure propose une fonctionnalité appelée accès juste-à-temps (JIT). Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire.

L'accès JIT vous permet de demander un accès élevé aux ressources d'une application managée à des fins de résolution des problèmes ou de maintenance. Vous bénéficiez toujours d'un accès en lecture seule aux ressources, mais pour une durée spécifique, votre accès est étendu.

Voici la procédure à suivre pour accorder l’accès :

1. Vous ajoutez une application managée à la Place de marché et précisez que l’accès JAT est disponible.

1. Pendant le déploiement, le consommateur active l’accès JAT pour cette instance de l’application managée.

1. Après le déploiement, le consommateur peut modifier les paramètres d’accès JAT.

1. Vous envoyez une requête d’accès lorsqu'il vous faut détecter un problème ou mettre à jour les ressources managées.

1. Le consommateur approuve votre requête.

Le présent article porte sur les mesures prises par les éditeurs pour permettre l’accès JAT et soumettre des requêtes. Pour en savoir plus sur l'approbation des requêtes d'accès JIT, consultez [Approuver l'accès juste-à-temps dans les applications managées Azure](approve-just-in-time-access.md).

## <a name="add-jit-access-step-to-ui"></a>Ajouter l’étape d’accès JIT à l’interface utilisateur

Dans votre fichier CreateUiDefinition.json, incluez une étape qui permet aux consommateurs d'activer l'accès JIT. Pour prendre en charge la fonctionnalité JIT dans votre offre, ajoutez le contenu suivant à votre fichier CreateUiDefinition.json.

Dans « steps » :

```json
{
    "name": "jitConfiguration",
    "label": "JIT Configuration",
    "subLabel": {
        "preValidation": "Configure JIT settings for your application",
        "postValidation": "Done"
    },
    "bladeTitle": "JIT Configuration",
    "elements": [
        {
          "name": "jitConfigurationControl",
          "type": "Microsoft.Solutions.JitConfigurator",
          "label": "JIT Configuration"
        }
    ]
}
```

Dans « outputs » :

```json
"jitAccessPolicy": "[steps('jitConfiguration').jitConfigurationControl]"
```

> [!NOTE]
> L'accès JIT est proposé en préversion. Le schéma de configuration de JIT peut changer dans les itérations à venir.

## <a name="enable-jit-access"></a>Activer l’accès JIT

Lors de la création de votre offre dans l'Espace partenaires, veillez à activer l'accès JIT.

1. Connectez-vous au portail Place de marché commerciale de l'[Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace/overview).

1. Pour créer une application managée, suivez la procédure décrite dans [Créer une offre d'application Azure](../../marketplace/create-new-azure-apps-offer.md).

1. Sur la page **Configuration technique**, cochez la case **Activer l'accès juste à temps (JIT)** .

   :::image type="content" source="./media/request-just-in-time-access/enable-just-in-time-access.png" alt-text="Activer l’accès juste-à-temps":::

Vous avez ajouté une étape de configuration JIT à votre interface utilisateur et activé l'accès JIT dans l'offre de la Place de marché commerciale. Lorsque les consommateurs déploient votre application managée, ils peuvent [activer l’accès JIT pour leur instance](approve-just-in-time-access.md#enable-during-deployment).

## <a name="request-access"></a>Demander l'accès

Lorsque vous avez besoin d'accéder aux ressources managées du consommateur, vous envoyez une requête portant sur un rôle, une heure et une durée spécifiques. Le consommateur doit ensuite approuver votre requête.

Pour envoyer une requête d’accès JIT :

1. Sélectionnez **Accès JIT** pour l’application managée à laquelle vous souhaitez accéder.

1. Sélectionnez **Rôles éligibles**, puis **Activer** dans la colonne ACTION du rôle souhaité.

   ![Activer la requête d’accès](./media/request-just-in-time-access/send-request.png)

1. Dans le formulaire **Activer le rôle** , sélectionnez une heure de début et la durée pendant laquelle votre rôle sera actif. Sélectionnez **Activer** pour envoyer la requête.

   ![Activer l'accès](./media/request-just-in-time-access/activate-access.png) 

1. Afficher les notifications pour vérifier que la nouvelle requête JIT a bien été envoyée au consommateur.

   ![Notification](./media/request-just-in-time-access/in-progress.png)

   Vous devez maintenant attendre que le consommateur [approuve votre requête](approve-just-in-time-access.md#approve-requests).

1. Pour afficher l’état de toutes les requêtes JIT d'une application managée, sélectionnez **Accès JIT** et **Historique des requêtes**.

   ![Afficher l’état](./media/request-just-in-time-access/view-status.png)

## <a name="known-issues"></a>Problèmes connus

L’ID du principal du compte qui demande l’accès JIT doit être explicitement inclus dans la définition de l'application managée. Le compte ne peut pas être inclus via un groupe spécifié dans le package. Cette limite sera corrigée dans une prochaine version.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l'approbation des requêtes d'accès JIT, consultez [Approuver l'accès juste-à-temps dans les applications managées Azure](approve-just-in-time-access.md).
