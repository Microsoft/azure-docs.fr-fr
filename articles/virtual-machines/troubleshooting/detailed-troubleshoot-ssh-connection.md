---
title: Dépannage détaillé de SSH pour une machine virtuelle Azure | Microsoft Docs
description: Étapes supplémentaires de dépannage détaillées des problèmes de connexion à une machine virtuelle Azure
keywords: refus de la connexion ssh,erreur ssh,ssh azure,échec de la connexion ssh
services: virtual-machines-linux
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.assetid: b8e8be5f-e8a6-489d-9922-9df8de32e839
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: c117f9697299b94a54d9184093e65c56822b8bd2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002627"
---
# <a name="detailed-ssh-troubleshooting-steps-for-issues-connecting-to-a-linux-vm-in-azure"></a>Étapes détaillées supplémentaires de résolution des problèmes SSH pour la connexion à une machine virtuelle Linux dans Azure
Il existe de nombreuses raisons pour lesquelles le client SSH peut ne pas pouvoir accéder au service SSH sur la machine virtuelle. Si vous avez suivi les [étapes de dépannage générales pour SSH](troubleshoot-ssh-connection.md), vous devez aussi résoudre le problème de connexion. Cet article vous guide tout au long des étapes de dépannage détaillées pour déterminer où la connexion SSH échoue et comment résoudre le problème.

## <a name="take-preliminary-steps"></a>Commencer par les étapes préliminaires
La figure suivante montre les composants concernés.

![Diagramme qui affiche les composants d’un service SSH](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot1.png)

Les étapes suivantes vous aident à isoler la source du problème et à déterminer différentes solutions.

1. Vérifiez l’état de la machine virtuelle sur le portail.
   Dans le [portail Azure](https://portal.azure.com), sélectionnez **Machines virtuelles** > *Nom de la machine virtuelle*.

   Le volet d’état de la machine virtuelle doit afficher **En cours d’exécution**. Faites défiler vers le bas pour voir l’activité récente des ressources de calcul, de stockage et réseau.

2. Sélectionnez **Paramètres** pour examiner les points de terminaison, les adresses IP, les groupes de sécurité réseau et les autres paramètres.

   La machine virtuelle doit avoir un point de terminaison défini pour le trafic SSH que vous pouvez consulter sous **Points de terminaison** ou **[Groupe de sécurité réseau](../../virtual-network/network-security-groups-overview.md)** . Les points de terminaison pour les machines virtuelles créées à l’aide de Resource Manager sont stockés dans un groupe de sécurité réseau. Vérifiez que les règles ont été appliquées au groupe de sécurité réseau et qu’elles sont référencées dans le sous-réseau.

Pour vérifier la connectivité réseau, contrôlez les points de terminaison configurés et déterminez si vous pouvez vous connecter à la machine virtuelle par le biais d’un autre protocole, comme HTTP ou un autre service.

Une fois ces étapes effectuées, essayez à nouveau la connexion SSH.

## <a name="find-the-source-of-the-issue"></a>Découvrir la source du problème
Le client SSH sur votre ordinateur n’a peut-être pas pu se connecter au service SSH sur la machine virtuelle Azure en raison de problèmes ou de mauvaises configurations dans les domaines suivants :

* [Ordinateur client SSH](#source-1-ssh-client-computer)
* [Appareil du périmètre de l’organisation](#source-2-organization-edge-device)
* [point de terminaison de service cloud et liste de contrôle d’accès (ACL) ;](#source-3-cloud-service-endpoint-and-acl)
* [Groupes de sécurité réseau](#source-4-network-security-groups)
* [Machine virtuelle Linux Azure](#source-5-linux-based-azure-virtual-machine)

## <a name="source-1-ssh-client-computer"></a>Source 1 : Ordinateur client SSH
Pour vérifier que votre ordinateur n’est pas la source du problème, vérifiez qu’il peut établir des connexions SSH avec un autre ordinateur Linux local.

![Diagramme qui indique les composants de l’ordinateur client SSH](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot2.png)

Si la connexion échoue, recherchez les problèmes suivants sur votre ordinateur :

* un paramètre de pare-feu local qui bloque le trafic SSH entrant ou sortant (TCP 22) ;
* un logiciel de proxy client installé localement qui empêche les connexions SSH ;
* un logiciel de surveillance réseau installé localement qui empêche les connexions SSH ;
* d’autres types de logiciels de sécurité qui surveillent le trafic ou autorisent/interdisent des types spécifiques de trafic.

Si une de ces conditions s’applique, désactivez temporairement le logiciel et tentez d’établir une connexion SSH à un ordinateur local pour connaître le motif du blocage de la connexion sur votre ordinateur. Ensuite, contactez votre administrateur réseau pour corriger les paramètres du logiciel et autoriser les connexions SSH.

Si vous utilisez l’authentification par certificat, vérifiez que vous avez ces autorisations sur le dossier .ssh dans votre répertoire de base :

* Chmod 700 ~/.ssh
* Chmod 644 ~/.ssh/\*.pub
* Chmod 600 ~/.ssh/id_rsa (ou tout autre fichier contenant vos clés privées)
* Chmod 644 ~/.ssh/known_hosts (contient les hôtes auxquels vous vous êtes connecté via SSH)

## <a name="source-2-organization-edge-device"></a>Source 2 : Appareil du périmètre de l’organisation
Pour vous assurer que votre appareil de périmètre de l’organisation n’est pas la cause du problème, vérifiez qu’un ordinateur directement connecté à Internet peut établir des connexions SSH à votre machine virtuelle Azure. Si vous accédez à la machine virtuelle par le biais d'un VPN site à site ou d'une connexion Azure ExpressRoute, passez à [Source 4 : Groupes de sécurité réseau](#nsg).

![Diagramme qui met en évidence un appareil du périmètre de l’organisation](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot3.png)

Si votre ordinateur n’est pas directement connecté à Internet, créez une machine virtuelle Azure dans son propre groupe de ressources ou service cloud, et utilisez-la. Pour plus d’informations, consultez [Créer une machine virtuelle exécutant Linux dans Azure](../linux/quick-create-cli.md). Une fois le test terminé, supprimez le groupe de ressources ou la machine virtuelle et le service cloud.

Si vous pouvez créer une connexion SSH avec un ordinateur directement connecté à Internet, vérifiez sur l’appareil de périmètre de l’organisation :

* un pare-feu interne qui bloque le trafic SSH avec Internet ;
* un serveur proxy qui empêche les connexions SSH ;
* un logiciel de détection d’intrusion ou de surveillance réseau s’exécutant sur les appareils de votre réseau de périmètre qui empêche les connexions SSH.

Contactez votre administrateur réseau pour corriger les paramètres de vos appareils du périmètre de l’organisation pour permettre le trafic SSH avec Internet.

## <a name="source-3-cloud-service-endpoint-and-acl"></a>Source 3 : Point de terminaison de service cloud et liste de contrôle d’accès

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

> [!NOTE]
> Cette source ne s’applique qu’aux machines virtuelles créées à l’aide du modèle de déploiement Classic. Pour les machines virtuelles créées à l'aide de Resource Manager, passez à [Source 4 : Groupes de sécurité réseau](#nsg).

Pour vous assurer que le point de terminaison du service cloud et la liste de contrôle d’accès ne sont pas la cause du problème, vérifiez qu’une autre machine virtuelle Azure dans le même réseau virtuel peut se connecter avec SSH.

![Diagramme qui met en évidence un point de terminaison de service cloud et une liste de contrôle d’accès](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot4.png)

Si le réseau virtuel ne contient pas une autre machine virtuelle, vous pouvez facilement en créer une. Pour plus d’informations, consultez [Création d’une machine virtuelle Linux sur Azure à l’aide de l’interface de ligne de commande](../linux/quick-create-cli.md). Une fois le test terminé, supprimez la machine virtuelle supplémentaire.

Si vous pouvez créer une connexion SSH avec une machine virtuelle dans le même réseau virtuel, vérifiez les éléments suivants :

* **la configuration du point de terminaison pour le trafic SSH sur la machine virtuelle cible ;** le port TCP privé du point de terminaison doit correspondre au port TCP écouté par le service SSH de la machine virtuelle. (Par défaut, il s’agit du port 22.) Vérifiez le numéro de port TCP SSH dans le portail Azure en sélectionnant **Machines virtuelles** > *Nom de la machine virtuelle* > **Paramètres** > **Points de terminaison**.
* **La liste de contrôle d’accès du point de terminaison du trafic SSH sur la machine virtuelle cible.** Une liste de contrôle d’accès vous permet de spécifier le trafic Internet entrant autorisé ou interdit, en fonction de l’adresse IP source. Une mauvaise configuration des listes de contrôle d’accès peut empêcher le trafic SSH entrant d’accéder au point de terminaison. Consultez vos listes de contrôle d’accès et vérifiez que le trafic entrant provenant des adresses IP publiques de votre proxy ou d’un autre serveur de périmètre est autorisé. Pour plus d'informations, consultez [À propos des listes de contrôle d'accès (ACL) réseau](/previous-versions/azure/virtual-network/virtual-networks-acl).

Pour vérifier que le point de terminaison n’est pas la source du problème, supprimez le point de terminaison actuel, créez-en un autre et spécifiez le nom SSH (port TCP 22 comme port public et privé). Pour plus d’informations, consultez [Configuration des points de terminaison sur une machine virtuelle dans Azure](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints?toc=/azure/virtual-machines/windows/classic/toc.json).

<a id="nsg"></a>

## <a name="source-4-network-security-groups"></a>Source 4 : Groupes de sécurité réseau
Les groupes de sécurité réseau vous permettent de mieux contrôler le trafic entrant et sortant autorisé. Vous pouvez créer des règles qui s’étendent aux sous-réseaux et aux services cloud d’un réseau virtuel Azure. Vérifiez les règles de votre groupe de sécurité réseau pour vous assurer que le trafic SSH vers et depuis Internet est autorisé.
Pour plus d'informations, consultez [À propos des groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md).

Vous pouvez également utiliser Vérification IP pour valider la configuration NSG. Pour plus d’informations, consultez [Vue d’ensemble de la surveillance réseau Azure](../../network-watcher/network-watcher-monitoring-overview.md). 

## <a name="source-5-linux-based-azure-virtual-machine"></a>Source 5 : Machine virtuelle Azure Linux
La dernière source des problèmes possibles est la machine virtuelle Azure elle-même.

![Diagramme qui met en évidence une machine virtuelle Azure Linux](./media/detailed-troubleshoot-ssh-connection/ssh-tshoot5.png)

Si ce n’est déjà fait, suivez les instructions permettant de [réinitialiser un mot de passe pour les machines virtuelles Linux](./reset-password.md).

Essayez une nouvelle fois de vous connecter à partir de votre ordinateur. Si l’échec se reproduit, l’une des raisons suivantes en est peut-être la cause :

* Le service SSH n’est pas en cours d’exécution sur la machine virtuelle cible.
* Le service SSH n’est pas à l’écoute sur le port TCP 22. Pour effectuer un test, installez un client telnet sur votre ordinateur local et exécutez « telnet *NomServiceCloud*.cloudapp.net 22 ». Cette étape détermine si la machine virtuelle autorise les communications entrantes et sortantes avec le point de terminaison SSH.
* Le pare-feu local sur la machine virtuelle cible a des règles qui empêchent le trafic SSH entrant ou sortant.
* Un logiciel de détection d’intrusion ou de surveillance réseau s’exécutant sur la machine virtuelle Azure empêche les connexions SSH.

## <a name="additional-resources"></a>Ressources supplémentaires
Pour plus d’informations sur la résolution des problèmes d’accès aux applications, consultez la page [Résolution des problèmes d’accès à une application exécutée sur une machine virtuelle Azure](./troubleshoot-app-connection.md)