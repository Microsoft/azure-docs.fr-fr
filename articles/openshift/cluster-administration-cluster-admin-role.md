---
title: Rôle d'administrateur de cluster Azure Red Hat OpenShift | Microsoft Docs
description: Attribution et utilisation du rôle d’administrateur de cluster Azure Red Hat OpenShift
services: container-service
author: mjudeikis
ms.author: jzim
ms.service: container-service
ms.topic: article
ms.date: 09/25/2019
ms.openlocfilehash: f8b84461cc96c64ce2f97a8b4bb69246f5a0dccc
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221177"
---
# <a name="azure-red-hat-openshift-customer-administrator-role"></a>Rôle d'administrateur de client Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service le 30 juin 2022. La prise en charge de la création de nouveaux clusters Azure Red Hat OpenShift 3.11 se poursuit jusqu’au 30 novembre 2020. Après la mise hors service, les clusters Azure Red Hat OpenShift 3.11 restants seront arrêtés pour éviter des failles de sécurité.
> 
> Suivez ce guide pour [créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, n’hésitez pas à [nous contacter](mailto:arofeedback@microsoft.com).

Vous êtes l’administrateur d’un cluster Azure Red Hat OpenShift. Votre compte dispose de plus en plus d’autorisations et d’un accès à tous les projets créés par l’utilisateur.

Quand le rôle d’autorisation customer-admin-cluster est lié à votre compte, celui-ci peut automatiquement gérer un projet.

> [!Note] 
> Le rôle de cluster customer-admin-cluster est différent de celui de cluster-admin.

Par exemple, vous pouvez exécuter des actions associées à un ensemble de verbes (`create`) pour agir sur un ensemble de noms de ressources (`templates`). Pour afficher les détails de ces rôles et de leurs ensembles de verbes et de ressources, exécutez la commande suivante :

`$ oc get clusterroles customer-admin-cluster -o yaml`

Les noms de verbes ne sont pas nécessairement tous mappés directement aux commandes `oc`. Ils s’assimilent plus généralement aux types d’opérations CLI que vous pouvez effectuer. 

Par exemple, le fait de disposer du verbe `list` signifie que vous pouvez afficher la liste de tous les objets d’un nom de ressource (`oc get`). Le verbe `get` signifie que vous pouvez afficher les détails d’un objet spécifique si vous en connaissez le nom (`oc describe`).

## <a name="configure-the-customer-administrator-role"></a>Configurer le rôle d’administrateur de client

Vous pouvez configurer le rôle de cluster uniquement au moment de créer le cluster en fournissant l’indicateur `--customer-admin-group-id`. Ce champ n’est actuellement pas configurable dans le portail Azure. Pour savoir comment configurer Azure Active Directory et le groupe Administrateurs, consultez [Intégration d’Azure Active Directory pour Azure Red Hat OpenShift](howto-aad-app-configuration.md).

## <a name="confirm-membership-in-the-customer-administrator-role"></a>Confirmer une appartenance au rôle d’administrateur de client

Pour confirmer votre appartenance au groupe d’administration de client, essayez les commandes CLI OpenShift `oc get nodes` ou `oc projects`. `oc get nodes` affiche une liste de nœuds si vous avez le rôle customer-admin-cluster, et une erreur d’autorisation si vous avez uniquement le rôle customer-admin-project. `oc projects` affiche tous les projets figurant dans le cluster, par opposition aux seuls projets auxquels vous collaborez.

Pour approfondir l’exploration des rôles et des autorisations dans votre cluster, vous pouvez utiliser la commande [`oc policy who-can <verb> <resource>`](https://docs.openshift.com/container-platform/3.11/admin_guide/manage_rbac.html#managing-role-bindings).

## <a name="next-steps"></a>Étapes suivantes

Configurez le rôle de cluster customer-admin-cluster :
> [!div class="nextstepaction"]
> [Intégration d’Azure Active Directory pour Azure Red Hat OpenShift](howto-aad-app-configuration.md)
