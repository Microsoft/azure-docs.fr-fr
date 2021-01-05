---
title: Créer un compte Azure Remote Rendering
description: Décrit les étapes de création d’un compte pour Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 02/11/2020
ms.topic: how-to
ms.openlocfilehash: 8169b277dfae918e86ac493259325ff84d0b6a4e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998530"
---
# <a name="create-an-azure-remote-rendering-account"></a>Créer un compte Azure Remote Rendering

Ce chapitre vous guide tout au long de la création d’un compte pour le service **Azure Remote Rendering**. Vous devez disposer d’un compte valide pour suivre les différents guides de démarrage rapide et tutoriels.

## <a name="create-an-account"></a>Créer un compte

Vous devez effectuer les étapes suivantes afin de créer un compte pour le service Azure Remote Rendering :

1. Accédez à la [page d’aperçu de réalité mixte](https://aka.ms/MixedRealityPrivatePreview).
1. Cliquez sur le bouton « Créer une ressource ».
1. Tapez « Remote Rendering » dans le champ de recherche (« Rechercher dans la Place de marché »), puis appuyez sur « Entrée ».
1. Dans la liste des résultats, cliquez sur la vignette « Remote Rendering ».
1. Dans l’écran suivant, cliquez sur le bouton « Créer ». Un formulaire s’ouvre. Il vous permet de créer un compte Remote Rendering :
    1. Sous « Nom de la ressource », spécifiez le nom du compte.
    1. Mettez à jour l’« Abonnement » si nécessaire.
    1. Sous « Groupe de ressources », spécifiez le groupe de ressources de votre choix.
    1. Sélectionnez une région dans la liste déroulante « Emplacement » dans laquelle cette ressource doit être créée. Consultez les notes sur [les régions de compte](create-an-account.md#account-regions) ci-dessous.
1. Accédez au compte dès lors que celui-ci est créé, puis effectuez les opérations suivantes :
    1. Dans l’onglet *Vue d’ensemble*, notez l’« ID de compte ».
    1. Dans l’onglet *Paramètres > Clés d’accès*, notez la « Clé primaire », qui est la clé de compte secrète du compte.

### <a name="account-regions"></a>Régions de compte
L’emplacement spécifié lors de la création d’un compte détermine la région à laquelle la ressource de compte est affectée. Cette valeur ne peut pas être modifiée après la création. Toutefois, le compte peut être utilisé pour se connecter à une session Remote Rendering dans n’importe quelle [région prise en charge](./../reference/regions.md), indépendamment de l’emplacement du compte.

### <a name="retrieve-the-account-information"></a>Récupérer les informations de compte

Pour accéder aux exemples et tutoriels, vous devez spécifier l’ID de compte et une clé. Par exemple, dans le fichier **arrconfig.json** utilisé pour les exemples de scripts PowerShell :

```json
"accountSettings": {
    "arrAccountId": "<fill in the account ID from the Azure portal>",
    "arrAccountKey": "<fill in the account key from the Azure portal>",
    "region": "<select from available regions>"
},
```

Consultez la [liste des régions disponibles](../reference/regions.md) pour définir l’option de *région*.

Vous trouverez les valeurs de **`arrAccountId`** et **`arrAccountKey`** dans le portail comme décrit dans les étapes suivantes :

* Accédez au [Portail Azure](https://www.portal.azure.com).
* Recherchez votre **« Compte Remote Rendering »** . Il doit figurer dans la liste **« Ressources récentes »** . Vous pouvez également le rechercher à l’aide de la barre de recherche en haut. Dans ce cas, assurez-vous que l’abonnement que vous souhaitez utiliser est sélectionné dans le filtre d’abonnement par défaut (icône de filtre en regard de la barre de recherche) :

![Filtre d’abonnement](./media/azure-subscription-filter.png)

Quand vous cliquez sur votre compte, l’écran suivant, dans lequel vous pouvez voir instantanément l’**ID de compte**, s’affiche :

![ID de compte Azure](./media/azure-account-id.png)

Pour la clé, sélectionnez **Clés d’accès** dans le panneau de gauche. La page suivante indique une clé primaire et une clé secondaire :

![Clés d’accès Azure](./media/azure-account-primary-key.png)

La valeur de **`arrAccountKey`** peut être une clé primaire ou secondaire.

## <a name="link-storage-accounts"></a>Lier des comptes de stockage

Ce paragraphe explique comment lier des comptes de stockage à votre compte Remote Rendering. Quand un compte de stockage est lié, il n’est pas nécessaire de générer un URI SAS chaque fois que vous souhaitez interagir avec les données du compte, par exemple au chargement d’un modèle. Au lieu de cela, vous pouvez utiliser les noms de compte de stockage directement comme décrit dans la section [Chargement d’un modèle](../concepts/models.md#loading-models).

Vous devez effectuer les étapes de ce paragraphe pour chaque compte de stockage devant utiliser cette méthode d’accès alternative. Si vous n’avez pas encore créé de comptes de stockage, vous pouvez consulter l’étape correspondante dans [Démarrage rapide : Convertir un modèle pour le rendu](../quickstarts/convert-model.md#storage-account-creation).

À présent, nous supposons que vous disposez d’un compte de stockage. Accédez au compte de stockage dans le portail, puis accédez à l’onglet **Contrôle d’accès (IAM)** pour ce compte de stockage :

![Compte de stockage IAM](./media/azure-storage-account.png)

 Veillez à disposer des autorisations de propriétaire sur ce compte de stockage pour vous assurer que vous pouvez ajouter des attributions de rôles. Si vous n’avez pas accès, l’option **Ajouter une attribution de rôle** est désactivée.

 Vous devez ajouter trois rôles distincts comme décrit dans les étapes suivantes. Si vous ne fournissez pas les trois niveaux d’accès, des problèmes d’autorisation se produiront quand vous tenterez d’accéder au compte de stockage.

 Cliquez sur le bouton **Ajouter** dans la vignette « Ajouter une attribution de rôle » pour ajouter le premier rôle :

![Compte de stockage IAM – Ajouter une attribution de rôle](./media/azure-add-role-assignment.png)

* Le premier rôle à attribuer est **Propriétaire** comme indiqué dans la capture d’écran ci-dessus.
* Sélectionnez **Compte Remote Rendering** dans la liste déroulante ***Attribuer l’accès à**.
* Sélectionnez votre abonnement et votre compte Remote Rendering dans les dernières listes déroulantes.

> [!WARNING]
> Si votre Remote Rendering n’est pas répertorié, consultez cette [section pour la résolution de problèmes](../resources/troubleshoot.md#cant-link-storage-account-to-arr-account).

Répétez deux fois la procédure d’ajout de rôles en effectuant les sélections suivantes dans la liste déroulante **Rôle** :

* **Contributeur de compte de stockage**
* **Contributeur aux données Blob du stockage**

Les sélections dans les autres listes déroulantes correspondent à celles de la première étape.

Si vous avez ajouté les trois rôles, votre compte Azure Remote Rendering a accès à votre compte de stockage avec les identités de service managées affectées par le système.
> [!IMPORTANT]
> Les attributions de rôles Azure étant mises en cache par le service Stockage Azure, jusqu’à 30 minutes peuvent s’écouler entre le moment où vous accordez l’accès à votre compte de rendu à distance et le moment où celui-ci peut être utilisé pour accéder à votre compte de stockage. Pour plus d’informations, consultez la [documentation sur le contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/troubleshooting.md#role-assignment-changes-are-not-being-detected).

## <a name="next-steps"></a>Étapes suivantes

* [Authentification](authentication.md)
* [Utiliser les API Azure Front-end pour l’authentification](frontend-apis.md)
* [Exemples de scripts PowerShell](../samples/powershell-example-scripts.md)