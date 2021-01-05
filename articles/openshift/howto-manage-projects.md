---
title: Gérer les ressources dans Azure Red Hat OpenShift | Microsoft Docs
description: Gérer les projets, modèles et flux d’images dans un cluster Azure Red Hat OpenShift
services: openshift
keywords: demandes de projets red hat openshift avec provisionnement automatique
author: mjudeikis
ms.author: gwallace
ms.date: 07/19/2019
ms.topic: conceptual
ms.service: container-service
ms.openlocfilehash: 0abc086553f5e903a71bcfd0b6322bcee56d2d8b
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92216927"
---
# <a name="manage-projects-templates-image-streams-in-an-azure-red-hat-openshift-cluster"></a>Gérer les projets, modèles et flux d’images dans un cluster Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service le 30 juin 2022. La prise en charge de la création de nouveaux clusters Azure Red Hat OpenShift 3.11 se poursuit jusqu’au 30 novembre 2020. Après la mise hors service, les clusters Azure Red Hat OpenShift 3.11 restants seront arrêtés pour éviter des failles de sécurité.
> 
> Suivez ce guide pour [créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, n’hésitez pas à [nous contacter](mailto:arofeedback@microsoft.com).

Dans une plateforme OpenShift Container, les projets sont utilisés pour regrouper et isoler les objets associés. En tant qu’administrateur, vous pouvez autoriser les développeurs à accéder à des projets spécifiques, les autoriser à créer leurs propres projets et leur accorder des droits d’administration sur des projets particuliers.

## <a name="self-provisioning-projects"></a>Provisionnement automatique de projets

Vous pouvez autoriser les développeurs à créer leurs propres projets. Un point de terminaison d’API est chargé de provisionner un projet d’après une demande de projet d’un modèle nommé. La console web et la commande `oc new-project` utilisent ce point de terminaison quand un développeur crée un projet.

À la soumission d’une demande de projet, l’API remplace les paramètres suivants dans le modèle :

| Paramètre               | Description                                    |
| ----------------------- | ---------------------------------------------- |
| PROJECT_NAME            | Nom du projet. Obligatoire.             |
| PROJECT_DISPLAYNAME     | Nom d’affichage du projet. Peut être vide. |
| PROJECT_DESCRIPTION     | Description du projet. Peut être vide.  |
| PROJECT_ADMIN_USER      | Nom d’utilisateur de l’administrateur.       |
| PROJECT_REQUESTING_USER | Nom d’utilisateur du demandeur.           |

L’accès à l’API est accordé aux développeurs avec la liaison de rôle de cluster du provisionnement automatique. Cette fonctionnalité est proposée par défaut à tous les développeurs authentifiés.

## <a name="modify-the-template-for-a-new-project"></a>Modifier le modèle d’un nouveau projet 

1. Connectez-vous en tant qu’utilisateur avec des privilèges `customer-admin`.

2. Modifiez le modèle de demande de projet par défaut.

   ```
   oc edit template project-request -n openshift
   ```

3. Supprimez le modèle de projet par défaut du processus de mise à jour d’Azure Red Hat OpenShift (ARO) en ajoutant l’annotation suivante : `openshift.io/reconcile-protect: "true"`

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

   Le modèle de demande de projet ne sera pas mis à jour par le processus de mise à jour d’ARO. Les clients peuvent ainsi personnaliser le modèle et conserver leurs personnalisations quand le cluster est mis à jour.

## <a name="disable-the-self-provisioning-role"></a>Désactiver le rôle de provisionnement automatique

Vous pouvez empêcher un groupe d’utilisateurs authentifiés de provisionner automatiquement de nouveaux projets.

1. Connectez-vous en tant qu’utilisateur avec des privilèges `customer-admin`.

2. Modifiez la liaison de rôle de cluster du provisionnement automatique.

   ```
   oc edit clusterrolebinding.rbac.authorization.k8s.io self-provisioners
   ```

3. Supprimez le rôle du processus de mise à jour d’ARO en ajoutant l’annotation suivante : `openshift.io/reconcile-protect: "true"`.

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

4. Changez la liaison de rôle de cluster pour empêcher `system:authenticated:oauth` de créer des projets :

   ```
   apiVersion: rbac.authorization.k8s.io/v1
   groupNames:
   - osa-customer-admins
   kind: ClusterRoleBinding
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
     labels:
       azure.openshift.io/owned-by-sync-pod: "true"
     name: self-provisioners
   roleRef:
     name: self-provisioner
   subjects:
   - kind: SystemGroup
     name: osa-customer-admins
   ```

## <a name="manage-default-templates-and-imagestreams"></a>Gérer les modèles et les flux d’images par défaut

Dans Azure Red Hat OpenShift, vous pouvez désactiver les mises à jour pour tous les modèles et flux d’images par défaut dans l’espace de noms `openshift`.
Pour désactiver les mises à jour pour tous les éléments `Templates` et `ImageStreams` dans l’espace de noms `openshift` :

1. Connectez-vous en tant qu’utilisateur avec des privilèges `customer-admin`.

2. Modifiez l’espace de noms `openshift` :

   ```
   oc edit namespace openshift
   ```

3. Supprimez l’espace de noms `openshift` du processus de mise à jour d’ARO en ajoutant l’annotation suivante : `openshift.io/reconcile-protect: "true"`

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

   Tout objet dans l’espace de noms `openshift` peut être supprimé du processus de mise à jour en lui ajoutant une annotation `openshift.io/reconcile-protect: "true"`.

## <a name="next-steps"></a>Étapes suivantes

Essayez le tutoriel :
> [!div class="nextstepaction"]
> [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md)
