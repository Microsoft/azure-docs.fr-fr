---
title: Sauvegarder et restaurer Active Directory
description: Découvrez comment sauvegarder et restaurer des contrôleurs de domaine Active Directory.
ms.topic: conceptual
ms.date: 07/08/2020
ms.openlocfilehash: afbc538b84c20f9baaa664c7d47140365c747cb0
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754509"
---
# <a name="back-up-and-restore-active-directory-domain-controllers"></a>Sauvegarder et restaurer des contrôleurs de domaine Active Directory

La sauvegarde d’Active Directory et la garantie d’une restauration réussie en cas de corruption, de compromission ou de catastrophe constituent un élément essentiel de la maintenance d’Active Directory.

Cet article décrit les procédures appropriées pour la sauvegarde et la restauration de contrôleurs de domaine Active Directory à l’aide de Sauvegarde Azure, qu’il s’agisse de machines virtuelles Azure ou de serveurs locaux. Il présente un scénario dans lequel vous devez restaurer l’intégralité d’un contrôleur de domaine à l’état qu’il avait au moment de la sauvegarde. Pour savoir quel scénario de restauration vous convient, consultez [cet article](https://docs.microsoft.com/windows-server/identity/ad-ds/manage/ad-forest-recovery-determine-how-to-recover).  

>[!NOTE]
> Cet article ne traite pas de la restauration d’éléments à partir d’[Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis). Pour plus d’informations sur la restauration des utilisateurs Azure Active Directory, consultez [cet article](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-users-restore).

## <a name="best-practices"></a>Meilleures pratiques

- Assurez-vous qu’au moins un contrôleur de domaine est sauvegardé. Si vous sauvegardez plusieurs contrôleurs de domaine, assurez-vous que tous ceux qui contiennent les [rôles FSMO (Flexible Single Master Operation)](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/planning-operations-master-role-placement) sont sauvegardés.

- Sauvegardez régulièrement Active Directory. La sauvegarde ne doit jamais être supérieure à la durée de vie de l’objet tombstone (par défaut, 60 jours), car les objets antérieurs à cette durée seront marqués « tombstone » et ne seront plus considérés comme valides.

- Ayez un plan clair de récupération d’urgence qui contient des instructions sur la façon de restaurer vos contrôleurs de domaine. Pour préparer la restauration d’une forêt Active Directory, consultez le [guide de récupération d’une forêt Active Directory](https://docs.microsoft.com/windows-server/identity/ad-ds/manage/ad-forest-recovery-guide).

- Si vous avez besoin de restaurer un contrôleur de domaine et qu’il vous reste un contrôleur de domaine fonctionnel dans le domaine, vous pouvez créer un nouveau serveur au lieu d’effectuer une restauration à partir d’une sauvegarde. Ajoutez le **rôle de serveur Active Directory Domain Services** au nouveau serveur pour en faire un contrôleur de domaine dans le domaine existant. Les données d’Active Directory seront alors répliquées sur le nouveau serveur. Pour supprimer le contrôleur de domaine précédent d’Active Directory, suivez les étapes de [cet article](https://docs.microsoft.com/windows-server/identity/ad-ds/deploy/ad-ds-metadata-cleanup) afin d’effacer les métadonnées.

>[!NOTE]
>Sauvegarde Azure n’inclut pas la restauration au niveau de l’élément pour Active Directory. Si vous souhaitez restaurer des objets supprimés et que vous pouvez accéder à un contrôleur de domaine, utilisez la [corbeille d’Active Directory](https://docs.microsoft.com/windows-server/identity/ad-ds/get-started/adac/introduction-to-active-directory-administrative-center-enhancements--level-100-#ad_recycle_bin_mgmt). Si cette méthode n’est pas disponible, vous pouvez utiliser la sauvegarde de votre contrôleur de domaine pour restaurer les objets supprimés à l’aide de l’outil **ntdsutil.exe**, comme expliqué [ici](https://support.microsoft.com/help/840001/how-to-restore-deleted-user-accounts-and-their-group-memberships-in-ac).
>
>Pour plus d’informations sur l’exécution d’une restauration de SYSVOL faisant autorité, consultez [cet article](https://docs.microsoft.com/windows-server/identity/ad-ds/manage/ad-forest-recovery-authoritative-recovery-sysvol).

## <a name="backing-up-azure-vm-domain-controllers"></a>Sauvegarde des contrôleurs de domaine de machine virtuelle Azure

Si le contrôleur de domaine est une machine virtuelle Azure, vous pouvez sauvegarder le serveur à l’aide de [Sauvegarde de machine virtuelle Azure](backup-azure-vms-introduction.md).

Pour garantir la réussite des sauvegardes (et des restaurations ultérieures) de vos contrôleurs de domaine de machine virtuelle Azure, consultez les [considérations opérationnelles pour les contrôleurs de domaine virtualisés](https://docs.microsoft.com/windows-server/identity/ad-ds/get-started/virtual-dc/virtualized-domain-controllers-hyper-v#operational-considerations-for-virtualized-domain-controllers).

## <a name="backing-up-on-premises-domain-controllers"></a>Sauvegarde des contrôleurs de domaine locaux

Pour sauvegarder un contrôleur de domaine local, vous devez sauvegarder les données de l’état du système du serveur.

- Si vous utilisez MARS, suivez [ces instructions](backup-azure-system-state.md).
- Si vous utilisez MABS (serveur de sauvegarde Azure), suivez [ces instructions](backup-mabs-system-state-and-bmr.md).

>[!NOTE]
> La restauration des contrôleurs de domaine locaux (à partir de l’état du système ou des machines virtuelles) vers le cloud Azure n’est pas prise en charge. Si vous souhaitez pouvoir basculer d’un environnement Active Directory local vers Azure, envisagez d’utiliser [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-active-directory).

## <a name="restoring-active-directory"></a>Restauration d’Active Directory

Les données d’Active Directory peuvent être restaurées dans l’un des deux modes suivants : **faisant autorité** ou **ne faisant pas autorité**. Dans une restauration faisant autorité, les données d’Active Directory restaurées remplacent les données trouvées sur les autres contrôleurs de domaine de la forêt.

Toutefois, dans ce scénario, nous régénérons un contrôleur de domaine dans un domaine existant, ce qui signifie qu’une restauration **ne faisant pas autorité** doit être effectuée.

Pendant la restauration, le serveur est démarré en mode de restauration des services d’annuaire (DSRM). Vous devez fournir le mot de passe Administrateur pour le mode de restauration des services d’annuaire.

>[!NOTE]
>Si vous avez oublié le mot de passe DSRM, vous pouvez le réinitialiser à l’aide de [ces instructions](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc754363(v=ws.11)).

### <a name="restoring-azure-vm-domain-controllers"></a>Restauration des contrôleurs de domaine de machine virtuelle Azure

Pour restaurer un contrôleur de domaine de machine virtuelle Azure, consultez [Restaurer les machines virtuelles d’un contrôleur de domaine](backup-azure-arm-restore-vms.md#restore-domain-controller-vms).

Si vous restaurez une seule machine virtuelle de contrôleur de domaine ou plusieurs machines virtuelles de contrôleur de domaine dans un domaine unique, restaurez-les comme n’importe quelle autre machine virtuelle. Le mode DSRM (Directory Restore Mode) étant également disponible, tous les scénarios de récupération Active Directory sont viables.

Si vous devez restaurer une seule machine virtuelle de contrôleur de domaine dans une configuration à plusieurs domaines, restaurez les disques et créez une machine virtuelle [en utilisant PowerShell](backup-azure-vms-automation.md#restore-the-disks).

Si vous restaurez le dernier contrôleur de domaine restant dans le domaine ou si vous restaurez plusieurs domaines dans une forêt, nous vous recommandons d’utiliser une [récupération de forêt](https://docs.microsoft.com/windows-server/identity/ad-ds/manage/ad-forest-recovery-single-domain-in-multidomain-recovery).

>[!NOTE]
> À partir de Windows 2012, les contrôleurs de domaine virtualisés utilisent des [protections basées sur la virtualisation](https://docs.microsoft.com/windows-server/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100#virtualization-based-safeguards). Grâce à ces garanties, Active Directory comprend si la machine virtuelle restaurée est un contrôleur de domaine et effectue les étapes nécessaires pour restaurer les données d’Active Directory.

### <a name="restoring-on-premises-domain-controllers"></a>Restauration des contrôleurs de domaine locaux

Pour restaurer un contrôleur de domaine local, suivez les instructions de restauration de l’état du système sur Windows Server, à l’aide des conseils relatifs aux [considérations spéciales pour la récupération de l’état du système sur un contrôleur de domaine](backup-azure-restore-system-state.md#special-considerations-for-system-state-recovery-on-a-domain-controller).

## <a name="next-steps"></a>Étapes suivantes

- [Matrice de prise en charge pour Sauvegarde Azure](backup-support-matrix.md)
