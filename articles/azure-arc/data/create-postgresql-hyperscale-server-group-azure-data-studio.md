---
title: Créer un PostgreSQL Hyperscale activé par Azure Arc à l’aide d’Azure Data Studio
description: Créer un PostgreSQL Hyperscale activé par Azure Arc à l’aide d’Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: e2007d8f0c558d35c0507b6e12bce6d6777fad52
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92310895"
---
# <a name="create-azure-arc-enabled-postgresql-hyperscale-using-azure-data-studio"></a>Créer un PostgreSQL Hyperscale activé par Azure Arc à l’aide d’Azure Data Studio

Ce document vous guide tout au long des étapes d’utilisation d’Azure Data Studio pour configurer des groupes de serveurs PostgreSQL Hyperscale activés pour Azure Arc.

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="connect-to-the-azure-arc-data-controller"></a>Se connecter au contrôleur de données Azure Arc

Avant de pouvoir créer une instance, connectez-vous au contrôleur de données Azure Arc si vous ne l’avez pas encore fait.

```console
azdata login
```

Vous êtes ensuite invité à entrer l’espace de noms dans lequel le contrôleur de données est déployé, le nom d’utilisateur et le mot de passe pour se connecter au contrôleur.

> Si vous devez valider l’espace de noms, vous pouvez exécuter ```kubectl get pods -A``` pour obtenir une liste de tous les espaces de noms sur le cluster.

```console
Username: arcadmin
Password:
Namespace: arc
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="preliminary-and-temporary-step-for-openshift-users-only"></a>Étape préliminaire et temporaire pour les utilisateurs OpenShift uniquement

Implémentez cette étape avant de passer à l’étape suivante. Pour déployer le groupe de serveurs PostgreSQL Hyperscale sur Red Hat OpenShift dans un projet autre que celui par défaut, vous devez exécuter les commandes suivantes sur votre cluster pour mettre à jour les contraintes de sécurité. Cette commande accorde les privilèges nécessaires aux comptes de service qui exécuteront votre groupe de serveurs PostgreSQL Hyperscale. La contrainte de contexte de sécurité (SCC) **_arc-data-scc_** est celle que vous avez ajoutée lors du déploiement du contrôleur de données Azure Arc.

```console
oc adm policy add-scc-to-user arc-data-scc -z <server-group-name> -n <namespace name>
```

_**Server-Group-Name** est le nom du groupe de serveurs que vous allez déployer à l’étape suivante._
   
Pour plus d’informations sur SCC dans OpenShift, consultez la documentation [OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html).
Vous pouvez maintenant implémenter l’étape suivante.

## <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Créer un groupe de serveurs PostgreSQL Hyperscale compatible avec Azure Arc

1. Lancer Azure Data Studio
1. Sous l’onglet Connexions, cliquez sur les trois points situés en haut à gauche, puis choisissez Nouveau déploiement
1. Dans les options de déploiement, sélectionnez **Groupe de serveurs PostgreSQL Hyperscale - Azure Arc**
    >[!NOTE]
    > Vous pouvez être invité à installer l’outil [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] ici s’il n’est pas actuellement installé.
1. Acceptez la déclaration de confidentialité et les termes du contrat de licence, puis cliquez sur **Sélectionner** en bas
1. Dans le panneau Déployer le groupe de serveurs PostgreSQL Hyperscale - Azure Arc, entrez les informations suivantes :
   - Entrez un nom pour le groupe de serveur
   - Entrez et confirmez un mot de passe pour l'utilisateur administrateur de _postgres_ du groupe de serveurs
   - Sélectionnez la classe de stockage appropriée pour les données
   - Sélectionnez la classe de stockage appropriée pour les journaux
   - Sélectionnez la classe de stockage appropriée pour les sauvegardes
   - Sélectionnez le nombre de nœuds Worker à approvisionner
1. Cliquez sur le bouton **Déployer**

Cela démarre la création du groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc sur le contrôleur de données.

Votre création ne devrait prendre que quelques minutes.

## <a name="next-steps"></a>Étapes suivantes
- [Gérer votre groupe de serveurs à l’aide d’Azure Data Studio](manage-postgresql-hyperscale-server-group-with-azure-data-studio.md)
- [Surveiller votre groupe de serveurs](monitor-grafana-kibana.md)
- Lisez les concepts et les guides pratiques d’Azure Database pour PostgreSQL Hyperscale pour distribuer vos données sur plusieurs nœuds PostgreSQL Hyperscale et tirer parti de toute la puissance d’Azure Database pour Postgres Hyperscale. :
    * [Nœuds et tables](../../postgresql/concepts-hyperscale-nodes.md)
    * [Déterminer le type d’application](../../postgresql/concepts-hyperscale-app-type.md)
    * [Choisir une colonne de distribution](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocalisation de table](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribuer et modifier des tables](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Concevoir une base de données multilocataire](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Concevoir un tableau de bord d’analytique en temps réel](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* Dans les documents ci-dessus, ignorez les sections **Se connecter au portail Azure** et **Créer un serveur Azure Database pour PostgreSQL - Hyperscale (Citus)** . Implémentez les étapes restantes dans votre déploiement Azure Arc. Ces sections sont spécifiques à Azure Database pour PostgreSQL Hyperscale (Citus) proposé en tant que service PaaS dans le cloud Azure, mais les autres parties des documents s’appliquent directement à votre PostgreSQL Hyperscale activé pour Azure Arc.

- [Effectuer un scale-out de votre groupe de serveurs Azure Database pour PostgreSQL Hyperscale](scale-out-postgresql-hyperscale-server-group.md)
- [Configuration de stockage et concepts de stockage Kubernetes](storage-configuration.md)
- [Modèle de ressources Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

