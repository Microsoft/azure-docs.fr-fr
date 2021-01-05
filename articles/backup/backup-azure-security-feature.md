---
title: Fonctionnalités de sécurité qui protègent des sauvegardes hybrides
description: Découvrez comment utiliser les fonctionnalités de sécurité dans la Sauvegarde Azure pour renforcer la sécurité des sauvegardes
ms.reviewer: utraghuv
ms.topic: conceptual
ms.date: 06/08/2017
ms.openlocfilehash: 8c671b1b54b937f518f7179bb6940f31a28a78d4
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94841016"
---
# <a name="security-features-to-help-protect-hybrid-backups-that-use-azure-backup"></a>Fonctionnalités de sécurité pour la protection des sauvegardes hybrides ayant recours à la Sauvegarde Azure

Les préoccupations en matière de risques de sécurité, comme les logiciels malveillants, le ransomware et les intrusions, sont de plus en plus nombreuses. Ces problèmes de sécurité peuvent coûter cher, à la fois en termes d’argent et de données. Pour vous protéger contre ces attaques, la Sauvegarde Azure fournit désormais des fonctionnalités de sécurité visant à protéger les sauvegardes hybrides. Cet article explique comment activer et utiliser ces fonctionnalités par le biais d’un agent Azure Recovery Services et du serveur de sauvegarde Azure. Voici quelques fonctionnalités :

- **Prévention**. Une couche supplémentaire d’authentification est ajoutée à chaque fois qu’une opération critique (par exemple, Modifier la phrase secrète) est effectuée. Cette validation permet de garantir que ces opérations ne peuvent être effectuées que par les utilisateurs ayant des informations d’identification Azure valides.
- **Alertes**. Un e-mail de notification est envoyé à l’administrateur de l’abonnement à chaque fois qu’une opération critique (par exemple, Supprimer les données de sauvegarde) est effectuée. Ce courrier électronique garantit que l’utilisateur est rapidement averti de ces actions.
- **Récupération**. Les données de sauvegarde supprimées sont conservées pendant 14 jours supplémentaires à compter de la date de suppression. Cela garantit la possibilité de les récupérer dans un délai donné afin d’éviter toute perte, même en cas d’attaque. En outre, les points de récupération minimum sont conservés en plus grand nombre pour offrir une protection contre les données corrompues.

> [!NOTE]
> Les fonctionnalités de sécurité ne doivent pas être activées si vous utilisez une sauvegarde de machine virtuelle IaaS (Infrastructure en tant que service). Ces fonctionnalités ne sont pas encore disponibles pour la sauvegarde de machine virtuelle IaaS, donc leur activation n’aura aucun impact. Vous ne devez activer les fonctionnalités de sécurité que si vous utilisez : <br/>
>
> - **Agent Azure Backup**. Version minimale de l’agent 2.0.9052. Une fois que vous avez activé ces fonctionnalités, vous devez effectuer une mise à niveau vers cette version de l’agent pour effectuer des opérations critiques. <br/>
> - **Serveur de sauvegarde Azure**. Version minimale de l’agent Azure Backup 2.0.9052 avec la mise à jour 1 du serveur de sauvegarde Azure. <br/>
> - **System Center Data Protection Manager**. Version minimale de l’agent Azure Backup version 2.0.9052 avec Data Protection Manager 2012 R2 UR12 ou Data Protection Manager 2016 UR2. <br/>

> [!NOTE]
> Ces fonctionnalités ne sont disponibles que pour le coffre Recovery Services. Tous les coffres Recovery Services nouvellement créés ont ces fonctionnalités activées par défaut. Pour les coffres Recovery Services existants, les utilisateurs activent ces fonctionnalités en suivant les étapes mentionnées dans la section ci-dessous. Une fois les fonctionnalités activées, elles s’appliquent à l’ensemble des ordinateurs agents Recovery Services, instances de serveur de sauvegarde Azure, et serveurs Data Protection Manager inscrits avec le coffre. L’activation de ce paramètre ne s’effectue qu’une seule fois et il est impossible de désactiver ces fonctionnalités après leur activation.
>

## <a name="enable-security-features"></a>Activer les fonctionnalités de sécurité

Si vous créez un coffre Recovery Services, vous pouvez utiliser toutes les fonctionnalités de sécurité. Si vous utilisez un coffre existant, activez les fonctionnalités de sécurité en procédant comme suit :

1. Connectez-vous au portail Azure à l’aide de vos informations d’identification Azure.
2. Sélectionnez **Parcourir**, puis entrez **Recovery Services**.

    ![Capture d’écran de l’option Parcourir du portail Azure](./media/backup-azure-security-feature/browse-to-rs-vaults.png) <br/>

    La liste des archivages de Recovery Services s’affiche. Dans cette liste, sélectionnez un coffre. Le tableau de bord de l’archivage sélectionné s'ouvre.
3. Dans la liste d’éléments qui s’affiche sous le coffre, sous **Paramètres**, sélectionnez **Propriétés**.

    ![Capture d’écran des options de coffre Recovery Services](./media/backup-azure-security-feature/vault-list-properties.png)
4. Sous **Paramètres de sécurité**, sélectionnez **Mettre à jour**.

    ![Capture d’écran des propriétés de coffre Recovery Services](./media/backup-azure-security-feature/security-settings-update.png)

    Le lien de mise à jour ouvre le volet **Paramètres de sécurité**, qui vous permet de découvrir une courte présentation de ces fonctionnalités et d’activer ces dernières.
5. Dans la liste déroulante **Avez-vous configuré Azure AD Multi-Factor Authentication ?** , sélectionnez une valeur pour confirmer que vous avez activé ou non [Azure AD Multi-Factor Authentication](../active-directory/authentication/concept-mfa-howitworks.md). Si tel est le cas, vous êtes invité à vous authentifier à partir d’un autre appareil (par exemple, un téléphone mobile) lors de votre connexion au portail Azure.

   Lorsque vous effectuez des opérations critiques dans la sauvegarde, vous devez entrer un code PIN de sécurité, disponible sur le portail Azure. L’activation de la fonctionnalité Azure AD Multi-Factor Authentication offre une couche de sécurité supplémentaire. Seuls les utilisateurs autorisés, disposant d’informations d’identification Azure valides et authentifiés à partir d’un deuxième appareil peuvent accéder au portail Azure.
6. Pour enregistrer les paramètres de sécurité, sélectionnez **Activer**, puis **Enregistrer**. Vous pouvez sélectionner **Activer** uniquement après avoir sélectionné une valeur dans la liste **Avez-vous configuré Azure AD Multi-Factor Authentication ?** à l’étape précédente.

    ![Capture d’écran des paramètres de sécurité](./media/backup-azure-security-feature/enable-security-settings-dpm-update.png)

## <a name="recover-deleted-backup-data"></a>Récupérer les données de sauvegarde supprimées

La sauvegarde conserve les données de sauvegarde supprimées pendant 14 jours supplémentaires et ne les supprime pas immédiatement si l’opération **Arrêter la sauvegarde avec suppression des données de sauvegarde** est effectuée. Pour restaurer ces données dans la période de 14 jours autorisée, procédez comme suit, selon ce que vous utilisez :

Pour les utilisateurs de l’**agent Azure Recovery Services** :

1. Si l’ordinateur sur lequel les sauvegardes étaient effectuées est toujours disponible, protégez à nouveau les sources de données supprimées et utilisez la fonctionnalité [Récupération des données sur le même ordinateur](backup-azure-restore-windows-server.md#use-instant-restore-to-recover-data-to-the-same-machine) d’Azure Recovery Services pour procéder à la récupération à partir de tous les anciens points de récupération.
2. Si l’ordinateur n’est pas disponible, utilisez [Récupération sur un autre ordinateur](backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine) pour utiliser un autre ordinateur Azure Recovery Services afin d’obtenir ces données.

Pour les utilisateurs **Serveur de sauvegarde Azure** :

1. Si le serveur sur lequel les sauvegardes étaient effectuées est toujours disponible, protégez à nouveau les sources de données supprimées et utilisez la fonctionnalité **Récupérer les données** pour effectuer la récupération à partir de tous les anciens points de récupération.
2. Si le serveur n’est pas disponible, utilisez [Récupérer les données à partir d’un autre serveur de sauvegarde Azure](backup-azure-alternate-dpm-server.md) pour utiliser une autre instance de serveur de sauvegarde Azure afin d’obtenir ces données.

Pour les utilisateurs de **Data Protection Manager** :

1. Si le serveur sur lequel les sauvegardes étaient effectuées est toujours disponible, protégez à nouveau les sources de données supprimées et utilisez la fonctionnalité **Récupérer les données** pour effectuer la récupération à partir de tous les anciens points de récupération.
2. Si le serveur n’est pas disponible, sélectionnez [Ajouter un serveur DPM externe](backup-azure-alternate-dpm-server.md) pour utiliser un autre serveur Data Protection Manager afin d’obtenir ces données.

## <a name="prevent-attacks"></a>Empêcher les attaques

Des vérifications ont été ajoutées pour que seuls les utilisateurs valides puissent effectuer diverses opérations. Ces vérifications incluent l’ajout d’une couche d’authentification et la conservation d’une durée de rétention minimale pour la récupération.

### <a name="authentication-to-perform-critical-operations"></a>Authentification pour exécuter des opérations critiques

Dans le cadre de l’ajout d’une couche d’authentification pour les opérations critiques, vous êtes invité à entrer un code PIN de sécurité lors de l’exécution des opérations d’**arrêt de la protection lors de la suppression de données** et de **modification de la phrase secrète**.

> [!NOTE]
>
> Actuellement, le code PIN de sécurité n’est pas pris en charge pour **Arrêter la protection avec les opérations Supprimer les données** pour DPM et MABS.

Pour recevoir ce code PIN :

1. Connectez-vous au portail Azure.
2. Accédez à **Coffre Recovery Services** > **Paramètres** > **Propriétés**.
3. Sous **Code PIN de sécurité**, sélectionnez **Générer**. Cela a pour effet d’ouvrir un panneau contenant le code PIN à entrer dans l’interface utilisateur de l’agent Azure Recovery Services.
    Ce code PIN n’est valide que pendant cinq minutes, après quoi il est généré automatiquement.

### <a name="maintain-a-minimum-retention-range"></a>Maintenir la durée de rétention minimale

Les vérifications suivantes ont été ajoutées pour garantir qu’il existe toujours un nombre valide de points de récupération disponibles :

- Pour une rétention quotidienne, une rétention de **sept** jours minimum doit être appliquée.
- Pour une rétention hebdomadaire, une rétention de **quatre** semaines minimum doit être appliquée.
- Pour une rétention mensuelle, une rétention de **trois** mois minimum doit être appliquée.
- Pour une rétention annuelle, une rétention d’**un** an minimum doit être appliquée.

## <a name="notifications-for-critical-operations"></a>Notifications pour les opérations critiques

Généralement, lorsqu’une opération critique est effectuée, l’administrateur de l’abonnement reçoit une notification électronique contenant les détails de l’opération. Vous pouvez configurer les autres destinataires de ces notifications électroniques sur le portail Azure.

Les fonctionnalités de sécurité mentionnées dans cet article fournissent des mécanismes de défense contre les attaques ciblées. Et surtout, en cas d’attaque, ces fonctionnalités vous permettent de récupérer vos données.

## <a name="troubleshooting-errors"></a>Dépannage des erreurs

| Opération | Détails de l’erreur | Résolution |
| --- | --- | --- |
| Modification de la stratégie |Impossible de modifier la stratégie de sauvegarde. Erreur : Échec de l’opération en cours en raison d’une erreur de service interne [0x29834]. Veuillez réessayer l’opération après un certain temps. Si le problème persiste, contactez le support technique Microsoft. |**Cause :**<br/>Cette erreur apparaît lorsque les paramètres de sécurité sont activés, que vous essayez de réduire la durée de rétention en deçà des valeurs minimales spécifiées ci-dessus et que vous utilisez une version non prise en charge (les versions prises en charge sont indiquées dans la première remarque de cet article). <br/>**Action recommandée :**<br/> Dans ce cas, vous devez définir la période de rétention sur une valeur supérieure à la période de rétention minimale spécifiée (sept jours pour la rétention quotidienne, quatre semaines pour la rétention hebdomadaire, trois semaines pour la rétention mensuelle ou une année pour la rétention annuelle) pour exécuter les mises à jour relatives à la stratégie. Vous pouvez également mettre à jour l’agent de sauvegarde, le serveur de sauvegarde Azure et/ou le correctif cumulatif de DPM pour tirer parti de toutes les mises à jour de sécurité. |
| Modification de la phrase secrète |Le code PIN de sécurité entré est incorrect. (ID : 100130) Indiquez le code PIN de sécurité approprié pour effectuer cette opération. |**Cause :**<br/> Cette erreur se produit lorsque vous entrez un code PIN de sécurité non valide ou qui a expiré lors d’une opération critique (par exemple, la modification de la phrase secrète). <br/>**Action recommandée :**<br/> Pour exécuter l’opération, vous devez entrer un code PIN de sécurité valide. Pour obtenir le code PIN, connectez-vous au portail Azure et accédez au coffre Recovery Services > Paramètres > Propriétés > Générer un code PIN de sécurité. Utilisez ce code PIN pour modifier la phrase secrète. |
| Modification de la phrase secrète |Échec de l’opération. ID : 120002 |**Cause :**<br/>Cette erreur apparaît lorsque les paramètres de sécurité sont activés, que vous essayez de modifier la phrase secrète et que vous utilisez une version non prise en charge (les versions valides sont indiquées dans la première remarque de cet article).<br/>**Action recommandée :**<br/> Pour modifier la phrase secrète, vous devez tout d’abord mettre à jour l’agent de sauvegarde vers la version 2.0.9052, le serveur de sauvegarde Azure vers la mise à jour minimale 1 et/ou DPM vers la version minimale DPM 2012 R2 UR12 ou DPM 2016 UR2 (liens de téléchargement disponibles plus bas), puis entrer un code PIN de sécurité valide. Pour obtenir le code PIN, connectez-vous au portail Azure et accédez au coffre Recovery Services > Paramètres > Propriétés > Générer un code PIN de sécurité. Utilisez ce code PIN pour modifier la phrase secrète. |

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer avec le coffre Azure Recovery Services](backup-azure-vms-first-look-arm.md) pour activer ces fonctionnalités.
- [Téléchargez le dernier Agent Azure Recovery Services](https://aka.ms/azurebackup_agent) pour protéger les ordinateurs Windows, ainsi que vos données de sauvegarde contre les attaques.
- [Téléchargez le dernier serveur de sauvegarde Azure](https://support.microsoft.com/help/4457852/microsoft-azure-backup-server-v3) pour protéger les charges de travail, ainsi que vos données de sauvegarde contre les attaques.
- [Téléchargez UR12 pour System Center 2012 R2 Data Protection Manager](https://support.microsoft.com/help/3209592/update-rollup-12-for-system-center-2012-r2-data-protection-manager) ou [téléchargez UR2 pour System Center 2016 Data Protection Manager](https://support.microsoft.com/help/3209593/update-rollup-2-for-system-center-2016-data-protection-manager) pour protéger les charges de travail et vos données de sauvegarde contre les attaques.
