---
title: Impossible de se connecter avec RDP à une machine virtuelle Windows dans Azure | Microsoft Docs
description: Résolution des problèmes lorsque vous ne pouvez pas vous connecter à votre machine virtuelle Windows dans Azure à l’aide du Bureau à distance
keywords: erreur bureau à distance,erreur de connexion bureau à distance,impossible se connecter à la machine virtuelle,résolution des problèmes de connexion bureau à distance
services: virtual-machines-windows
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.assetid: 0d740f8e-98b8-4e55-bb02-520f604f5b18
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 03/23/2018
ms.author: akjosh
ms.openlocfilehash: ba6dda86475456b6797d27e11727d70261be2e1a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91439995"
---
# <a name="troubleshoot-remote-desktop-connections-to-an-azure-virtual-machine"></a>Résolution des problèmes de connexion Bureau à distance avec une machine virtuelle Azure
La connexion RDP (Remote Desktop Protocol) à votre machine virtuelle Azure Windows peut échouer pour diverses raisons, rendant votre machine virtuelle inaccessible. Le problème peut être lié au service Bureau à distance sur la machine virtuelle, à la connexion réseau ou encore au client Bureau à distance sur votre ordinateur hôte. Cet article vous guide à travers certaines des méthodes plus courantes pour résoudre les problèmes de connexion RDP. 

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.

 

<a id="quickfixrdp"></a>

## <a name="quick-troubleshooting-steps"></a>Étapes de dépannage rapide
Après chaque étape de résolution des problèmes, essayez de vous reconnecter à la machine virtuelle :

1. Réinitialisez la configuration du Bureau à distance.
2. Vérifiez les règles de groupe de sécurité réseau / de point de terminaison de services cloud.
3. Examinez les journaux d’activité de console de la machine virtuelle.
4. Réinitialisez la carte d’interface réseau pour la machine virtuelle.
5. Vérifiez l’intégrité des ressources de la machine virtuelle.
6. Réinitialisez le mot de passe de votre machine virtuelle.
7. Redémarrez votre machine virtuelle.
8. Redéployez votre machine virtuelle.

Si vous avez besoin de procédures plus détaillées et d’explications, poursuivez la lecture. Vérifiez que l’équipement réseau local, notamment les routeurs et les pare-feu, ne bloque pas le port TCP 3389 sortant, comme indiqué dans les [scénarios détaillés de dépannage RDP](detailed-troubleshoot-rdp.md).

> [!TIP]
> Si le bouton **Connecter** de votre machine virtuelle est grisé dans le portail et que vous n’êtes pas connecté à Azure avec une connexion [Express Route](../../expressroute/expressroute-introduction.md) ou [réseau privé virtuel de site à site](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md), vous devez créer votre machine virtuelle et lui attribuer une adresse IP publique pour pouvoir utiliser le protocole RDP. Pour en savoir plus sur les adresses IP publiques dans Azure, consultez [cet article](../../virtual-network/public-ip-addresses.md).


## <a name="ways-to-troubleshoot-rdp-issues"></a>Comment résoudre les problèmes de connexion RDP
Vous pouvez résoudre les problèmes affectant les machines virtuelles créées à l’aide du modèle de déploiement Resource Manager en utilisant l’une des méthodes suivantes :

* Portail Azure : utile si vous devez rapidement rétablir la configuration RDP ou les informations d’identification utilisateur et que vous n’avez pas installé les outils Azure.
* Azure PowerShell : si vous êtes familiarisé avec les invites PowerShell, réinitialisez rapidement la configuration RDP ou les informations d’identification utilisateur à l’aide des applets de commande Azure PowerShell.

Vous trouverez également des étapes sur la résolution des problèmes affectant les machines virtuelles créées à l’aide du [modèle de déploiement classique](#troubleshoot-vms-created-using-the-classic-deployment-model).

<a id="fix-common-remote-desktop-errors"></a>

## <a name="troubleshoot-using-the-azure-portal"></a>Résolution des problèmes à l’aide du portail Azure
Après chaque étape de résolution des problèmes, essayez de nouveau de vous connecter à la machine virtuelle. Si vous ne parvenez toujours pas à vous connecter, essayez l’étape suivante.

1. **Réinitialisez votre connexion RDP**. Cette étape de dépannage réinitialise la configuration RDP lorsque des connexions à distance sont désactivées ou si des règles du pare-feu Windows bloquent la connexion RDP, par exemple.
   
    Sélectionnez votre machine virtuelle dans le portail Azure. Faites défiler le volet des paramètres jusqu’à la section **Support + dépannage** en bas de la liste. Cliquez sur le bouton **Réinitialiser le mot de passe**. Définissez l’option **Mode** sur **Réinitialiser la configuration uniquement** puis cliquez sur le bouton **Mettre à jour** :
   
    ![Réinitialiser la configuration RDP dans le portail Azure.](./media/troubleshoot-rdp-connection/reset-rdp.png)
2. **Vérifiez les règles du groupe de sécurité réseau**. Utilisez la [vérification des flux IP](../../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) pour savoir si une règle d’un groupe de sécurité réseau bloque le trafic depuis ou vers une machine virtuelle. Vous pouvez également vérifier les règles de groupe de sécurité effectives pour vous assurer que la règle « Allow » entrante du groupe de sécurité réseau existe pour le port RDP (par défaut, 3389). Pour en savoir plus, voir [Utilisation de règles de sécurité effectives pour résoudre des problèmes de flux de trafic de machine virtuelle](../../virtual-network/diagnose-network-traffic-filter-problem.md).

3. **Passez en revue les diagnostics de démarrage de la machine virtuelle**. Cette étape de dépannage passe en revue les journaux d’activité de la console de la machine virtuelle afin de déterminer si celle-ci signale un problème. Les diagnostics de démarrage ne sont pas activés sur toutes les machines virtuelles et cette étape de dépannage peut être facultative.
   
    Les étapes de dépannage spécifiques ne sont pas abordées dans cet article, mais elles peuvent indiquer un problème plus large affectant la connectivité RDP. Pour plus d’informations sur l’examen des journaux d’activité de la console et pour obtenir une capture d’écran de la machine virtuelle, consultez [Diagnostics de démarrage pour les machines virtuelles](boot-diagnostics.md).

4. **Réinitialisez la carte d’interface réseau pour la machine virtuelle**. Pour en savoir plus, consultez la page [How to reset NIC for Azure Windows VM](./reset-network-interface.md) (Réinitialiser une carte d’interface réseau pour une machine virtuelle Windows Azure).
5. **Vérifiez l’intégrité des ressources de la machine virtuelle**. Cette étape de dépannage vérifie qu’il n’existe aucun problème connu sur la plateforme Azure susceptible d’avoir un impact sur la connectivité à la machine virtuelle.
   
    Sélectionnez votre machine virtuelle dans le portail Azure. Faites défiler le volet des paramètres jusqu’à la section **Support + dépannage** en bas de la liste. Cliquez sur le bouton **Intégrité des ressources**. Une machine virtuelle saine est indiquée comme étant **Disponible** :
   
    ![Vérifier l’intégrité des ressources de machine virtuelle dans le portail Azure.](./media/troubleshoot-rdp-connection/check-resource-health.png)
6. **Réinitialisez les informations d’identification de l’utilisateur**. Cette étape de dépannage réinitialise le mot de passe sur un compte d’administrateur local lorsque vous ne connaissez pas ou si vous avez oublié les informations d’identification.  Une fois que vous êtes connecté à la machine virtuelle, vous devez réinitialiser le mot de passe pour cet utilisateur.
   
    Sélectionnez votre machine virtuelle dans le portail Azure. Faites défiler le volet des paramètres jusqu’à la section **Support + dépannage** en bas de la liste. Cliquez sur le bouton **Réinitialiser le mot de passe**. Assurez-vous que le **Mode** est défini sur **Réinitialiser le mot de passe**, puis entrez votre nom d’utilisateur et un nouveau mot de passe. Enfin, cliquez sur le bouton **Mettre à jour** :
   
    ![Réinitialiser les informations d’identification de l’utilisateur dans le portail Azure.](./media/troubleshoot-rdp-connection/reset-password.png)
7. **Redémarrez votre machine virtuelle**. Cette étape de dépannage permet de corriger tout problème sous-jacent rencontré par la machine virtuelle.
   
    Sélectionnez votre machine virtuelle dans le portail Azure et cliquez sur l’onglet **Vue d’ensemble**. Cliquez sur le bouton **Redémarrer** :
   
    ![Redémarrer la machine virtuelle dans le portail Azure.](./media/troubleshoot-rdp-connection/restart-vm.png)
8. **Redéployez votre machine virtuelle**. Cette étape de dépannage redéploie votre machine virtuelle sur un autre hôte dans Azure pour corriger tout problème sous-jacent lié au réseau ou à la plateforme.
   
    Sélectionnez votre machine virtuelle dans le portail Azure. Faites défiler le volet des paramètres jusqu’à la section **Support + dépannage** en bas de la liste. Cliquez sur le bouton **Redéployer**, puis sur **Redéployer** :
   
    ![Redéployer la machine virtuelle dans le portail Azure](./media/troubleshoot-rdp-connection/redeploy-vm.png)
   
    Une fois cette opération terminée, les données de disque éphémères sont perdues et les adresses IP dynamiques associées à la machine virtuelle sont mises à jour.

9. **Vérifiez le routage**. Utilisez la fonctionnalité [Tronçon suivant](../../network-watcher/diagnose-vm-network-routing-problem.md) de Network Watcher pour vérifier qu’un itinéraire n’empêche pas le trafic d’être routé à destination ou en provenance d’une machine virtuelle. Vous pouvez également examiner les itinéraires effectifs pour voir tous les itinéraires effectifs pour une interface réseau. Pour plus d’informations, consultez [Utilisation d’itinéraires effectifs pour résoudre des problèmes de flux de trafic de machine virtuelle](../../virtual-network/diagnose-network-routing-problem.md).

10. Vérifiez que le pare-feu de votre ordinateur ou vos pare-feu locaux autorisent le trafic sortant vers Azure via le port TCP 3389.

Si vous rencontrez toujours des problèmes liés au protocole RDP, vous pouvez [ouvrir une demande de support](https://azure.microsoft.com/support/options/) ou lire [des concepts et des étapes de résolution des problèmes RDP plus détaillés](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="troubleshoot-using-azure-powershell"></a>Résolution des problèmes à l’aide d’Azure PowerShell
Si vous ne l’avez pas déjà fait, [installez et configurez la dernière version d’Azure PowerShell](/powershell/azure/).

Les exemples suivants utilisent des variables telles que `myResourceGroup`, `myVM` et `myVMAccessExtension`. Remplacez ces noms de variables et les emplacements par vos propres valeurs.

> [!NOTE]
> Vous pouvez réinitialiser les informations d’identification de l’utilisateur et la configuration RDP en utilisant la cmdlet PowerShell [Set-AzVMAccessExtension](/powershell/module/az.compute/set-azvmaccessextension). Dans les exemples suivants, `myVMAccessExtension` est un nom que vous spécifiez dans le cadre du processus. Si vous avez déjà utilisé VMAccessAgent, vous pouvez obtenir le nom de l’extension existante à l’aide de `Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"` pour vérifier les propriétés de la machine virtuelle. Pour afficher le nom, examinez la section « Extensions » de la sortie.

Après chaque étape de résolution des problèmes, essayez de nouveau de vous connecter à la machine virtuelle. Si vous ne parvenez toujours pas à vous connecter, essayez l’étape suivante.

1. **Réinitialisez votre connexion RDP**. Cette étape de dépannage réinitialise la configuration RDP lorsque des connexions à distance sont désactivées ou si des règles du pare-feu Windows bloquent la connexion RDP, par exemple.
   
    L’exemple suivant réinitialise la connexion RDP sur une machine virtuelle nommée `myVM` dans l’emplacement `WestUS` et dans le groupe de ressources nommé `myResourceGroup` :
   
    ```powershell
    Set-AzVMAccessExtension -ResourceGroupName "myResourceGroup" `
        -VMName "myVM" -Location Westus -Name "myVMAccessExtension"
    ```
2. **Vérifiez les règles du groupe de sécurité réseau**. Cette étape de dépannage permet de vérifier que vous disposez d’une règle autorisant le trafic RDP dans votre groupe de sécurité réseau. Le port par défaut pour RDP est le port TCP 3389. Une règle autorisant le trafic RDP peut ne pas être créée automatiquement lorsque vous créez votre machine virtuelle.
   
    Tout d’abord, affectez toutes les données de configuration de votre groupe de sécurité réseau à la variable `$rules`. L’exemple suivant obtient des informations sur le groupe de sécurité réseau nommé `myNetworkSecurityGroup` dans le groupe de ressources nommé `myResourceGroup` :
   
    ```powershell
    $rules = Get-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" `
        -Name "myNetworkSecurityGroup"
    ```
   
    À présent, affichez les règles qui sont configurées pour ce groupe de sécurité réseau. Vérifiez qu’il existe une règle pour autoriser le port TCP 3389 pour les connexions entrantes comme suit :
   
    ```powershell
    $rules.SecurityRules
    ```
   
    L’exemple suivant montre une règle de sécurité valide qui autorise le trafic RDP. Vous pouvez voir que `Protocol`, `DestinationPortRange`, `Access` et `Direction` sont correctement configurés :
   
    ```powershell
    Name                     : default-allow-rdp
    Id                       : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/default-allow-rdp
    Etag                     : 
    ProvisioningState        : Succeeded
    Description              : 
    Protocol                 : TCP
    SourcePortRange          : *
    DestinationPortRange     : 3389
    SourceAddressPrefix      : *
    DestinationAddressPrefix : *
    Access                   : Allow
    Priority                 : 1000
    Direction                : Inbound
    ```
   
    Si vous ne disposez pas de règle autorisant le trafic RDP, [créez une règle de groupe de sécurité réseau](../windows/nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Autorisez le port TCP 3389.
3. **Réinitialisez les informations d’identification de l’utilisateur**. Cette étape de dépannage réinitialise le mot de passe du compte de l’administrateur local que vous spécifiez lorsque vous ne connaissez pas ou si vous avez oublié les informations d’identification.
   
    Tout d’abord, spécifiez le nom d’utilisateur et un nouveau mot de passe en affectant des informations d’identification à la variable `$cred` comme suit :
   
    ```powershell
    $cred=Get-Credential
    ```
   
    À présent, mettez à jour les informations d’identification sur votre machine virtuelle. L’exemple suivant met à jour les informations d’identification sur une machine virtuelle nommée `myVM` dans l’emplacement `WestUS` et dans le groupe de ressources nommé `myResourceGroup` :
   
    ```powershell
    Set-AzVMAccessExtension -ResourceGroupName "myResourceGroup" `
        -VMName "myVM" -Location WestUS -Name "myVMAccessExtension" `
        -UserName $cred.GetNetworkCredential().Username `
        -Password $cred.GetNetworkCredential().Password
    ```
4. **Redémarrez votre machine virtuelle**. Cette étape de dépannage permet de corriger tout problème sous-jacent rencontré par la machine virtuelle.
   
    L’exemple suivant redémarre la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` :
   
    ```powershell
    Restart-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
    ```
5. **Redéployez votre machine virtuelle**. Cette étape de dépannage redéploie votre machine virtuelle sur un autre hôte dans Azure pour corriger tout problème sous-jacent lié au réseau ou à la plateforme.
   
    L’exemple suivant redéploie la machine virtuelle nommée `myVM` dans l’emplacement `WestUS` et dans le groupe de ressources nommé `myResourceGroup` :
   
    ```powershell
    Set-AzVM -Redeploy -ResourceGroupName "myResourceGroup" -Name "myVM"
    ```

6. **Vérifiez le routage**. Utilisez la fonctionnalité [Tronçon suivant](../../network-watcher/diagnose-vm-network-routing-problem.md) de Network Watcher pour vérifier qu’un itinéraire n’empêche pas le trafic d’être routé à destination ou en provenance d’une machine virtuelle. Vous pouvez également examiner les itinéraires effectifs pour voir tous les itinéraires effectifs pour une interface réseau. Pour plus d’informations, consultez [Utilisation d’itinéraires effectifs pour résoudre des problèmes de flux de trafic de machine virtuelle](../../virtual-network/diagnose-network-routing-problem.md).

7. Vérifiez que le pare-feu de votre ordinateur ou vos pare-feu locaux autorisent le trafic sortant vers Azure via le port TCP 3389.

Si vous rencontrez toujours des problèmes liés au protocole RDP, vous pouvez [ouvrir une demande de support](https://azure.microsoft.com/support/options/) ou lire [des concepts et des étapes de résolution des problèmes RDP plus détaillés](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="troubleshoot-vms-created-using-the-classic-deployment-model"></a>Résolution des problèmes des machines virtuelles créées à l’aide du modèle de déploiement classique

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]


Après chaque étape de résolution des problèmes, essayez de vous reconnecter à la machine virtuelle.

1. **Réinitialisez votre connexion RDP**. Cette étape de dépannage réinitialise la configuration RDP lorsque des connexions à distance sont désactivées ou si des règles du pare-feu Windows bloquent la connexion RDP, par exemple.
   
    Sélectionnez votre machine virtuelle dans le portail Azure. Cliquez sur le bouton **...Plus** puis cliquez sur **Réinitialiser l’accès à distance** :
   
    ![Réinitialiser la configuration RDP dans le portail Azure pour machine virtuelle à l’aide du modèle de déploiement classique.](./media/troubleshoot-rdp-connection/classic-reset-rdp.png)
2. **Vérifiez les points de terminaison des Services cloud**. Cette étape de dépannage vérifie que vous avez des points de terminaison dans vos Services cloud pour autoriser le trafic RDP. Le port par défaut pour RDP est le port TCP 3389. Une règle autorisant le trafic RDP peut ne pas être créée automatiquement lorsque vous créez votre machine virtuelle.
   
   Sélectionnez votre machine virtuelle dans le portail Azure. Cliquez sur le bouton **Points de terminaison** pour afficher les points de terminaison actuellement configurés pour votre machine virtuelle. Vérifiez la présence de points de terminaison autorisant le trafic RDP sur le port TCP 3389.
   
   L’exemple suivant montre les points de terminaison valides qui autorisent le trafic RDP :
   
   ![Vérifier les points de terminaison des services cloud dans le portail Azure pour machine virtuelle à l’aide du modèle de déploiement classique.](./media/troubleshoot-rdp-connection/classic-verify-cloud-services-endpoints.png)
   
   Si vous ne disposez pas d’un point de terminaison autorisant le trafic RDP, [créez un point de terminaison de Services cloud](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints). Autorisez TCP sur le port privé 3389.
3. **Passez en revue les diagnostics de démarrage de la machine virtuelle**. Cette étape de dépannage passe en revue les journaux d’activité de la console de la machine virtuelle afin de déterminer si celle-ci signale un problème. Les diagnostics de démarrage ne sont pas activés sur toutes les machines virtuelles et cette étape de dépannage peut être facultative.
   
    Les étapes de dépannage spécifiques ne sont pas abordées dans cet article, mais elles peuvent indiquer un problème plus large affectant la connectivité RDP. Pour plus d’informations sur l’examen des journaux d’activité de la console et pour obtenir une capture d’écran de la machine virtuelle, consultez [Diagnostics de démarrage pour les machines virtuelles](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/).
4. **Vérifiez l’intégrité des ressources de la machine virtuelle**. Cette étape de dépannage vérifie qu’il n’existe aucun problème connu sur la plateforme Azure susceptible d’avoir un impact sur la connectivité à la machine virtuelle.
   
    Sélectionnez votre machine virtuelle dans le portail Azure. Faites défiler le volet des paramètres jusqu’à la section **Support + dépannage** en bas de la liste. Cliquez sur le bouton **Intégrité des ressources**. Une machine virtuelle saine est indiquée comme étant **Disponible** :
   
    ![Vérifier l’intégrité d’une ressource de machine virtuelle dans le portail Azure pour machine virtuelle à l’aide du modèle de déploiement classique.](./media/troubleshoot-rdp-connection/classic-check-resource-health.png)
5. **Réinitialisez les informations d’identification de l’utilisateur**. Cette étape de dépannage réinitialise le mot de passe du compte de l’administrateur local que vous spécifiez lorsque vous ne connaissez pas ou si vous avez oublié les informations d’identification.  Une fois que vous êtes connecté à la machine virtuelle, vous devez réinitialiser le mot de passe pour cet utilisateur.
   
    Sélectionnez votre machine virtuelle dans le portail Azure. Faites défiler le volet des paramètres jusqu’à la section **Support + dépannage** en bas de la liste. Cliquez sur le bouton **Réinitialiser le mot de passe**. Entrez votre nom d’utilisateur et un nouveau mot de passe. Enfin, cliquez sur le bouton **Enregistrer** :
   
    ![Réinitialiser les informations d’identification de l’utilisateur dans le portail Azure pour machine virtuelle à l’aide du modèle de déploiement classique.](./media/troubleshoot-rdp-connection/classic-reset-password.png)
6. **Redémarrez votre machine virtuelle**. Cette étape de dépannage permet de corriger tout problème sous-jacent rencontré par la machine virtuelle.
   
    Sélectionnez votre machine virtuelle dans le portail Azure et cliquez sur l’onglet **Vue d’ensemble**. Cliquez sur le bouton **Redémarrer** :
   
    ![Redémarrer la machine virtuelle dans le portail Azure pour machine virtuelle à l’aide du modèle de déploiement classique.](./media/troubleshoot-rdp-connection/classic-restart-vm.png)

7. Vérifiez que le pare-feu de votre ordinateur ou vos pare-feu locaux autorisent le trafic sortant vers Azure via le port TCP 3389.

Si vous rencontrez toujours des problèmes liés au protocole RDP, vous pouvez [ouvrir une demande de support](https://azure.microsoft.com/support/options/) ou lire [des concepts et des étapes de résolution des problèmes RDP plus détaillés](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="troubleshoot-specific-rdp-errors"></a>Dépannage d’erreurs RDP spécifiques
Vous pouvez recevoir un message d’erreur spécifique lorsque vous tentez de vous connecter à votre machine virtuelle suivant le protocole RDP. Voici les messages d’erreur les plus courants :

* [La session distante a été déconnectée, car aucun serveur de licences Bureau à distance n’est disponible pour fournir une licence](troubleshoot-specific-rdp-errors.md#rdplicense).
* [Le bureau à distance ne trouve pas le « name » de l’ordinateur](troubleshoot-specific-rdp-errors.md#rdpname).
* [Une erreur d’authentification s’est produite. L’autorité de sécurité locale ne peut pas être contactée](troubleshoot-specific-rdp-errors.md#rdpauth).
* [Message d’erreur de sécurité Windows : Vos informations d’identification n’ont pas fonctionné](troubleshoot-specific-rdp-errors.md#wincred).
* [Cet ordinateur ne peut pas se connecter à l’ordinateur distant](troubleshoot-specific-rdp-errors.md#rdpconnect).

## <a name="additional-resources"></a>Ressources supplémentaires
Si aucune de ces erreurs ne s’est produite et que vous ne parvenez toujours pas à vous connecter à la machine virtuelle avec le Bureau à distance, consultez le [guide de résolution des problèmes du Bureau à distance](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
* Pour connaître les étapes de résolution des problèmes d’accès aux applications exécutées sur une machine virtuelle, consultez [Résolution des problèmes d’accès à une application exécutée sur une machine virtuelle Azure](./troubleshoot-app-connection.md?toc=/azure/virtual-machines/linux/toc.json).
* Si vous rencontrez des problèmes pour vous connecter à une machine virtuelle Linux dans Azure avec SSH (Secure Shell), consultez [Résolution des problèmes de connexions SSH à une machine virtuelle de Linux dans Azure](./troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json).
