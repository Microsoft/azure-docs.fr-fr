---
title: Dépannage détaillé du Bureau à distance | Microsoft Docs
description: Revoir les étapes de dépannage détaillées pour les erreurs liées au Bureau à distance, quand vous ne pouvez pas vous connecter aux machines virtuelles Windows dans Azure
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
keywords: impossible de se connecter au bureau à distance, dépanner le bureau à distance, le bureau à distance ne peut pas se connecter, erreurs du bureau à distance, dépannage du bureau à distance, problèmes du bureau à distance
ms.assetid: 9da36f3d-30dd-44af-824b-8ce5ef07e5e0
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 91f15e32866cca008553286f7585247909d9a4ba
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002678"
---
# <a name="detailed-troubleshooting-steps-for-remote-desktop-connection-issues-to-windows-vms-in-azure"></a>Étapes de dépannage détaillées pour les problèmes de connexion du Bureau à distance aux machines virtuelles Windows dans Azure
Cet article décrit les étapes de dépannage détaillées pour diagnostiquer et résoudre les erreurs complexes du Bureau à distance pour les machines virtuelles basées Azure sur Windows.

> [!IMPORTANT]
> Pour éliminer les erreurs du Bureau à distance les plus courantes, veillez à lire [l’article sur la résolution des problèmes de base du Bureau à distance](troubleshoot-rdp-connection.md) avant de continuer.

Vous pouvez rencontrer un message d’erreur du Bureau à distance qui ne ressemble à aucun des messages d’erreur spécifiques couverts dans [le guide de résolution des problèmes de base du Bureau à distance](troubleshoot-rdp-connection.md). Suivez ces étapes pour déterminer pourquoi le client Bureau à distance ne parvient pas à se connecter au service Bureau à distance sur la machine virtuelle Azure.


Si vous avez besoin d'aide supplémentaire concernant n'importe quel point de cet article, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site de support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**. Pour plus d’informations sur l’utilisation du support Azure, lisez la [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="components-of-a-remote-desktop-connection"></a>Composants d’une connexion Bureau à distance
Voici les composants impliqués dans une connexion Bureau à distance :

![Diagramme montrant les composants impliqués dans une connexion Bureau à distance.](./media/detailed-troubleshoot-rdp/tshootrdp_0.png)

Avant de poursuivre, nous vous recommandons de réfléchir à tout ce qui a changé depuis que vous avez créé avec succès une connexion Bureau à distance à la machine virtuelle. Par exemple :

* Si l’adresse IP publique de la machine virtuelle ou du service cloud contenant la machine virtuelle (également appelée adresse IP virtuelle [VIP](https://en.wikipedia.org/wiki/Virtual_IP_address)) a changé. L’erreur de Bureau à distance peut indiquer que le cache client DNS a toujours *l’ancienne adresse IP* enregistrée pour le nom DNS. Videz le cache client DNS et essayez de vous reconnecter à la machine virtuelle. Ou essayez de vous connecter directement avec la nouvelle adresse IP virtuelle.
* Vous utilisez une application tierce pour gérer vos connexions Bureau à distance au lieu d’utiliser la connexion générée par le portail Azure. Vérifiez que la configuration de l’application inclut bien le port TCP approprié pour le trafic de Bureau à distance. Vous pouvez vérifier ce port pour une machine virtuelle classique dans le [portail Azure](https://portal.azure.com), en cliquant sur Paramètres de la machine virtuelle > Points de terminaison.

## <a name="preliminary-steps"></a>Étapes préliminaires
Avant de passer à la procédure de dépannage détaillé :

* Vérifiez l’état de la machine virtuelle sur le Portail Azure pour identifier d’éventuels problèmes flagrants.
* Suivez les [étapes du guide de résolution des problèmes de base pour corriger rapidement les erreurs de Bureau à distance courantes](troubleshoot-rdp-connection.md#quick-troubleshooting-steps).
* Pour les images personnalisées, assurez-vous que votre VHD est préparé correctement avant de le charger. Pour plus d’informations, consultez [Préparer un VHD ou VHDX Windows à charger sur Azure](../windows/prepare-for-upload-vhd-image.md).


Essayez de vous reconnecter à la machine virtuelle via le Bureau à distance après ces étapes.

## <a name="detailed-troubleshooting-steps"></a>Étapes de dépannage détaillées
Le client Bureau à distance peut ne pas être en mesure d’atteindre le service Bureau à distance sur la machine virtuelle Azure en raison de problèmes au niveau des sources suivantes :

* [ordinateur client de Bureau à distance ;](#source-1-remote-desktop-client-computer)
* [périphérique de périmètre intranet de l’entreprise ;](#source-2-organization-intranet-edge-device)
* [point de terminaison de service cloud et liste de contrôle d’accès (ACL) ;](#source-3-cloud-service-endpoint-and-acl)
* [Groupes de sécurité réseau](#source-4-network-security-groups)
* [Machine virtuelle Windows sur Azure](#source-5-windows-based-azure-vm)

## <a name="source-1-remote-desktop-client-computer"></a>Source 1 : ordinateur client de Bureau à distance ;
Vérifiez que votre ordinateur peut établir des connexions Bureau à distance avec un autre ordinateur Windows local.

![Diagramme des composants d’une connexion Bureau à distance avec le client Bureau à distance mis en surbrillance et une flèche pointant vers un autre ordinateur local indiquant une connexion.](./media/detailed-troubleshoot-rdp/tshootrdp_1.png)

Si vous n’y parvenez pas, recherchez les paramètres suivants sur votre ordinateur :

* un paramètre de pare-feu local qui bloque le trafic de Bureau à distance ;
* un logiciel proxy client installé localement qui empêche les connexions Bureau à distance ;
* un logiciel de surveillance réseau installé localement qui empêche les connexions Bureau à distance ;
* d’autres types de logiciel de sécurité qui analysent le trafic ou autorisent/interdisent des types spécifiques de trafic empêchant les connexions Bureau à distance.

Dans tous ces cas, désactivez temporairement le logiciel concerné et essayez d’établir une connexion avec un ordinateur local via le Bureau à distance. Si vous ne parvenez pas à identifier l’origine du problème de cette façon, contactez votre administrateur réseau pour corriger les paramètres logiciels afin d’autoriser les connexions Bureau à distance.

## <a name="source-2-organization-intranet-edge-device"></a>Source 2 : périphérique de périmètre intranet de l’entreprise ;
Vérifiez qu’un ordinateur directement connecté à Internet peut établir des connexions Bureau à distance avec votre machine virtuelle Azure.

![Diagramme des composants d’une connexion Bureau à distance avec le client Bureau à distance connecté à Internet mis en surbrillance et une flèche pointant vers une machine virtuelle Azure indiquant une connexion.](./media/detailed-troubleshoot-rdp/tshootrdp_2.png)

Si vous n’avez pas d’ordinateur directement connecté à Internet, créez et testez une nouvelle machine virtuelle Azure dans un groupe de ressources ou service cloud. Pour plus d’informations, consultez [Création d’une machine virtuelle exécutant Windows dans Azure](../windows/quick-create-portal.md). Une fois le test terminé, supprimez la machine virtuelle et le groupe de ressources ou le service cloud.

Si vous pouvez créer une connexion Bureau à distance avec un ordinateur directement connecté à Internet, recherchez sur votre périphérique de périmètre intranet d’entreprise :

* un pare-feu qui bloque les connexions HTTPS à Internet ;
* un serveur proxy qui empêche les connexions Bureau à distance ;
* un logiciel de détection d’intrusion ou de surveillance réseau s’exécutant sur les appareils de votre réseau de périmètre et qui empêche les connexions Bureau à distance.

Contactez votre administrateur réseau pour corriger les paramètres de votre périphérique de périmètre intranet d’entreprise afin d’autoriser les connexions Bureau à distance basées sur HTTPS à Internet.

## <a name="source-3-cloud-service-endpoint-and-acl"></a>Source 3 : Point de terminaison de service cloud et liste de contrôle d’accès

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Pour les machines virtuelles créées à l’aide du modèle de déploiement classique, vérifiez qu’une autre machine virtuelle Azure du même service cloud ou réseau virtuel peut établir des connexions Bureau à distance avec votre machine virtuelle Azure.

![Diagramme des composants d’une connexion Bureau à distance avec une machine virtuelle Azure mise en surbrillance et une flèche pointant vers une autre machine virtuelle Azure au sein du même service cloud indiquant une connexion.](./media/detailed-troubleshoot-rdp/tshootrdp_3.png)

> [!NOTE]
> Pour les machines virtuelles créées dans Resource Manager, passez à [Source 4 : groupes de sécurité réseau](#source-4-network-security-groups).

Si vous ne disposez pas d’une autre machine virtuelle dans le même service cloud ou réseau virtuel, créez-en une. Suivez les étapes de [création d’une machine virtuelle exécutant Windows dans Azure](../windows/quick-create-portal.md). Une fois le test terminé, supprimez la machine virtuelle de test.

Si vous pouvez vous connecter à une machine virtuelle via le Bureau à distance dans le même service cloud ou réseau virtuel, vérifiez les paramètres suivants :

* La configuration du point de terminaison pour le trafic de Bureau à distance sur la machine virtuelle cible : le port TCP privé du point de terminaison doit correspondre au port TCP sur lequel le service Bureau à distance de la machine virtuelle écoute (par défaut, le port 3389).
* La liste de contrôle d’accès du point de terminaison du trafic de Bureau à distance sur la machine virtuelle cible : les listes de contrôle d’accès vous permettent de spécifier le trafic Internet entrant autorisé et interdit en fonction de l’adresse IP source. Une mauvaise configuration des listes de contrôle d’accès peut empêcher le trafic du Bureau à distance d’accéder au point de terminaison. Examinez vos listes de contrôle d’accès pour vous assurer que le trafic entrant provenant des adresses IP publiques de votre proxy ou d’un autre serveur Edge est autorisé. Pour plus d’informations, consultez [Qu’est-ce qu’une liste de contrôle d’accès (ACL) réseau ?](/previous-versions/azure/virtual-network/virtual-networks-acl)

Pour vérifier si le point de terminaison est la source du problème, supprimez le point de terminaison actuel et créez un autre point en choisissant un port aléatoire dont le numéro externe se situe entre 49152 et 65535. Pour plus d’informations, consultez [Configuration des points de terminaison sur une machine virtuelle](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints?toc=/azure/virtual-machines/windows/classic/toc.json).

## <a name="source-4-network-security-groups"></a>Source 4 : Network Security Group
Les groupes de sécurité réseau vous permettent de contrôler plus précisément le trafic entrant et sortant autorisé. Vous pouvez créer des règles qui s’étendent aux sous-réseaux et aux services cloud d’un réseau virtuel Azure.

Utilisez la [vérification des flux IP](../../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) pour savoir si une règle d’un groupe de sécurité réseau bloque le trafic depuis ou vers une machine virtuelle. Vous pouvez également vérifier les règles de groupe de sécurité effectives pour vous assurer que la règle « Allow » entrante du groupe de sécurité réseau existe pour le port RDP (par défaut, 3389). Pour en savoir plus, voir [Utilisation de règles de sécurité effectives pour résoudre des problèmes de flux de trafic de machine virtuelle](../../virtual-network/diagnose-network-traffic-filter-problem.md).

## <a name="source-5-windows-based-azure-vm"></a>Source 5 : Machine virtuelle Windows sur Azure
![Diagramme des composants d’une connexion Bureau à distance avec une machine virtuelle Azure mise en surbrillance au sein d’un service cloud et un message indiquant qu’il peut s’agir d’une « potentielle source de problèmes ». Une ligne bleue indique que les règles du groupe de sécurité réseau peuvent bloquer le trafic vers ou depuis la machine virtuelle Azure.](./media/detailed-troubleshoot-rdp/tshootrdp_5.png)

Suivez les instructions de [cet article](./reset-rdp.md). Cet article est consacré à la réinitialisation du service Bureau à distance sur la machine virtuelle :

* activer la règle par défaut du pare-feu Windows Bureau à distance (port TCP 3389) ;
* activer les connexions Bureau à distance en définissant la valeur de registre HKLM\System\CurrentControlSet\Control\Terminal Server\fDenyTSConnections sur 0.

Essayez une nouvelle fois de vous connecter à partir de votre ordinateur. Si vous ne réussissez toujours pas à vous connecter via Bureau à distance, cela peut être dû à l’une des raisons suivantes :

* Le service Bureau à distance ne fonctionne pas sur la machine virtuelle cible.
* Le service Bureau à distance n’est pas compatible avec l’écoute sur le port TCP 3389.
* Le pare-feu Windows ou un autre pare-feu local comporte une règle sortante qui empêche le trafic du Bureau à distance.
* Le logiciel de détection d’intrusion ou de surveillance réseau s’exécutant sur la machine virtuelle Azure empêche les connexions Bureau à distance.

Pour les machines virtuelles créées à l’aide du modèle de déploiement classique, vous pouvez utiliser une session Azure PowerShell distante vers la machine virtuelle Azure. Tout d’abord, vous devez installer un certificat pour le service cloud d’hébergement de la machine virtuelle. Accédez à [Configurer l’accès à distance sécurisé de PowerShell aux machines virtuelles Azure](https://gallery.technet.microsoft.com/scriptcenter/Configures-Secure-Remote-b137f2fe) et téléchargez le fichier de script **InstallWinRMCertAzureVM.ps1** sur votre ordinateur local.

Installez ensuite Azure PowerShell si ce n’est pas déjà fait. Consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/).

Ouvrez ensuite une invite de commandes Azure PowerShell, puis remplacez le dossier actif par l’emplacement du fichier de script **InstallWinRMCertAzureVM.ps1** . Pour exécuter un script Azure PowerShell, vous devez définir la bonne stratégie d’exécution. Exécutez la commande **Get-ExecutionPolicy** afin de déterminer votre niveau de stratégie actuel. Pour plus d’informations sur la définition du niveau approprié, consultez [Set-ExecutionPolicy](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-5.1).

Indiquez ensuite le nom de votre abonnement Azure, le nom du service cloud et le nom de votre machine virtuelle (en supprimant les caractères < et >), puis exécutez les commandes suivantes.

```powershell
$subscr="<Name of your Azure subscription>"
$serviceName="<Name of the cloud service that contains the target virtual machine>"
$vmName="<Name of the target virtual machine>"
.\InstallWinRMCertAzureVM.ps1 -SubscriptionName $subscr -ServiceName $serviceName -Name $vmName
```

Le nom d’abonnement correct apparaît dans la propriété *SubscriptionName* de l’affichage de la commande **Get-AzureSubscription** . Le nom du service cloud pour la machine virtuelle apparaît dans la colonne *ServiceName* de l’affichage de la commande **Get-AzureVM**.

Vérifiez si vous disposez du nouveau certificat. Ouvrez un composant logiciel enfichable Certificats pour l’utilisateur actuel, puis examinez le dossier **Autorités de certification racines de confiance\Certificats**. Vous devriez voir un certificat portant le nom DNS de votre service cloud doit apparaître dans la colonne Issued To (exemple : cloudservice4testing.cloudapp.net).

Lancez ensuite une session Azure PowerShell distante à l’aide de ces commandes.

```powershell
$uri = Get-AzureWinRMUri -ServiceName $serviceName -Name $vmName
$creds = Get-Credential
Enter-PSSession -ConnectionUri $uri -Credential $creds
```

Une fois que vous avez entré les informations d’identification administrateur valides, vous devez visualiser une invite de commandes Azure PowerShell similaire à celle-ci :

```powershell
[cloudservice4testing.cloudapp.net]: PS C:\Users\User1\Documents>
```

La première partie de l’invite de commande représente le nom de votre service cloud qui contient la machine virtuelle cible, qui peut être différent de « cloudservice4testing.cloudapp.net ». Vous pouvez maintenant émettre des commandes Azure PowerShell pour ce service cloud afin d’examiner les problèmes et de corriger la configuration.

### <a name="to-manually-correct-the-remote-desktop-services-listening-tcp-port"></a>Correction manuelle des Services Bureau à distance permettant l’écoute du port TCP
À l’invite de la session Azure PowerShell à distance, exécutez cette commande.

```powershell
Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber"
```

La propriété PortNumber indique le numéro de port actuel. Si nécessaire, utilisez cette commande pour modifier le numéro de port du Bureau à distance afin de revenir à sa valeur par défaut (3389).

```powershell
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber" -Value 3389
```

Vérifiez que le port affiche désormais la valeur 3389 à l’aide de cette commande.

```powershell
Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber"
```

Quittez la session Azure PowerShell distante à l’aide de cette commande.

```powershell
Exit-PSSession
```

Vérifiez que le point de terminaison du Bureau à distance de la machine virtuelle Azure utilise également le port TCP 3398 comme port interne. Redémarrez la machine virtuelle Azure puis testez de nouveau la connexion Bureau à distance.

## <a name="additional-resources"></a>Ressources supplémentaires
[Réinitialisation d’un mot de passe ou du service Bureau à distance pour les machines virtuelles Windows](./reset-rdp.md)

[Installation et configuration d’Azure PowerShell](/powershell/azure/)

[Résolution des problèmes des connexions SSH avec une machine virtuelle Azure Linux](./troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json)

[Résolution des problèmes d’accès à une application exécutée sur une machine virtuelle Azure](./troubleshoot-app-connection.md?toc=/azure/virtual-machines/linux/toc.json)
