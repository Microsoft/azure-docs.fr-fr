---
title: Console série Azure | Microsoft Docs
description: La console série Azure vous permet de vous connecter à votre machine virtuelle lorsque SSH ou RDP n’est pas disponible.
services: virtual-machines
documentationcenter: ''
author: asinn826
manager: borisb
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm
ms.workload: infrastructure-services
ms.date: 02/10/2020
ms.author: alsin
ms.openlocfilehash: 28c5a3085d84b25deb7c5ee09a9c9cc4d7a06819
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87374063"
---
# <a name="azure-serial-console"></a>Console série Azure

La console série du portail Azure permet aux machines virtuelles et aux instances de groupe de machines virtuelles identiques exécutant Linux ou Windows d’accéder à une console texte. Cette connexion série s’effectue par le biais du port série ttyS0 ou COM1 de la machine virtuelle ou de l’instance du groupe de machines virtuelles identiques. Elle fournit un accès qui n’est pas lié au réseau ni à l’état du système d’exploitation. La console série est accessible seulement avec le portail Azure et est autorisée seulement pour les utilisateurs disposant d’un rôle d’accès de contributeur ou supérieur à la machine virtuelle ou au groupe de machines virtuelles identiques.

La console série fonctionne de la même manière pour les machines virtuelles et les instances de groupe de machines virtuelles identiques. Dans ce document, toutes les mentions aux machines virtuelles incluent implicitement les instances de groupe de machines virtuelles identiques, sauf indication contraire.

La console série est généralement disponible dans les régions Azure mondiales et en préversion publique dans Azure Government. Elle n’est pas encore disponible dans le cloud Azure Chine.

## <a name="prerequisites-to-access-the-azure-serial-console"></a>Prérequis pour l’accès à la console série Azure
Pour accéder à la console série sur votre machine virtuelle ou instance de groupe de machines virtuelles identiques, vous devez disposer des éléments suivants :

- Les diagnostics de démarrage doivent être activés pour la machine virtuelle.
- Un compte utilisateur qui utilise l’authentification par mot de passe doit exister dans la machine virtuelle. Vous pouvez créer un utilisateur basé sur mot de passe avec la fonction [Réinitialiser le mot de passe](../extensions/vmaccess.md#reset-password) de l’extension d’accès aux machines virtuelles. Sélectionnez **Réinitialiser le mot de passe** dans la section **Support + dépannage**.
- Le compte Azure qui accède à la console série doit disposer du [rôle Contributeur de machine virtuelle](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) pour la machine virtuelle et le compte de stockage des [diagnostics de démarrage](boot-diagnostics.md)
- Les déploiements classiques ne sont pas pris en charge. Votre machine virtuelle ou votre instance de groupe de machines virtuelles identiques doit utiliser le modèle de déploiement Azure Resource Manager.

> [!NOTE]
> La console série est actuellement incompatible avec un compte de stockage de diagnostics à démarrage managé. Pour utiliser la console série, veillez à utiliser un compte de stockage personnalisé.

## <a name="get-started-with-the-serial-console"></a>Bien démarrer avec la console série
Pour les machines virtuelles et les groupes de machines virtuelles identiques, la console série est accessible uniquement sur le portail Azure :

### <a name="serial-console-for-virtual-machines"></a>Console série pour machines virtuelles
La console série pour machines virtuelles est accessible en cliquant simplement sur **Console série** dans la section **Support + dépannage** dans le portail Azure.
  1. Ouvrez le [portail Azure](https://portal.azure.com).

  1. Accédez à **Toutes les ressources** et sélectionnez une machine virtuelle. La page de présentation de la machine virtuelle s’ouvre.

  1. Faites défiler l’écran jusqu’à la section **Support + dépannage**, puis sélectionnez **Console série**. Un nouveau volet s’ouvre avec la console série, puis démarre la connexion.

     ![Fenêtre de console série Linux](./media/virtual-machines-serial-console/virtual-machine-linux-serial-console-connect.gif)

### <a name="serial-console-for-virtual-machine-scale-sets"></a>Console série pour les groupes de machines virtuelles identiques
La console série est disponible pour les groupes de machines virtuelles identiques, accessible sur chaque instance au sein du groupe identique. Vous devez accéder à l’instance individuelle d’un groupe de machines virtuelles identiques avant de voir le bouton **Console série**. Si les diagnostics de démarrage ne sont pas activés sur votre groupe de machines virtuelles identiques, veillez à mettre à jour votre modèle de groupe de machines virtuelles identiques pour activer les diagnostics de démarrage, puis mettez à niveau toutes les instances vers le nouveau modèle afin d’accéder à la console série.
  1. Ouvrez le [portail Azure](https://portal.azure.com).

  1. Accédez à **Toutes les ressources** et sélectionnez un groupe de machines virtuelles identiques. La page Vue d’ensemble du groupe de machines virtuelles identiques s’ouvre.

  1. Accédez à **Instances**

  1. Sélectionner une instance de groupe de machines virtuelles identiques

  1. Dans la section **Support + dépannage**, sélectionnez **Console série**. Un nouveau volet s’ouvre avec la console série, puis démarre la connexion.

     ![Console série de groupe de machines virtuelles identiques Linux](./media/virtual-machines-serial-console/vmss-start-console.gif)


### <a name="tls-12-in-serial-console"></a>TLS 1.2 dans la console série
La console série utilise le protocole TLS 1.2 de bout en bout pour sécuriser toutes les communications au sein du service. La console série dépend d’un compte de stockage des diagnostics de démarrage géré par l’utilisateur, et le protocole TLS 1.2 doit être configuré séparément pour le compte de stockage. Les instructions pour ce faire sont accessibles [ici](../../storage/common/transport-layer-security-configure-minimum-version.md).

## <a name="advanced-uses-for-serial-console"></a>Utilisations avancées de la console série
Outre l’accès de la console à votre machine virtuelle, vous pouvez également utiliser la console série Azure pour ce qui suit :
* Envoi [d’une commande de requête système à votre machine virtuelle](./serial-console-nmi-sysrq.md)
* Envoi [d’une interruption non masquable à votre machine virtuelle](./serial-console-nmi-sysrq.md)
* [Redémarrage ou mises hors tension et sous tension successives forcées de votre machine virtuelle](./serial-console-power-options.md)


## <a name="next-steps"></a>Étapes suivantes
Vous trouverez de la documentation supplémentaire sur la console série dans la barre latérale.
- Des informations supplémentaires sont disponibles pour la [console série pour les machines virtuelles Linux](./serial-console-linux.md).
- Des informations supplémentaires sont disponibles pour la [console série pour les machines virtuelles Windows](./serial-console-windows.md).
