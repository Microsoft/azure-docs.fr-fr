---
title: Envoyer des paramètres vers App Configuration avec Azure Pipelines
description: Découvrez comment utiliser Azure Pipelines pour envoyer des paires clé-valeur à un magasin App Configuration.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.topic: how-to
ms.date: 07/27/2020
ms.author: alkemper
ms.openlocfilehash: fd3f7dbfd824360dcba9f8a166c579e3b55527ba
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932113"
---
# <a name="push-settings-to-app-configuration-with-azure-pipelines"></a>Envoyer des paramètres vers App Configuration avec Azure Pipelines

La tâche [Azure App Configuration Push](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task-push) envoie les paires clé-valeur d’un fichier de configuration dans votre magasin App Configuration. Cette tâche permet de bénéficier d’une fonctionnalité circulaire complète dans le pipeline, car vous pouvez désormais extraire des paramètres du magasin App Configuration et envoyer des paramètres au magasin App Configuration.

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
- Ressource App Configuration : créez-en une gratuitement dans le [portail Azure](https://portal.azure.com)
- Projet Azure DevOps : [créez-en un gratuitement](https://go.microsoft.com/fwlink/?LinkId=2014881)
- Tâche Azure App Configuration Push : téléchargez-la gratuitement à partir de [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task-push)

## <a name="create-a-service-connection"></a>Créer une connexion de service

Une [connexion de service](/azure/devops/pipelines/library/service-endpoints) vous permet d’accéder aux ressources de votre abonnement Azure à partir de votre projet Azure DevOps.

1. Dans Azure DevOps, accédez au projet contenant votre pipeline cible et ouvrez les **Paramètres du projet** en bas à gauche.
1. Sous **Pipelines** sélectionnez **Connexions de service** puis **Nouvelle connexion de service** en haut à droite.
1. Sélectionnez **Azure Resource Manager**.
1. Sélectionnez **Principal de service (automatique)** .
1. Renseignez votre abonnement et votre ressource. Donnez un nom à votre connexion de service.

Maintenant que votre connexion de service est créée, recherchez le nom du principal de service qui lui est assigné. Vous ajouterez une nouvelle attribution de rôle à ce principal de service à l’étape suivante.

1. Accédez à **Paramètres du projet** > **Connexions de service**.
1. Sélectionnez la connexion de service que vous avez créée dans la section précédente.
1. Sélectionnez **Gérer le principal du service**.
1. Notez le **Nom complet** listé.

## <a name="add-role-assignment"></a>Ajouter une attribution de rôle

Affectez les attributions de rôle App Configuration appropriées aux informations d’identification utilisées dans la tâche, afin que la tâche puisse accéder au magasin App Configuration.

1. Accédez à votre magasin App Configuration cible. 
1. Sur la gauche, sélectionnez **Contrôle d’accès (IAM)** .
1. En haut, sélectionnez **+ Ajouter** et choisissez **Ajouter une attribution de rôle**.
1. Sous **Rôle**, sélectionnez **Propriétaire des données App Configuration**. Ce rôle permet à la tâche de lire et d’écrire dans le magasin App Configuration. 
1. Sélectionnez le principal de service associé à la connexion de service que vous avez créée dans la section précédente.
  
## <a name="use-in-builds"></a>Utiliser dans les builds

Cette section explique comment utiliser la tâche Azure App Configuration Push dans un pipeline de build Azure DevOps.

1. Accédez à la page de pipeline de build en cliquant sur **Pipelines** > **Pipelines**. Vous trouverez la documentation sur les pipelines de build [ici](/azure/devops/pipelines/create-first-pipeline?tabs=tfs-2018-2&view=azure-devops).
      - Si vous créez un pipeline de build, sélectionnez **Afficher l’Assistant** sur le côté droit du pipeline, puis recherchez la tâche **Azure App Configuration Push**.
      - Si vous utilisez un pipeline de build existant, accédez à l’onglet **Tâches** lors de la modification du pipeline, puis recherchez la tâche **Azure App Configuration Push**.
2. Configurez les paramètres nécessaires pour que la tâche envoie les paires clé-valeur du fichier de configuration vers le magasin App Configuration. Le paramètre de **chemin du fichier de configuration**  commence à la racine du référentiel de fichiers.
3. Enregistrez et mettez en file d’attente une build. Le journal de génération affiche les erreurs qui se sont produites pendant l’exécution de la tâche.

## <a name="use-in-releases"></a>Utiliser dans les mises en production

Cette section explique comment utiliser la tâche Azure App Configuration Push dans des pipelines de mise en production Azure DevOps.

1. Accédez à la page de pipeline de mise en production en sélectionnant **Pipelines** > **Mises en production**. La documentation sur les pipelines de mise en production est disponible [ici](/azure/devops/pipelines/release?view=azure-devops).
1. Choisissez un pipeline de mise en production existant. Si vous n’en avez pas, sélectionnez **+ Nouveau** pour en créer un.
1. Sélectionnez le bouton **Modifier** dans le coin supérieur droit pour modifier le pipeline de mise en production.
1. Choisissez la **phase** à laquelle ajouter la tâche. Vous trouverez des informations supplémentaires sur les phases [ici](/azure/devops/pipelines/release/environments?view=azure-devops).
1. Sélectionnez **+** pour ce travail, puis ajoutez la tâche **Azure App Configuration Push** sous l’onglet **Déployer**.
1. Configurez les paramètres nécessaires dans la tâche afin d’envoyer les paires clé-valeur du fichier de configuration vers votre magasin App Configuration. Vous trouverez des explications sur les paramètres dans la section **Paramètres** ci-dessous ainsi que dans les info-bulles en regard de chaque paramètre.
1. Enregistrez et mettez en file d’attente une mise en production. Le journal de mise en production affiche les erreurs rencontrées pendant l’exécution de la tâche.

## <a name="parameters"></a>Paramètres

Les paramètres suivants sont utilisés par la tâche App Configuration Push :

- **Azure subscription** : liste déroulante contenant vos connexions de service Azure disponibles. Pour mettre à jour et actualiser votre liste de connexions de service Azure disponibles, appuyez sur le bouton **Refresh Azure subscription** (Actualiser l’abonnement Azure) à droite de la zone de texte.
- **App Configuration Name** : liste déroulante qui charge vos magasins de configuration disponibles sous l’abonnement sélectionné. Pour mettre à jour et actualiser votre liste de magasins de configuration disponibles, appuyez sur le bouton **Refresh App Configuration Name** (Actualiser le nom App Configuration) à droite de la zone de texte.
- **Configuration File Path** : chemin de votre fichier de configuration. Vous pouvez parcourir votre artefact de build pour sélectionner un fichier de configuration. (bouton `...` à droite de la zone de texte).
- **Separator** : séparateur utilisé pour aplatir les fichiers .JSON et .yml.
- **Depth** : profondeur à laquelle les fichiers .JSON et .yml seront aplatis.
- **Prefix** : chaîne ajoutée au début de chaque clé envoyée au magasin App Configuration.
- **Label** : chaîne ajoutée à chaque paire clé-valeur en tant qu’étiquette dans le magasin App Configuration.
- **Content Type** : chaîne ajoutée à chaque paire clé-valeur en tant que type de contenu dans le magasin App Configuration.
- **Tags** : objet JSON au format `{"tag1":"val1", "tag2":"val2"}`, qui définit les étiquettes ajoutées à chaque paire clé-valeur envoyée à votre magasin App Configuration.
- **Delete all other Key-Values in store with the specified prefix and label** : la valeur par défaut est **Désactivé**.
  - **Activé** : supprime toutes les paires clé-valeur dans le magasin App Configuration qui correspondent à la fois au préfixe et à l’étiquette spécifiés avant d’envoyer de nouvelles paires clé-valeur à partir du fichier de configuration.
  - **Désactivé** : envoie toutes les paires clé-valeur du fichier de configuration dans le magasin App Configuration, et laisse intact tout le reste dans le magasin App Configuration.

Après avoir renseigné les paramètres requis, exécutez le pipeline. Toutes les paires clé-valeur dans le fichier de configuration spécifié seront chargées vers App Configuration.

## <a name="troubleshooting"></a>Dépannage

Si une erreur inattendue se produit, les journaux de débogage peuvent être activés en affectant la valeur `true` à la variable de pipeline `system.debug`.

## <a name="faq"></a>Questions fréquentes (FAQ)

**Comment faire pour charger plusieurs fichiers de configuration ?**

Créez plusieurs instances de la tâche Azure App Configuration Push dans le même pipeline pour envoyer plusieurs fichiers de configuration au magasin App Configuration.

**Pourquoi est-ce que je reçois une erreur 409 lors de la tentative d’envoi de paires clé-valeur vers mon magasin de configuration ?**

Une erreur de conflit 409 se produit si la tâche tente de supprimer ou de remplacer une paire clé-valeur qui est verrouillée dans le magasin App Configuration.
