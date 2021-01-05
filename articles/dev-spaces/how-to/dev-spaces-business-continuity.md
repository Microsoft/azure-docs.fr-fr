---
title: Continuité d’activité et reprise d’activité
services: azure-dev-spaces
author: AlexandraKemperMS
ms.author: alkemper
ms.date: 01/28/2019
ms.topic: conceptual
description: Apprenez à utiliser Azure Dev Spaces et Azure Kubernetes Service pour assurer la continuité des activités et préparer la récupération d’urgence.
keywords: 'Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs, Helm, service Mesh, routage du service Mesh, kubectl, k8s '
manager: gwallace
ms.openlocfilehash: 77425d294dcea0546259b152b793cec291940c53
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929597"
---
# <a name="business-continuity-and-disaster-recovery-in-azure-dev-spaces"></a>Continuité d’activité et reprise d’activité dans Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

## <a name="review-disaster-recovery-guidance-for-azure-kubernetes-service-aks"></a>Passer en revue les conseils sur la reprise d’activité pour Azure Kubernetes Service (AKS)

Azure Dev Spaces est une fonctionnalité d’Azure Kubernetes Service. Vous devez connaître les recommandations relatives à la reprise d’activité dans AKS et déterminer si elles s’appliquent aux clusters AKS que vous utilisez pour Dev Spaces. Pour plus d’informations, reportez-vous à [Bonnes pratiques pour la continuité d’activité et la reprise d’activité dans AKS (Azure Kubernetes Services)](../../aks/operator-best-practices-multi-region.md)

## <a name="enable-dev-spaces-on-aks-clusters-in-different-regions"></a>Activer Dev Spaces sur des clusters AKS dans différentes régions

L’activation de Dev Spaces sur des clusters AKS dans différentes régions vous permet de reprendre l’utilisation de Dev Spaces juste après un incident dans une région Azure.

Pour obtenir des informations générales sur les déploiements d’AKS dans plusieurs régions, consultez [Planifier le déploiement multi-région](../../aks/operator-best-practices-multi-region.md#plan-for-multiregion-deployment)

### <a name="enable-dev-spaces-via-the-azure-cli"></a>Activer Dev Spaces via Azure CLI

Vous pouvez également activer Dev Spaces au niveau de la ligne de commande :

```azurecli
az aks use-dev-spaces -g <resource group name> -n <cluster name>
```

## <a name="deploy-your-teams-baseline-to-each-cluster"></a>Déployer la base de référence de votre équipe sur chaque cluster

Lorsque vous utilisez Dev Spaces, vous déployez généralement l’intégralité de l’application dans un espace de développement parent sur votre cluster Kubernetes. Par défaut, l’espace `default` est utilisé. Le déploiement initial inclut tous les services, ainsi que les ressources externes dont ces services dépendent, telles que les bases de données ou les files d’attente. Cela s’appelle la *base de référence*. Une fois que vous avez configuré une base de référence dans l’espace de développement parent, vous procédez à une itération et déboguez les services individuels dans les espaces de développement enfant.

Vous devez déployer les versions les plus récentes de votre ensemble de services de base de référence sur des clusters dans plusieurs régions. Mettre à jour vos services de base de référence de cette façon garantit que vous pouvez continuer à utiliser Dev Spaces en cas d’incident dans une région Azure. Par exemple, si vous déployez votre base de référence via un pipeline CI/CD, vous devez modifier le pipeline afin qu’il soit déployé sur plusieurs clusters dans différentes régions.

## <a name="select-the-correct-aks-cluster-to-use-for-dev-spaces"></a>Sélectionner le cluster AKS correct à utiliser pour Dev Spaces

Une fois que vous avez correctement configuré un cluster de sauvegarde exécutant la base de référence de votre équipe, vous pouvez rapidement basculer sur le cluster de sauvegarde à tout moment. Ensuite, vous pouvez réexécuter les services individuels sur lesquels vous travaillez dans les espaces de développement enfants.

Sélectionnez un autre cluster avec la commande CLI suivante :

```azurecli
az aks use-dev-spaces -g <new resource group name> -n <new cluster name>
```

Vous pouvez répertorier les espaces de développement disponibles sur le nouveau cluster avec la commande suivante :

```cmd
azds space list
```

Vous pouvez créer un nouvel espace de développement à utiliser ou en sélectionner un existant à l’aide de la commande suivante :

```cmd
azds space select -n <space name>
```

Après avoir exécuté ces commandes, le cluster sélectionné et l’espace de développement seront utilisés pour les opérations CLI suivantes et pour le débogage de projets à l’aide de l’extension de Visual Studio Code pour Azure Dev Spaces.

Si vous utilisez Visual Studio, vous pouvez changer le cluster utilisé par un projet existant avec la procédure suivante :

1. Ouvrez votre projet dans Visual Studio.
1. Cliquez avec le bouton droit sur le nom du projet dans l’Explorateur de solutions, puis cliquez sur **Propriétés**.
1. Dans le volet gauche, cliquez sur **Déboguer**.
1. Dans la page de propriétés de débogage, cliquez sur la liste déroulante **Profil** et choisissez **Azure Dev Spaces**.
1. Cliquez sur le bouton **Modifier**.
1. Dans la boîte de dialogue qui s’affiche, sélectionnez le cluster AKS que vous souhaitez utiliser. Si vous le souhaitez, choisissez un espace de développement différent à utiliser, ou créez-en un en sélectionnant l’option appropriée dans la liste déroulante **Espace**.

Une fois que vous avez sélectionné le cluster et l’espace appropriés, appuyez sur F5 pour exécuter le service dans Dev Spaces.

Répétez ces étapes pour tous les autres projets configurés pour utiliser le cluster d’origine.

## <a name="access-a-service-on-a-backup-cluster"></a>Accéder à un service sur un cluster de sauvegarde

Si vous avez configuré votre service pour qu’il utilise un nom DNS public, le service aura une URL différente si vous l’exécutez sur un cluster de sauvegarde. Les noms DNS publics sont toujours au format `<space name>.s.<root space name>.<service name>.<cluster GUID>.<region>.azds.io`. Si vous basculez vers un autre cluster, le GUID du cluster et éventuellement la région sont modifiés.

Dev Spaces affiche toujours l’URL correcte du service lors de l’exécution de `azds up`, ou dans la fenêtre Sortie dans Visual Studio sous **Azure Dev Spaces**.

Vous pouvez également trouver l’URL en exécutant la commande `azds list-uris` :
```
$ azds list-uris
Uri                                                     Status
------------------------------------------------------  ---------
http://default.mywebapi.d05afe7e006a4fddb73c.eus.azds.io/  Available
```

Utilisez cette URL lors de l’accès au service.
