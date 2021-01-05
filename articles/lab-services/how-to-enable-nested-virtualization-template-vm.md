---
title: Activer la virtualisation imbriquée pour une machine virtuelle modèle dans Azure Lab Services | Microsoft Docs
description: Cet article explique comment configurer la virtualisation imbriquée sur une machine modèle dans Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 319695088e967dc3156ecab5c1b9458e77fcc186
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91251472"
---
# <a name="enable-nested-virtualization-on-a-template-virtual-machine-in-azure-lab-services"></a>Activer la virtualisation imbriquée pour une machine virtuelle modèle dans Azure Lab Services

Actuellement, Azure Lab Services vous permet de configurer un seul modèle de machine virtuelle dans un labo et de rendre disponible une seule copie pour chacun de vos utilisateurs. Si vous êtes formateur enseignant des classes de mise en réseau, de sécurité ou d’informatique, il peut être nécessaire de fournir à chacun de vos étudiants un environnement où plusieurs machines virtuelles peuvent communiquer entre elles sur un réseau.

La virtualisation imbriquée vous permet de créer un environnement avec plusieurs machines virtuelles au sein de la machine virtuelle d’un modèle de labo. La publication du modèle fournit à chaque utilisateur du labo une machine virtuelle configurée avec plusieurs machines virtuelles en son sein.  Cet article explique comment configurer la virtualisation imbriquée pour une machine modèle dans Azure Lab Services.

## <a name="what-is-nested-virtualization"></a>Qu’est-ce que la virtualisation imbriquée ?

La virtualisation imbriquée vous permet de créer des machines virtuelles au sein d’une machine virtuelle. La virtualisation imbriquée est effectuée via Hyper-V et elle est disponible seulement sur les machines virtuelles Windows.

Pour plus d’informations sur la visualisation imbriquée, consultez les articles suivants :

- [Virtualisation imbriquée dans Azure](https://azure.microsoft.com/blog/nested-virtualization-in-azure/)
- [Guide pratique pour activer la virtualisation imbriquée dans une machine virtuelle Azure](../virtual-machines/windows/nested-virtualization.md)

## <a name="considerations"></a>Considérations

Avant de configurer un laboratoire pour la virtualisation imbriquée, voici quelques éléments à prendre en compte.

- Lorsque vous créez un laboratoire, sélectionnez **Moyenne (virtualisation imbriquée)** ou **Grande (virtualisation imbriquée)** pour la taille de la machine virtuelle. Ces tailles de machine virtuelle prennent en charge la virtualisation imbriquée.
- Choisissez une taille qui fournira de bonnes performances pour les machines virtuelles clientes et les machines virtuelles hôtes.  N’oubliez pas que quand vous utilisez la virtualisation, la taille que vous choisissez doit être adaptée non seulement à la machine, mais également à l’hôte ainsi qu’aux machines Hyper-V s’exécutant simultanément.
- Les machines virtuelles clientes n’ont pas accès aux ressources Azure, telles que les serveurs DNS, sur réseau virtuel Azure.
- La machine virtuelle hôte nécessite que la configuration autorise la machine cliente à être connectée à Internet.
- Les machines virtuelles clientes sont concédées sous licence comme des machines indépendantes. Pour plus d’informations sur les licences des systèmes d’exploitation et des produits Microsoft, consultez [Licences Microsoft](https://www.microsoft.com/licensing/default). Vérifiez les contrats de licence de tous les autres logiciels que vous utilisiez avant de configurer la machine modèle.

## <a name="enable-nested-virtualization-on-a-template-vm"></a>Activer la virtualisation imbriquée sur une machine virtuelle modèle

Cet article suppose que vous avez déjà créé un compte de labo et un labo.  Pour plus d’informations sur la création d’un compte de labo, consultez le [tutoriel Configurer un compte de labo](tutorial-setup-lab-account.md). Pour plus d’informations sur la création d’un labo, consultez le [tutoriel Configurer un labo de salle de classe](tutorial-setup-classroom-lab.md).

>[!IMPORTANT]
>Sélectionnez **Grande (virtualisation imbriquée)** ou **Moyenne (virtualisation imbriquée)** pour la taille de la machine virtuelle lorsque vous créez le labo.  Sinon, la virtualisation imbriquée ne fonctionnera pas.  

Pour vous connecter à la machine modèle, consultez [Créer et gérer un modèle de salle de classe](how-to-create-manage-template.md).

Pour activer la virtualisation imbriquée, vous devez accomplir quelques tâches.  

- **Activation du rôle Hyper-V**. Le rôle Hyper-V doit être activé pour la création et l’exécution de machines virtuelles Hyper-V sur la machine virtuelle Lab Services.
- **Activation de DHCP**.  Quand le rôle DHCP est activé sur la machine virtuelle Lab Services, les machines virtuelles Hyper-V peuvent recevoir automatiquement une adresse IP.
- **Création d’un réseau NAT pour les machines virtuelles Hyper-V**.  Le réseau NAT est configuré pour permettre aux machines virtuelles Hyper-V d’accéder à Internet.  Les machines virtuelles Hyper-V peuvent communiquer entre elles.

>[!NOTE]
>Le réseau NAT créé sur la machine virtuelle Lab Services permet à une machine virtuelle Hyper-V d’accéder à Internet et à d’autres machines virtuelles Hyper-V sur la même machine virtuelle Lab Services.  La machine virtuelle Hyper-V ne peut pas accéder aux ressources Azure, telles que les serveurs DNS, sur le réseau virtuel Azure.

Vous pouvez accomplir les tâches ci-dessus à l’aide d’un script ou d’outils Windows.  Pour plus d’informations, lisez les sections ci-dessous.

### <a name="using-script-to-enable-nested-virtualization"></a>Utilisation d’un script pour activer la virtualisation imbriquée

Afin d’utiliser le programme d’installation automatisée pour la virtualisation imbriquée avec Windows Server 2016 ou Windows Server 2019, consultez [Activer la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services à l’aide d’un script](how-to-enable-nested-virtualization-template-vm-using-script.md). Vous utilisez les scripts de [Scripts Hyper-V Lab Services](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Scripts/HyperV) pour installer le rôle Hyper-V.  Les scripts configureront également la mise en réseau afin que les machines virtuelles Hyper-V puissent accéder à Internet.

### <a name="using-windows-tools-to-enable-nested-virtualization"></a>Utilisation des outils Windows pour activer la virtualisation imbriquée

Afin de configurer la virtualisation imbriquée pour Windows Server 2016 ou Windows Server 2019 à l’aide de rôles et d’outils d’administration Windows, consultez [Activer la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services manuellement](how-to-enable-nested-virtualization-template-vm-ui.md).  Les instructions expliqueront également comment configurer la mise en réseau afin que les machines virtuelles Hyper-V puissent accéder à Internet.
