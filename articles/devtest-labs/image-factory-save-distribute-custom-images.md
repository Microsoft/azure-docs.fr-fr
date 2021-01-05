---
title: Enregistrer et distribuer des images dans Azure DevTest Labs | Microsoft Docs
description: Cet article décrit les étapes à suivre pour enregistrer des images personnalisées à partir de machines virtuelles existantes dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a5278626f8cdd4299912f3c952786422436fe916
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85476238"
---
# <a name="save-custom-images-and-distribute-to-multiple-labs"></a>Enregistrer les images personnalisées et les distribuer à plusieurs laboratoires
Cet article décrit les étapes à suivre pour enregistrer des images personnalisées à partir de machines virtuelles existantes. Il explique également comment distribuer ces images personnalisées dans d’autres labs DevTest Labs de l’organisation.

## <a name="prerequisites"></a>Prérequis
Les éléments suivants doivent déjà être en place :

- Un lab pour la fabrique d’images dans Azure DevTest Labs.
- Un projet Azure DevOps utilisé pour automatiser la fabrique d’images.
- Un emplacement pour le code source, contenant les scripts et la configuration (dans notre exemple, il se trouve dans le même projet DevOps que celui indiqué à l’étape précédente).
- Une définition de build pour orchestrer les tâches Azure PowerShell.

Si vous devez créer ou configurer un ou plusieurs de ces éléments, effectuez les étapes décrites dans [Exécuter une fabrique d’images à partir d’Azure DevOps](image-factory-set-up-devops-lab.md). 

## <a name="save-vms-as-generalized-vhds"></a>Enregistrer des machines virtuelles en tant que disques durs virtuels généralisés
Enregistrez les machines virtuelles existantes en tant que disques durs virtuels généralisés.  Vous pouvez utiliser l’exemple de script PowerShell fourni pour enregistrer les machines virtuelles existantes en tant que disques durs virtuels généralisés. Pour l’utiliser, ajoutez d’abord une autre tâche **Azure PowerShell** à la définition de build, comme illustré ci-dessous :

![Ajouter une étape Azure PowerShell](./media/save-distribute-custom-images/powershell-step.png)

Quand vous voyez la nouvelle tâche dans la liste, sélectionnez-la, puis renseignez tous les détails comme indiqué dans l’image suivante : 

![Paramètres PowerShell](./media/save-distribute-custom-images/powershell-settings.png)


## <a name="generalized-vs-specialized-custom-images"></a>Images personnalisées généralisées et spécialisées
Dans le [portail Azure](https://portal.azure.com), quand vous créez une image personnalisée à partir d’une machine virtuelle, vous avez le choix de créer une image personnalisée généralisée ou spécialisée.

- **Image personnalisée spécialisée :** Sysprep ou le déprovisionnement n’a PAS été exécuté sur la machine. L’image est donc une copie exacte du disque du système d’exploitation sur la machine virtuelle existante (un instantané).  Quand une machine virtuelle est créée à partir de cette image personnalisée, elle comporte les mêmes fichiers, applications, comptes d’utilisateur, nom d’ordinateur, etc.
- **Image personnalisée généralisée :** Sysprep ou un déprovisionnement a été exécuté sur la machine.  Quand ce processus est exécuté, il supprime les comptes d’utilisateur, le nom d’ordinateur, les ruches du Registre utilisateur, etc. dans le but de généraliser l’image, c’est-à-dire de la rendre personnalisable lors de la création d’une autre machine virtuelle.  Quand vous généralisez une machine virtuelle (en exécutant Sysprep), le processus détruit la machine virtuelle actuelle, qui n’est alors plus opérationnelle.

Le script d’alignement des images personnalisées dans la fabrique d’images enregistre les disques durs virtuels des machines virtuelles qui ont été créées à l’étape précédente (celles identifiées par une étiquette placée sur la ressource dans Azure).

## <a name="update-configuration-for-distributing-images"></a>Mettre à jour la configuration pour distribuer les images
L’étape suivante du processus consiste à envoyer (push) les images personnalisées du lab de la fabrique d’images aux autres labs qui en ont besoin. L’élément principal de ce processus est le fichier de configuration **labs.json**. Ce fichier se trouve dans le dossier **Configuration** inclus dans la fabrique d’images.

Le fichier de configuration labs.json contient deux éléments clés :

- L’identifiant unique d’un lab de destination, composé de l’ID de l’abonnement et du nom du lab.
- L’ensemble des images à envoyer au lab sous forme de chemins relatifs à la racine de la configuration. Vous pouvez également spécifier un dossier entier (pour prendre toutes les images présentes dans ce dossier).

Voici un exemple de fichier Labs.json où deux labs sont listés. Dans ce cas, vous distribuez les images à deux labs différents.

```json
{
   "Labs": [
      {
         "SubscriptionId": "<subscription ID that contains the lab>",
         "LabName": "<Name of the DevTest Lab>",
         "ImagePaths": [
               "Win2012R2",
               "Win2016/Datacenter.json"
         ]
      },
      {
         "SubscriptionId": "<subscription ID that contains the lab>",
         "LabName": "<Name of the DevTest Lab>",
         "ImagePaths": [
               "Win2016/Datacenter.json"
         ]
      }
   ]
}
```

## <a name="create-a-build-task"></a>Créer une tâche de génération
En suivant les mêmes étapes que celles décrites plus haut dans cet article, ajoutez une autre tâche de build **Azure PowerShell** à la définition de build existante. Renseignez les détails comme indiqué dans l’image suivante : 

![Tâche de build pour distribuer les images](./media/save-distribute-custom-images/second-build-task-powershell.png)

Les paramètres sont : `-ConfigurationLocation $(System.DefaultWorkingDirectory)$(ConfigurationLocation) -SubscriptionId $(SubscriptionId) -DevTestLabName $(DevTestLabName) -maxConcurrentJobs 20`

Cette tâche prend les images personnalisées présentes dans la fabrique d’images et les envoie (push) aux labs définis dans le fichier Labs.json.

## <a name="queue-the-build"></a>Mettre la build en file d’attente
Une fois que la tâche de build de la distribution est terminée, mettez en file d’attente une nouvelle build pour vérifier que tout fonctionne correctement. Une fois que la build est terminée, les nouvelles images personnalisées sont affichées dans le lab de destination qui a été spécifié dans le fichier de configuration Labs.json.

## <a name="next-steps"></a>Étapes suivantes
Dans l’article suivant de la série, vous mettez à jour la fabrique d’images par le biais d’une stratégie de conservation et d’un processus de nettoyage : [Définir la stratégie de conservation et exécuter des scripts de nettoyage](image-factory-set-retention-policy-cleanup.md).
