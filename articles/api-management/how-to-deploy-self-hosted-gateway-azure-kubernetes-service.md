---
title: Déployer une passerelle auto-hébergée sur Azure Kubernetes Service | Microsoft Docs
description: Découvrez comment déployer la passerelle auto-hébergée de la Gestion des API Azure sur Azure Kubernetes Service.
services: api-management
documentationcenter: ''
author: miaojiang
manager: gwallace
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 04/26/2020
ms.author: apimpm
ms.openlocfilehash: 02962e9c5be2c4b73d121a53a7b595c573ad6cd0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87015219"
---
# <a name="deploy-to-azure-kubernetes-service"></a>Déployer sur Azure Kubernetes Service

Cet article explique comment déployer la passerelle auto-hébergée de la Gestion des API Azure sur [Azure Kubernetes Service](https://azure.microsoft.com/services/kubernetes-service/). Pour déployer la passerelle auto-hébergée sur un cluster Kubernetes, consultez [ce document](how-to-deploy-self-hosted-gateway-kubernetes.md).

## <a name="prerequisites"></a>Prérequis

- [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md)
- [Créer un cluster Azure Kubernetes](../aks/kubernetes-walkthrough-portal.md)
- [Provisionnez une ressource de passerelle dans votre instance de gestion des API](api-management-howto-provision-self-hosted-gateway.md).

## <a name="deploy-the-self-hosted-gateway-to-aks"></a>Déployer une passerelle auto-hébergée sur AKS

1. Sélectionnez **Gateways** (Passerelles) sous **Deployment and infrastructure** (Déploiement et infrastructure).
2. Sélectionnez la ressource de passerelle auto-hébergée que vous souhaitez déployer.
3. Sélectionnez **Déploiement**.
4. Notez qu’un nouveau jeton dans la zone de texte **Jeton** a été généré automatiquement pour vous avec les valeurs par défaut de **Expiration** et **Clé secrète**. Ajustez l’un ou l’autre, ou les deux, si vous le souhaitez, puis sélectionnez **Générer** pour créer un autre jeton.
5. Assurez-vous que **Kubernetes** est sélectionné sous **Scripts de déploiement**.
6. Sélectionnez le lien du fichier **<nom-passerelle>.yml** en regard de **Déploiement** pour télécharger le fichier.
7. Ajustez les mappages de port et le nom de conteneur dans le fichier yml, si nécessaire.
8. Selon votre scénario, vous pouvez avoir besoin de changer le [type de service](../aks/concepts-network.md#services). La valeur par défaut est `NodePort`.
9. Sélectionnez l’icône **Copier** située à l’extrémité droite de la zone de texte **Déployer** pour enregistrer la commande `kubectl` dans le Presse-papiers.
10. Collez la commande dans la fenêtre (ou la commande) du terminal. Notez que la commande s’attend à ce que le fichier d’environnement téléchargé soit présent dans le répertoire actif.
```console
    kubectl apply -f <gateway-name>.yaml
```
11. Exécutez la commande. La commande indique à votre cluster AKS d’exécuter le conteneur à l’aide de l’image de la passerelle auto-hébergée qui a été téléchargée à partir de Microsoft Container Registry, et de configurer le conteneur pour qu’il expose les ports HTTP (8080) et HTTPS (443).
12. Exécutez la commande ci-dessous pour vérifier que le pod de la passerelle est en cours d’exécution. Notez que le nom de votre pod va être différent.
```console
kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
contoso-apim-gateway-59f5fb94c-s9stz   1/1       Running   0          1m
```
13. Exécutez la commande ci-dessous pour vérifier que le service de passerelle est en cours d’exécution. Notez que le nom de votre service et les adresses IP seront différents.
```console
kubectl get services
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
contosogateway   NodePort    10.110.230.87   <none>        80:32504/TCP,443:30043/TCP   1m
```
14. Revenez au portail Azure et vérifiez que le nœud de la passerelle que vous venez de déployer signale l’état d’intégrité.

> [!TIP]
> Utilisez la commande <code>kubectl logs <gateway-pod-name></code> pour afficher un instantané du journal de la passerelle auto-hébergée.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la passerelle auto-hébergée, consultez [Vue d’ensemble de la passerelle auto-hébergée Gestion des API](self-hosted-gateway-overview.md)
* En savoir plus sur [Azure Kubernetes Service](../aks/intro-kubernetes.md)
* Découvrez comment [configurer et conserver les journaux dans le cloud](how-to-configure-cloud-metrics-logs.md).
* * Découvrez comment [configurer et conserver les journaux localement](how-to-configure-local-metrics-logs.md).
