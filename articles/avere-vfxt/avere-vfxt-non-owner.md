---
title: Solution de contournement pour les utilisateurs d’Avere vFXT non propriétaires - Azure
description: Découvrez une solution de contournement pour permettre aux utilisateurs sans autorisation de propriétaire sur l’abonnement de déployer Avere vFXT pour Azure
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 12/19/2019
ms.author: rohogue
ms.openlocfilehash: 0d9b1060ee35af6cbc2e1b95b0f7813072c52d2e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85505373"
---
# <a name="authorize-non-owners-to-deploy-avere-vfxt"></a>Autoriser les utilisateurs non propriétaires à déployer Avere vFXT

Ces instructions sont une solution de contournement qui permet à un utilisateur n’ayant pas les privilèges de propriétaire sur l’abonnement de créer un système Avere vFXT pour Azure.

(La méthode recommandée pour déployer le système Avere vFXT est d’octroyer les privilèges de propriétaire à l’utilisateur chargé de la création du système, comme expliqué dans [Étapes de préparation à la création d’un système Avere vFXT](avere-vfxt-prereqs.md).)  

La solution de contournement consiste à créer un rôle d’accès supplémentaire qui donne aux utilisateurs les autorisations nécessaires pour installer le cluster. Le rôle doit être créé par un propriétaire d’abonnement, et un propriétaire doit assigner le rôle aux utilisateurs appropriés.

Le propriétaire d’abonnement doit également [accepter les conditions d’utilisation](avere-vfxt-prereqs.md) de l’image de la Place de marché Avere vFXT.

> [!IMPORTANT]
> Toutes ces étapes doivent être effectuées par un utilisateur qui a les privilèges de propriétaire sur l’abonnement utilisé pour le cluster.

1. Copiez ces lignes et enregistrez-les dans un fichier (par exemple, `averecreatecluster.json`). Spécifiez votre ID d’abonnement dans l’instruction `AssignableScopes`.

   ```json
   {
       "AssignableScopes": ["/subscriptions/<SUBSCRIPTION_ID>"],
       "Name": "avere-create-cluster",
       "IsCustom": "true"
       "Description": "Can create Avere vFXT clusters",
       "NotActions": [],
       "Actions": [
           "Microsoft.Authorization/*/read",
           "Microsoft.Authorization/roleAssignments/*",
           "Microsoft.Authorization/roleDefinitions/*",
           "Microsoft.Compute/*/read",
           "Microsoft.Compute/availabilitySets/*",
           "Microsoft.Compute/virtualMachines/*",
           "Microsoft.Network/*/read",
           "Microsoft.Network/networkInterfaces/*",
           "Microsoft.Network/routeTables/write",
           "Microsoft.Network/routeTables/delete",
           "Microsoft.Network/routeTables/routes/delete",
           "Microsoft.Network/virtualNetworks/subnets/join/action",
           "Microsoft.Network/virtualNetworks/subnets/read",

           "Microsoft.Resources/subscriptions/resourceGroups/read",
           "Microsoft.Resources/subscriptions/resourceGroups/resources/read",
           "Microsoft.Storage/*/read",
           "Microsoft.Storage/storageAccounts/listKeys/action"
       ],
   }
   ```

1. Exécutez cette commande pour créer le rôle :

   `az role definition create --role-definition <PATH_TO_FILE>`

    Exemple :

    ```azurecli
    az role definition create --role-definition ./averecreatecluster.json
    ```

1. Assignez ce rôle à l’utilisateur chargé de créer le cluster :

   `az role assignment create --assignee <USERNAME> --scope /subscriptions/<SUBSCRIPTION_ID> --role 'avere-create-cluster'`

Une fois ce processus terminé, tout utilisateur qui reçoit ce rôle a les autorisations suivantes sur l’abonnement :

* Créer et configurer l’infrastructure réseau
* Créer le contrôleur de cluster
* Exécuter des scripts de création de cluster à partir du contrôleur de cluster pour créer le cluster
