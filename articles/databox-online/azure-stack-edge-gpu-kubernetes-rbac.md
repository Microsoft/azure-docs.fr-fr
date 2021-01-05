---
title: Comprendre le contrôle d’accès en fonction du rôle Kubernetes sur les appareils Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment le contrôle d’accès en fonction du rôle Kubernetes se produit sur un appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: alkohli
ms.openlocfilehash: 9a9625dcf40ae7d11e1154fc89b7f04652c8ca16
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94635838"
---
# <a name="kubernetes-role-based-access-control-on-your-azure-stack-edge-pro-gpu-device"></a>Contrôle d’accès en fonction du rôle Kubernetes sur votre appareil Azure Stack Edge Pro avec GPU


Sur votre appareil Azure Stack Edge Pro, quand vous configurez le rôle de calcul, un cluster Kubernetes est créé. Vous pouvez utiliser le contrôle d’accès en fonction du rôle Kubernetes (Kubernetes RBAC) pour limiter l’accès aux ressources de cluster sur votre appareil.

Cet article fournit une vue d’ensemble du système Kubernetes RBAC fourni par Kubernetes et explique comment Kubernetes RBAC est implémenté sur votre appareil Azure Stack Edge Pro. 

## <a name="kubernetes-rbac"></a>Contrôle RBAC Kubernetes

Kubernetes RBAC vous permet d’affecter à des utilisateurs ou à des groupes d’utilisateurs, l’autorisation d’accomplir des opérations, telles que la création ou la modification de ressources, ou encore l’affichage de journaux d’activité à partir de charges de travail d’applications en cours d’exécution. La portée de ces autorisations peut être limitée à un seul espace de noms, ou accordée à l’ensemble du cluster. 

Quand vous configurez le cluster Kubernetes, un seul utilisateur correspondant à ce cluster est créé ; il est appelé utilisateur administrateur de cluster.  Un fichier `kubeconfig` est associé à l’utilisateur administrateur de cluster. Le fichier `kubeconfig` est un fichier texte qui contient toutes les informations de configuration requises pour se connecter au cluster afin d’authentifier l’utilisateur.

## <a name="namespaces-types"></a>Types d’espaces de noms

Les ressources Kubernetes, telles que les pods et les déploiements, sont regroupées logiquement dans un espace de noms. Ces regroupements permettent de scinder un cluster Kubernetes logiquement et de restreindre l’accès pour la création, l’affichage ou la gestion des ressources. Les utilisateurs ne peuvent interagir qu’avec les ressources appartenant aux espaces de noms qui leur sont attribués.

Les espaces de noms sont destinés à être utilisés dans les environnements où de nombreux utilisateurs sont répartis entre plusieurs équipes ou projets. Pour plus d’informations, consultez la section [Espace de noms Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).

Votre appareil Azure Stack Edge Pro a les espaces de noms suivants :

- **Espace de noms du système** : espace de noms où se trouvent les principales ressources, telles que les fonctionnalités réseau, comme le DNS et le proxy, ou bien le tableau de bord Kubernetes. En règle générale, vous ne déployez pas vos propres applications dans cet espace de noms. Utilisez cet espace de noms pour déboguer tous les problèmes de cluster Kubernetes. 

    Il y a plusieurs espaces de noms système sur votre appareil et les noms correspondant à ces espaces de noms système sont réservés. Voici une liste des espaces de noms système réservés : 
    - kube-system
    - metallb-system
    - dbe-namespace
    - default
    - kubernetes-dashboard
    - kube-node-lease
    - kube-public


    Veillez à ne pas utiliser de noms réservés pour les espaces de noms utilisateur que vous créez. 
<!--- **default namespace** - This namespace is where pods and deployments are created by default when none is provided and you have admin access to this namespace. When you interact with the Kubernetes API, such as with `kubectl get pods`, the default namespace is used when none is specified.-->

- **Espace de noms utilisateur** : il s’agit des espaces de noms que vous pouvez créer via **kubectl** ou via l’interface PowerShell de l’appareil pour déployer localement des applications.
 
- **Espace de noms IoT Edge** : vous vous connectez à cet espace de noms `iotedge` pour gérer les applications déployées via IoT Edge.

- **Espace de noms Azure Arc** : vous vous connectez à cet espace de noms `azure-arc` pour gérer les applications déployées via Azure Arc. Avec Azure Arc, vous pouvez également déployer des applications dans d’autres espaces de noms utilisateur. 

## <a name="namespaces-and-users"></a>Espaces de noms et utilisateurs

Dans le monde réel, il est important de diviser le cluster en plusieurs espaces de noms. 

- **Plusieurs utilisateurs** : Si vous avez plusieurs utilisateurs, plusieurs espaces de noms permettent à ces utilisateurs de déployer leurs applications et services dans leurs espaces de noms spécifiques de manière isolée les uns des autres. 
- **Utilisateur unique** : Même s’il existe un seul utilisateur, plusieurs espaces de noms permettent à cet utilisateur d’exécuter plusieurs versions des applications dans le même cluster Kubernetes.

### <a name="roles-and-rolebindings"></a>Rôles et RoleBindings

Kubernetes offre le concept de rôle et de liaison de rôle pour vous permettre de fournir des autorisations à l’utilisateur ou aux ressources au niveau de l’espace de noms et au niveau du cluster. 

- **Rôles** : Vous pouvez définir des autorisations pour les utilisateurs en tant que **Rôle**, puis utiliser les **Rôles** pour accorder des autorisations au sein d’un espace de noms. 
- **RoleBindings** : Une fois que vous avez défini les rôles, vous pouvez utiliser **RoleBindings** pour assigner des rôles pour un espace de noms donné. 

Cette approche vous permet de séparer logiquement un cluster Kubernetes unique, avec des utilisateurs uniquement capable d’accéder aux ressources d’application dans leur espace de noms attribué. 

## <a name="kubernetes-rbac-on-azure-stack-edge-pro"></a>Kubernetes RBAC sur Azure Stack Edge Pro

Dans l’implémentation actuelle de Kubernetes RBAC, Azure Stack Edge Pro vous permet d’effectuer les actions suivantes à partir d’une instance d’exécution PowerShell restreinte :

- Créer des espaces de noms.  
- Créez des utilisateurs supplémentaires.
- Accorder à votre administrateur l’accès aux espaces de noms que vous avez créés. N’oubliez pas que vous n’avez pas accès au rôle d’administrateur de cluster ou à une vue des ressources sur l’ensemble du cluster.
- Obtenez le fichier `kubeconfig` avec les informations permettant d’accéder au cluster Kubernetes.


L’appareil Azure Stack Edge Pro possède plusieurs espaces de noms système, et vous pouvez créer des espaces de noms utilisateur avec des fichiers `kubeconfig` pour accéder à ces espaces de noms. Les utilisateurs disposent d’un contrôle total sur ces espaces de noms et peuvent créer ou modifier des utilisateurs, ou accorder l’accès aux utilisateurs. Seul l’administrateur de cluster dispose d’un accès complet aux espaces de noms système et aux ressources sur l’ensemble du cluster. Un `aseuser` dispose d’un accès en lecture seule aux espaces de noms système.

Voici un diagramme qui illustre l’implémentation de Kubernetes RBAC sur les appareils Azure Stack Edge Pro.

![Kubernetes RBAC sur appareil Azure Stack Edge Pro](./media/azure-stack-edge-gpu-kubernetes-rbac/rbac-view-1.png)

Dans ce diagramme, Alice, Bob et Chuck ont accès uniquement aux espaces de noms utilisateur affectés, qui dans ce cas sont `ns1`, `ns2` et `ns3` respectivement. Au sein de ces espaces de noms, ils disposent d’un accès administrateur. En revanche, l’administrateur de cluster dispose d’un accès administrateur aux espaces de noms système et aux ressources sur l’ensemble du cluster.

En tant qu’utilisateur, vous pouvez créer des espaces de noms et des utilisateurs, attribuer des utilisateurs à des espaces de noms ou télécharger des fichiers `kubeconfig`. Pour obtenir des instructions pas à pas détaillées, consultez [Accéder au cluster Kubernetes via kuebctl sur votre appareil Edge Azure Stack Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md).


Lorsque vous travaillez avec des espaces de noms et des utilisateurs sur vos appareils Azure Stack Edge Pro, les avertissements suivants s’appliquent :

- Vous n’êtes pas autorisé à effectuer des opérations comme créer des utilisateurs, accorder ou révoquer l’accès à l’espace de noms à l’utilisateur pour tous les espaces de noms système. Les exemples d’espaces de noms système incluent `kube-system`, `metallb-system`, `kubernetes-dashboard`, `default`, `kube-node-lease` et `kube-public`. Les espaces de noms système incluent également les espaces de noms réservés pour les types de déploiement tels que `iotedge` (espace de noms IoT Edge) et `azure-arc` (espace de noms Azure Arc).
- Vous pouvez créer des espaces de noms utilisateur et, dans ces espaces de noms, créer des utilisateurs supplémentaires et accorder ou révoquer l’accès à l’espace de noms à ces utilisateurs.
- Vous n’êtes pas autorisé à créer des espaces de noms dont les noms sont identiques à ceux d’un espace de noms système. Les noms des espaces de noms système sont réservés.  
- Vous n’êtes pas autorisé à créer des espaces de noms utilisateur dont le nom est déjà utilisé par d’autres espaces de noms utilisateur. Par exemple, si vous avez créé un `test-ns`, vous ne pouvez pas créer un autre espace de noms `test-ns`.
- Vous n’êtes pas autorisé à créer des utilisateurs dont le nom est déjà réservé. Par exemple, `aseuser` est un utilisateur réservé et ne peut pas être utilisé.


## <a name="next-steps"></a>Étapes suivantes

Pour comprendre comment vous pouvez créer un utilisateur, créer un espace de noms et accorder à un utilisateur l’accès à l’espace de noms, consultez [Accéder à un cluster Kubernetes via kubectl](azure-stack-edge-gpu-create-kubernetes-cluster.md).

