---
title: Mettre à niveau le contrôleur d’entrée avec Helm
description: Cet article fournit des informations sur la façon de mettre à niveau une entrée Application Gateway à l’aide de Helm.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: f20302a4993da1754255254ce6d69c000750d4ab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84806778"
---
# <a name="how-to-upgrade-application-gateway-ingress-controller-using-helm"></a>Comment mettre à niveau le contrôleur d’entrée d’Application Gateway à l’aide de Helm 

Le contrôleur d’entrée d’Azure Application Gateway pour Kubernetes (AGIC) peut être mis à niveau à l’aide d’un référentiel Helm hébergé sur Stockage Azure.

Avant de commencer la procédure de mise à niveau, assurez-vous que vous avez ajouté le référentiel requis :

- Affichez vos référentiels Helm actuellement ajoutés avec :

    ```bash
    helm repo list
    ```

- Ajoutez le référentiel AGIC avec :

    ```bash
    helm repo add \
        application-gateway-kubernetes-ingress \
        https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
    ```

## <a name="upgrade"></a>Mettre à niveau

1. Actualisez le référentiel Helm AGIC pour bénéficier de la dernière version :

    ```bash
    helm repo update
    ```

1. Affichez les versions disponibles du graphique `application-gateway-kubernetes-ingress` :

    ``` bash
    helm search -l application-gateway-kubernetes-ingress
    ```

    Exemple de réponse :

    ```bash
    NAME                                                    CHART VERSION   APP VERSION     DESCRIPTION
    application-gateway-kubernetes-ingress/ingress-azure    0.7.0-rc1       0.7.0-rc1       Use Azure Application Gateway as the ingress for an Azure...
    application-gateway-kubernetes-ingress/ingress-azure    0.6.0           0.6.0           Use Azure Application Gateway as the ingress for an Azure...
    ```

    La dernière version disponible dans la liste ci-dessus est la suivante : `0.7.0-rc1`

1. Affichez les graphiques Helm actuellement installés :

    ```bash
    helm list
    ```

    Exemple de réponse :

    ```bash
    NAME            REVISION        UPDATED                         STATUS  CHART                   APP VERSION     NAMESPACE
    odd-billygoat   22              Fri Jun 21 15:56:06 2019        FAILED  ingress-azure-0.7.0-rc1 0.7.0-rc1       default
    ```

    L’installation du graphique Helm à partir de l’exemple de réponse ci-dessus est nommée `odd-billygoat`. Nous utiliserons ce nom pour le reste des commandes. Le nom réel de votre déploiement sera probablement différent.

1. Mettez à niveau le déploiement Helm vers une nouvelle version :

    ```bash
    helm upgrade \
        odd-billygoat \
        application-gateway-kubernetes-ingress/ingress-azure \
        --version 0.9.0-rc2
    ```

## <a name="rollback"></a>Restauration

Si le déploiement Helm échoue, vous pouvez restaurer une version précédente.

1. Obtenez le numéro de la dernière version saine connue :

    ```bash
    helm history odd-billygoat
    ```

    Exemple de sortie :

    ```bash
    REVISION        UPDATED                         STATUS          CHART                   DESCRIPTION
    1               Mon Jun 17 13:49:42 2019        DEPLOYED        ingress-azure-0.6.0     Install complete
    2               Fri Jun 21 15:56:06 2019        FAILED          ingress-azure-xx        xxxx
    ```

    Dans l’exemple de sortie de la commande `helm history`, il semble que le dernier déploiement réussi de notre `odd-billygoat` soit la révision `1`

1. Restaurer à la dernière révision réussie :

    ```bash
    helm rollback odd-billygoat 1
    ```
