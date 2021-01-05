---
title: Ressources prises en charge pour Azure Red Hat OpenShift 3.11
description: Découvrez les régions Azure et les tailles de machine virtuelle qui sont prises en charge par Microsoft Azure Red Hat OpenShift.
author: jimzim
ms.author: jzim
ms.service: container-service
ms.topic: conceptual
ms.date: 05/15/2019
ms.openlocfilehash: 38203cede755d776ba9142ad16e1bea1fd1637a6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82203673"
---
# <a name="azure-red-hat-openshift-resources"></a>Ressources Azure Red Hat OpenShift

Cette rubrique liste les régions Azure et les tailles de machine virtuelle qui sont prises en charge par le service Microsoft Azure Red Hat OpenShift 3.11.

## <a name="azure-regions"></a>Régions Azure

Pour obtenir la liste actuelle des régions où vous pouvez déployer des clusters Azure Red Hat OpenShift, consultez [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=openshift&regions=all).

## <a name="virtual-machine-sizes"></a>Tailles de machines virtuelles

Voici les tailles de machine virtuelle prises en charge que vous pouvez spécifier pour les nœuds de calcul dans votre cluster Azure Red Hat OpenShift.

> [!Important]
> Un nombre différent de disques peut être attachés à chaque machine virtuelle. Cela peut ne pas être aussi immédiatement évident que la taille de la mémoire ou de l’unité centrale.
> Certaines tailles de machine virtuelle ne sont pas disponibles dans toutes les régions. Même si l’API prend en charge la taille spécifiée, vous pouvez obtenir une erreur si cette taille n’est pas disponible dans la région que vous spécifiez.
> Pour plus d’informations, consultez la [liste actuelle des tailles de machine virtuelle prises en charge par région](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines).

## <a name="compute-node-sizes"></a>Tailles de nœud de calcul

Les tailles de nœud de calcul suivantes sont prises en charge par l’API REST Azure Red Hat OpenShift :

|Taille|Processeurs virtuels|RAM|
|-|-|-|
|Standard D4s v3|4|16 Go|
|Standard D8s v3|8|32 Go|
|Standard D16s v3|16|64 Go|
|Standard D32s v3|32|128 Go|
|-|-|-|
|Standard E4s v3|4|32 Go|
|Standard E8s v3|8|64 Go|
|Standard E16s v3|16|128 Go|
|Standard E32s v3|32|256 Go|
|-|-|-|
|Standard F8s v2|8|16 Go|
|Standard F16s v2|16|32 Go|
|Standard F32s v2|32|64 Go|

## <a name="master-node-sizes"></a>Tailles de nœud principal

Les tailles de nœud principal/d’infrastructure suivantes sont prises en charge par l’API REST Azure Red Hat OpenShift :

|Taille|Processeurs virtuels|RAM|
|-|-|-|
|Standard D4s v3|4|16 Go|
|Standard D8s v3|8|32 Go|
|Standard D16s v3|16|64 Go|
|Standard D32s v3|32|128 Go|

## <a name="next-steps"></a>Étapes suivantes

Essayez le tutoriel [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md).
