---
title: 'Tutoriel : Implémenter Azure Policy en tant que code avec GitHub'
description: Dans ce tutoriel, vous allez implémenter un workflow Azure Policy sous forme de code, avec l’exportation, les actions GitHub et les workflows GitHub
ms.date: 10/20/2020
ms.topic: tutorial
ms.openlocfilehash: 76a46adc3fc8efab4f7a2d6e656e83c2537dd037
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92325874"
---
# <a name="tutorial-implement-azure-policy-as-code-with-github"></a>Tutoriel : Implémenter Azure Policy en tant que code avec GitHub

Un workflow Azure Policy sous forme de code permet de gérer vos définitions et affectations de stratégies en tant que code, de contrôler le cycle de vie de la mise à jour de ces définitions, et d’automatiser la validation des résultats de conformité. Dans ce tutoriel, vous allez apprendre à utiliser les fonctionnalités Azure Policy avec GitHub pour créer un processus de cycle de vie. Il s’agit notamment des tâches suivantes :

> [!div class="checklist"]
> - Exporter les définitions et les affectations de stratégie vers GitHub
> - Pousser vers Azure les objets de stratégie mis à jour dans GitHub
> - Déclencher une analyse de conformité à partir de l’action GitHub

Si vous souhaitez affecter une stratégie pour identifier l’état de conformité actuel de vos ressources existantes, les articles de démarrage rapide vous expliquent la marche à suivre.

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Passez en revue [Concevoir un workflow Azure Policy sous forme de code](../concepts/policy-as-code.md) pour comprendre les modèles de conception utilisés dans ce tutoriel.

### <a name="export-azure-policy-objects-from-the-azure-portal"></a>Exporter les objets Azure Policy à partir du portail Azure

Pour exporter une définition de stratégie à partir du Portail Azure, procédez comme suit :

1. Lancez le service Azure Policy dans le portail Azure en cliquant sur **Tous les services** , puis en recherchant et en cliquant sur **Stratégie** .

1. Sélectionnez **Définitions** sur le côté gauche de la page Azure Policy.

1. Utilisez le bouton **Exporter les définitions** ou sélectionnez les points de suspension sur la ligne d’une définition de stratégie, puis sélectionnez **Exporter la définition** .

1. Sélectionnez le bouton **Sign in with GitHub** (Se connecter avec GitHub). Si vous n’avez pas encore été authentifié avec GitHub pour autoriser Azure Policy à exporter la ressource, passez en revue les besoins [GitHub Actions](https://github.com/features/actions) dans la nouvelle fenêtre qui s’ouvre, puis sélectionnez **Autoriser AzureGitHubActions** pour poursuivre le processus d’exportation. Une fois l’opération terminée, la nouvelle fenêtre se ferme automatiquement.

1. Sous l’onglet **Général** , définissez les options suivantes, puis sélectionnez l’onglet **Stratégies** ou **Suivant : Stratégies** en bas de la page.

   - **Filtre de référentiel**  : Affectez la valeur _Mes référentiels_ pour afficher uniquement les référentiels que vous possédez, ou _Tous les référentiels_ pour voir tous les référentiels auxquels vous avez accordé l’accès à GitHub Action.
   - **Référentiel** : Définissez sur le référentiel dans lequel vous souhaitez exporter les ressources Azure Policy.
   - **Branche** : Définissez la branche dans le référentiel. L’utilisation d’une autre branche que la branche par défaut est un bon moyen de valider vos mises à jour avant de les fusionner dans votre code source.
   - **Répertoire** : Le _dossier racine_ vers lequel exporter les ressources Azure Policy. Les sous-dossiers de ce répertoire sont créés en fonction des ressources exportées.

1. Sous l’onglet **Stratégies** , définissez l’étendue à rechercher en sélectionnant les points de suspension et en choisissant une combinaison de groupes d’administration, d’abonnements ou de groupes de ressources.
   
1. Utilisez le bouton **Ajouter une ou plusieurs définitions de stratégie** pour rechercher l’étendue des objets à exporter. Dans la fenêtre latérale qui s’ouvre, sélectionnez chaque objet à exporter. Filtrez la sélection avec la zone de recherche ou en sélectionnant un type. Une fois que vous avez sélectionné tous les objets à exporter, utilisez le bouton **Ajouter** au bas de la page.

1. Pour chaque objet sélectionné, sélectionnez les options d’exportation souhaitées, par exemple _Définition seulement_ ou _Définition et affectation(s)_ pour une définition de stratégie. Sélectionnez ensuite l’onglet **Vérifier + exporter** ou le bouton **Suivant : Vérifier + exporter** en bas de la page.

   > [!NOTE]
   > Si vous avez choisi l’option _Définition et affectation(s)_ , seules les affectations de stratégie au sein de l’étendue définie par le filtre lors de l’ajout de la définition de stratégie sont exportées.

1. Sous l’onglet **Vérifier + exporter** , vérifiez que les détails correspondent, puis utilisez le bouton **Exporter** au bas de la page.

1. Vérifiez votre référentiel, votre branche et votre _dossier de niveau racine_ GitHub pour vérifier que les ressources sélectionnées sont désormais exportées vers votre contrôle de code source.

Les ressources Azure Policy sont exportées dans la structure suivante dans le référentiel et le _dossier de niveau racine_ GitHub sélectionnés :

```text
|
|- <root level folder>/  ________________ # Root level folder set by Directory property
|  |- policies/  ________________________ # Subfolder for policy objects
|     |- <displayName>_<name>____________ # Subfolder based on policy displayName and name properties
|        |- policy.json _________________ # Policy definition
|        |- assign.<displayName>_<name>__ # Each assignment (if selected) based on displayName and name properties
|
```

### <a name="push-policy-objects-updated-in-github-to-azure"></a>Pousser vers Azure les objets de stratégie mis à jour dans GitHub

1. Lorsque des objets de stratégie sont exportés, un fichier de [workflow GitHub](https://docs.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow#about-workflows) nommé `.github/workflows/manage-azure-policy-<randomLetters>.yml` est également créé pour vous aider à démarrer.

   > [!NOTE]
   > Le fichier de workflow GitHub est créé chaque fois que l’exportation est utilisée. Chaque instance du fichier est spécifique aux options utilisées durant cette action d’exportation.

1. Ce fichier de workflow utilise l’action [Gérer Azure Policy](https://github.com/marketplace/actions/manage-azure-policy) pour transmettre à Azure Policy les modifications apportées aux objets de stratégie exportés dans le dépôt GitHub. Par défaut, l’action prend en compte et synchronise uniquement les fichiers qui sont différents de ceux existants dans Azure. Vous pouvez également utiliser le paramètre `assignments` dans l’action pour synchroniser uniquement les modifications apportées à des fichiers d’affectation spécifiques. Ce paramètre peut être utilisé pour appliquer des affectations de stratégie uniquement pour un environnement particulier. Pour plus d’informations, consultez le [fichier readme du dépôt Manage Azure Policy](https://github.com/Azure/manage-azure-policy).

1. Par défaut, le workflow doit être déclenché manuellement. Pour ce faire, utilisez les **Actions** dans GitHub, sélectionnez le workflow `manage-azure-policy-<randomLetters>`, puis sélectionnez **Exécuter le workflow** et de nouveau **Exécuter le workflow** .

   :::image type="content" source="../media/policy-as-code-github/manually-trigger-workflow.png" alt-text="Capture d’écran de l’onglet Action, du workflow et des boutons Exécuter le workflow dans l’interface web GitHub.":::

   > [!NOTE]
   > Le fichier de workflow doit se trouver dans la branche par défaut pour être détecté et exécuté manuellement.

1. Le workflow synchronise avec Azure les modifications apportées aux objets de stratégie, et vous donne l’état dans les journaux.

   :::image type="content" source="../media/policy-as-code-github/workflow-logging.png" alt-text="Capture d’écran de l’onglet Action, du workflow et des boutons Exécuter le workflow dans l’interface web GitHub.":::

1. Le workflow ajoute également des détails dans les objets Azure Policy `properties.metadata` pour vous permettre le suivi.

   :::image type="content" source="../media/policy-as-code-github/updated-definition-metadata.png" alt-text="Capture d’écran de l’onglet Action, du workflow et des boutons Exécuter le workflow dans l’interface web GitHub.":::

### <a name="trigger-compliance-scans-using-github-action"></a>Déclencher des analyses de conformité au moyen de l’action GitHub

En utilisant l’[action Analyse de conformité Azure Policy](https://github.com/marketplace/actions/azure-policy-compliance-scan), vous pouvez déclencher une analyse d’évaluation de conformité à la demande depuis votre [workflow GitHub](https://docs.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow#about-workflows) sur un nombre quelconque de ressources, de groupes de ressources ou d’abonnements, et contrôler le chemin de workflow en fonction de l’état de conformité de ces ressources. Vous pouvez également configurer le workflow pour qu’il s’exécute à une heure planifiée afin d’obtenir l’état de conformité le plus récent au moment opportun. Si vous le souhaitez, cette action GitHub peut également générer un rapport sur l’état de conformité des ressources analysées en vue d’une analyse plus poussée ou de l’archivage.

L’exemple suivant exécute une analyse de conformité pour un abonnement. 

```yaml

on:
  schedule:    
    - cron:  '0 8 * * *'  # runs every morning 8am
jobs:
  assess-policy-compliance:    
    runs-on: ubuntu-latest
    steps:         
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{secrets.AZURE_CREDENTIALS}} 

    
    - name: Check for resource compliance
      uses: azure/policy-compliance-scan@v0
      with:
        scopes: |
          /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

```

## <a name="review"></a>Révision

Dans ce didacticiel, vous avez effectué avec succès les tâches suivantes :

> [!div class="checklist"]
> - Exportation des définitions et des affectations de stratégie vers GitHub
> - Envoi vers Azure d’objets de stratégie mis à jour dans GitHub
> - Déclenchement d’une analyse de conformité à partir de l’action GitHub

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les structures des définitions de stratégie, consultez cet article :

> [!div class="nextstepaction"]
> [Structure de définition Azure Policy](../concepts/definition-structure.md)