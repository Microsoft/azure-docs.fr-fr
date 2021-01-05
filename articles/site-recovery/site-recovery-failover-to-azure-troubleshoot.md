---
title: Résoudre les problèmes d’échec de basculement vers Azure | Microsoft Docs
description: Cet article explique comment résoudre les erreurs courantes qui surviennent lors d’un basculement vers Azure
author: ponatara
manager: abhemraj
ms.service: site-recovery
services: site-recovery
ms.topic: article
ms.workload: storage-backup-recovery
ms.date: 01/08/2020
ms.author: mayg
ms.openlocfilehash: d2a0444483c382da7c54accf7dca49d097671771
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92371985"
---
# <a name="troubleshoot-errors-when-failing-over-vmware-vm-or-physical-machine-to-azure"></a>Résoudre les erreurs se produisant lors du basculement d’une machine virtuelle VMware ou d'une machine physique vers Azure

Vous pouvez recevoir les erreurs suivantes lorsque vous procédez au basculement d’une machine virtuelle vers Azure. Pour résoudre les problèmes, servez-vous des étapes décrites pour chaque condition d’erreur.

## <a name="failover-failed-with-error-id-28031"></a>Le basculement a échoué avec l’ID d’erreur 28031

Site Recovery n’a pas pu créer de machine virtuelle basculée dans Azure. L’une des raisons suivantes peut en être la cause :

* Le quota est suffisant pour créer la machine virtuelle : Vous pouvez vérifier le quota disponible en accédant à Abonnement -> Utilisation + quotas. Vous pouvez ouvrir une [nouvelle demande de support](https://aka.ms/getazuresupport) pour augmenter le quota.

* Vous tentez de basculer des machines virtuelles de familles de taille différentes dans le même groupe à haute disponibilité. Veillez à choisir le même ordre de grandeur de taille pour toutes les machines virtuelles d’un même groupe à haute disponibilité. Changez la taille en accédant aux paramètres Calcul et réseau de la machine virtuelle, puis réessayez le basculement.

* Il existe une stratégie sur l’abonnement qui empêche la création d’une machine virtuelle. Modifiez la stratégie pour permettre la création d’une machine virtuelle, puis réessayez le basculement.

## <a name="failover-failed-with-error-id-28092"></a>Le basculement a échoué avec l’ID d’erreur 28092

Site Recovery n’a pas pu créer d’interface réseau pour la machine virtuelle basculée. Assurez-vous que vous avez un quota suffisant pour créer des interfaces réseau dans l’abonnement. Vous pouvez vérifier le quota disponible en accédant à Abonnement -> Utilisation + quotas. Vous pouvez ouvrir une [nouvelle demande de support](https://aka.ms/getazuresupport) pour augmenter le quota. Si vous disposez d’un quota suffisant, le problème peut être occasionnel, recommencez l’opération. Si le problème persiste même après plusieurs tentatives, laissez un commentaire à la fin de ce document.  

## <a name="failover-failed-with-error-id-70038"></a>Le basculement a échoué avec l’ID d’erreur 70038

Site Recovery n’a pas pu créer de machine virtuelle classique basculée dans Azure. Les raisons suivantes peuvent en être la cause :

* Une des ressources, comme un réseau virtuel, nécessaire à la création de la machine virtuelle n’existe pas. Créez le réseau virtuel, tel que proposé sous les paramètres Calcul et réseau de la machine virtuelle, ou modifiez le paramètre sur un réseau virtuel qui existe déjà et réessayez le basculement.

## <a name="failover-failed-with-error-id-170010"></a>Le basculement a échoué avec l’ID d’erreur 170010

Site Recovery n’a pas pu créer de machine virtuelle basculée dans Azure. Cela peut être causé par l’échec d’une activité interne d’alimentation pour la machine virtuelle locale.

Pour afficher une machine dans Azure, l’environnement Azure exige que certains pilotes soient à l’état Démarrage et que des services comme DHCP soient à l’état Démarrage automatique. Au moment du basculement, l’activité d’alimentation convertit donc le type de démarrage des **pilotes atapi, intelide, storflt, vmbus et storvsc** en Démarrage, et celui de certains services comme DHCP en Démarrage automatique. Cette activité peut échouer en raison de problèmes propres à l’environnement. 

Pour modifier manuellement le type de démarrage des pilotes pour **SE invité Windows** , suivez les étapes ci-dessous :

1. [Téléchargez](https://download.microsoft.com/download/5/D/6/5D60E67C-2B4F-4C51-B291-A97732F92369/Script-no-hydration.ps1) le script de non-alimentation, puis exécutez-le de la façon suivante. Ce script détermine si la machine virtuelle a besoin d’une alimentation.

    `.\Script-no-hydration.ps1`

    Il donne le résultat suivant si une alimentation est nécessaire :

    ```output
    REGISTRY::HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\services\storvsc           start =  3 expected value =  0

    This system doesn't meet no-hydration requirement.
    ```

    Dans le cas où la machine virtuelle remplit l’exigence de non-alimentation, le script donne le résultat « This system meets no-hydration requirement ». Tous les pilotes et services se trouvent à l’état requis par Azure ; l’alimentation n’est pas nécessaire sur la machine virtuelle.

2. Exécutez ainsi le script no-hydration-set si la machine virtuelle ne répond pas à l’exigence de non-alimentation.

    `.\Script-no-hydration.ps1 -set`
    
    Il convertit le type de démarrage des pilotes et donne le résultat ci-dessous :

    ```output
    REGISTRY::HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\services\storvsc           start =  3 expected value =  0

    Updating registry:  REGISTRY::HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\services\storvsc   start =  0

    This system is now no-hydration compatible.
    ```

## <a name="unable-to-connectrdpssh-to-the-failed-over-virtual-machine-due-to-grayed-out-connect-button-on-the-virtual-machine"></a>Impossible de se connecter ou d’établir une liaison RDP ou SSH à la machine virtuelle après son basculement, car le bouton Se connecter est grisé

Pour obtenir des instructions détaillées de résolution des problèmes liés à RDP, consultez notre documentation [ici](../virtual-machines/troubleshooting/troubleshoot-rdp-connection.md).

Pour obtenir des instructions détaillées sur la résolution des problèmes liés à SSH, consultez notre documentation [ici](../virtual-machines/troubleshooting/troubleshoot-ssh-connection.md).

Si le bouton **Se connecter** de la machine virtuelle basculée dans Azure est grisé et que vous n’avez pas établi de connexion ExpressRoute ou réseau privé virtuel de site à site à Azure :

1. Accédez à **Machine virtuelle** > **Réseaux** , cliquez sur le nom de l’interface réseau concernée.  ![Capture d'écran représentant la page Réseaux d'une machine virtuelle, sur laquelle le nom de l'interface réseau est sélectionné.](media/site-recovery-failover-to-azure-troubleshoot/network-interface.PNG)
2. Accédez à **Configurations d’adresses IP** , puis cliquez sur le champ Nom de la configuration d’adresse IP souhaitée. ![Capture d'écran représentant la page Configurations d'adresses IP de l'interface réseau, sur laquelle le nom de la configuration d'adresses IP est sélectionné.](media/site-recovery-failover-to-azure-troubleshoot/IpConfigurations.png)
3. Pour activer l’adresse IP publique, cliquez sur **Activer** . ![Activer l’adresse IP](media/site-recovery-failover-to-azure-troubleshoot/Enable-Public-IP.png)
4. Cliquez sur **Configurer les paramètres requis** > **Créer** . ![Créer](media/site-recovery-failover-to-azure-troubleshoot/Create-New-Public-IP.png)
5. Entrez le nom de l’adresse publique, choisissez les options par défaut pour **SKU** et **Affectation** , puis cliquez sur **OK** .
6. Pour enregistrer les modifications, cliquez sur **Enregistrer** .
7. Fermez les panneaux et accédez à la section **Vue d’ensemble** de la machine virtuelle pour vous connecter/établir une liaison RDP.

## <a name="unable-to-connectrdpssh---vm-connect-button-available"></a>Impossible de se connecter/RDP/SSH – Le bouton Se connecter de la machine virtuelle est disponible

Si le bouton **Se connecter** de la machine virtuelle basculée dans Azure est disponible (s’il n’est pas grisé), cochez **Diagnostics de démarrage** sur votre machine virtuelle et recherchez les erreurs mentionnées dans [cet article](../virtual-machines/troubleshooting/boot-diagnostics.md).

1. Si la machine virtuelle n’a pas démarré, tentez de basculer vers un point de récupération plus ancien.
2. Si l’application de la machine virtuelle n’est pas disponible, tentez de basculer vers un point de récupération cohérent au niveau application.
3. Si la machine virtuelle est jointe au domaine, vérifiez que le contrôleur de domaine fonctionne bien. Pour cela, vous pouvez suivre les étapes ci-dessous :

    a. Créez une machine virtuelle dans le même réseau.

    b.  Veillez à ce qu’elle puisse être jointe au domaine sur lequel la machine virtuelle devrait apparaître après son basculement.

    c. Si le contrôleur de domaine ne fonctionne **pas** bien, essayez de vous connecter à la machine virtuelle basculée à l’aide d’un compte Administrateur local.
4. Si vous utilisez un serveur DNS personnalisé, vérifiez qu’il est accessible. Pour cela, vous pouvez suivre les étapes ci-dessous :

    a. Créez une machine virtuelle dans le même réseau et

    b. Vérifiez que la machine virtuelle parvient à effectuer la résolution de noms à l’aide du serveur DNS personnalisé

>[!Note]
>Tout paramètre autre que les diagnostics de démarrage nécessite l’installation de l’agent de machine virtuelle Azure sur la machine virtuelle avant le basculement.

## <a name="unable-to-open-serial-console-after-failover-of-a-uefi-based-machine-into-azure"></a>Impossible d’ouvrir la Serial console après le basculement d’une machine UEFI dans Azure

Si vous êtes en mesure de vous connecter à la machine à l’aide du protocole RDP, mais que vous ne pouvez pas ouvrir la Serial console, procédez comme suit :

* Si le système d’exploitation de le la machine est Red Hat ou Oracle Linux 7.*/8.0, exécutez la commande suivante sur la machine virtuelle Azure de basculement avec les autorisations racine. Redémarrez la machine virtuelle après avoir exécuté la commande.

  ```console
  grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
  ```

* Si le système d’exploitation de le la machine est CentOS 7*, exécutez la commande suivante sur la machine virtuelle Azure de basculement avec les autorisations racine. Redémarrez la machine virtuelle après avoir exécuté la commande.

  ```console
  grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
  ```

## <a name="unexpected-shutdown-message-event-id-6008"></a>Message d’arrêt inattendu (ID d’événement 6008)

Lors du démarrage d’un machine virtuelle Windows après le basculement, si un message d’arrêt inattendu s’affiche sur la machine virtuelle récupérée, cela indique qu’un état d’arrêt de la machine virtuelle n’a pas été capturé dans le point de récupération utilisé pour le basculement. Cela se produit lorsque vous récupérez à un point où la machine virtuelle n'avait pas été complètement arrêtée.

Il ne s’agit normalement pas d’une source de préoccupation et cela peut généralement être ignoré pour les basculements non planifiés. Si le basculement est planifié, assurez-vous que la machine virtuelle a été correctement arrêtée avant le basculement et patientez suffisamment pour que les données de réplication locales en attente soient envoyées à Azure. Utilisez ensuite l’option **Plus récent** sur l’ [écran de basculement](site-recovery-failover.md#run-a-failover) pour que toutes les données en attente sur Azure soient traitées en un point de récupération, qui est ensuite utilisé pour le basculement de la machine virtuelle.

## <a name="unable-to-select-the-datastore"></a>Impossible de sélectionner la banque de données

Ce problème est indiqué quand vous ne voyez pas la banque de données dans le portail Azure alors que vous essayez de reprotéger la machine virtuelle qui a fait l’objet d’un basculement. Il est dû au fait que la cible maître n’est pas reconnue en tant que machine virtuelle sous les vCenters ajoutés à Azure Site Recovery.

Pour plus d’informations sur la reprotection d’une machine virtuelle, consultez [Reprotéger et effectuer une reprise automatique des machines vers un site local après le basculement vers Azure](vmware-azure-reprotect.md).

Pour résoudre le problème :

Créez manuellement la cible maître dans le vCenter qui gère votre machine source. La banque de données est disponible à l’issue des prochaines opérations de découverte et d’actualisation vCenter.

> [!Note]
> 
> Les opérations de structure fabric de découverte et d’actualisation peuvent prendre jusqu’à 30 minutes. 

## <a name="linux-master-target-registration-with-cs-fails-with-a-tls-error-35"></a>L’inscription du serveur cible maître Linux auprès de CS échoue avec une erreur TLS 35 

L’inscription de la cible maître Azure Site Recovery auprès du serveur de configuration échoue en raison de l’activation du proxy authentifié sur la cible maître. 
 
Cette erreur est indiquée par les chaînes suivantes dans le journal d’installation : 

```
RegisterHostStaticInfo encountered exception config/talwrapper.cpp(107)[post] CurlWrapper Post failed : server : 10.38.229.221, port : 443, phpUrl : request_handler.php, secure : true, ignoreCurlPartialError : false with error: [at curlwrapperlib/curlwrapper.cpp:processCurlResponse:231]   failed to post request: (35) - SSL connect error. 
```

Pour résoudre le problème :
 
1. Sur la machine virtuelle du serveur de configuration, ouvrez une invite de commandes pour vérifier les paramètres de proxy à l’aide des commandes suivantes :

    cat /etc/environment  echo $http_proxy  echo $https_proxy 

2. Si la sortie des commandes précédentes indique que des paramètres http_proxy ou https_proxy sont définis, utilisez une des méthodes suivantes pour débloquer les communications de la cible maître avec le serveur de configuration :
   
   - Téléchargez l’[outil PsExec](/sysinternals/downloads/psexec).
   - Utilisez l’outil pour accéder au contexte d’utilisateur système et déterminez si l’adresse du proxy est configurée. 
   - Si le proxy est configuré, ouvrez Internet Explorer dans un contexte d’utilisateur système à l’aide de l’outil PsExec.
  
     **psexec -s -i "%programfiles%\Internet Explorer\iexplore.exe"**

   - Pour vérifier que le serveur cible maître peut communiquer avec le serveur de configuration :
  
     - Modifiez les paramètres du proxy dans Internet Explorer pour contourner l’adresse IP du serveur cible maître par le biais du proxy.   
     ou
     - Désactivez le proxy sur le serveur cible maître. 


## <a name="next-steps"></a>Étapes suivantes
- Résoudre les problèmes de [connexion à la machine virtuelle Windows via RDP](../virtual-machines/troubleshooting/troubleshoot-rdp-connection.md)
- Résoudre les problèmes de [connexion à la machine virtuelle Linux via SSH](../virtual-machines/troubleshooting/detailed-troubleshoot-ssh-connection.md)

Si vous avez besoin d’aide supplémentaire, posez votre question sur la [Page de questions Microsoft Q&A pour Site Recovery](/answers/topics/azure-site-recovery.html) ou laissez un commentaire à la fin de ce document. Notre communauté est active et doit être en mesure de vous aider.