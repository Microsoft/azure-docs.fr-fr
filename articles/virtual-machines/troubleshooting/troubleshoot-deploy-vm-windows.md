---
title: Résolution des problèmes de déploiement de la machine virtuelle Windows dans Azure | Microsoft Docs
description: Résolution des problèmes de déploiement de la machine virtuelle Windows dans le modèle de déploiement Azure Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 4e383427-4aff-4bf3-a0f4-dbff5c6f0c81
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 11/01/2018
ms.author: genli
ms.openlocfilehash: 8ed567ee20835cfe076c9b06234f2cba23f3a8bd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002554"
---
# <a name="troubleshoot-deploying-windows-virtual-machine-issues-in-azure"></a>Résolution des problèmes de déploiement de la machine virtuelle Windows dans Azure

Pour résoudre les problèmes de déploiement de la machine virtuelle (VM) dans Azure, passez en revue les [principaux problèmes](#top-issues) pour voir les erreurs courantes et les solutions.

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.

## <a name="top-issues"></a>Problèmes principaux
[!INCLUDE [virtual-machines-windows-troubleshoot-deploy-vm-top](../../../includes/virtual-machines-windows-troubleshoot-deploy-vm-top.md)]

## <a name="the-cluster-cannot-support-the-requested-vm-size"></a>Le cluster ne peut pas prendre en charge la taille de machine virtuelle demandée
\<properties
supportTopicIds="123456789"
resourceTags="windows"
productPesIds="1234, 5678"
/>
- Relancez la requête en utilisant une taille inférieure pour la machine virtuelle.
- Si la taille de la machine virtuelle requise ne peut pas être modifiée :
    - Arrêtez toutes les machines virtuelles dans le groupe à haute disponibilité. Cliquez sur **Groupes de ressources** > votre groupe de ressources > **Ressources** > votre groupe à haute disponibilité > **Machines virtuelles** > votre machine virtuelle > **Arrêter**.
    - Une fois que toutes les machines virtuelles sont arrêtées, créez une machine virtuelle à la taille souhaitée.
    - Démarrez la nouvelle machine virtuelle en premier, puis sélectionnez chacune des machines virtuelles arrêtées et cliquez sur Démarrer.


## <a name="the-cluster-does-not-have-free-resources"></a>Le cluster n’a pas de ressources libres
\<properties
supportTopicIds="123456789"
resourceTags="windows"
productPesIds="1234, 5678"
/>
- Relancez la requête ultérieurement.
- Si la nouvelle machine virtuelle peut faire partie d’un autre groupe à haute disponibilité
    - Créez une machine virtuelle dans un autre groupe à haute disponibilité (dans la même région).
    - Ajoutez la nouvelle machine virtuelle au même réseau virtuel.

## <a name="how-can-i-use-and-deploy-a-windows-client-image-into-azure"></a>Comment utiliser et déployer des images de client Windows dans Azure ?

Vous pouvez utiliser Windows 7, Windows 8 ou Windows 10 dans Azure pour des scénarios de développement /de test à condition de disposer d'un abonnement Visual Studio (anciennement MSDN) approprié. Cet [article](../windows/client-images.md) décrit les conditions d’éligibilité pour les clients Windows en cours d’exécution dans Azure et l’utilisation des images de galerie Azure.

## <a name="how-can-i-deploy-a-virtual-machine-using-the-hybrid-use-benefit-hub"></a>Comment déployer une machine virtuelle à l’aide du Hybrid Use Benefit (HUB) ?

Il existe de nombreuses manières de déployer des machines virtuelles Windows avec Azure Hybrid Use Benefit.

Un abonnement avec un contrat Entreprise :

•   Déployez des machines virtuelles à partir d’images de la Place de marché spécifiques qui sont préconfigurées avec Azure Hybrid Use Benefit.

Contrat Entreprise :

•  Téléchargez une machine virtuelle personnalisée et effectuez le déploiement à l’aide d’un modèle Resource Manager ou d’Azure PowerShell.

Pour plus d’informations, consultez les ressources suivantes :

 - [Présentation d’Azure Hybrid Use Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/)

 - [FAQ téléchargeable](https://download.microsoft.com/download/4/2/1/4211AC94-D607-4A45-B472-4B30EDF437DE/Windows_Server_Azure_Hybrid_Use_FAQ_EN_US.pdf)

 - [Azure Hybrid Use Benefit pour Windows Server et client Windows](../windows/hybrid-use-benefit-licensing.md).

 - [Utilisation de Hybrid Use Benefit dans Azure](/archive/blogs/azureedu/how-can-i-use-the-hybrid-use-benefit-in-azure)

## <a name="how-do-i-activate-my-monthly-credit-for-visual-studio-enterprise-bizspark"></a>Comment activer mon crédit mensuel pour Visual Studio Enterprise (BizSpark) ?

Pour activer votre crédit mensuel, consultez cet [article](https://azure.microsoft.com/offers/ms-azr-0064p/).

## <a name="how-to-add-enterprise-devtest-to-my-enterprise-agreement-ea-to-get-access-to-window-client-images"></a>Comment ajouter Enterprise Dev/Test à mon Contrat Entreprise (EA) pour accéder aux images de client Windows ?

La possibilité de créer des abonnements basés sur l’offre Enterprise Dev/Test est limitée aux propriétaires de comptes qui y ont été autorisés par un administrateur d’entreprise. Le propriétaire de compte crée les abonnements par le biais du portail des comptes Azure. Il doit ensuite ajouter les abonnés Visual Studio actifs en tant que coadministrateurs. Ils pourront alors gérer et utiliser les ressources nécessaires au développement et au test. Pour plus d’informations, consultez [Enterprise Dev/Test](https://azure.microsoft.com/offers/ms-azr-0148p/).

## <a name="my-drivers-are-missing-for-my-windows-n-series-vm"></a>Il manque des pilotes sur ma machine virtuelle Windows Série N

Vous trouverez des instructions relatives à l’installation des pilotes pour machines virtuelles Windows [ici](../sizes-gpu.md#supported-operating-systems-and-drivers).

## <a name="i-cant-find-a-gpu-instance-within-my-n-series-vm"></a>Impossible de trouver une instance GPU dans ma machine virtuelle Série N

Pour tirer parti des fonctionnalités GPU des machines virtuelles de la série N, vous devez installer un pilote graphique sur chaque machine virtuelle après le déploiement. Vous trouverez des informations sur la configuration des pilotes [ici](../sizes-gpu.md#supported-operating-systems-and-drivers).

## <a name="are-n-series-vms-available-in-my-region"></a>Les machines virtuelles de la série N sont-elles disponibles dans ma région ?

Vous pouvez vérifier la disponibilité à l’aide de la [Table des produits disponibles par région](https://azure.microsoft.com/regions/services) ainsi que les tarifications [ici](https://azure.microsoft.com/pricing/details/virtual-machines/series/#n-series).

## <a name="what-client-images-can-i-use-and-deploy-in-azure-and-how-to-i-get-them"></a>Quelles sont les images de client que je peux utiliser et déployer dans Azure, et comment les obtenir ?

Vous pouvez utiliser Windows 7, Windows 8 ou Windows 10 dans Azure pour des scénarios de développement / de test à condition de disposer d'un abonnement Visual Studio (anciennement MSDN) approprié. 

- Les images Windows 10 sont disponibles dans la galerie Azure sous [eligible dev/test offers](../windows/client-images.md#eligible-offers). 
- Les abonnés Visual Studio dans n’importe quel type d’offre peuvent également [préparer et créer correctement](../windows/prepare-for-upload-vhd-image.md) une image 64 bits de Windows 7, Windows 8 ou Windows 10, puis la [charger dans Azure](../windows/upload-generalized-managed.md). L’utilisation reste limitée au développement/test par les abonnés Visual Studio actifs.

Cet [article](../windows/client-images.md) décrit les conditions d’éligibilité pour les clients Windows en cours d’exécution dans Azure et l’utilisation des images de galerie Azure.

## <a name="i-am-not-able-to-see-vm-size-family-that-i-want-when-resizing-my-vm"></a>Impossible de voir les différentes familles de taille de machines virtuelles quand je redimensionne ma machine virtuelle.

Lorsqu’un ordinateur virtuel est en cours d’exécution, il est déployé sur un serveur physique. Les serveurs physiques dans les régions Azure sont regroupés dans des clusters de matériel physique commun. La méthode pour redimensionner une machine virtuelle qui doit être transférée vers différents clusters de matériel physique peut varier en fonction du modèle de déploiement utilisé pour déployer la machine virtuelle.

- Si les machines virtuelles sont déployées à partir d’un modèle de déploiement classique, le déploiement du service cloud doit être supprimé et redéployé afin de changer la taille de la machine virtuelle pour une autre famille de taille.

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

- Si les machines virtuelles sont déployées à partir du modèle de déploiement Resource Manager, vous devez arrêter toutes les machines virtuelles dans le groupe à haute disponibilité avant de changer la taille d’une machine dans le groupe à haute disponibilité.

## <a name="the-listed-vm-size-is-not-supported-while-deploying-in-availability-set"></a>La taille de machine virtuelle répertoriée n’est pas prise en charge lors du déploiement dans le groupe à haute disponibilité.

Choisissez une taille prise en charge par le cluster du groupe à haute disponibilité. Lors de la création d’un groupe à haute disponibilité, il est recommandé de choisir la plus grande taille de machine virtuelle dont vous pensez avoir besoin. Ce sera votre premier déploiement dans le groupe à haute disponibilité.

## <a name="can-i-add-an-existing-classic-vm-to-an-availability-set"></a>Est-il possible d’ajouter une machine virtuelle classique à un groupe à haute disponibilité ?

Oui. Vous pouvez ajouter une machine virtuelle classique existante à un nouveau groupe ou à un groupe à haute disponibilité déjà existant. Pour plus d’informations, consultez [Ajouter une machine virtuelle existante à un groupe à haute disponibilité](/previous-versions/azure/virtual-machines/windows/classic/configure-availability-classic#addmachine).


## <a name="next-steps"></a>Étapes suivantes
Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/).

Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.
