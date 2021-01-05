---
title: Chiffrement des données de sauvegarde à l’aide de clés gérées par le client
description: Découvrez comment Sauvegarde Azure vous permet de chiffrer vos données de sauvegarde à l’aide de clés gérées par le client (CMK).
ms.topic: conceptual
ms.date: 07/08/2020
ms.openlocfilehash: 6e3eea4b5f44203b68c1263c0fb3ae843cabbe72
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895985"
---
# <a name="encryption-of-backup-data-using-customer-managed-keys"></a>Chiffrement des données de sauvegarde à l’aide de clés gérées par le client

Sauvegarde Azure vous permet de chiffrer vos données de sauvegarde à l’aide de clés gérées par le client (CMK) au lieu d’utiliser des clés gérées par la plateforme, qui sont activées par défaut. Les clés que vous utilisez pour le chiffrement des données de sauvegarde doivent être stockées dans [Azure Key Vault](../key-vault/index.yml).

La clé de chiffrement utilisée pour le chiffrement des sauvegardes peut être différente de celle utilisée pour la source. Les données sont protégées à l’aide d’une clé de chiffrement des données (DEK) basée sur l’algorithme AES 256, qui est à son tour protégée par vos clés (KEK). Cela vous donne un contrôle total sur les données et les clés. Pour permettre le chiffrement, il est nécessaire que le coffre Recovery Services soit autorisé à accéder à la clé de chiffrement dans le coffre Azure Key Vault. Vous pouvez éventuellement changer de clé, si nécessaire.

Cet article aborde les sujets suivants :

- Création d’un coffre Recovery Services
- Configuration de votre coffre Recovery Services pour chiffrer les données de sauvegarde à l’aide de clés gérées par le client
- Sauvegarde dans des coffres chiffrés à l’aide de clés gérées par le client
- Restauration de données à partir de sauvegardes

## <a name="before-you-start"></a>Avant de commencer

- Cette fonctionnalité vous permet de chiffrer les **nouveaux coffres Recovery Services uniquement** . Les coffres contenant des éléments existants inscrits ou en attente de l’être ne sont pas pris en charge.

- Une fois activé pour un coffre Recovery Services, le chiffrement à base de clés gérées par le client ne peut pas revenir à une utilisation de clés gérées par la plateforme (option par défaut). Vous pouvez changer les clés de chiffrement en fonction de vos besoins.

- Pour l’heure, cette fonctionnalité **ne prend pas en charge la sauvegarde à l’aide de l’agent MARS** , et vous ne pourrez peut-être pas utiliser de coffre chiffré par une clé CMK à cette fin. L’agent MARS utilise un chiffrement basé sur une phrase secrète utilisateur. Cette fonctionnalité ne prend pas non plus en charge la sauvegarde de machines virtuelles classiques.

- Elle n’est pas liée à [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md), qui utilise le chiffrement basé sur l’invité des disques d’une machine virtuelle à l’aide de BitLocker (pour Windows) et de DM-Crypt (pour Linux)

- Le coffre Recovery Services ne peut être chiffré qu’avec des clés stockées dans un coffre de clés Azure, situé dans la **même région** . De même, les clés doivent être de type **RSA 2048** uniquement et doivent être à l’état **activé** .

- Le déplacement d’un coffre Recovery Services chiffré par clé CMK d’un groupe de ressources ou d’un abonnement vers un autre n’est pas actuellement pris en charge.

- Pour l’heure, cette fonctionnalité ne peut être configurée qu’à partir du portail Azure.

Si vous n'avez pas encore créé et configuré votre coffre Recovery Services, [cliquez ici pour accéder aux instructions correspondantes](backup-create-rs-vault.md).

## <a name="configuring-a-vault-to-encrypt-using-customer-managed-keys"></a>Configuration d’un coffre à chiffrer à l’aide de clés gérées par le client

Cette section implique les étapes suivantes :

1. Activer une identité managée pour votre coffre Recovery Services

1. Attribuer des autorisations au coffre pour lui permettre d’accéder à la clé de chiffrement dans le coffre de clés Azure

1. Activer la suppression réversible et la protection contre le vidage sur le coffre de clés Azure

1. Affecter la clé de chiffrement au coffre Recovery Services

Il est nécessaire de suivre toutes ces étapes dans l’ordre indiqué ci-dessus pour obtenir les résultats souhaités. Chaque étape est décrite en détail ci-dessous.

### <a name="enable-managed-identity-for-your-recovery-services-vault"></a>Activer une identité managée pour votre coffre Recovery Services

Sauvegarde Azure utilise l’identité managée affectée par le système pour authentifier le coffre Recovery Services et lui permettre d’accéder aux clés de chiffrement stockées dans le coffre de clés Azure. Pour activer l’identité managée pour votre coffre Recovery Services, suivez les étapes indiquées ci-dessous.

>[!NOTE]
>Une fois activée, l'identité managée ne doit **pas** être désactivée (même temporairement). La désactivation de l’identité managée peut entraîner un comportement incohérent.

1. Accédez à votre coffre Recovery Services -> **Identité** .

    ![Paramètres d’identité](./media/encryption-at-rest-with-cmk/managed-identity.png)

1. Cliquez sur le bouton bascule sous **État** pour le définir sur la valeur **Activé** , puis sélectionnez **Enregistrer** .

1. Un ID d’objet est généré, qui correspond à l’identité managée affectée par le système du coffre.

### <a name="assign-permissions-to-the-recovery-services-vault-to-access-the-encryption-key-in-the-azure-key-vault"></a>Attribuer des autorisations au coffre Recovery Services pour lui permettre d’accéder à la clé de chiffrement dans le coffre de clés Azure

Vous devez maintenant autoriser le coffre Recovery Services à accéder au coffre de clés Azure qui contient la clé de chiffrement. Pour cela, vous devez autoriser l’identité managée du coffre Recovery Services à accéder au coffre de clés.

1. Accédez à votre coffre de clés Azure -> **Stratégies d’accès** . Continuez pour **+Ajouter une stratégie d’accès** .

    ![Ajouter des stratégies d’accès](./media/encryption-at-rest-with-cmk/access-policies.png)

1. Sous **Autorisations de clé** , sélectionnez les opérations **Obtenir** , **Lister** , **Ne pas inclure la clé** et **Inclure la clé** . Cela indique les actions qui seront autorisées sur la clé.

    ![Attribuer des autorisations de clé](./media/encryption-at-rest-with-cmk/key-permissions.png)

1. Accédez à **Sélectionner le principal** et recherchez votre coffre dans la zone de recherche à partir de son nom ou de son identité managée. Une fois le coffre affiché, sélectionnez-le, puis choisissez **Sélectionner** au bas du volet.

    ![Sélectionner le principal](./media/encryption-at-rest-with-cmk/select-principal.png)

1. Après quoi, sélectionnez **Ajouter** pour ajouter la nouvelle stratégie d’accès.

1. Sélectionnez **Enregistrer** pour enregistrer les modifications apportées à la stratégie d’accès du coffre de clés Azure.

### <a name="enable-soft-delete-and-purge-protection-on-the-azure-key-vault"></a>Activer la suppression réversible et la protection contre le vidage sur le coffre de clés Azure

Vous devez **activer la suppression réversible et la protection contre le vidage**  sur votre coffre de clés Azure qui stocke votre clé de chiffrement. Cette opération s’effectue dans l’interface utilisateur d’Azure Key Vault comme indiqué ci-dessous. (Vous pouvez aussi définir ces propriétés au moment de créer le coffre de clés). Pour en savoir plus sur ces propriétés Key Vault, consultez [cette page](../key-vault/general/soft-delete-overview.md).

![Activer la suppression réversible et la protection contre le vidage](./media/encryption-at-rest-with-cmk/soft-delete-purge-protection.png)

Vous pouvez aussi activer la suppression réversible et la protection contre le vidage via PowerShell en suivant les étapes ci-dessous :

1. Connectez-vous à votre compte Azure.

    ```azurepowershell
    Login-AzAccount
    ```

1. Sélectionnez l’abonnement qui contient votre coffre.

    ```azurepowershell
    Set-AzContext -SubscriptionId SubscriptionId
    ```

1. Activer la suppression réversible

    ```azurepowershell
    ($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "AzureKeyVaultName").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"
    ```

    ```azurepowershell
    Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
    ```

1. Activer la protection contre le vidage

    ```azurepowershell
    ($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "AzureKeyVaultName").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true"
    ```

    ```azurepowershell
    Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
    ```

### <a name="assign-encryption-key-to-the-rs-vault"></a>Attribuer une clé de chiffrement au coffre RS

>[!NOTE]
> Avant de continuer, vérifiez les points suivants :
>
> - Toutes les étapes mentionnées ci-dessus ont été effectuées avec succès :
>   - L’identité managée du coffre Recovery Services a été activée et les autorisations nécessaires lui ont été attribuées
>   - La suppression réversible et la protection contre le vidage ont été activées sur le coffre de clés Azure
> - **Aucun** élément protégé ou inscrit ne se trouve dans le coffre Recovery Services pour lequel vous souhaitez activer le chiffrement CMK

Une fois les éléments ci-dessus vérifiés, poursuivez en sélectionnant la clé de chiffrement de votre coffre.

Pour affecter la clé :

1. Accédez à votre coffre Recovery Services -> **Propriétés** .

    ![Paramètres de chiffrement](./media/encryption-at-rest-with-cmk/encryption-settings.png)

1. Sélectionnez **Mettre à jour** en dessous de **Paramètres de chiffrement** .

1. Dans le volet Paramètres de chiffrement, sélectionnez **Utiliser votre propre clé** , puis continuez à spécifier la clé de l’une des façons suivantes. **Vérifiez que la clé que vous souhaitez utiliser est une clé RSA 2048, qui est à l’état activé.**

    1. Entrez l’ **URI de clé** avec lequel vous souhaitez chiffrer les données dans ce coffre Recovery Services. Vous devez aussi spécifier l’abonnement dans lequel le coffre de clés Azure (qui contient cette clé) est présent. Cet URI de clé peut être obtenu à partir de la clé correspondante dans votre coffre de clés Azure. Vérifiez que l’URI de la clé est correctement copié. Il est recommandé d’utiliser le bouton **Copier dans le Presse-papiers** fourni avec l’identificateur de clé.

        ![Entrer l’URI de la clé](./media/encryption-at-rest-with-cmk/key-uri.png)

    1. Recherchez la clé dans le coffre de clés et sélectionnez-la dans le volet de sélection de clé.

        ![Sélectionner une clé dans le coffre de clés](./media/encryption-at-rest-with-cmk/key-vault.png)

1. Sélectionnez **Enregistrer** .

1. **Suivi de la progression de la mise à jour de la clé de chiffrement :** vous pouvez suivre la progression de l’affectation de la clé à l’aide du **Journal d’activité**  dans le coffre Recovery Services. L’état doit passer rapidement à **Réussite** . Votre coffre chiffre alors toutes les données avec la clé spécifiée en tant que KEK.

    ![Suivre la progression avec le journal d’activité](./media/encryption-at-rest-with-cmk/activity-log.png)

    ![État Réussite](./media/encryption-at-rest-with-cmk/status-succeeded.png)

>[!NOTE]
> Ce processus est le même quand vous souhaitez mettre à jour/modifier la clé de chiffrement. Si vous souhaitez mettre à jour et utiliser une clé d’un autre coffre de clés (différent de celui actuellement utilisé), vérifiez que :
>
> - Le coffre de clés se trouve dans la même région que le coffre Recovery Services
>
> - La suppression réversible et la protection contre le vidage ont été activées pour le coffre de clés
>
> - Le coffre Recovery Services dispose des autorisations nécessaires pour accéder au coffre de clés.

## <a name="backing-up-to-a-vault-encrypted-with-customer-managed-keys"></a>Sauvegarde dans un coffre chiffré à l’aide de clés gérées par le client

Avant de poursuivre la configuration de la protection, nous vous recommandons vivement de vous assurer que les éléments de la liste ci-dessous ont bien été vérifiés. Ceci est important, car une fois qu’un élément a été configuré pour être sauvegardé (ou tenté de l’être) dans un coffre qui n’a pas été chiffré avec des clés gérées par le client, le chiffrement à base de clés de ce type peut pas être activé pour le coffre et des clés gérées par la plateforme continuent d’être utilisées.

>[!IMPORTANT]
> Avant de poursuivre la configuration de la protection, vous devez avoir **correctement** effectué les étapes suivantes :
>
>1. Création de votre coffre de sauvegarde.
>1. Activation de l’identité managée affectée par le système du coffre de sauvegarde.
>1. Attribution d’autorisations à votre coffre de sauvegarde pour lui permettre d’accéder aux clés de chiffrement à partir de votre coffre de clés.
>1. Activation de la suppression définitive et de la protection contre le vidage pour votre coffre de clés.
>1. Affectation d’une clé de chiffrement valide à votre coffre de sauvegarde.
>
>Si toutes les étapes ci-dessus ont été confirmées, poursuivez alors la configuration de la sauvegarde.

Les processus de configuration et d’exécution de sauvegardes dans un coffre Recovery Services chiffré avec des clés gérées par le client sont les mêmes que pour un coffre qui utilise des clés gérées par la plateforme, sans **aucun changement au niveau de l’expérience** . Cela est vrai pour la [sauvegarde de machines virtuelles Azure](./quick-backup-vm-portal.md), mais aussi pour la sauvegarde des charges de travail s’exécutant dans une machine virtuelle (par exemple, les bases de données [SAP HANA](./tutorial-backup-sap-hana-db.md), [SQL Server](./tutorial-sql-backup.md)).

## <a name="restoring-data-from-backup"></a>Restauration de données à partir d’une sauvegarde

### <a name="vm-backup"></a>Sauvegarde de machines virtuelles

Les données stockées dans le coffre Recovery Services peuvent être restaurées selon les étapes décrites [ici](./backup-azure-arm-restore-vms.md). Si vous restaurez à partir d’un coffre Recovery Services chiffré à l’aide de clés gérées par le client, vous pouvez choisir de chiffrer les données restaurées avec un jeu de chiffrement de disque (DES).

#### <a name="restoring-vm--disk"></a>Restauration de machine virtuelle/disque

1. Si vous récupérez un disque ou une machine virtuelle à partir d’un point de récupération de « capture instantané », les données restaurées sont chiffrées avec le jeu de chiffrement de disque (DES) utilisé pour le chiffrement des disques de la machine virtuelle source.

1. Si vous restaurez un disque ou une machine virtuelle à partir d’un point de récupération avec le type de récupération « Coffre », vous pouvez choisir de chiffrer les données restaurées à l’aide d’un jeu de chiffrement de disque, spécifié au moment de la restauration. Vous pouvez aussi choisir de poursuivre la restauration des données sans spécifier de jeu de chiffrement de disque. Auquel cas, elles sont chiffrées à l’aide de clés gérées par Microsoft.

Vous pouvez chiffrer la machine virtuelle ou le disque restauré une fois la restauration terminée, quelle que soit la sélection effectuée pendant le lancement de la restauration.

![Points de restauration](./media/encryption-at-rest-with-cmk/restore-points.png)

#### <a name="select-a-disk-encryption-set-while-restoring-from-vault-recovery-point"></a>Sélectionner un jeu de chiffrement de disque pendant la restauration à partir du point de récupération du coffre

Le jeu de chiffrement de disque est spécifié en dessous de Paramètres de chiffrement dans le volet de restauration, comme le montre l’image ci-dessous :

1. Dans **Chiffrer le(s) disque(s) à l’aide de votre clé** , sélectionnez **Oui** .

1. Dans la liste déroulante, sélectionnez le jeu de chiffrement que vous voulez utiliser pour le ou les disques restaurés. **Vérifiez que vous avez accès au jeu de chiffrement de disque.**

>[!NOTE]
>Vous n'avez pas la possibilité de choisir un jeu de chiffrement de disque pendant la restauration si vous restaurez une machine virtuelle qui utilise Azure Disk Encryption.

![Chiffrer le disque à l’aide de votre clé](./media/encryption-at-rest-with-cmk/encrypt-disk-using-your-key.png)

#### <a name="restoring-files"></a>Restauration de fichiers

Quand vous effectuez une restauration de fichiers, les données restaurées sont chiffrées à l’aide de la clé utilisée pour chiffrer l’emplacement cible.

### <a name="restoring-sap-hanasql-databases-in-azure-vms"></a>Restauration de bases de données SAP HANA/SQL dans des machines virtuelles Azure

Quand vous restaurez à partir d’une base de données SAP HANA/SQL sauvegardée qui s’exécute dans une machine virtuelle Azure, les données restaurées sont chiffrées à l’aide de la clé de chiffrement utilisée à l’emplacement de stockage cible. Il peut s’agir d’une clé gérée par le client ou d’une clé gérée par la plateforme utilisée pour chiffrer les disques de la machine virtuelle.

## <a name="frequently-asked-questions"></a>Questions fréquentes (FAQ)

### <a name="can-i-encrypt-an-existing-backup-vault-with-customer-managed-keys"></a>Puis-je chiffrer un coffre de sauvegarde existant avec des clés gérées par le client ?

Non, le chiffrement CMK ne peut être activé que pour les nouveaux coffres. Le coffre ne doit donc jamais contenir d’éléments protégés. En fait, vous ne devez en aucun cas tenter de protéger des éléments dans le coffre avant d’avoir activé le chiffrement à l’aide de clés gérées par le client.

### <a name="i-tried-to-protect-an-item-to-my-vault-but-it-failed-and-the-vault-still-doesnt-contain-any-items-protected-to-it-can-i-enable-cmk-encryption-for-this-vault"></a>J’ai essayé en vain de protéger un élément dans mon coffre, et le coffre ne contient toujours aucun élément protégé. Puis-je activer le chiffrement CMK pour ce coffre ?

Non, le coffre ne doit pas avoir subi de tentatives de protection.

### <a name="i-have-a-vault-thats-using-cmk-encryption-can-i-later-revert-to-encryption-using-platform-managed-keys-even-if-i-have-backup-items-protected-to-the-vault"></a>J’ai un coffre qui utilise le chiffrement CMK. Puis-je revenir par la suite à un chiffrement à l’aide de clés gérées par la plateforme, même si le coffre contient des éléments de sauvegarde protégés ?

Non, une fois que vous avez activé le chiffrement CMK, vous ne pouvez pas revenir à une utilisation de clés gérées par la plateforme. Vous pouvez modifier les clés en fonction de vos besoins.

### <a name="does-cmk-encryption-for-azure-backup-also-apply-to-azure-site-recovery"></a>Le chiffrement CMK pour Sauvegarde Azure s’applique-t-il aussi à Azure Site Recovery ?

Non, cet article traite du chiffrement de données de sauvegarde uniquement. Pour Azure Site Recovery, vous devez définir la propriété séparément comme étant disponible à partir du service.

### <a name="i-missed-one-of-the-steps-in-this-article-and-went-on-to-protect-my-data-source-can-i-still-use-cmk-encryption"></a>Je n’ai pas suivi l’une des étapes de cet article, mais j’ai pu protéger ma source de données. Puis-je continuer d’utiliser le chiffrement CMK ?

Si vous ne suivez pas les étapes de cet article et que vous continuez à protéger des éléments, le coffre risque de ne pas pouvoir utiliser le chiffrement à l'aide de clés gérées par le client. C’est pourquoi nous vous recommandons de vous référer à [cette liste de vérification](#backing-up-to-a-vault-encrypted-with-customer-managed-keys) avant de procéder à la protection d’éléments.

### <a name="does-using-cmk-encryption-add-to-the-cost-of-my-backups"></a>L’utilisation du chiffrement CMK augmente-elle le coût de mes sauvegardes ?

L’utilisation du chiffrement CMK pour la sauvegarde n’entraîne pas de frais supplémentaires. En revanche, l’utilisation de votre coffre de clés Azure Key où est stockée votre clé peut continuer d’occasionner des frais.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des fonctionnalités de sécurité de Sauvegarde Azure](security-overview.md)
