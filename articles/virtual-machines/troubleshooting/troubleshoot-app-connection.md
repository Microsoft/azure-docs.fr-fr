---
title: Résoudre les problèmes d’accès à l’application de la machine virtuelle dans Azure | Microsoft Docs
description: Utilisez ces étapes détaillées de résolution de problèmes pour isoler les problèmes de connexion aux applications s’exécutant sur des machines virtuelles dans Azure.
services: virtual-machines
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
keywords: impossible de démarrer l’application, le programme ne s’ouvre pas, port d’écoute bloqué, impossible de démarrer le programme, port d’écoute bloqué
ms.assetid: b9ff7cd0-0c5d-4c3c-a6be-3ac47abf31ba
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: dec5aeaac6f39a106899094e674864d3bd10dc02
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91966336"
---
# <a name="troubleshoot-application-connectivity-issues-on-virtual-machines-in-azure"></a>Résoudre les problèmes de connectivité des applications sur des machines virtuelles dans Azure

Plusieurs raisons peuvent expliquer le fait que vous ne puissiez pas démarrer ou vous connecter à une application exécutée sur une machine virtuelle Azure. Parmi elles figurent le fait que l’application ne s’exécute pas ou ne surveille pas les ports attendus, que le port d’écoute soit bloqué ou que les règles réseau ne transmettent pas correctement le trafic à l’application. Cet article décrit une approche méthodique permettant d’identifier et de corriger le problème.

Si vous rencontrez des problèmes de connexion à votre machine virtuelle à l’aide de RDP ou de SSH, consultez l’un des articles suivants :

* [Résolution des problèmes de connexion Bureau à distance avec une machine virtuelle Azure exécutant Windows](troubleshoot-rdp-connection.md)
* [Résolution des problèmes des connexions SSH avec une machine virtuelle Azure Linux](troubleshoot-ssh-connection.md)

Si vous avez besoin d'aide supplémentaire concernant n'importe quel point de cet article, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.

## <a name="quick-start-troubleshooting-steps"></a>Étapes de dépannage rapide
Si vous avez des problèmes de connexion à une application, essayez les étapes génériques suivantes. Après chaque étape, essayez de vous connecter à votre application :

* Redémarrez la machine virtuelle.
* Recréez le point de terminaison, les règles de pare-feu et les règles de groupe de sécurité réseau.
  * [Modèle Resource Manager : gérer les groupes de sécurité réseau](../../virtual-network/manage-network-security-group.md)
  * [Modèle Classic : gérer les points de terminaison Cloud Services](../../cloud-services/cloud-services-enable-communication-role-instances.md)
* Connectez-vous à partir d’un emplacement différent, tel qu’un autre réseau virtuel Azure.
* Redéployez la machine virtuelle.
  * [Redéployez la machine virtuelle Windows.](redeploy-to-new-node-windows.md)
  * [Redéployer la machine virtuelle Linux.](redeploy-to-new-node-linux.md)
* Recréez la machine virtuelle.

Pour plus d'informations, consultez [Résolution des problèmes de connectivité de point de terminaison (échecs RDP/SSH/HTTP, etc.)](https://social.msdn.microsoft.com/Forums/azure/en-US/538a8f18-7c1f-4d6e-b81c-70c00e25c93d/troubleshooting-endpoint-connectivity-rdpsshhttp-etc-failures?forum=WAVirtualMachinesforWindows).

## <a name="detailed-troubleshooting-overview"></a>Vue d’ensemble détaillée de la résolution de problèmes
La résolution des problèmes d’accès à une application exécutée sur une machine virtuelle Azure comporte quatre domaines principaux.

![dépannage - impossible de démarrer l’application](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access1.png)

1. L’application exécutée sur la machine virtuelle Azure.
   * L’application fonctionne-t-elle correctement ?
2. La machine virtuelle Azure.
   * La machine virtuelle fonctionne-t-elle correctement et répond-elle aux demandes ?
3. Points de terminaison réseau Azure.
   * Points de terminaison de service cloud pour les machines virtuelles dans le modèle de déploiement classique.
   * Groupes de sécurité réseau et règles NAT de trafic entrant pour les machines virtuelles dans le modèle de déploiement Resource Manager.
   * Le trafic peut-il transiter depuis les utilisateurs jusqu’à la machine virtuelle/l’application sur les ports prévus ?
4. Votre périphérique de périmètre Internet.
   * Les règles de pare-feu en place empêchent-elles la circulation correcte du trafic ?

Pour les ordinateurs clients qui accèdent à l’application par le biais d’une connexion de réseau privé virtuel de site à site ou d’une connexion ExpressRoute, les principaux éléments susceptibles de poser problème sont l’application et la machine virtuelle Azure.

Pour déterminer la source du problème et sa correction, procédez comme suit.

## <a name="step-1-access-application-from-target-vm"></a>Étape 1 : Accéder à une application à partir de la machine virtuelle cible
Essayez d’accéder à l’application avec le programme client approprié à partir de la machine virtuelle sur laquelle il est en cours d’exécution. Utilisez le nom d’hôte local, l’adresse IP locale ou l’adresse de bouclage (127.0.0.1).

![démarrer l’application directement à partir de la machine virtuelle](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access2.png)

Par exemple, si l’application est un serveur web, ouvrez un navigateur sur la machine virtuelle et essayez d’accéder à une page web hébergée sur la machine virtuelle.

Si vous pouvez accéder à l’application, allez à l’ [étape 2](#step2).

Si vous ne pouvez pas accéder à l’application, vérifiez les paramètres suivants :

* L’application est exécutée sur la machine virtuelle cible.
* L’application écoute sur les ports TCP et UDP attendus.

Sur les machines virtuelles Windows et Linux, utilisez la commande **netstat** pour afficher les ports d’écoute actifs. Examinez la sortie pour les ports attendus sur lesquels votre application doit être à l’écoute. Redémarrez l’application ou configurez-la pour utiliser les ports prévus selon les besoins et essayez d’accéder à nouveau localement à l’application.

## <a name="step-2-access-application-from-another-vm-in-the-same-virtual-network"></a><a id="step2"></a>Étape 2 : Accéder à l’application à partir d’une autre machine virtuelle dans le même réseau virtuel
Essayez d’accéder à l’application à partir d’une machine virtuelle différente, mais sur le même réseau virtuel, à l’aide du nom d’hôte de la machine virtuelle ou de son adresse IP publique, privée ou de fournisseur attribuée par Azure. Pour les machines virtuelles créées à l’aide du modèle de déploiement classique, n’utilisez pas l’adresse IP publique du service cloud.

![démarrer l’application à partir d’une autre machine virtuelle](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access3.png)

Par exemple, si l’application est un serveur web, essayez d’accéder à un navigateur sur une autre machine virtuelle du même réseau virtuel.

Si vous pouvez accéder à l’application, allez à l’ [étape 3](#step3).

Si vous ne pouvez pas accéder à l’application, vérifiez les paramètres suivants :

* Le pare-feu de l’hôte sur la machine virtuelle cible autorise le trafic de demandes entrantes et de réponses sortantes.
* Un logiciel de détection d’intrusion ou de surveillance réseau s’exécutant sur la machine virtuelle cible autorise le trafic.
* Les points de terminaison Cloud Services et les groupes de sécurité réseau autorisent le trafic :
  * [Modèle Classic : gérer les points de terminaison Cloud Services](../../cloud-services/cloud-services-enable-communication-role-instances.md)
  * [Modèle Resource Manager : gérer les groupes de sécurité réseau](../../virtual-network/manage-network-security-group.md)
* Un composant distinct qui s’exécute dans votre réseau virtuel dans le chemin d’accès entre la machine virtuelle de test et la machine virtuelle, par exemple un équilibreur de charge ou un pare-feu, autorise le trafic.

Sur une machine virtuelle Windows, utilisez le pare-feu Windows avec fonctions avancées de sécurité pour déterminer si les règles de pare-feu excluent le trafic entrant et sortant de votre application

## <a name="step-3-access-application-from-outside-the-virtual-network"></a><a id="step3"></a>Étape 3 : Accéder à l’application à partir de l’extérieur du réseau virtuel
Essayez d’accéder à l’application à partir d’un ordinateur en dehors du réseau virtuel de la machine virtuelle sur laquelle l’application est exécutée. Utilisez un réseau différent de celui de votre ordinateur client d’origine.

![démarrer l’application à partir d’un ordinateur situé hors du réseau virtuel](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access4.png)

Par exemple, si l’application est un serveur web, essayez d’accéder à la page web à partir d’un navigateur sur un autre ordinateur ne faisant pas partie de ce réseau virtuel.

Si vous ne pouvez pas accéder à l’application, vérifiez les paramètres suivants :

* Pour les machines virtuelles créées à l’aide du modèle de déploiement Classic :
  
  * Vérifiez que la configuration de points de terminaison de la machine virtuelle autorise le trafic entrant, notamment le protocole (TCP ou UDP) et les numéros de port public et privé.
  * Assurez-vous que les ACL (listes de contrôle d’accès) sur le point de terminaison n’empêchent pas le trafic entrant à partir d’Internet.
  * Pour plus d’informations, consultez [Configuration des points de terminaison sur une machine virtuelle](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints).
* Pour les machines virtuelles créées à l’aide du modèle de déploiement Resource Manager :
  
  * Vérifiez que la configuration de la règle NAT de trafic entrant de la machine virtuelle autorise le trafic entrant, notamment le protocole (TCP ou UDP) et les numéros de port public et privé.
  * Assurez-vous que les groupes de sécurité réseau autorisent le trafic de demandes entrantes et de réponses sortantes.
  * Pour plus d’informations, consultez l’article de [présentation des groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md).

Si la machine virtuelle ou le point de terminaison est membre d’un jeu à charge équilibrée :

* Vérifiez que le protocole de la sonde (TCP ou UDP) et le numéro de port sont corrects.
* Si le protocole de la sonde et le port sont différents du protocole de jeu à charge équilibrée et du port :
  * Vérifiez que l’application écoute le protocole de la sonde (TCP ou UDP) et le numéro de port (utilisez **netstat –a** sur la machine virtuelle cible).
  * Le pare-feu de l’hôte sur la machine virtuelle cible autorise le trafic de demandes de sonde entrantes et de réponses de sonde sortantes.

Si vous pouvez accéder à l’application, assurez-vous que votre périphérique de périmètre Internet autorise :

* Le trafic des demandes des applications sortantes, de votre ordinateur client à la machine virtuelle Azure.
* Le trafic des réponses des applications entrantes, de la machine virtuelle Azure.

## <a name="step-4-if-you-cannot-access-the-application-use-ip-verify-to-check-the-settings"></a>Étape 4 Si vous ne pouvez pas accéder à l’application, utilisez Vérification IP pour vérifier les paramètres. 

Pour plus d’informations, consultez [Vue d’ensemble de la surveillance réseau Azure](../../network-watcher/network-watcher-monitoring-overview.md). 

## <a name="additional-resources"></a>Ressources supplémentaires
[Résolution des problèmes de connexion Bureau à distance avec une machine virtuelle Azure exécutant Windows](troubleshoot-rdp-connection.md)

[Résolution des problèmes des connexions SSH avec une machine virtuelle Azure Linux](troubleshoot-ssh-connection.md)