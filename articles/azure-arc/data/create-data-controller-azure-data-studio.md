---
title: Créer un contrôleur de données dans Azure Data Studio
description: Créer un contrôleur de données dans Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 208c9b4172719b876766f0c4d07a17caa24bfd63
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92310950"
---
# <a name="create-data-controller-in-azure-data-studio"></a>Créer un contrôleur de données dans Azure Data Studio

Vous pouvez créer un contrôleur de données en utilisant Azure Data Studio via l’Assistant Déploiement et des notebooks.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Prérequis

- Vous devez avoir accès à un cluster Kubernetes et votre fichier kubeconfig doit être configuré pour pointer vers le cluster Kubernetes sur lequel vous voulez déployer.
- Vous devez [installer les outils clients](install-client-tools.md), notamment **Azure Data Studio** et les extensions Azure Data Studio appelées **Azure Arc** et **[!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]** .
- Vous devez vous connecter à Azure dans Azure Data Studio.  Pour cela, tapez Ctrl/Cmd+Maj+P pour ouvrir la fenêtre de texte de commande, puis tapez **Azure** .  Choisissez **Azure : Sign in** .   Dans le volet qui s’affiche, cliquez sur l’icône + en haut à droite pour ajouter un compte Azure.

## <a name="use-the-deployment-wizard-to-create-azure-arc-data-controller"></a>Utiliser l’Assistant Déploiement pour créer un contrôleur de données Azure Arc

Suivez ces étapes pour créer un contrôleur de données Azure Arc en utilisant l’Assistant Déploiement.

1. Dans Azure Data Studio, cliquez sur l’onglet Connexions dans le volet de navigation gauche.
2. Cliquez sur le bouton **...** en haut du panneau Connexions, puis choisissez **Nouveau déploiement...**
3. Dans l’Assistant Nouveau déploiement, choisissez **Contrôleur de données Azure Arc** , cochez la case d’acceptation de la licence, puis cliquez sur le bouton **Sélectionner** dans le bas.
4. Utilisez le fichier kubeconfig par défaut ou sélectionnez-en un autre.  Cliquez sur **Suivant** .
5. Choisissez un contexte de cluster Kubernetes. Cliquez sur **Suivant** .
6. Choisissez un fichier de profil de configuration de déploiement en fonction de votre cluster Kubernetes cible. Cliquez sur **Suivant** .
8. Choisissez l’abonnement et le groupe de ressources souhaités.
9. Entrez un nom pour le contrôleur de données et pour l’espace de noms dans lequel le contrôleur de données sera créé.  

> [!NOTE]
> Si l’espace de noms existe déjà, il sera utilisé si l’espace de noms ne contient pas déjà d’autres objets Kubernetes, comme des pods, etc.  Si l’espace de noms n’existe pas, une tentative de création de l’espace de noms sera effectuée.  La création d’un espace de noms dans un cluster Kubernetes nécessite des privilèges d’administrateur du cluster Kubernetes.  Si vous ne disposez pas de privilèges d’administrateur du cluster Kubernetes, demandez à l’administrateur de votre cluster Kubernetes d’effectuer les premières étapes décrites dans l’article [Créer un contrôleur de données en utilisant des outils natifs Kubernetes](./create-data-controller-using-kubernetes-native-tools.md) qui doivent être exécutées par un administrateur Kubernetes avant que vous puissiez utiliser cet Assistant.

> [!NOTE]
> Remarque : Le nom du contrôleur de données et de l’espace de noms sont utilisés pour créer une ressource personnalisée dans le cluster Kubernetes : ils doivent donc être conformes aux [conventions de nommage de Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names).

10. Sélectionnez un emplacement Azure.
   
> [!NOTE]
> L’emplacement Azure sélectionné ici est l’emplacement dans Azure où les *métadonnées* sur le contrôleur de données et les instances de base de données qu’il gère seront stockées.  Les instances de contrôleur de données et de base de données seront créées sur votre cluster Kubernetes, où qu’il soit.

11.  Entrez un nom d’utilisateur et un mot de passe, puis confirmez le mot de passe pour le compte d’administrateur du contrôleur de données.

> [!NOTE]
> Le mot de passe doit comporter au moins 8 caractères.

1.  Cliquez sur **Suivant** .
2.  Passez en revue et cliquez sur **Exécuter un script sur un notebook** .
3.  **Passez en revue le notebook généré** .  Apportez les modifications nécessaires, comme les noms des classes de stockage ou les types de services.
4.  Cliquez sur **Exécuter tout** en haut du notebook.

## <a name="monitoring-the-creation-status"></a>Surveillance de l’état de la création

La création du contrôleur prend plusieurs minutes. Vous pouvez superviser la progression dans une autre fenêtre de terminal avec les commandes suivantes :

> [!NOTE]
>  Les exemples de commandes ci-dessous supposent que vous avez créé un contrôleur de données et un espace de noms Kubernetes avec le nom « arc ».  Si vous avez utilisé un autre nom pour le contrôleur de données/espace de noms, vous pouvez remplacer « arc » par ce nom.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

Vous pouvez également vérifier l’état de la création de n’importe quel pod en exécutant une commande comme celle qui figure ci-dessous.  C’est particulièrement pratique pour résoudre les éventuels problèmes.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Résolution des problèmes de création

Si vous rencontrez des problèmes avec la création, consultez le [Guide de résolution des problèmes](troubleshoot-guide.md).
