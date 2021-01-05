---
title: Migrer un espace de travail et des projets Microsoft Translator Hub - Custom Translator
titleSuffix: Azure Cognitive Services
description: Cet article explique comment migrer votre espace de travail et vos projets Hub vers Azure Cognitive Services Custom Translator.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 05/26/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 64c449bdefe6fb067a7c0e26b155b58b6f00c399
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368282"
---
# <a name="migrate-hub-workspace-and-projects-to-custom-translator"></a>Migrer un espace de travail et des projets Microsoft Hub vers Custom Translator

Vous pouvez facilement migrer votre espace de travail et vos projets [Microsoft Translator Hub](https://hub.microsofttranslator.com/) vers Custom Translator. Pour lancer la migration à partir de Microsoft Hub, sélectionnez successivement un espace de travail ou un projet, un espace de travail dans Custom Translator, puis les entraînements à transférer. Une fois la migration démarrée, les paramètres d’entraînement sélectionnés sont transférés avec tous les documents pertinents. Les modèles déployés sont entraînés et peuvent faire l’objet d’un déploiement automatique une fois l’opération terminée.

Les actions suivantes sont effectuées pendant la migration :
* Les noms de l’ensemble des documents et des définitions de projet sont transférés avec le préfixe « hub_ ». Les données des réglages et des tests générés automatiquement sont nommées hub_systemtune_\<modelid> ou hub_systemtest_\<modelid>.
* Les entraînements qui se trouvent dans l’état déployé quand la migration a lieu sont automatiquement entraînés à l’aide des documents de l’entraînement Hub. Cet entraînement n’est pas facturé à votre abonnement. Si le déploiement automatique est sélectionné pour la migration, le modèle entraîné est déployé une fois l’opération terminée. Des frais normaux d’hébergement sont appliqués.
* Les entraînements migrés qui ne se trouvent pas dans l’état déployé sont placés dans l’état de migration provisoire. Vous pouvez dans cet état entraîner un modèle avec la définition migrée, mais des frais normaux d’entraînement sont appliqués.
* Vous pouvez à tout moment accéder au score BLEU migré à partir de l’entraînement Hub dans la page TrainingDetails du modèle sous le titre « Bleu score in MT Hub ».

> [!Note] 
> Pour une formation réussie, Custom Translator nécessite au moins 10 000 phrases extraites uniques. Custom Translator ne peut pas effectuer l’apprentissage avec moins du [minimum suggéré](./sentence-alignment.md#suggested-minimum-number-of-sentences).

## <a name="find-custom-translator-workspace-id"></a>Rechercher l’ID de l’espace de travail Custom Translator

Pour migrer un espace de travail [Microsoft Translator Hub](https://hub.microsofttranslator.com/), vous avez besoin de l’ID de l’espace de travail de destination dans Custom Translator. L’espace de travail de destination dans Custom Translator représente l’endroit vers lequel tous vos espaces de travail et projets du hub doivent être migrés.

Vous trouverez votre ID d’espace de travail de destination dans la page des paramètres de Custom Translator :

1. Accédez à la page « Settings » (Paramètres) du portail Custom Translator.

2. Vous trouverez l’ID de l’espace de travail dans la section sur les informations de base.

    ![Comment trouver l’ID de l’espace de travail de destination](media/how-to/how-to-find-destination-ws-id.png)

3. Conservez votre ID d’espace de travail de destination pour y faire référence au cours du processus de migration.

## <a name="migrate-a-project"></a>Migrer un projet

Si vous souhaitez migrer vos projets de manière sélective, Microsoft Translator Hub vous offre cette possibilité.

Pour migrer un projet :

1. Connectez-vous à Microsoft Translator Hub.

2. Accédez à la page des projets (Projects).

3. Cliquez sur le lien « Migrate » (Migrer) du projet correspondant.

    ![Capture d’écran mettant en évidence le bouton Migrer pour le projet sélectionné.](media/how-to/how-to-migrate-from-hub.png)

4. Quand vous appuyez sur le lien de migration, vous obtenez un formulaire dans lequel vous pouvez effectuer les tâches suivantes :
   * Spécifier l’espace de travail vers lequel effectuer un transfert dans Custom Translator.
   * Indiquer si vous souhaitez transférer tous les entraînements avec les entraînements réussis ou simplement les entraînements déployés. Par défaut, tous les entraînements réussis sont transférés.
   * Indiquer si vous souhaitez que votre entraînement soit déployé automatiquement à l’issue de l’entraînement. Par défaut, votre entraînement n’est pas automatiquement déployé une fois l’opération terminée.

5. Cliquez sur « Envoyer une demande ».

## <a name="migrate-a-workspace"></a>Migrer un espace de travail

En plus de la migration d’un projet unique, vous pouvez migrer tous les projets avec des entraînements réussis dans un espace de travail. Chaque projet dans l’espace de travail est alors évalué comme si vous aviez appuyé sur le lien de migration. Cette fonctionnalité permet aux utilisateurs de migrer plusieurs projets vers Custom Translator avec les mêmes paramètres. La migration d’un espace de travail peut être lancée à partir de la page des paramètres de Translator Hub.

Pour migrer un espace de travail :

1. Connectez-vous à Microsoft Translator Hub.

2. Accédez à la page des paramètres (Settings).

3. Dans la page « Settings » (Paramètres), cliquez sur « Migrate Workspace data to Custom Translator » (Migrer les données de l’espace de travail vers Custom Translator).

    ![Capture d’écran mettant en évidence l’option Migrer les données de l’espace de travail vers Custom Translator.](media/how-to/how-to-migrate-workspace-from-hub.png)

4. Dans la page suivante, sélectionnez une de ces deux options :

    a. Deployed Trainings only (Entraînements déployés uniquement) : cette option migre uniquement vos systèmes déployés et les documents associés.

    b. All Successful Trainings (Tous les entraînements réussis) : cette option migre tous vos entraînements réussis et les documents associés.

    c. Entrez votre ID d’espace de travail de destination dans Custom Translator.

    ![Migrer à partir du hub](media/how-to/how-to-migrate-from-hub-screen.png)

5. Cliquez sur Envoyer une demande.

## <a name="migration-history"></a>Historique des migrations

Quand vous demandez la migration d’un espace de travail/projet à partir du Hub, l’historique des migrations se trouve dans la page des paramètres de Custom Translator.

Pour afficher l’historique des migrations, procédez comme suit :

1. Accédez à la page « Settings » (Paramètres) du portail Custom Translator.

2. Dans la section de l’historique des migrations de la page des paramètres, cliquez sur Migration History (Historique des migrations).

    ![Historique des migrations](media/how-to/how-to-migration-history.png)

La page de l’historique des migrations s’affiche après les informations suivantes pour récapituler chaque migration que vous avez demandée.

1. Migrated By (Migré par) : nom et adresse e-mail de l’utilisateur ayant soumis cette requête de migration

2. Migrated On (Migré le) : date et heure de la migration

3. Projets : nombre de projets demandés pour la migration par rapport au nombre de projets effectivement migrés.

4. Trainings (Entraînements) : nombre d’entraînements demandés pour la migration par rapport au nombre d’entraînements effectivement migrés.

5. Documents : nombre de documents demandés pour la migration par rapport au nombre de documents effectivement migrés.

    ![Détails de l’historique des migrations](media/how-to/how-to-migration-history-details.png)

Si vous souhaitez obtenir un rapport plus détaillé sur la migration de vos projets, formations et documents, vous pouvez exporter ces informations au format CSV.

## <a name="implementation-notes"></a>Remarques relatives à l’implémentation
* Les systèmes avec des paires de langues PAS encore disponibles dans Custom Translator seront disponibles uniquement pour accéder aux données ou annuler le déploiement via Custom Translator. Ces projets sont marqués comme étant « Non disponible » dans la page Projets. Au fur et à mesure de l’activation de nouvelles paires de langage avec Custom Translator, les projets deviennent actifs pour entraîner et déployer. 
* La migration d’un projet Hub vers Custom Translator n’a aucun impact sur vos entraînements ou projets Hub. Nous ne supprimons pas les projets ou documents Hub durant une migration et n’annulons pas le déploiement de modèles.
* Vous ne pouvez migrer un projet qu’une seule fois. Si vous avez besoin de répéter la migration d’un projet, contactez-nous.
* Custom Translator prend en charge les paires de langues NMT depuis et vers l’anglais. [Affichez la liste complète des langues prises en charge](../language-support.md#customization). Hub ne nécessite pas de modèles de base et prend donc en charge plusieurs milliers de langues. Vous pouvez migrer une paire de langues non prise en charge, mais nous migrons uniquement les documents et définitions de projet. Nous ne pouvons pas entraîner le nouveau modèle. Par ailleurs, ces documents et projets apparaissent comme inactifs pour indiquer que vous ne pouvez pas les utiliser. Si ces projets et/ou documents sont un jour pris en charge, ils deviennent actifs et peuvent être entraînés.
* Actuellement, Custom Translator ne gère pas les données d’entraînement monolingues. Vous pouvez, au même titre que les paires de langues non prises en charge, migrer des documents monolingues. Toutefois, ils apparaissent comme inactifs jusqu’à ce que les données monolingues soient pris en charge.
* L’entraînement de Custom Translator nécessite 10 000 phrases parallèles. Microsoft Hub peut s’entraîner sur un plus petit jeu de données. Si un entraînement ne respectant pas cette condition est migré, la procédure d’entraînement n’est pas effectuée.

## <a name="custom-translator-versus-hub"></a>Custom Translator ou hub

Ce tableau compare les fonctionnalités de Microsoft Translator Hub et celles de Custom Translator.

| Fonctionnalité | Hub | Custom Translator |
| ------- | :-: | :---------------: |
| État de la fonctionnalité de personnalisation    | Disponibilité générale    | Disponibilité générale |
| Version de l'API de texte    | V2     | V3  |
| Personnalisation TA statistique    | Oui    | Non |
| Personnalisation TA neuronale    | Non    | Oui |
| Nouvelle personnalisation unifiée des services vocaux    | Non    | Oui |
| Sans trace | Oui | Oui |

## <a name="new-languages"></a>Nouvelles langues

Si vous êtes une communauté ou une organisation travaillant sur la création d’un nouveau système de langue pour Translator, contactez [custommt@microsoft.com](mailto:custommt@microsoft.com) pour obtenir plus d’informations.

## <a name="next-steps"></a>Étapes suivantes

- [Effectuer l’apprentissage d’un modèle](how-to-train-model.md).
- Commencez à utiliser votre modèle de traduction personnalisé déployé via [Translator V3](../reference/v3-0-translate.md?tabs=curl).