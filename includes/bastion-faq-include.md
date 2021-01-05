---
title: Fichier include
description: Fichier include
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: include
ms.date: 11/05/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 48a7912b1a6375111b6f9af1496cbbd41966202c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183888"
---
### <a name="which-regions-are-available"></a><a name="regions"></a>Quelles régions sont disponibles ?

[!INCLUDE [Azure Bastion region availability](bastion-regions-include.md)]

### <a name="do-i-need-a-public-ip-on-my-virtual-machine-to-connect-via-azure-bastion"></a><a name="publicip"></a>Ai-je besoin d’une IP publique sur ma machine virtuelle pour me connecter via Azure Bastion ?

Non. Quand vous vous connectez à une machine virtuelle avec Azure Bastion, vous n’avez pas besoin d’une IP publique sur la machine virtuelle Azure à laquelle vous vous connectez. Le service Bastion va ouvrir la session/connexion RDP/SSH à votre machine virtuelle sur l’adresse IP privée de celle-ci, au sein de votre réseau virtuel.

### <a name="is-ipv6-supported"></a>IPv6 est-il pris en charge ?

IPv6 n’est actuellement pas pris en charge. Azure Bastion prend en charge IPv4 uniquement.

### <a name="do-i-need-an-rdp-or-ssh-client"></a><a name="rdpssh"></a>Ai-je besoin d’un client RDP ou SSH ?

Non. Vous n’avez pas besoin d’un client RDP ou SSH pour accéder par RDP/SSH à votre machine virtuelle Azure dans votre portail Azure. Utilisez le [portail Azure](https://portal.azure.com) pour obtenir un accès RDP/SSH à votre machine virtuelle directement dans le navigateur.

### <a name="do-i-need-an-agent-running-in-the-azure-virtual-machine"></a><a name="agent"></a>Ai-je besoin d’un agent exécuté sur la machine virtuelle Azure ?

Non. Vous n’avez pas besoin d’installer un agent ou un logiciel sur votre navigateur ou sur votre machine virtuelle Azure. Le service Bastion est sans agent et ne nécessite aucun logiciel supplémentaire pour RDP/SSH.

### <a name="how-many-concurrent-rdp-and-ssh-sessions-does-each-azure-bastion-support"></a><a name="limits"></a>Combien de sessions RDP et SSH simultanées chaque bastion Azure prend-il en charge ?

RDP et SSH sont tous deux des protocoles basés sur l’utilisation. Une utilisation intensive des sessions entraîne la prise en charge d’un nombre total inférieur de sessions par l’hôte bastion. Les nombres ci-dessous concernent des workflows quotidiens normaux.

[!INCLUDE [limits](bastion-limits.md)]

### <a name="what-features-are-supported-in-an-rdp-session"></a><a name="rdpfeaturesupport"></a>Quelles sont les fonctionnalités prises en charge dans une session RDP ?

À l’heure actuelle, seul le copier-coller de texte est pris en charge. Les fonctionnalités telles que la copie de fichiers ne sont pas prises en charge. N’hésitez pas à nous faire part de vos commentaires sur les nouvelles fonctionnalités dans la [page des commentaires Azure Bastion](https://feedback.azure.com/forums/217313-networking?category_id=367303).

### <a name="does-bastion-hardening-work-with-aadj-vm-extension-joined-vms"></a><a name="aadj"></a>Le durcissement Bastion fonctionne-t-il avec des machines virtuelles jointes à l’extension de machine virtuelle AADJ ?

Cette fonctionnalité ne fonctionne pas avec les ordinateurs joints à l’extension de machine virtuelle AADJ à l’aide d’utilisateurs Azure AD. Pour plus d’informations, consultez [Machines virtuelles Windows et Azure AD](../articles/active-directory/devices/howto-vm-sign-in-azure-ad-windows.md#requirements).

### <a name="which-browsers-are-supported"></a><a name="browsers"></a>Quels sont les navigateurs pris en charge ?

Utilisez le navigateur Microsoft Edge ou Google Chrome sous Windows. Pour Apple Mac, utilisez le navigateur Google Chrome. Microsoft Edge Chromium est également pris en charge sur Windows et Mac, respectivement.

### <a name="where-does-azure-bastion-store-customer-data"></a><a name="data"></a>Où Azure bastion stocke-t-il les données client ?

Azure Bastion ne déplace pas et ne stocke pas les données client en dehors de la région dans laquelle il est déployé.

### <a name="are-any-roles-required-to-access-a-virtual-machine"></a><a name="roles"></a>Certains rôles sont-ils obligatoires pour accéder à une machine virtuelle ?

Pour établir une connexion, les rôles suivants sont nécessaires :

* Rôle de lecteur sur la machine virtuelle
* Rôle de lecteur sur la carte réseau avec adresse IP privée de la machine virtuelle
* Rôle de lecteur sur la ressource Azure Bastion

### <a name="what-is-the-pricing"></a><a name="pricingpage"></a>Quel est la tarification ?

Pour plus d’informations, consultez la [page relative aux prix appliqués](https://aka.ms/BastionHostPricing).

### <a name="does-azure-bastion-require-an-rds-cal-for-administrative-purposes-on-azure-hosted-vms"></a><a name="rdscal"></a>Est-ce qu’Azure Bastion nécessite une licence d’accès client aux services Bureau à distance à des fins d’administration sur les machines virtuelles hébergées par Azure ?

Non, l’accès aux machines virtuelles Windows Server par Azure Bastion ne nécessite pas de [licence d’accès client aux services Bureau à distance](https://www.microsoft.com/p/windows-server-remote-desktop-services-cal/dg7gmgf0dvsv?activetab=pivot:overviewtab) quand elles sont utilisées exclusivement à des fins d’administration.

### <a name="which-keyboard-layouts-are-supported-during-the-bastion-remote-session"></a><a name="keyboard"></a>Quelles sont les dispositions de clavier prises en charge pendant la session à distance Bastion ?

Azure Bastion prend actuellement en charge la disposition de clavier en-US-QWERTY dans la machine virtuelle.  La prise en charge d’autres paramètres régionaux pour la disposition de clavier est en cours de développement.

### <a name="is-user-defined-routing-udr-supported-on-an-azure-bastion-subnet"></a><a name="udr"></a>Le routage défini par l’utilisateur (UDR) est-il pris en charge sur un sous-réseau Azure Bastion ?

Non. Le routage UDR n’est pas pris en charge sur un sous-réseau Azure Bastion.

Pour les scénarios qui incluent Azure Bastion et Pare-feu Azure/Appliance virtuelle réseau (NVA) dans le même réseau virtuel, vous n’avez pas besoin de forcer le trafic d’un sous-réseau Azure Bastion vers le Pare-feu Azure, car la communication entre Azure Bastion et vos machines virtuelles est privée. Pour plus d’informations, consultez [Accès aux machines virtuelles derrière le pare-feu Azure avec Bastion](https://azure.microsoft.com/blog/accessing-virtual-machines-behind-azure-firewall-with-azure-bastion/).

### <a name="why-do-i-get-your-session-has-expired-error-message-before-the-bastion-session-starts"></a><a name="session"></a>Pourquoi le message d’erreur « Votre session a expiré » s’affiche avant le démarrage de la session Bastion ?

Une session ne doit être lancée qu’à partir du portail Azure. Connectez-vous au portail Azure, puis redémarrez votre session. Si vous accédez à l’URL directement à partir d’une autre session de navigateur ou d’un autre onglet, cette erreur est normale. Cela permet de sécuriser votre session, en évitant qu’elle soit accessible en dehors du portail Azure.

### <a name="how-do-i-handle-deployment-failures"></a><a name="udr"></a>Comment gérer les échecs de déploiement ?

Passez en revue les messages d’erreur et [déclenchez une demande de support dans le portail Azure](../articles/azure-portal/supportability/how-to-create-azure-support-request.md) en fonction des besoins. Les échecs de déploiement peuvent être dus aux [limites, quotas et contraintes de l’abonnement Azure](../articles/azure-resource-manager/management/azure-subscription-service-limits.md). Plus précisément, les clients peuvent rencontrer une limite relative au nombre d’adresses IP publiques autorisées par abonnement, qui entraîne l’échec du déploiement Azure Bastion.

### <a name="how-do-i-incorporate-azure-bastion-in-my-disaster-recovery-plan"></a><a name="dr"></a>Comment incorporer Azure Bastion dans mon plan de reprise d’activité après sinistre ?

Azure Bastion est déployé sur des réseaux virtuels ou des réseaux virtuels appairés, et il est associé à une région Azure. Il vous revient de déployer Azure Bastion sur un réseau virtuel de site de reprise d’activité après sinistre. En cas de défaillance d’une région Azure, basculez vos machines virtuelles vers la région de reprise d’activité. Ensuite, utilisez l’hôte Azure Bastion qui est déployé dans la région de reprise d’activité pour vous connecter aux machines virtuelles qui y sont désormais maintenant déployées.