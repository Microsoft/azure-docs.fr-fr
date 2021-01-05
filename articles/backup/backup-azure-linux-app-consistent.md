---
title: Sauvegardes cohérentes par rapport à l’application des machines virtuelles Linux
description: Créez des sauvegardes cohérentes des applications de vos machines virtuelles Linux sur Azure. Cet article explique la configuration de l’infrastructure de script pour sauvegarder les machines virtuelles Linux déployées par Azure. Il contient également des informations de dépannage.
ms.topic: conceptual
ms.date: 01/12/2018
ms.openlocfilehash: 22053004026a2dd8976027359f11d50a5663b334
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88999238"
---
# <a name="application-consistent-backup-of-azure-linux-vms"></a>Sauvegarde cohérente des applications des machines virtuelles Linux Azure

Lorsque vous prenez des instantanés de sauvegarde de vos machines virtuelles, la cohérence des applications signifie que vos applications démarrent en même temps que les machines virtuelles, une fois celles-ci restaurées. Comme vous pouvez l’imaginer, la cohérence des applications est extrêmement importante. Pour vous assurer que vos machines virtuelles Linux bénéficient de la cohérence des applications, vous pouvez utiliser l’infrastructure de pré-script et de post-script afin d’effectuer des sauvegardes cohérentes des applications. L’infrastructure de pré-script et de post-script prend en charge les machines virtuelles Linux déployées par Azure Resource Manager. Les scripts de cohérence des applications ne prennent pas en charge les machines virtuelles déployées par Service Manager et les machines virtuelles Windows.

## <a name="how-the-framework-works"></a>Fonctionnement de l’infrastructure

L’infrastructure fournit une option permettant d’exécuter des pré/post-scripts personnalisés lors de la capture d’instantanés de machines virtuelles. Les pré-scripts s’exécutent juste avant la capture instantanée de machines virtuelles ; et les post-scripts, juste après. Les pré-scripts et post-scripts vous permettent de contrôler votre application et votre environnement, pendant que vous prenez des captures instantanées de machines virtuelles.

Les pré-scripts appellent les API natives de l’application, qui suspendent les E/S et vident le contenu de la mémoire sur le disque. Ces actions garantissent que la capture instantanée est cohérente des applications. Les post-scripts utilisent les API natives de l’application pour libérer les E/S, qui permettent à l’application de reprendre ses opérations normales après la capture instantanée des machines virtuelles.

## <a name="steps-to-configure-pre-script-and-post-script"></a>Procédure de configuration du pré-script et du post-script

1. Connectez-vous en tant qu’utilisateur root de la machine virtuelle Linux que vous souhaitez sauvegarder.

2. Dans [GitHub](https://github.com/MicrosoftAzureBackup/VMSnapshotPluginConfig), téléchargez le fichier **VMSnapshotScriptPluginConfig.json** et copiez-le dans le dossier **/etc/azure** de toutes les machines virtuelles à sauvegarder. Si le dossier **/etc/azure** n’existe pas, créez-le.

3. Copiez le pré-script et le post-script de votre application sur toutes les machines virtuelles que vous prévoyez de sauvegarder. Vous pouvez copier les scripts vers n’importe quel emplacement sur la machine virtuelle. Veillez à mettre à jour le chemin d’accès complet des fichiers de script dans le fichier **VMSnapshotScriptPluginConfig.json**.

4. Vérifiez les autorisations suivantes pour ces fichiers :

   - **VMSnapshotScriptPluginConfig.json** : autorisation « 600 ». Par exemple, seul l’utilisateur « racine » doit avoir des autorisations de « lecture » et « d’écriture » pour ce fichier, et aucun utilisateur ne doit disposer des autorisations « d’exécution ».

   - **Fichier de pré-script** : autorisation « 700 ».  Par exemple, seul l’utilisateur « racine » doit avoir les autorisations de « lecture », « d’écriture » et « d’exécution » pour ce fichier.

   - **Post-script** : autorisation « 700 ». Par exemple, seul l’utilisateur « racine » doit avoir les autorisations de « lecture », « d’écriture » et « d’exécution » pour ce fichier.

   > [!IMPORTANT]
   > L’infrastructure offre beaucoup de puissance aux utilisateurs. Sécurisez l’infrastructure et vérifiez que seul l’utilisateur « root » a accès au fichier JSON et aux fichiers de script.
   > Si ces conditions ne sont pas remplies, le script ne s’exécute pas, ce qui bloque le système de fichiers et produit une sauvegarde incohérente.
   >

5. Configurez **VMSnapshoScriptPluginConfig.json** comme décrit ici :
    - **pluginName** : laissez ce champ tel quel, sinon vos scripts ne fonctionneront pas comme prévu.

    - **preScriptLocation** : fournissez le chemin complet du pré-script sur la machine virtuelle qui sera sauvegardée.

    - **postScriptLocation** : fournissez le chemin complet du post-script sur la machine virtuelle qui sera sauvegardée.

    - **preScriptParams** : fournissez les paramètres facultatifs qui doivent être transmis au pré-script. Tous les paramètres doivent être entre guillemets. Si vous utilisez plusieurs paramètres, séparez-les par une virgule.

    - **postScriptParams** : fournissez les paramètres facultatifs qui doivent être transmis au post-script. Tous les paramètres doivent être entre guillemets. Si vous utilisez plusieurs paramètres, séparez-les par une virgule.

    - **preScriptNoOfRetries** : définissez le nombre de fois que l’exécution du pré-script doit être réessayée en cas d’erreur avant qu’elle se termine. Zéro signifie une seule tentative et aucune nouvelle tentative en cas d’échec.

    - **postScriptNoOfRetries** :  définissez le nombre de fois que l’exécution du post-script doit être réessayée en cas d’erreur avant qu’elle se termine. Zéro signifie une seule tentative et aucune nouvelle tentative en cas d’échec.

    - **timeoutInSeconds** : spécifiez des délais d’attente individuels pour le pré-script et le post-script (la valeur maximale s’élève à 1 800).

    - **continueBackupOnFailure** : définissez cette valeur sur **true** si vous voulez que Sauvegarde Azure effectue une restauration vers une sauvegarde cohérente en cas d’incident/cohérente dans le système de fichiers en cas d’échec du pré-script ou du post-script. Si cette valeur est définie sur **false**, la sauvegarde échoue en cas d’échec du script (sauf si vous avez une machine virtuelle à un seul disque qui restaure vers une sauvegarde cohérente en cas d’incident, indépendamment de ce paramètre). Lorsque la valeur de **continueBackupOnFailure** est définie sur false, si la sauvegarde échoue, l’opération de sauvegarde est tentée à nouveau en fonction d’une logique de nouvelle tentative en service (pour le nombre spécifié de tentatives).

    - **fsFreezeEnabled** : spécifiez si fsfreeze Linux doit être appelé pendant la prise d’instantané de la machine virtuelle pour garantir la cohérence du système de fichiers. Nous vous recommandons de laisser cette valeur définie sur **true**, sauf si votre application comporte des dépendances sur la désactivation de fsfreeze.

    - **ScriptsExecutionPollTimeSeconds** : définissez le temps de veille de l’extension entre chaque sondage d’exécution du script. Par exemple, si la valeur est 2, l’extension vérifie si l’exécution du pré-script/post-script s’est effectuée toutes les 2 secondes. La valeur minimale est 1 et la valeur maximale 5. La valeur peut être un entier uniquement.

6. L’infrastructure de script est désormais configurée. Si la sauvegarde de la machine virtuelle est déjà configurée, la sauvegarde suivante appelle les scripts et déclenche la sauvegarde cohérente avec les applications. Si la sauvegarde de machine virtuelle n’est pas configurée, configurez-la en procédant de la manière décrite dans [Sauvegarde de machines virtuelles Azure dans des coffres Recovery Services](./backup-azure-vms-first-look-arm.md).

## <a name="troubleshooting"></a>Dépannage

Veillez à ajouter un enregistrement approprié lors de l’écriture de votre pré-script et post-script et passez en revue vos journaux d’activité de script pour résoudre les problèmes de script. Si vous rencontrez toujours des problèmes pour exécuter des scripts, reportez-vous au tableau suivant pour plus d’informations.

| Error | Message d’erreur | Action recommandée |
| ------------------------ | -------------- | ------------------ |
| Pre-ScriptExecutionFailed |Le pré-script a renvoyé une erreur ; la sauvegarde peut ne pas être cohérente avec les applications.| Examinez les journaux d’activité d’erreur de votre script pour résoudre le problème.|  
|Post-ScriptExecutionFailed |Le post-script a renvoyé une erreur qui peut affecter l’état de l’application. |Examinez les journaux d’activité d’erreur de votre script pour résoudre le problème et vérifiez l’état de l’application. |
| Pre-ScriptNotFound |Le pré-script est introuvable à l’emplacement spécifié dans le fichier de configuration **VMSnapshotScriptPluginConfig.json**. |Assurez-vous que ce pré-script est présent au niveau du chemin d’accès spécifié dans le fichier de configuration pour garantir la sauvegarde cohérente des applications.|
| Post-ScriptNotFound |Le post-script est introuvable à l’emplacement spécifié dans le fichier de configuration **VMSnapshotScriptPluginConfig.json**. |Assurez-vous que ce post-script est présent au niveau du chemin d’accès spécifié dans le fichier de configuration pour garantir la sauvegarde cohérente des applications.|
| IncorrectPluginhostFile |Le fichier **Pluginhost** fourni avec l’extension VmSnapshotLinux est endommagé. Le pré-script et le post-script ne peuvent donc pas être exécutés et la sauvegarde ne sera pas cohérente avec les applications.| Désinstallez l’extension **VmSnapshotLinux**, et elle sera automatiquement réinstallée avec la sauvegarde suivante pour résoudre le problème. |
| IncorrectJSONConfigFile | Le fichier **VMSnapshotScriptPluginConfig.json** est incorrect. Le pré-script et le post-script ne peuvent donc pas être exécutés et la sauvegarde ne sera pas cohérente avec les applications. | Téléchargez la copie à partir de [GitHub](https://github.com/MicrosoftAzureBackup/VMSnapshotPluginConfig) et la configurer à nouveau. |
| InsufficientPermissionforPre-Script | Pour l’exécution de scripts, l’utilisateur racine doit être le propriétaire du fichier et le fichier doit avoir des autorisations « 700 », (seul le propriétaire doit avoir des autorisations de « lecture », « d’écriture » et « d’exécution »). | Assurez-vous que l’utilisateur « racine » est le « propriétaire » du fichier de script et que seul le propriétaire dispose des autorisations de « lecture », « d’écriture » et « d’exécution ». |
| InsufficientPermissionforPost-Script | Pour l’exécution de scripts, l’utilisateur racine doit être le propriétaire du fichier et le fichier doit avoir des autorisations « 700 » (c.-à-d. que seul le propriétaire doit avoir des autorisations de « lecture », « d’écriture » et « d’exécution »). | Assurez-vous que l’utilisateur « racine » est le « propriétaire » du fichier de script et que seul le propriétaire dispose des autorisations de « lecture », « d’écriture » et « d’exécution ». |
| Pre-ScriptTimeout | L’exécution du pré-script de sauvegarde cohérente des applications a expiré. | Vérifiez le script et augmentez le délai d’expiration dans le fichier **VMSnapshotScriptPluginConfig.json** situé à l’emplacement **/etc/azure**. |
| Post-ScriptTimeout | L’exécution du post-script de sauvegarde cohérente des applications a expiré. | Vérifiez le script et augmentez le délai d’expiration dans le fichier **VMSnapshotScriptPluginConfig.json** situé à l’emplacement **/etc/azure**. |

## <a name="next-steps"></a>Étapes suivantes

[Sauvegarder des machines virtuelles Azure dans un coffre Recovery Services](./backup-azure-vms-first-look-arm.md)
