---
title: Le serveur de licences des services Bureau à distance n’est pas disponible lorsque vous vous connectez à une machine virtuelle Azure | Microsoft Docs
description: Découvrez comment résoudre les problèmes d’échec de RDP dus à l’indisponibilité d’un serveur de licences des services Bureau à distance | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/23/2018
ms.author: genli
ms.openlocfilehash: 6b5864d40d2a4d8f8d6cf404df29f909a73f04e2
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97832041"
---
# <a name="remote-desktop-license-server-isnt-available-when-you-connect-to-an-azure-vm"></a>Le serveur de licences des services Bureau à distance n’est pas disponible lorsque vous vous connectez à une machine virtuelle Azure

Cet article vous aide à résoudre le problème lorsque vous ne pouvez pas vous connecter à une machine virtuelle Azure, car aucun serveur de licences des services Bureau à distance n’est disponible pour fournir une licence.

## <a name="symptoms"></a>Symptômes

Lorsque vous essayez de vous connecter à une machine virtuelle (VM), vous rencontrez les situations suivantes :

- La capture d’écran de la VM montre que le système d’exploitation est entièrement chargé et attend des informations d’identification.
- Lorsque vous essayez d’établir une connexion de Microsoft RDP (Remote Desktop Protocol), les messages d’erreur suivant s’affichent :

  - La session distante a été déconnectée, car aucun serveur de licences des services Bureau à distance n’est disponible pour fournir une licence.

  - Aucun serveur de licences des services Bureau à distance n’est disponible. Les services Bureau à distance cesseront de fonctionner, car cet ordinateur a dépassé la période de grâce et n’a pas contacté au moins un serveur de licences Windows Server 2008 valide. Sélectionnez ce message pour ouvrir la configuration du serveur hôte de session Bureau à distance et utiliser le diagnostic des licences.

Toutefois, vous pouvez vous connecter à la machine virtuelle normalement à l’aide d’une session d’administration :

```
mstsc /v:<Server>[:<Port>] /admin
```

## <a name="cause"></a>Cause

Ce problème survient si un serveur de licences des services Bureau à distance n’est pas disponible pour fournir une licence permettant de démarrer une session à distance. Il peut avoir plusieurs origines, même si un rôle Hôte de session Bureau à distance a été configuré sur la machine virtuelle :

- Il n’a jamais eu de rôle de licences Bureau à distance dans l’environnement et la période de grâce de 180 jours est écoulée.
- Une licence Bureau à distance a été installée dans l’environnement, mais elle n’a jamais été activée.
- Aucune licence d’accès client (CAL) n’a été injectée dans une licence Bureau à distance de l’environnement pour configurer la connexion.
- Une licence Bureau à distance a été installée dans l’environnement. Des licences d’accès client sont disponibles, mais elles n’ont pas été configurées correctement.
- Une licence Bureau à distance a des licences d’accès client et elle a été activée. Toutefois, d’autres problèmes sur le serveur de licences des services Bureau à distance l’empêchent de fournir des licences dans l’environnement.

## <a name="solution"></a>Solution

Pour résoudre ce problème, [sauvegardez le disque du système d’exploitation](../windows/snapshot-copy-managed-disk.md) et procédez comme suit :

1. Connectez-vous à la machine virtuelle à l’aide d’une session d’administration :

   ```
   mstsc /v:<Server>[:<Port>] /admin
   ```

    Si vous ne pouvez pas vous connecter à la machine virtuelle à l’aide d’une session d’administration, vous pouvez utiliser la [console série de la machine virtuelle sur Azure](serial-console-windows.md) pour accéder à la machine virtuelle comme suit :

    1. Accéder à la Console série en sélectionnant **Support & Troubleshooting (Support et dépannage)**  > **Console série (préversion)** . Si la fonctionnalité est activée sur la machine virtuelle, vous pouvez connecter la machine.

    2. Créez un canal pour une instance CMD. Entrez **CMD** pour démarrer le canal et en obtenir le nom.

    3. Basculez vers le canal qui exécute l’instance CMD. Dans ce cas, il devrait s’agir du canal 1 :

       ```
       ch -si 1
       ```

    4. Sélectionnez de nouveau **Entrée** et entrez un nom d’utilisateur et un mot de passe valides ainsi qu’un ID local ou de domaine pour la machine virtuelle.

2. Vérifiez si un rôle Hôte de session Bureau à distance est activé sur la machine virtuelle. Si le rôle est activé, assurez-vous qu’il fonctionne correctement. Ouvrez une instance CMD avec élévation de privilèges et procédez comme suit :

    1. Utilisez la commande suivante pour vérifier l’état du rôle Hôte de session Bureau à distance :

       ```
        reg query "HKLM\SOFTWARE\Microsoft\ServerManager\ServicingStorage\ServerComponentCache\RDS-RD-Server" /v InstallState
        ```

        Si cette commande renvoie la valeur 0, cela signifie que le rôle est désactivé. Passez à l’étape 3.

    2. Utilisez la commande suivante pour vérifier les stratégies et, au besoin, reconfigurez-les :

       ```
        reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\RCM\Licensing Core" /v LicensingMode

        reg query "HKLM\SYSTEM\CurrentControlSet\Services\TermService\Parameters" /v SpecifiedLicenseServers
       ```

        Si **LicensingMode** a une valeur autre que 4, par utilisateur, réglez-le sur 4 :

         ```
        reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\RCM\Licensing Core" /v LicensingMode /t REG_DWORD /d 4
        ```

       Si la valeur **SpecifiedLicenseServers** n’existe pas ou si les informations du serveur de licences sont incorrectes, modifiez-la comme suit :

       ```
        reg add "HKLM\SYSTEM\CurrentControlSet\Services\TermService\Parameters" /v SpecifiedLicenseServers /t REG_MULTI_SZ /d "<FQDN / IP License server>"
       ```

    3. Après avoir modifié le registre, redémarrez la machine virtuelle.

    4. Si vous n’avez pas de licences d’accès client, supprimez le rôle Hôte de session Bureau à distance. Ensuite, le protocole RDP retourne à la normale. Il n’autorise que deux connexions RDP simultanées à la machine virtuelle :

        ```
       dism /ONLINE /Disable-feature /FeatureName:Remote-Desktop-Services
        ```

        Si la machine virtuelle a le rôle de licences Bureau à distance et qu’il n’est pas utilisé, vous pouvez également le supprimer :

       ```
        dism /ONLINE /Disable-feature /FeatureName:Licensing
       ```

    5. Assurez-vous que la machine virtuelle peut se connecter au serveur de licences de services Bureau à distance. Vous pouvez tester la connectivité au port 135 entre la machine virtuelle et le serveur de licences : 

       ```
       telnet <FQDN / IP License Server> 135
       ```

3. S’il n’y a aucun serveur de licences des services Bureau à distance dans l’environnement et que vous en souhaitez un, vous pouvez [installer un service de rôle Gestionnaire de licences des services Bureau à distance](/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc731765(v=ws.11)). [Configurer ensuite les licences des services Bureau à distance](https://techcommunity.microsoft.com/t5/Ask-The-Performance-Team/RD-Licensing-Configuration-on-Windows-Server-2012/ba-p/375383).

4. Si un serveur de licences des services Bureau à distance est configuré et intègre, assurez-vous qu’il est activé avec des licences d’accès client.

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez toujours besoin d’aide, [contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre votre problème.
