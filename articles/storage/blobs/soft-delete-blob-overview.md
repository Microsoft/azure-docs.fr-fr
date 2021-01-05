---
title: Suppression réversible pour les objets blob
titleSuffix: Azure Storage
description: La fonctionnalité de suppression réversible pour les objets blob protège vos données, ce qui vous permet de récupérer plus facilement vos données en cas de modification ou de suppression malencontreuses de celles-ci par une application ou un autre utilisateur du compte de stockage.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 07/15/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: bb90c5776e67c1ba8fecdbf394a8098e96ca0652
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96022375"
---
# <a name="soft-delete-for-blobs"></a>Suppression réversible pour les objets blob

La suppression réversible pour les objets blob protège vos données contre les modifications ou suppressions accidentelles ou erronées. Lorsque la suppression réversible pour les objets blob est activée pour un compte de stockage, les blobs, les versions des blobs et les captures instantanées de ce compte de stockage peuvent être récupérés après leur suppression, au cours d’une période de rétention que vous spécifiez.

S’il existe une possibilité de modification ou de suppression accidentelles de vos données par une application ou un autre utilisateur du compte de stockage, Microsoft vous recommande d’activer la suppression réversible. Pour plus d’informations sur l’activation de la suppression réversible, consultez [Activer et gérer la suppression réversible pour les blobs](./soft-delete-blob-enable.md).

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="about-soft-delete-for-blobs"></a>À propos de la suppression réversible pour les objets blob

Lorsque la suppression réversible pour les objets blob est activée sur un compte de stockage, vous pouvez récupérer les objets une fois qu’ils ont été supprimés, au cours de la période de conservation des données spécifiée. Cette protection s’étend à tous les blobs (objets blob de blocs, objets blob d’ajout ou objets blob de pages) qui sont effacées à la suite d’un remplacement.

Si les données d’un objet blob ou d’une capture instantanée existant sont supprimées alors que la suppression réversible d’objet blob est activée, mais que le contrôle de version des blobs n’est pas activé, une capture instantanée supprimée de manière réversible est générée pour enregistrer l’état des données remplacées. Une fois la période de rétention spécifiée expirée, l’objet est supprimé définitivement.

Si le contrôle de version des objets blob et la suppression réversible d’objets blob sont tous deux activés sur le compte de stockage, la suppression d’un objet blob crée une nouvelle version au lieu d’un instantané supprimé de manière réversible. La nouvelle version n’est pas supprimée de manière réversible et n’est pas supprimée à l’expiration de la période de rétention de la suppression réversible. Les versions supprimées de manière réversible d’un objet BLOB peuvent être restaurées au cours de la période de rétention en appelant l’opération [Undelete Blob](/rest/api/storageservices/undelete-blob) (Annuler la suppression d’un objet BLOB). L’objet BLOB peut ensuite être restauré à partir de l’une de ses versions en appelant l’opération [Copy Blob](/rest/api/storageservices/copy-blob) (Copier l’objet BLOB). Pour plus d’informations sur l’utilisation conjointe de la gestion des versions d’objets BLOB et de la suppression réversible, consultez [Contrôle de version d’objet BLOB et suppression réversible](versioning-overview.md#blob-versioning-and-soft-delete).

Les objets supprimés de manière réversible sont invisibles, sauf s’ils sont répertoriés de façon explicite.

La suppression réversible d’objet blob étant rétrocompatible, il est inutile d’apporter des modifications à vos applications pour tirer parti des protections qu’offre cette fonctionnalité. Toutefois, la fonctionnalité de [récupération des données](#recovery) introduit une nouvelle API de **rétablissement d’objet blob supprimé**.

La suppression réversible d’objet blob est disponible pour les comptes de stockage à usage général v2, à usage général v1, ainsi que pour les comptes de stockage d’objets blob. Les types de compte de stockage standard et premium sont pris en charge. La suppression réversible d’objet blob est disponible pour tous les niveaux de stockage : chaud, froid et archive. La suppression réversible est disponible pour les disques non managés, qui sont des objets blob de pages en arrière-plan, mais n’est pas disponible pour les disques managés.

### <a name="configuration-settings"></a>Paramètres de configuration

Lorsque vous créez un compte, la suppression réversible est désactivée par défaut. Elle est également désactivée par défaut pour les comptes de stockage existants. Vous pouvez activer ou désactiver la suppression réversible pour un compte de stockage à tout moment.

Lorsque vous activez la suppression réversible, vous devez configurer la période de rétention. La période de rétention indique la durée pendant laquelle les données supprimées de manière réversible sont stockées et disponibles pour récupération. Pour les objets supprimés explicitement, le délai la période de rétention démarre dès la suppression des données. Pour les versions ou instantanés supprimés de manière réversible générés par la fonctionnalité de suppression réversible lors du remplacement de données, l’horloge démarre lors de la génération de la version ou de l’instantané. La période de rétention peut être comprise entre 1 et 365 jours.

Vous pouvez modifier la période de rétention de suppression réversible à tout moment. Une mise à jour de la période de rétention s’applique uniquement aux données supprimées par la suite. Les données supprimées antérieurement expirent à l’issue de la période de rétention qui était configurée lors de leur suppression. Une tentative de suppression d’un objet supprimé de manière réversible n’affecte pas son délai d’expiration.

Si vous désactivez la suppression réversible, vous pouvez continuer à accéder aux données supprimées de manière réversible dans votre compte de stockage et à les récupérer lors de l’activation de la fonctionnalité.

### <a name="saving-deleted-data"></a>Enregistrement de données supprimées

La suppression réversible préserve vos données dans les nombreux cas où les objets sont supprimés ou remplacés.

Quand un objet blob est remplacé à l’aide d’une des commandes **Put Blob**, **Put Block List**, ou **Copy Blob**, une version ou un instantané de l’état de l’objet blob avant l’opération d’écriture est générée automatiquement. Cet objet est invisible, sauf si les objets supprimés de manière réversible sont répertoriés de façon explicite. Pour savoir comment répertorier des objets supprimés de manière réversible, voir la section [Récupération](#recovery).

![Diagramme montrant comment les captures instantanées d’objets blob sont stockées lorsqu’elles sont remplacées à l’aide de Placer Blob, Placer une liste de blocage ou Copier le Blob.](media/soft-delete-blob-overview/storage-blob-soft-delete-overwrite.png)

*Les données supprimées de manière réversible s’affichent en gris, et les données actives en bleu. Les données écrites plus récemment s’affichent sous les données plus anciennes. Si B0 est remplacé par B1, un instantané d’objet blob supprimé de manière réversible de B0 est généré. Si B1 est remplacé par B2, un instantané d’objet blob supprimé de manière réversible de B1 est généré.*

> [!NOTE]  
> La fonctionnalité de suppression réversible offre une protection en cas de remplacement pour les opérations de copie uniquement quand elle est activée pour le compte de l’objet blob de destination.

> [!NOTE]  
> La fonctionnalité de suppression réversible n’offre pas de protection en cas de remplacement pour les objets blob au niveau archive. Si un objet blob dans l’archive est remplacé par un nouvel objet blob à un niveau quelconque, l’objet blob remplacé expire définitivement.

Lors de l’appel de la commande **Delete Blob** sur un instantané, celui-ci est marqué comme supprimé de manière réversible. Aucun nouvel instantané n’est généré.

![Diagramme montrant comment les captures instantanées d’objets blob sont supprimées de manière réversible en utilisant Supprimer le blob.](media/soft-delete-blob-overview/storage-blob-soft-delete-explicit-delete-snapshot.png)

*Les données supprimées de manière réversible s’affichent en gris, et les données actives en bleu. Les données écrites plus récemment s’affichent sous les données plus anciennes. Lors de l’appel de la commande **Snapshot Blob**, B0 devient un instantané et B1 est l’état actif de l’objet blob. Quand l’instantané B0 est supprimé, il est marqué comme supprimé de manière réversible.*

Lors de l’appel de la commande **Delete Blob** sur un objet blob de base (tout objet blob qui est pas un instantané), celui-ci est marqué comme supprimé de manière réversible. Conformément au comportement précédent, l’appel de la commande **Delete Blob** sur un objet blob ayant des instantanés actifs renvoie une erreur. L’appel de la commande **Delete Blob** sur un objet blob ayant un instantané d’objet blob supprimé de manière réversible ne retourne pas d’erreur. Quand la suppression réversible est activée, vous pouvez toujours supprimer un objet blob et tous ses instantanés en une seule opération. Cela a pour effet de marquer l’objet blob de base et les instantanés comme étant supprimés de manière réversible.

![Diagramme montrant ce qu’il se passe lorsque l’option Supprimer le blog est appelée sur un objet blob de base.](media/soft-delete-blob-overview/storage-blob-soft-delete-explicit-include.png)

*Les données supprimées de manière réversible s’affichent en gris, et les données actives en bleu. Les données écrites plus récemment s’affichent sous les données plus anciennes. Ici, un appel de la commande **Delete Blob** est effectué pour supprimer B2 et tous les instantanés qui y sont associés. L’objet blob actif, B2, ainsi que tous les instantanés associés à celui-ci, sont marqués comme étant supprimés de manière réversible.*

> [!NOTE]  
> Quand un objet blob supprimé de manière réversible est remplacé, un instantané d’objet blob supprimé de manière réversible reflétant l’état de l’objet blob avant l’opération d’écriture est généré automatiquement. Le nouvel objet blob hérite le niveau de l’objet blob remplacé.

La fonctionnalité de suppression réversible n’enregistre pas vos données en cas de suppression de conteneur ou de compte, ou en cas de remplacement de métadonnées et propriétés d’objet blob. Pour protéger un compte de stockage contre une suppression malencontreuse, vous pouvez configurer un verrou à l’aide d’Azure Resource Manager. Pour plus d’informations, voir l’article sur Azure Resource Manager intitulé [Verrouiller les ressources pour empêcher les modifications inattendues](../../azure-resource-manager/management/lock-resources.md).

Le tableau suivant indique le comportement attendu quand la suppression réversible est activée :

| Opération d’API REST | Type de ressource | Description | Modification du comportement |
|--------------------|---------------|-------------|--------------------|
| [Supprimer](/rest/api/storagerp/StorageAccounts/Delete) | Compte | Supprime le compte de stockage avec l’ensemble des conteneurs et objets blob qui y figurent.                           | Aucune modification. Les conteneurs et objets blob dans le compte supprimé ne sont pas récupérables. |
| [Delete Container](/rest/api/storageservices/delete-container) | Conteneur | Supprime le conteneur avec tous les objets blob qui y figurent. | Aucune modification. Les objets blob figurant dans le conteneur supprimé ne sont pas récupérables. |
| [Put Blob](/rest/api/storageservices/put-blob) | Objets blob de blocs, d’ajouts et de pages | Crée ou remplace un objet blob dans un conteneur. | Si cette opération est utilisée pour remplacer un objet blob existant, un instantané de l’état de l’objet blob avant l’appel est généré automatiquement. Cela s’applique également à un objet blob précédemment supprimé de manière réversible si et seulement si celui-ci est remplacé par un objet blob du même type (objet blob de blocs, d’ajouts ou de pages). En cas de remplacement par un objet blob d’un autre type, toutes les données supprimées de manière réversible expireront définitivement. |
| [Delete Blob](/rest/api/storageservices/delete-blob) | Objets blob de blocs, d’ajouts et de pages | Marque un objet blob ou un instantané d’objet blob pour suppression. L’objet blob ou l’instantané est ensuite supprimé pendant le nettoyage de la mémoire | Si cette opération est utilisée pour supprimer un instantané d’objet blob, celui-ci est marqué comme étant supprimé de manière réversible. Si cette opération est utilisée pour supprimer un objet blob, celui-ci est marqué comme étant supprimé de manière réversible. |
| [Copy Blob](/rest/api/storageservices/copy-blob) | Objets blob de blocs, d’ajouts et de pages | Copie un objet blob source vers un objet blob de destination dans le même ou dans un autre compte de stockage. | Si cette opération est utilisée pour remplacer un objet blob existant, un instantané de l’état de l’objet blob avant l’appel est généré automatiquement. Cela s’applique également à un objet blob précédemment supprimé de manière réversible si et seulement si celui-ci est remplacé par un objet blob du même type (objet blob de blocs, d’ajouts ou de pages). En cas de remplacement par un objet blob d’un autre type, toutes les données supprimées de manière réversible expireront définitivement. |
| [Put Block](/rest/api/storageservices/put-block) | Objets blob de blocs | Crée un bloc à valider en tant qu’élément d’un objet blob de blocs. | Si cette opération est utilisée pour valider un bloc dans un objet blob actif, rien ne change. Si elle est utilisée pour valider un bloc dans un objet blob supprimé de manière réversible, un nouvel objet blob est créé, et un instantané est généré automatiquement pour capturer l’état de l’objet blob supprimé de manière réversible. |
| [Put Block List](/rest/api/storageservices/put-block-list) | Objets blob de blocs | Valide un objet blob en spécifiant le jeu d’ID de bloc composant cet objet blob de blocs. | Si cette opération est utilisée pour remplacer un objet blob existant, un instantané de l’état de l’objet blob avant l’appel est généré automatiquement. Cela s’applique également à tout objet blob précédemment supprimé de manière réversible si et seulement s’il s’agit d’un objet blob de blocs. En cas de remplacement par un objet blob d’un autre type, toutes les données supprimées de manière réversible expireront définitivement. |
| [Put Page](/rest/api/storageservices/put-page) | Objets blob de pages | Écrit une plage de pages dans un objet blob de pages. | Aucune modification. Les données de l’objet blob de pages remplacées ou effacées par cette opération n’étant pas enregistrées, elles sont irrécupérables. |
| [Append Block](/rest/api/storageservices/append-block) | Objets blob d’ajouts | Écrit un bloc de données à la fin d’un objet blob d’ajouts | Aucune modification. |
| [Set Blob Properties](/rest/api/storageservices/set-blob-properties) | Objets blob de blocs, d’ajouts et de pages | Définit les valeurs des propriétés système pour un objet blob. | Aucune modification. Les propriétés d’objet blob remplacées ne sont pas récupérables. |
| [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata) | Objets blob de blocs, d’ajouts et de pages | Définit les métadonnées définies par l’utilisateur pour l’objet blob spécifié en tant qu’une ou plusieurs paires nom-valeur. | Aucune modification. Les métadonnées de l’objet blob remplacé ne sont pas récupérables. |

Il est important de noter que l’appel de la commande **Put Page** pour remplacer ou effacer des plages de pages dans un objet blob de pages n’a pas pour effet de générer automatiquement des instantanés. Les disques de machine virtuelle s’appuient sur des objets blob de pages et utilisent la commande **Put Page** pour écrire des données.

### <a name="recovery"></a>Récupération

L’appel de l’opération [Undelete Blob](/rest/api/storageservices/undelete-blob) sur un objet blob de base supprimé de manière réversible a pour effet de restaurer comme actifs cet objet blob et tous les instantanés supprimés de manière réversible qui lui sont associés. L’appel de l’opération **Undelete Blob** sur un objet blob de base actif a pour effet de restaurer comme actifs tous les instantanés supprimés de manière réversible qui lui sont associés. Quand des instantanés sont restaurés comme actifs, ils ressemblent à des instantanés générés par l’utilisateur et ne remplacent pas l’objet blob de base.

Pour restaurer un objet blob en instantané d’objet blob supprimé de manière réversible, vous pouvez appeler la commande **Undelete Blob** sur l’objet blob de base. Ensuite, vous pouvez copier l’instantané sur l’objet blob ainsi activé. Vous pouvez également copier l’instantané vers un nouvel objet blob.

![Diagramme montrant ce qu’il se passe lorsque l’opération Annulation de la suppression d’un objet blob est utilisée.](media/soft-delete-blob-overview/storage-blob-soft-delete-recover.png)

*Les données supprimées de manière réversible s’affichent en gris, et les données actives en bleu. Les données écrites plus récemment s’affichent sous les données plus anciennes. Ici, la commande **Undelete Blob** est appelée sur l’objet blob B, ce qui a pour effet de restaurer comme actifs l’objet blob de base B1 ainsi que tous les instantanés qui y sont associés, en l’occurrence uniquement B0. Dans la deuxième étape, B0 est copié sur l’objet blob de base. Cette opération de copie génère un instantané supprimé de manière réversible de B1.*

Pour afficher les objets blob et instantanés d’objets blob supprimés de manière réversible, vous pouvez inclure des données supprimées dans la commande **List Blobs**. Vous pouvez afficher uniquement les objets blob de base supprimés de manière réversible ou inclure les instantanés d’objets blob supprimés. Pour toutes les données supprimées de manière réversible, vous pouvez afficher l’heure de leur suppression, ainsi que le nombre de jours avant leur expiration définitive.

### <a name="example"></a>Exemple

Voici la sortie de console d’un script .NET qui charge, remplace, crée un instantané, supprime et restaure un objet blob nommé *HelloWorld* quand la suppression réversible est activée :

```bash
Upload:
- HelloWorld (is soft deleted: False, is snapshot: False)

Overwrite:
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Snapshot:
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Delete (including snapshots):
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: False)

Undelete:
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Copy a snapshot over the base blob:
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)
```

Pour obtenir un pointeur vers l’application qui a produit cette sortie, voir la section [Étapes suivantes](#next-steps).

## <a name="pricing-and-billing"></a>Tarification et facturation

Toutes les données supprimées de manière réversible sont facturées au même tarif que des données actives. Vous ne serez pas facturé pour des données supprimées définitivement à l’issue de la période de rétention configurée. Pour plus de détails sur les instantanés et la manière dont ils augmentent les coûts, voir [Présentation des frais liés aux instantanés](./snapshots-overview.md).

Vous ne serez pas facturé pour les transactions associées à la génération automatique d’instantanés. Les transactions **Undelete Blob** vous seront facturées au taux des opérations d’écriture.

Pour plus de détails sur les prix du Stockage Blob Azure en général, voir la [page relative aux tarifs de Stockage Blob Azure](https://azure.microsoft.com/pricing/details/storage/blobs/).

Lorsque vous activez initialement la suppression réversible, Microsoft vous recommande d’utiliser une courte période de rétention pour mieux comprendre l’impact de cette fonctionnalité sur votre facture.

L’activation de la suppression réversible pour les données fréquemment remplacées peut entraîner une augmentation des frais en termes de capacité de stockage, ainsi qu’une latence plus élevée lors de la classification des objets blob. Vous pouvez réduire ce coût et cette latence supplémentaires en stockant les données fréquemment remplacées dans un compte de stockage distinct où la suppression réversible est désactivée.

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="can-i-use-the-set-blob-tier-api-to-tier-blobs-with-soft-deleted-snapshots"></a>Puis-je utiliser l’API de définition d’un niveau d’objet blob pour différencier les objets blob des instantanés supprimés de manière réversible ?

Oui. Les instantanés supprimés de manière réversible restent dans le niveau d’origine, mais l’objet blob de base est déplacé dans le nouveau niveau.

### <a name="premium-storage-accounts-have-a-per-blob-snapshot-limit-of-100-do-soft-deleted-snapshots-count-toward-this-limit"></a>Les comptes de stockage Premium sont limités à 100 instantanés par objet blob. Les instantanés d’objets blob supprimés de manière réversible sont-ils pris en compte par rapport à cette limite ?

Non, ils ne le sont pas.

### <a name="if-i-delete-an-entire-account-or-container-with-soft-delete-turned-on-will-all-associated-blobs-be-saved"></a>Si je supprime un compte ou un conteneur entier alors que la suppression réversible est activée, les objets blob associés sont-ils tous enregistrés ?

Non, si vous supprimez un compte ou un conteneur entiers, tous les objets blob associés sont supprimés définitivement. Pour plus d’informations sur la protection d’un compte de stockage contre les suppressions accidentelles, consultez [Verrouiller les ressources pour empêcher les modifications inattendues](../../azure-resource-manager/management/lock-resources.md).

### <a name="can-i-view-capacity-metrics-for-deleted-data"></a>Puis-je afficher les métriques de capacité de données supprimées ?

Les données supprimées de manière réversible sont incluses dans la capacité totale de votre compte de stockage. Pour plus d’informations sur le suivi et l’analyse de la capacité de stockage, consultez [Storage Analytics](../common/storage-analytics.md).

### <a name="can-i-read-and-copy-out-soft-deleted-snapshots-of-my-blob"></a>Puis-je lire et copier les instantanés supprimés de manière réversible de mon objet blob ?  

Oui, mais vous devez commencer par appeler la commande Undelete sur l’objet blob.

### <a name="is-soft-delete-available-for-virtual-machine-disks"></a>La suppression réversible est-elle disponible pour les disques de machine virtuelle ?  

La suppression réversible est disponible pour les disques non managés Standard et Premium, qui sont des objets blob de pages en arrière-plan. La suppression réversible vous permettra de récupérer uniquement des données supprimées par les opérations **Delete Blob**, **Put Blob**, **Put Block List**, et **Copy Blob**. Les données remplacées par un appel de la commande **Put Page** ne sont pas récupérables.

Étant donné qu’une machine virtuelle Azure écrit sur un disque non managé à l'aide d'appels à **Put Page**, la suppression réversible pour annuler des écritures sur un disque non managé à partir d'une machine virtuelle Azure n'est pas un scénario pris en charge.

### <a name="do-i-need-to-change-my-existing-applications-to-use-soft-delete"></a>Dois-je modifier mes applications existantes pour utiliser la suppression réversible ?

Vous pouvez tirer parti de la suppression réversible, quelle que soit la version d’API que vous utilisez. Toutefois, pour répertorier et récupérer des objets blob et instantanés d’objets blob supprimés de manière réversible, vous devez utiliser la version 2017-07-29 de l’[API REST Stockage Azure](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services) ou une version supérieure. Microsoft recommande de toujours utiliser la dernière version de l’API de stockage Azure.

## <a name="next-steps"></a>Étapes suivantes

- [Activer la suppression réversible pour les objets blob](./soft-delete-blob-enable.md)
- [Contrôle de version des blobs](versioning-overview.md)