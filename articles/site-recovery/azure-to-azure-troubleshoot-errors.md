---
title: Détecter un problème de réplication de machine virtuelle Azure dans Azure Site Recovery
description: Résoudre les erreurs rencontrées lors de la réplication de machines virtuelles Azure pour une reprise d’activité après sinistre.
author: rochakm
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 04/07/2020
ms.author: rochakm
ms.openlocfilehash: b71c6b834a6217007134b3be961a0ffa103e2706
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368041"
---
# <a name="troubleshoot-azure-to-azure-vm-replication-errors"></a>Résoudre les erreurs rencontrées lors de la réplication de machines virtuelles Azure vers Azure

Cet article explique comment résoudre les erreurs courantes rencontrées dans Azure Site Recovery pendant la réplication et la récupération de machines virtuelles Azure d'une région vers une autre. Pour plus d’informations sur les configurations prises en charge, consultez la [matrice de prise en charge pour la réplication des machines virtuelles Azure](azure-to-azure-support-matrix.md).

## <a name="azure-resource-quota-issues-error-code-150097"></a>Problèmes liés aux quotas de ressources Azure (code d’erreur 150097)

Vérifiez que votre abonnement est activé pour créer des machines virtuelles Azure dans la région cible que vous prévoyez d’utiliser comme région de reprise d’activité après sinistre. Vérifiez que votre abonnement dispose d’un quota suffisant pour créer des machines virtuelles des tailles nécessaires. Par défaut, Site Recovery choisit une taille de machine virtuelle cible identique à celle de la machine virtuelle source. Si la taille correspondante n’est pas disponible, Site Recovery choisit automatiquement la taille disponible la plus proche.

S’il n’existe pas de taille qui prend en charge la configuration de la machine virtuelle source, le message d’erreur suivant s’affiche :

```Output
Replication couldn't be enabled for the virtual machine <VmName>.
```

### <a name="possible-causes"></a>Causes possibles

- Votre ID d’abonnement n’est pas activé pour créer des machines virtuelles dans l’emplacement de la région cible.
- Votre ID d’abonnement n’est pas activé ou ne dispose pas d’un quota suffisant pour créer des tailles de machine virtuelle spécifiques dans l’emplacement de la région cible.
- Aucune taille de machine virtuelle cible appropriée correspondant au nombre de cartes réseau de la machine virtuelle source (2) n’a été trouvée pour l’ID d’abonnement dans l’emplacement de la région cible.

### <a name="fix-the-problem"></a>Résoudre le problème

Contactez le [support de facturation Azure](../azure-portal/supportability/resource-manager-core-quotas-request.md) pour activer votre abonnement et créer des machines virtuelles des tailles nécessaires dans l’emplacement cible. Retentez alors l’opération ayant échouée.

Si l’emplacement cible a une contrainte de capacité, désactivez la réplication sur cet emplacement. Activez ensuite la réplication sur un autre emplacement où votre abonnement dispose d’un quota suffisant pour créer des machines virtuelles des tailles nécessaires.

## <a name="trusted-root-certificates-error-code-151066"></a>Certificats racines approuvés (code d’erreur 151066)

Si tous les certificats racines approuvés les plus récents ne sont pas présents sur la machine virtuelle, votre travail Site Recovery « Activer la réplication » peut échouer. L’authentification et l’autorisation des appels du service Site Recovery à partir de la machine virtuelle échouent sans ces certificats.

Si le travail « Activer la réplication » échoue, le message suivant s’affiche :

```Output
Site Recovery configuration failed.
```

### <a name="possible-cause"></a>Cause probable

Les certificats racines approuvés nécessaires pour l’autorisation et l’authentification ne sont pas présents sur la machine virtuelle.

### <a name="fix-the-problem"></a>Résoudre le problème

#### <a name="windows"></a>Windows

Pour une machine virtuelle exécutant le système d’exploitation Windows, installez toutes les mises à jour de Windows les plus récentes afin que tous les certificats racines approuvés soient présents sur la machine virtuelle. Suivez le processus classique de gestion des mises à jour Windows ou de gestion des mises à jour de certificats en vigueur dans votre organisation pour obtenir les certificats racines les plus récents et la liste de révocation de certificats mise à jour sur les machines virtuelles.

- Si vous êtes dans un environnement déconnecté, suivez le processus de mise à jour de Windows standard en vigueur dans votre organisation pour obtenir les certificats.
- Si les certificats nécessaires ne sont pas présents sur la machine virtuelle, les appels au service Site Recovery échouent pour des raisons de sécurité.

Pour vérifier que le problème est résolu, accédez à `login.microsoftonline.com` à partir d’un navigateur sur votre machine virtuelle.

Pour plus d’informations, consultez [Configurer des racines de confiance et des certificats non autorisés](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn265983(v=ws.11)).

#### <a name="linux"></a>Linux

Suivez les instructions fournies par le distributeur de votre version du système d’exploitation Linux pour obtenir les certificats racines approuvés les plus récents et la dernière liste de révocation de certificats sur la machine virtuelle.

Étant donné que SuSE Linux utilise des liens symboliques pour tenir à jour une liste de certificats, vous devez effectuer les étapes suivantes :

1. Connectez-vous en tant qu’utilisateur **racine** . Le symbole dièse (`#`) est l’invite de commandes par défaut.

1. Exécutez cette commande pour changer de répertoire :

   `cd /etc/ssl/certs`

1. Vérifiez si le certificat d’autorité de certification racine Symantec est présent :

   `ls VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem`

   - Si le certificat d’autorité de certification racine Symantec est introuvable, exécutez la commande suivante pour télécharger le fichier. Vérifiez la présence d’éventuelles erreurs et suivez les actions recommandées concernant les défaillances réseau.

     `wget https://docs.broadcom.com/docs-and-downloads/content/dam/symantec/docs/other-resources/verisign-class-3-public-primary-certification-authority-g5-en.pem -O VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem`

1. Vérifiez si le certificat d’autorité de certification racine Baltimore est présent :

   `ls Baltimore_CyberTrust_Root.pem`

   - Si le certificat d’autorité de certification racine Baltimore est introuvable, exécutez cette commande pour télécharger le certificat :

     `wget https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem -O Baltimore_CyberTrust_Root.pem`

1. Vérifiez si le certificat DigiCert_Global_Root_CA est présent :

   `ls DigiCert_Global_Root_CA.pem`

    - Si le certificat DigiCert_Global_Root_CA est introuvable, exécutez les commandes suivantes pour télécharger le certificat :

      ```shell
      wget http://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt

      openssl x509 -in DigiCertGlobalRootCA.crt -inform der -outform pem -out DigiCert_Global_Root_CA.pem
      ```

1. Exécutez le script de rehachage (rehash) afin de mettre à jour les hachages d’objets pour les certificats qui viennent d’être téléchargés :

   `c_rehash`

1. Exécutez les commandes suivantes afin de vérifier si des hachages d’objets sous forme de liens symboliques ont été créés pour les certificats :

   ```shell
   ls -l | grep Baltimore
   ```

   ```Output
   lrwxrwxrwx 1 root root   29 Jan  8 09:48 3ad48a91.0 -> Baltimore_CyberTrust_Root.pem

   -rw-r--r-- 1 root root 1303 Jun  5  2014 Baltimore_CyberTrust_Root.pem
   ```

   ```shell
   ls -l | grep VeriSign_Class_3_Public_Primary_Certification_Authority_G5
   ```

   ```Output
   -rw-r--r-- 1 root root 1774 Jun  5  2014 VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem

   lrwxrwxrwx 1 root root   62 Jan  8 09:48 facacbc6.0 -> VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem
   ```

   ```shell
   ls -l | grep DigiCert_Global_Root
   ```

   ```Output
   lrwxrwxrwx 1 root root   27 Jan  8 09:48 399e7759.0 -> DigiCert_Global_Root_CA.pem

   -rw-r--r-- 1 root root 1380 Jun  5  2014 DigiCert_Global_Root_CA.pem
   ```

1. Créez une copie du fichier _VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem_ et donnez-lui le nom _b204d74a.0_  :

   `cp VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem b204d74a.0`

1. Créez une copie du fichier _Baltimore_CyberTrust_Root.pem_ et donnez-lui le nom _653b494a.0_  :

   `cp Baltimore_CyberTrust_Root.pem 653b494a.0`

1. Créez une copie du fichier _DigiCert_Global_Root_CA.pem_ et donnez-lui le nom _3513523f.0_  :

   `cp DigiCert_Global_Root_CA.pem 3513523f.0`

1. Vérifiez que les fichiers sont présents :

   ```shell
   ls -l 653b494a.0 b204d74a.0 3513523f.0
   ```

   ```Output
   -rw-r--r-- 1 root root 1774 Jan  8 09:52 3513523f.0

   -rw-r--r-- 1 root root 1303 Jan  8 09:52 653b494a.0

   -rw-r--r-- 1 root root 1774 Jan  8 09:52 b204d74a.0
   ```

## <a name="outbound-urls-or-ip-ranges-error-code-151037-or-151072"></a>URL ou plages d'adresses IP sortantes (code d'erreur 151037 ou 151072)

Pour que la réplication Site Recovery fonctionne, une connectivité sortante vers des URL spécifiques est nécessaire à partir de la machine virtuelle. Si votre machine virtuelle se trouve derrière un pare-feu ou utilise des règles de groupe de sécurité réseau pour contrôler la connectivité sortante, vous pouvez rencontrer l’un des problèmes ci-après. Même si nous continuons de prendre en charge l’accès sortant via des URL, l’utilisation d’une liste verte de plages d’adresses IP n’est plus prise en charge.

#### <a name="possible-causes"></a>Causes possibles

- Il est impossible d’établir une connexion aux points de terminaison Site Recovery en raison d’un échec de la résolution DNS.
- Ce problème survient le plus souvent pendant la reprotection, quand vous avez procédé au basculement de la machine virtuelle mais que le serveur DNS n’est pas accessible à partir de la région de reprise d’activité après sinistre.

#### <a name="fix-the-problem"></a>Résoudre le problème

Si vous utilisez DNS personnalisé, assurez-vous que le serveur DNS est accessible à partir de la région de récupération d’urgence.

Pour vérifier si la machine virtuelle utilise un paramètre DNS personnalisé :

1. Ouvrez **Machines virtuelles** et sélectionnez la machine virtuelle.
1. Accédez aux **Paramètres** de la machine virtuelle et sélectionnez **Réseau** .
1. Dans **Réseau/sous-réseau virtuel** , sélectionnez le lien pour ouvrir la page des ressources du réseau virtuel.
1. Accédez à **Paramètres** et sélectionnez **Serveurs DNS** .

Essayez d’accéder au serveur DNS à partir de la machine virtuelle. Si le serveur DNS n’est pas accessible, rendez-le accessible en faisant basculer le serveur DNS ou en créant la ligne de site entre le réseau de récupération d’urgence et le DNS.

:::image type="content" source="./media/azure-to-azure-troubleshoot-errors/custom_dns.png" alt-text="com-error.":::

### <a name="issue-2-site-recovery-configuration-failed-151196"></a>Problème2 : Échec de la configuration de Site Recovery (151196)

#### <a name="possible-cause"></a>Cause probable

Impossible d'établir une connexion avec les points de terminaison IP4 d'identité et d'authentification Microsoft 365.

#### <a name="fix-the-problem"></a>Résoudre le problème

Azure Site Recovery exigeait l'accès aux plages d'adresses IP Microsoft 365 pour l'authentification.
Si vous utilisez un proxy de règles/pare-feu de groupe de sécurité réseau Azure pour contrôler la connectivité réseau sortante sur la machine virtuelle, assurez-vous d’utiliser une règle de groupe de sécurité réseau basée sur les [balises de service Azure Active Directory (AAD)](../virtual-network/network-security-groups-overview.md#service-tags) pour autoriser l’accès à AAD. Nous ne prenons plus en charge les règles de groupe de sécurité réseau basées sur les adresses IP.

### <a name="issue-3-site-recovery-configuration-failed-151197"></a>Problème 3 : Échec de la configuration de Site Recovery (151197)

#### <a name="possible-cause"></a>Cause probable

Il est impossible d’établir une connexion aux points de terminaison de service Azure Site Recovery.

#### <a name="fix-the-problem"></a>Résoudre le problème

Si vous utilisez un proxy de règles/pare-feu de groupe de sécurité réseau Azure pour contrôler la connectivité réseau sortante sur la machine virtuelle, assurez-vous d’utiliser les balises de service. Nous ne prenons plus en charge l’utilisation d’une liste verte d’adresses IP via les groupes de sécurité réseau pour Azure Site Recovery.

### <a name="issue-4-replication-fails-when-network-traffic-uses-on-premises-proxy-server-151072"></a>Problème 4 : La réplication échoue lorsque le trafic réseau utilise un serveur proxy local (151072)

#### <a name="possible-cause"></a>Cause probable

Les paramètres de proxy personnalisés sont incorrects, et l’agent Mobility Service n’a pas détecté automatiquement les paramètres de proxy d’Internet Explorer (IE).

#### <a name="fix-the-problem"></a>Résoudre le problème

1. L’agent du service Mobilité détecte les paramètres de proxy à partir d’Internet Explorer sur Windows et à l’emplacement `/etc/environment` sur Linux.
1. Si vous préférez définir un proxy uniquement pour le service Mobility, vous pouvez fournir les détails du proxy dans le fichier _ProxyInfo.conf_ situé aux emplacements suivants :

   - **Linux** : `/usr/local/InMage/config/`
   - **Windows** : `C:\ProgramData\Microsoft Azure Site Recovery\Config`

1. Le fichier _ProxyInfo.conf_ doit inclure les paramètres de proxy au format _INI_ suivant.

   ```plaintext
   [proxy]
   Address=http://1.2.3.4
   Port=567
   ```

> [!NOTE]
> L’agent du service Mobility ne prend en charge que les **proxys non authentifiés** .

### <a name="more-information"></a>Informations complémentaires

Pour spécifier les [URL nécessaires](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) ou les [plages d’adresses IP nécessaires](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags), suivez les instructions mentionnées dans [Mise en réseau dans la réplication Azure vers Azure](azure-to-azure-about-networking.md).

## <a name="disk-not-found-in-vm-error-code-150039"></a>Disque introuvable sur la machine virtuelle (code d'erreur 150039)

Un nouveau disque attaché à la machine virtuelle doit être initialisé. Si le disque est introuvable, le message suivant s’affiche :

```Output
Azure data disk <DiskName> <DiskURI> with logical unit number <LUN> <LUNValue> was not mapped to a corresponding disk being reported from within the VM that has the same LUN value.
```

### <a name="possible-causes"></a>Causes possibles

- Un nouveau disque de données a été attaché à la machine virtuelle, mais n’a pas été initialisé.
- Le disque de données à l’intérieur de la machine virtuelle ne signale pas correctement la valeur de numéro d’unité logique (LUN) à laquelle le disque a été attaché à la machine virtuelle.

### <a name="fix-the-problem"></a>Résoudre le problème

Vérifiez que les disques de données sont initialisés, puis retentez l’opération.

- **Windows**  : [Attacher et initialiser un nouveau disque](../virtual-machines/windows/attach-managed-disk-portal.md).
- **Linux**  : [Initialiser un nouveau disque de données sous Linux](../virtual-machines/linux/add-disk.md).

Si le problème persiste, contactez le support technique.

## <a name="multiple-disks-available-for-protection-error-code-153039"></a>Plusieurs disques disponibles pour la protection (code d'erreur 153039)

### <a name="possible-causes"></a>Causes possibles

- Un ou plusieurs disques ont récemment été ajoutés à la machine virtuelle après la protection.
- Un ou plusieurs disques ont été initialisés après la protection de la machine virtuelle.

### <a name="fix-the-problem"></a>Résoudre le problème

Pour rétablir l’intégrité de l’état de réplication à la machine virtuelle, vous pouvez choisir de protéger les disques ou d’ignorer l’avertissement.

#### <a name="to-protect-the-disks"></a>Pour protéger les disques

1. Accédez à **Éléments répliqués** > _Nom de la machine virtuelle_ > **Disques** .
1. Sélectionnez le disque non protégé, puis sélectionnez **Activer la réplication**  :

   :::image type="content" source="./media/azure-to-azure-troubleshoot-errors/add-disk.png" alt-text="com-error.":::

#### <a name="to-dismiss-the-warning"></a>Pour ignorer l’avertissement

1. Accédez à **Éléments répliqués** > _Nom de la machine virtuelle_ .
1. Sélectionnez l’avertissement dans la section **Vue d’ensemble** , puis sélectionnez **OK** .

   :::image type="content" source="./media/azure-to-azure-troubleshoot-errors/dismiss-warning.png" alt-text="com-error.":::

## <a name="vm-removed-from-vault-completed-with-information-error-code-150225"></a>Suppression de la machine virtuelle du coffre terminée avec des informations (code d'erreur 150225)

Quand il protège la machine virtuelle, Site Recovery crée des liens sur la machine virtuelle source. Quand vous supprimez la protection ou que vous désactivez la réplication, Site Recovery supprime ces liens dans le cadre du travail de nettoyage. Si la machine virtuelle dispose d’un verrou de ressource, le travail de nettoyage est terminé avec les informations. Ces informations indiquent que la machine virtuelle a été supprimée du coffre Recovery Services, mais que certains des liens obsolètes n’ont pas pu être nettoyés sur la machine source.

Vous pouvez ignorer cet avertissement si vous n’envisagez pas de protéger à nouveau cette machine virtuelle. Cependant, si vous devez protéger cette machine virtuelle plus tard, suivez les étapes mentionnées dans cette section pour nettoyer les liens.

> [!WARNING]
> Si vous ne faites pas le nettoyage :
>
> - Quand vous activerez la réplication au moyen du coffre Recovery Services, la machine virtuelle ne sera pas listée.
> - Si vous essayez de protéger la machine virtuelle en utilisant **Machine virtuelle** > **Paramètres** > **Reprise d’activité** , l’opération échouera avec le message **Impossible d’activer la réplication en raison de liens de ressource périmés existants sur la machine virtuelle** .

### <a name="fix-the-problem"></a>Résoudre le problème

> [!NOTE]
> Site Recovery ne supprime pas la machine virtuelle source ou n’a aucun impact sur elle pendant que vous exécutez ces étapes.

1. Supprimez le verrou de la machine virtuelle ou du groupe de ressources. Par exemple, dans l’image suivante, le verrou de ressource sur la machine virtuelle nommée `MoveDemo` doit être supprimé :

   :::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/vm-locks.png" alt-text="com-error.":::

1. Téléchargez le script pour [retirer une configuration Site Recovery obsolète](https://github.com/AsrOneSdk/published-scripts/blob/master/Cleanup-Stale-ASR-Config-Azure-VM.ps1).
1. Exécutez le script, _Cleanup-stale-asr-config-Azure-VM.ps1_ . Indiquez **l’ID d’abonnement** , le **groupe de ressources de la machine virtuelle** et le **nom de la machine virtuelle** en tant que paramètres.
1. Si vous êtes invité à fournir des informations d’identification Azure, indiquez-les. Vérifiez ensuite que le script s’exécute sans échec.

## <a name="replication-not-enabled-on-vm-with-stale-resources-error-code-150226"></a>Réplication non activée sur la machine virtuelle avec ressources périmées (code d'erreur 150226)

### <a name="possible-causes"></a>Causes possibles

La configuration de la machine virtuelle est obsolète par rapport à la protection Site Recovery précédente.

Une configuration obsolète peut se produire sur une machine virtuelle Azure si vous avez activé la réplication pour la machine virtuelle Azure en utilisant Site Recovery, puis que :

- Vous avez désactivé la réplication, mais la machine virtuelle source comportait un verrou de ressource.
- Vous avez supprimé le coffre Site Recovery sans désactiver explicitement la réplication sur la machine virtuelle.
- Vous avez supprimé le groupe de ressources contenant le coffre Site Recovery sans désactiver explicitement la réplication sur la machine virtuelle.

### <a name="fix-the-problem"></a>Résoudre le problème

> [!NOTE]
> Site Recovery ne supprime pas la machine virtuelle source ou n’a aucun impact sur elle pendant que vous exécutez ces étapes.

1. Supprimez le verrou de la machine virtuelle ou du groupe de ressources. Par exemple, dans l’image suivante, le verrou de ressource sur la machine virtuelle nommée `MoveDemo` doit être supprimé :

   :::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/vm-locks.png" alt-text="com-error.":::

1. Téléchargez le script pour [retirer une configuration Site Recovery obsolète](https://github.com/AsrOneSdk/published-scripts/blob/master/Cleanup-Stale-ASR-Config-Azure-VM.ps1).
1. Exécutez le script, _Cleanup-stale-asr-config-Azure-VM.ps1_ . Indiquez **l’ID d’abonnement** , le **groupe de ressources de la machine virtuelle** et le **nom de la machine virtuelle** en tant que paramètres.
1. Si vous êtes invité à fournir des informations d’identification Azure, indiquez-les. Vérifiez ensuite que le script s’exécute sans échec.

## <a name="cant-select-vm-or-resource-group-in-enable-replication-job"></a>Impossible de sélectionner la machine virtuelle ou le groupe de ressources dans le travail d'activation de la réplication

### <a name="issue-1-the-resource-group-and-source-vm-are-in-different-locations"></a>Problème 1 : Le groupe de ressources et la machine virtuelle source se trouvent à des emplacements différents

Site Recovery exige actuellement que le groupe de ressources et les machines virtuelles de la région source se trouvent dans le même emplacement. Si ce n’est pas le cas, vous ne pourrez pas trouver la machine virtuelle ou le groupe de ressources quand vous essaierez d’appliquer la protection.

Comme solution de contournement, vous pouvez activer la réplication à partir de la machine virtuelle plutôt qu’à partir du coffre Recovery Services. Accédez à **Machine virtuelle source** > **Propriétés** > **Reprise d’activité** , puis activez la réplication.

### <a name="issue-2-the-resource-group-isnt-part-of-the-selected-subscription"></a>Problème2 : Le groupe de ressources ne fait pas partie de l’abonnement sélectionné

Il se peut que vous ne puissiez pas trouver le groupe de ressources au moment de la protection s’il ne fait pas partie de l’abonnement sélectionné. Vérifiez que le groupe de ressources appartient à l’abonnement que vous utilisez.

### <a name="issue-3-stale-configuration"></a>Problème 3 : Configuration obsolète

Il est possible que la machine virtuelle que vous voulez activer pour la réplication ne s’affiche pas si une configuration Site Recovery obsolète existe sur la machine virtuelle Azure. Cette condition peut se produire si vous avez activé la réplication pour la machine virtuelle Azure en utilisant Site Recovery, puis que :

- Vous avez supprimé le coffre Site Recovery sans désactiver explicitement la réplication sur la machine virtuelle.
- Vous avez supprimé le groupe de ressources contenant le coffre Site Recovery sans désactiver explicitement la réplication sur la machine virtuelle.
- Vous avez désactivé la réplication, mais la machine virtuelle source comportait un verrou de ressource.

### <a name="fix-the-problem"></a>Résoudre le problème

> [!NOTE]
> Veillez à mettre à jour le module `AzureRM.Resources` avant d’utiliser le script mentionné dans cette section. Site Recovery ne supprime pas la machine virtuelle source ou n’a aucun impact sur elle pendant que vous exécutez ces étapes.

1. Supprimez le verrou, le cas échéant, de la machine virtuelle ou du groupe de ressources. Par exemple, dans l’image suivante, le verrou de ressource sur la machine virtuelle nommée `MoveDemo` doit être supprimé :

   :::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/vm-locks.png" alt-text="com-error.":::

1. Téléchargez le script pour [retirer une configuration Site Recovery obsolète](https://github.com/AsrOneSdk/published-scripts/blob/master/Cleanup-Stale-ASR-Config-Azure-VM.ps1).
1. Exécutez le script, _Cleanup-stale-asr-config-Azure-VM.ps1_ . Indiquez **l’ID d’abonnement** , le **groupe de ressources de la machine virtuelle** et le **nom de la machine virtuelle** en tant que paramètres.
1. Si vous êtes invité à fournir des informations d’identification Azure, indiquez-les. Vérifiez ensuite que le script s’exécute sans échec.

## <a name="unable-to-select-a-vm-for-protection"></a>Impossible de sélectionner une machine virtuelle pour la protection

### <a name="possible-cause"></a>Cause probable

La machine virtuelle a une extension installée dans un état d’échec ou sans réponse

### <a name="fix-the-problem"></a>Résoudre le problème

Accédez à **Machines virtuelles** > **Paramètres** > **Extensions** , puis recherchez les éventuelles extensions en état d’échec. Désinstallez toute extension en état d’échec, puis réessayez de protéger la machine virtuelle.

## <a name="vm-provisioning-state-isnt-valid-error-code-150019"></a>L'état d'approvisionnement de la machine virtuelle n'est pas valide (code d'erreur 150019)

Pour activer la réplication sur la machine virtuelle, son état de provisionnement doit être **Réussi** . Pour vérifier l’état de provisionnement, effectuez les étapes suivantes :

1. Dans le portail Azure, sélectionnez l’ **Explorateur de ressources** sous **Tous les services** .
1. Développez la liste **Abonnements** et sélectionnez votre abonnement.
1. Développez la liste **Groupes de ressources** et sélectionnez le groupe de ressources de la machine virtuelle.
1. Développez la liste **Ressources** , puis sélectionnez votre machine virtuelle.
1. Vérifiez le champ **provisioningState** dans la vue Instance sur le côté droit.

### <a name="fix-the-problem"></a>Résoudre le problème

- Si **provisioningState** a la valeur **Échec** , contactez le support avec les détails pour résoudre les problèmes.
- Si **provisioningState** a la valeur **Mise à jour** , une autre extension est peut-être en cours de déploiement. Vérifiez si des opérations sont en cours sur la machine virtuelle, attendez qu’elles se terminent, puis réessayez la tâche Site Recovery pour activer la réplication ayant échoué.

## <a name="unable-to-select-target-vm"></a>Impossible de sélectionner la machine virtuelle cible

### <a name="issue-1-vm-is-attached-to-a-network-thats-already-mapped-to-a-target-network"></a>Problème 1 : La machine virtuelle est jointe à un réseau qui est déjà mappé à un réseau cible

Lors de la configuration de la récupération d'urgence, si la machine virtuelle source fait partie d'un réseau virtuel et qu'une autre machine virtuelle du même réseau virtuel est déjà mappée à un réseau dans le groupe de ressources cible, la zone de liste déroulante de sélection du réseau est indisponible (grisée) par défaut.

:::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/unabletoselectnw.png" alt-text="com-error.":::

### <a name="issue-2-you-previously-protected-the-vm-and-then-you-disabled-the-replication"></a>Problème2 : Vous avez précédemment protégé la machine virtuelle, puis désactivé la réplication

La désactivation de la réplication d’une machine virtuelle ne supprime pas le mappage réseau. Le mappage doit être supprimé du coffre Recovery Services dans lequel la machine virtuelle a été protégée. Sélectionnez le **coffre Recovery Services** et accédez à **Gérer** > **Infrastructure Site Recovery** > **Pour les machines virtuelles Azure** > **Mappage réseau** .

:::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/delete_nw_mapping.png" alt-text="com-error.":::

Le réseau cible qui a été défini lors de la configuration de la récupération d'urgence peut être modifié après la configuration initiale, une fois la machine virtuelle protégée. Pour **modifier le mappage réseau** , sélectionnez le nom du réseau :

:::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/modify_nw_mapping.png" alt-text="com-error.":::


## <a name="com-or-vss-error-code-151025"></a>COM+ ou VSS (code d'erreur 151025)

Lorsque l'erreur COM+ ou VSS (Volume Shadow Copy Service) se produit, le message suivant s'affiche :

```Output
Site Recovery extension failed to install.
```

### <a name="possible-causes"></a>Causes possibles

- Le service Application système COM+ est désactivé.
- Le service Cliché instantané de volume est désactivé.

### <a name="fix-the-problem"></a>Résoudre le problème

Définissez les services Application système COM+ et Cliché instantané de volume en mode de démarrage manuel ou automatique.

1. Ouvrez la console Services dans Windows.
1. Vérifiez que les services Application système COM+ et Cliché instantané de volume ne sont pas définis sur **Désactivé** pour leur **Type de démarrage** .

   :::image type="content" source="./media/azure-to-azure-troubleshoot-errors/com-error.png" alt-text="com-error.":::

## <a name="unsupported-managed-disk-size-error-code-150172"></a>Taille de disque managé non prise en charge (code d’erreur 150172)

Quand cette erreur se produit, le message suivant s’affiche :

```Output
Protection couldn't be enabled for the virtual machine as it has <DiskName> with size <DiskSize> that is lesser than the minimum supported size 1024 MB.
```

### <a name="possible-cause"></a>Cause probable

La taille du disque est inférieure à la taille de 1 024 Mo prise en charge.

### <a name="fix-the-problem"></a>Résoudre le problème

Vérifiez que la taille de disque est dans la plage des tailles prises en charge, puis retentez l’opération.

## <a name="protection-not-enabled-when-grub-uses-device-name-error-code-151126"></a>Protection non activée lorsque GRUB utilise le nom de l'appareil (code d'erreur 151126)

### <a name="possible-causes"></a>Causes possibles

Les fichiers de configuration GRUB Linux ( _/boot/grub/menu.lst_ , _/boot/grub/grub.cfg_ , _/boot/grub2/grub.cfg_ ou _/etc/default/grub_ ) peuvent spécifier le nom réel des appareils au lieu des valeurs UUID pour les paramètres `root` et `resume`. Site Recovery exige les UUID car les noms d’appareils peuvent changer. Au redémarrage, une machine virtuelle peut ne pas trouver le même nom lors du basculement, ce qui entraîne des problèmes.

Les exemples suivants sont des lignes de fichiers GRUB où les noms d’appareils apparaissent à la place des UUID nécessaires :

- Fichier _/boot/grub2/grub.cfg_ :

  `linux /boot/vmlinuz-3.12.49-11-default root=/dev/sda2  ${extra_cmdline} resume=/dev/sda1 splash=silent quiet showopts`

- Fichier : _/boot/grub/menu.lst_

  `kernel /boot/vmlinuz-3.0.101-63-default root=/dev/sda2 resume=/dev/sda1 splash=silent crashkernel=256M-:128M showopts vga=0x314`

### <a name="fix-the-problem"></a>Résoudre le problème

Remplacez chaque nom de l’appareil par l’UUID correspondant :

1. Recherchez l'UUID de l'appareil en exécutant la commande `blkid <device name>`. Par exemple :

   ```shell
   blkid /dev/sda1
   /dev/sda1: UUID="6f614b44-433b-431b-9ca1-4dd2f6f74f6b" TYPE="swap"
   blkid /dev/sda2
   /dev/sda2: UUID="62927e85-f7ba-40bc-9993-cc1feeb191e4" TYPE="ext3"
   ```

1. Remplacez le nom de l’appareil par son UUID dans les formats `root=UUID=<UUID>` et `resume=UUID=<UUID>` . Par exemple, après le remplacement, la ligne de _/boot/grub/menu.lst_ ressemble à celle-ci :

   `kernel /boot/vmlinuz-3.0.101-63-default root=UUID=62927e85-f7ba-40bc-9993-cc1feeb191e4 resume=UUID=6f614b44-433b-431b-9ca1-4dd2f6f74f6b splash=silent crashkernel=256M-:128M showopts vga=0x314`

1. Réessayez la protection.

## <a name="protection-failed-because-grub-device-doesnt-exist-error-code-151124"></a>La protection a échoué parce que l'appareil GRUB n'existe pas (code d'erreur 151124)

### <a name="possible-cause"></a>Cause probable

Les fichiers de configuration GRUB ( _/boot/grub/menu.lst_ , _/boot/grub/grub.cfg_ , _/boot/grub2/grub.cfg_ ou _/etc/default/grub_ ) peuvent contenir les paramètres `rd.lvm.lv` ou `rd_LVM_LV`. Ces paramètres identifient les appareils LVM (Logical Volume Manager) qui doivent être découverts au démarrage. Si ces appareils LVM n’existent pas, le système protégé proprement dit ne démarrera pas et sera bloqué lors du processus de démarrage. Le même problème sera également observé avec la machine virtuelle de basculement. Voici quelques exemples :

- Fichier : _/boot/grub2/grub.cfg_ sur RHEL7 :

  `linux16 /vmlinuz-3.10.0-957.el7.x86_64 root=/dev/mapper/rhel_mup--rhel7u6-root ro crashkernel=128M\@64M rd.lvm.lv=rootvg/root rd.lvm.lv=rootvg/swap rhgb quiet LANG=en_US.UTF-8`

- Fichier : _/etc/default/grub_ sur RHEL7 :

  `GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=rootvg/root rd.lvm.lv=rootvg/swap rhgb quiet`

- Fichier : _/boot/grub/menu.lst_ sur RHEL6 :

  `kernel /vmlinuz-2.6.32-754.el6.x86_64 ro root=UUID=36dd8b45-e90d-40d6-81ac-ad0d0725d69e rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=rootvg/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_LVM_LV=rootvg/lv_swap rd_NO_DM rhgb quiet`

Dans chaque exemple, GRUB doit détecter deux appareils LVM avec les noms `root` et `swap` à partir du groupe de volumes `rootvg`.

### <a name="fix-the-problem"></a>Résoudre le problème

Si l’appareil LVM n’existe pas, créez-le ou supprimez les paramètres correspondants des fichiers de configuration GRUB. Ensuite, réessayez d’activer la protection.

## <a name="mobility-service-update-finished-with-warnings-error-code-151083"></a>La mise à jour du service Mobility s'est terminée avec des avertissements (code d'erreur 151083)

Mobility Service de Site Recovery comprend de nombreux composants, notamment le pilote de filtre. Le pilote de filtre n’est chargé dans la mémoire système que pendant le redémarrage du système. Chaque fois qu’une mise à jour de Mobility Service comprend des changements de pilote de filtre, l’ordinateur est mis à jour, mais un avertissement s’affiche pour vous avertir que certains correctifs nécessitent un redémarrage. Cet avertissement s’affiche car les correctifs du pilote de filtre ne peuvent prendre effet que lorsque le nouveau pilote de filtre est chargé, ce qui se produit uniquement lors d’un redémarrage.

> [!NOTE]
> Il s’agit simplement d’un avertissement. La réplication existante continue à fonctionner même après la nouvelle mise à jour de l’agent. Vous pouvez choisir d’effectuer un redémarrage chaque fois que vous souhaitez profiter du nouveau pilote de filtre, mais en l’absence de redémarrage, l’ancien pilote de filtre continue de fonctionner.
>
> Hormis le pilote de filtre, les autres améliorations et correctifs de Mobility Service prennent effet sans nécessiter un redémarrage.

## <a name="protection-not-enabled-if-replica-managed-disk-exists"></a>Protection non activée si le disque managé de réplica existe

Cette erreur se produit lorsque le disque managé de réplica existe déjà, sans les balises attendues, dans le groupe de ressources cible.

### <a name="possible-cause"></a>Cause probable

Ce problème peut se produire si la machine virtuelle a été précédemment protégée et qu’au moment de la désactivation de la réplication, le disque de réplica n’avait pas été retiré.

### <a name="fix-the-problem"></a>Résoudre le problème

Supprimez le disque de réplica identifié dans le message d’erreur, puis retentez la tâche de protection qui a échoué.

## <a name="enable-protection-failed-as-the-installer-is-unable-to-find-the-root-disk-error-code-151137"></a>L’activation de la protection a échoué, car le programme d’installation ne parvient pas à trouver le disque racine (code d’erreur 151137)

Cette erreur se produit pour les machines Linux dont le disque du système d’exploitation est chiffré à l’aide d’Azure Disk Encryption (ADE). Il s’agit d’un problème valide uniquement dans la version 9.35 de l’agent.

### <a name="possible-causes"></a>Causes possibles

Le programme d’installation ne parvient pas à trouver le disque racine qui héberge le système de fichiers racine.

### <a name="fix-the-problem"></a>Résoudre le problème

Suivez les étapes ci-dessous pour corriger le problème :

1. Recherchez les bits de l’agent sous le répertoire _/var/lib/waagent_ sur les machines RHEL et CentOS à l’aide de la commande ci-dessous : <br>

    `# find /var/lib/ -name Micro\*.gz`

   Sortie attendue :

    `/var/lib/waagent/Microsoft.Azure.RecoveryServices.SiteRecovery.LinuxRHEL7-1.0.0.9139/UnifiedAgent/Microsoft-ASR_UA_9.35.0.0_RHEL7-64_GA_30Jun2020_release.tar.gz`

2. Créez un répertoire, et remplacez le répertoire actuel par ce nouveau répertoire.
3. Extrayez le fichier de l’agent trouvé à la première étape à l’aide de la commande ci-dessous :

    `tar -xf <Tar Ball File>`

4. Ouvrez le fichier _prereq_check_installer.json_ et supprimez les lignes suivantes. Enregistrez ensuite le fichier.

    ```
       {
          "CheckName": "SystemDiskAvailable",
          "CheckType": "MobilityService"
       },
    ```
5. Appelez le programme d’installation à l’aide de la commande suivante : <br>

    `./install -d /usr/local/ASR -r MS -q -v Azure`
6. Si le programme d’installation s’exécute correctement, relancez la tâche d’activation de la réplication.

## <a name="next-steps"></a>Étapes suivantes

[Répliquer des machines virtuelles Azure dans une autre région Azure](azure-to-azure-how-to-enable-replication.md)