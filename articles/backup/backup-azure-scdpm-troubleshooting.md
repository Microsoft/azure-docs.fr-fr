---
title: Résoudre les problèmes liés à System Center Data Protection Manager
description: Dans cet article, découvrez des solutions pour les problèmes que vous pouvez rencontrer lors de l’utilisation de System Center Data Protection Manager.
ms.topic: troubleshooting
ms.date: 01/30/2019
ms.openlocfilehash: 84646f34d905b570855f655465529eb19b717cab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86513810"
---
# <a name="troubleshoot-system-center-data-protection-manager"></a>Résoudre les problèmes liés à System Center Data Protection Manager

Cet article décrit les solutions aux problèmes que vous pouvez rencontrer lors de l’utilisation de Data Protection Manager.

Pour lire les dernières notes de publication de System Center Data Protection Manager, consultez la [documentation de System Center](/system-center/dpm/dpm-release-notes). Plus d’informations sur la prise en charge pour Data Protection Manager dans [cette matrice](/system-center/dpm/dpm-protection-matrix).

## <a name="error-replica-is-inconsistent"></a>Erreur : le réplica est incohérent

Un réplica peut être incohérent pour les raisons suivantes :

- Le travail de création du réplica a échoué.
- Le journal des modifications a des problèmes.
- La bitmap de filtre au niveau du volume contient des erreurs.
- La machine source s’arrête de manière inattendue.
- Le journal de synchronisation est plein.
- Le réplica est vraiment incohérent.

Pour résoudre ce problème, procédez comme suit :

- Pour supprimer l’état incohérent, exécutez la vérification de cohérence manuellement ou planifiez une vérification de cohérence quotidienne.
- Vérifiez que vous utilisez la version la plus récente du serveur de sauvegarde Microsoft Azure et de Data Protection Manager.
- Vérifiez que le paramètre de **cohérence automatique** est activé.
- Essayez de redémarrer les services à partir de l’invite de commandes. Utilisez la commande `net stop dpmra` suivie de `net start dpmra`.
- Vérifiez que les exigences de connectivité et de bande passante réseau sont respectées.
- Vérifiez si la machine source a été arrêtée de manière inattendue.
- Vérifiez l’intégrité du disque et assurez-vous que l’espace disponible est suffisant pour le réplica.
- Vérifiez qu’aucun travail de sauvegarde n’est en cours d’exécution en même temps qu’un autre.

## <a name="error-online-recovery-point-creation-failed"></a>Erreur : échec lors de la création de points de récupération en ligne

Pour résoudre ce problème, procédez comme suit :

- Vérifiez que vous disposez de la dernière version de l’agent de sauvegarde Azure.
- Essayez de créer manuellement un point de récupération dans la zone de tâches Protection.
- Assurez-vous d’exécuter la vérification de cohérence sur la source de données.
- Vérifiez que les exigences de connectivité et de bande passante réseau sont respectées.
- Si les données du réplica sont dans un état incohérent, créez un point de récupération de disque de cette source de données.
- Vérifiez que le réplica est bien présent et qu’il ne manque pas.
- Assurez-vous que le réplica dispose d’un espace suffisant pour créer le journal de nombre de séquences de mise à jour (USN).

## <a name="error-unable-to-configure-protection"></a>Erreur : configuration de la protection impossible

Cette erreur se produit lorsque le serveur Data Protection Manager ne parvient pas à contacter le serveur protégé.

Pour résoudre ce problème, procédez comme suit :

- Vérifiez que vous disposez de la dernière version de l’agent de sauvegarde Azure.
- Vérifiez que votre serveur Data Protection Manager et le serveur protégé sont connectés (réseau/pare-feu/proxy).
- Si vous protégez un serveur SQL, vérifiez que la propriété **Propriétés de la connexion** > **NT AUTHORITY\SYSTEM** indique que le paramètre **sysadmin** est activé.

## <a name="error-server-not-registered-as-specified-in-vault-credential-file"></a>Erreur : serveur non inscrit comme spécifié dans le fichier d'informations d'identification du coffre

Cette erreur se produit pendant le processus de récupération des données du serveur de sauvegarde Azure/Data Protection Manager. Le fichier d'informations d'identification du coffre qui est utilisé lors du processus de récupération n’appartient pas au coffre Recovery Services pour le serveur de sauvegarde Azure/Data Protection Manager.

Pour résoudre ce problème, effectuez les étapes suivantes :

1. Téléchargez le fichier des informations d’identification du coffre Recovery Services auquel le serveur de sauvegarde Azure/Data Protection Manager est inscrit.
2. Essayez d’inscrire le serveur auprès du coffre en utilisant le dernier fichier d’informations d’identification du coffre téléchargé.

## <a name="error-no-recoverable-data-or-selected-server-not-a-data-protection-manager-server"></a>Erreur : aucune donnée récupérable ou le serveur sélectionné n’est pas un serveur Data Protection Manager

Cette erreur peut se produire pour les raisons suivantes :

- Aucun autre serveur de sauvegarde Azure/Data Protection Manager n’est inscrit auprès du coffre Recovery Services.
- Les serveurs n’ont pas encore téléchargé les métadonnées.
- Le serveur sélectionné n’est pas un serveur de sauvegarde Azure/Data Protection Manager.

Lorsque d’autres serveurs de sauvegarde Azure/Data Protection Manager sont inscrits auprès du coffre Recovery Services, suivez les étapes suivantes pour résoudre le problème :

1. Vérifiez que le dernier agent de sauvegarde Azure est installé.
2. Après avoir vérifié que le dernier agent est installé, patientez un jour avant de démarrer le processus de récupération. Le travail de sauvegarde nocturne charge les métadonnées de toutes les sauvegardes protégées sur le cloud. Vous pouvez ensuite récupérer les données de sauvegarde.

## <a name="error-provided-encryption-passphrase-doesnt-match-passphrase-for-server"></a>Erreur : la phrase secrète de chiffrement fournie ne correspond pas à la phrase secrète du serveur

Cette erreur se produit pendant le processus de chiffrement lors de la récupération des données du serveur de sauvegarde Azure/Data Protection Manager. La phrase secrète de chiffrement qui est utilisée lors du processus de récupération ne correspond pas à la phrase secrète de chiffrement du serveur. Par conséquent, l’agent ne peut pas déchiffrer les données et la récupération échoue.

> [!IMPORTANT]
> Si vous perdez ou oubliez la phrase secrète de chiffrement, il n’existe aucune autre méthode pour récupérer les données. La seule option consiste à générer une nouvelle phrase secrète. Utilisez la nouvelle phrase secrète pour chiffrer les futures données de sauvegarde.
>
> Lorsque vous récupérez des données, indiquez toujours la même phrase secrète de chiffrement associée au serveur de sauvegarde Azure/Data Protection Manager.
>
