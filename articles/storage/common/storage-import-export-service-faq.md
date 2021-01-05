---
title: Questions fréquentes (FAQ) sur le service Azure Import/Export | Microsoft Docs
description: Lisez les réponses aux Questions fréquentes (FAQ) sur le service Azure Import/Export.
author: alkohli
services: storage
ms.service: storage
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: 17a29621d3d70d56745d2e51e0ce80974876a9ef
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92782159"
---
# <a name="azure-importexport-service-frequently-asked-questions"></a>Service Azure Import/Export : Questions fréquentes (FAQ)

Vous trouverez ci-dessous des questions et réponses relatives à l’utilisation du service Azure Import/Export pour le transfert de données vers le Stockage Azure. Les questions/réponses sont classées dans les catégories suivantes :

- À propos du service Import/Export
- Préparation des disques pour l’importation/exportation
- Tâches d’importation/exportation
- Expédition des disques
- Divers

## <a name="about-importexport-service"></a>À propos du service Import/Export

### <a name="can-i-copy-azure-file-storage-using-the-azure-importexport-service"></a>Puis-je copier des données vers Stockage Fichier Azure à l’aide du service Azure Import/Export ?

Oui. Le service Azure Import/Export prend en charge l’importation de données dans Stockage Fichier Azure. L’exportation de fichiers Azure n’est pas prise en charge pour l’instant.

### <a name="is-the-azure-importexport-service-available-for-csp-subscriptions"></a>Le service Azure Import/Export est-il disponible pour les abonnements aux fournisseurs de solutions Cloud ?

Oui. Le service Azure Import/Export prend en charge les abonnements aux fournisseurs de solutions Cloud.

### <a name="can-i-use-the-azure-importexport-service-to-copy-pst-mailboxes-and-sharepoint-data-to-microsoft-365"></a>Puis-je utiliser le service Azure Import/Export pour copier des boîtes aux lettres PST et des données SharePoint dans Microsoft 365 ?

Oui. Pour plus d’informations, accédez à [Vue d’ensemble de l’importation des fichiers PST de votre organisation](/microsoft-365/compliance/importing-pst-files-to-office-365).

### <a name="can-i-use-the-azure-importexport-service-to-copy-my-backups-offline-to-the-azure-backup-service"></a>Puis-je utiliser le service Azure Import/Export pour copier mes sauvegardes en mode hors connexion vers le service Sauvegarde Azure ?

Oui. Pour plus d’informations, accédez à [Flux de travail de la sauvegarde hors connexion dans Sauvegarde Azure](../../backup/backup-azure-backup-import-export.md).

### <a name="can-i-purchase-drives-for-importexport-jobs-from-microsoft"></a>Puis-je acheter des disques auprès de Microsoft pour les tâches d’importation/exportation ?

Non. Vous devez expédier vos propres disques pour les tâches d’importation et d’exportation.

## <a name="preparing-disks-for-importexport"></a>Préparation des disques pour l’importation/exportation

### <a name="can-i-skip-the-drive-preparation-step-for-an-import-job-can-i-prepare-a-drive-without-copying"></a>Puis-je ignorer l’étape de préparation de disque pour une tâche d’importation ? Puis-je préparer un disque sans effectuer de copie ?

Non. Tout disque utilisé pour importer des données doit être préparé à l’aide de l’outil Azure WAImportExport. Utilisez l’outil pour copier également les données sur le disque.

### <a name="do-i-need-to-perform-any-disk-preparation-when-creating-an-export-job"></a>Dois-je effectuer une préparation du disque durant la création d’une tâche d’exportation ?

Non. Certaines vérifications préalables sont recommandées. Pour vérifier le nombre de disques nécessaires, utilisez la commande PreviewExport de l’outil WAImportExport. Pour plus d’informations, consultez [Aperçu de l’utilisation des lecteurs pour un travail d’exportation](/previous-versions/azure/storage/common/storage-import-export-tool-previewing-drive-usage-export-v1). La commande vous permet d’afficher un aperçu de l’utilisation des disques pour les objets blob sélectionnés, en fonction de la taille des disques à utiliser. Vérifiez également que vous pouvez accéder en lecture/écriture au disque dur expédié pour la tâche d’exportation.

## <a name="importexport-jobs"></a>Tâches d’importation/exportation

### <a name="can-i-cancel-my-job"></a>Puis-je annuler ma tâche ?

Oui. Vous pouvez annuler une tâche dont l’état est **Création** ou **Expédition** . Au-delà de ces étapes, la tâche ne peut être annulée. Elle se poursuit jusqu’à l’étape finale.

### <a name="how-long-can-i-view-the-status-of-completed-jobs-in-the-azure-portal"></a>Pendant combien de temps puis-je voir l’état des tâches effectuées dans le Portail Azure ?

L’état des travaux terminés est visible pendant 90 jours. Les tâches effectuées sont supprimées après 90 jours.

### <a name="if-i-want-to-import-or-export-more-than-10-drives-what-should-i-do"></a>Si je souhaite importer ou exporter plus de 10 disques, que dois-je faire ?

Une tâche d’importation ou d’exportation peut référencer uniquement 10 disques dans une seule tâche. Pour expédier plus de 10 disques, vous devez créer plusieurs tâches. Les disques associés à la même tâche doivent être expédiés ensemble dans le même colis.
Pour plus d’informations et pour obtenir de l’aide quand la capacité de données s’étend sur plusieurs tâches d’importation de disques, contactez l’assistance Microsoft.

### <a name="the-uploaded-blob-shows-status-as-lease-expired-what-should-i-do"></a>L’objet blob chargé affiche l’état « Bail expiré ». Que dois-je faire ?

Vous pouvez ignorer le champ « Bail expiré ». Import/Export prend le bail sur l’objet blob lors du chargement afin de s’assurer qu’aucun autre processus ne peut mettre à jour l’objet blob en parallèle. Bail expiré signifie qu’Import/export ne charge plus sur cet objet et que vous pouvez l’utiliser.

## <a name="shipping-disks"></a>Expédition des disques

### <a name="what-is-the-maximum-number-of-hdd-for-in-one-shipment"></a>Quel est le nombre maximal de disques durs par envoi ?

Il n’existe aucune limite au nombre de disques durs dans un envoi. Si les disques font partie de plusieurs tâches, nous vous recommandons de procéder comme suit :

- Étiquetez les disques avec les noms de tâches correspondants.
- Mettez à jour les tâches avec un numéro de suivi auquel vous ajoutez -1, -2, etc.

### <a name="should-i-include-anything-other-than-the-hdd-in-my-package"></a>Dois-je ajouter autre chose que le disque dur dans mon colis ?

Mettez uniquement vos disques durs dans le colis à expédier. N’incluez pas d’accessoires tels que des câbles d’alimentation ou USB.

### <a name="do-i-have-to-ship-my-drives-using-fedex-or-dhl"></a>Dois-je expédier mes disques par FedEx ou DHL ?

Vous pouvez expédier des disques au centre de données Azure en faisant appel à n’importe quel transporteur connu, par exemple FedEx, DHL, UPS ou La Poste. Toutefois, pour la réexpédition de disques depuis le centre de données, vous devez fournir :

- Un numéro de compte FedEx aux États-Unis et dans l’UE, ou
- Un numéro de compte DHL en Asie et en Australie

> [!NOTE]
> Les centres de données en Inde exigent une lettre de déclaration sur papier à en-tête au nom de votre organisation (bon de livraison connu sous le nom de « challan ») pour le renvoi des disques. Pour obtenir le laissez-passer obligatoire, vous devez aussi organiser l’enlèvement avec le transporteur choisi et partager les détails avec le centre de données.

### <a name="are-there-any-restrictions-with-shipping-and-returning-my-drive-internationally"></a>Existe-t-il des restrictions concernant l’expédition et le retour de mon disque à l’étranger ?

Notez que le support physique que vous expédiez devra peut-être franchir des frontières. Vous êtes responsable de l'application des lois applicables lorsque vous importez et/ou exportez vos données et supports physiques. Avant d'expédier le support physique, demandez à vos conseillers juridiques de vérifier que vos supports multimédias et données peuvent être envoyés légalement vers le centre de données identifié. Cela vous assurera d'atteindre Microsoft dans les délais.

Une fois le téléchargement terminé, le processus de renvoi d’un ou plusieurs lecteurs à une adresse internationale peut prendre plus de temps que les 2-3 jours nécessaires à une livraison locale. Au cours de la phase indiquée dans le portail Azure par Packaging, l’équipe Data Box vérifie que la documentation appropriée est fournie pour s'assurer que l'expédition est conforme aux différentes exigences internationales en matière d’importation et d’exportation.

### <a name="are-there-any-special-requirements-for-delivering-my-disks-to-a-datacenter"></a>Y a-t-il des exigences particulières à respecter pour l’envoi de disques à un centre de données ?

Certains centres de données Azure ont des restrictions spécifiques.

- Sur certains sites, tels que l'Australie, l'Allemagne et le Royaume-Uni Sud, le numéro d’identification entrant du centre de données Microsoft doit figurer sur le colis pour des raisons de sécurité. Avant d’expédier vos lecteurs ou disques au centre de données, contactez l’équipe des opérations Azure DataBox (adbops@microsoft.com) pour obtenir ce numéro. Sans ce numéro, le colis sera refusé.
- En Inde, les centres de données exigent les détails personnels du chauffeur, comme le numéro de la carte d’identité ou d’un autre document d’identité (par exemple, PAN, AADHAR, DL), le nom, les coordonnées et le numéro de plaque d’immatriculation du véhicule. Ces informations font office de laissez-passer. Pour éviter tout retard de livraison, portez ces exigences à la connaissance de votre transporteur.

### <a name="when-creating-a-job-the-shipping-address-is-a-location-that-is-different-from-my-storage-account-location-what-should-i-do"></a>Au moment de créer une tâche, l’adresse de livraison ne correspond pas à l’emplacement associé à mon compte de stockage. Que dois-je faire ?

Certains emplacements de compte de stockage sont associés à d’autres emplacements d’expédition. Les emplacements d’expédition auparavant disponibles peuvent être temporairement associés à d’autres emplacements. Vérifiez toujours l’adresse d’expédition fournie lors de la création du travail avant d’expédier vos disques.

### <a name="when-shipping-my-drive-the-carrier-asks-for-the-data-center-contact-address-and-phone-number-what-should-i-provide"></a>Au moment d’expédier mon disque, le transporteur me demande l’adresse de contact et le numéro de téléphone du centre de données. Que dois-je indiquer ?

Le numéro de téléphone et l’adresse du centre de données sont fournis dans le cadre de la création de la tâche.

## <a name="miscellaneous"></a>Divers

### <a name="what-happens-if-i-accidentally-send-an-hdd-that-does-not-conform-to-the-supported-requirements"></a>Que se passe-t-il si j’envoie par erreur un disque dur non conforme aux exigences prises en charge ?

Le centre de données Azure vous renvoie le lecteur non conforme au type pris en charge. Si seule une partie des lecteurs contenus dans le colis respecte la configuration requise, ceux-ci sont traités ; ceux qui ne la respectent vous sont renvoyés.

### <a name="does-the-service-format-the-drives-before-returning-them"></a>Est-ce que le service formate les disques avant de les retourner ?

Non. Tous les disques sont chiffrés avec BitLocker.

### <a name="how-can-i-access-data-that-is-imported-by-this-service"></a>Comment accéder aux données importées par ce service ?

Utilisez le Portail Azure ou [Explorateur Stockage](../../vs-azure-tools-storage-manage-with-storage-explorer.md) pour accéder aux données de votre compte de stockage Azure.  

### <a name="after-the-import-is-complete-what-does-my-data-look-like-in-the-storage-account-is-my-directory-hierarchy-preserved"></a>Une fois l’importation effectuée, à quoi ressemblent mes données dans le compte de stockage ? La hiérarchie de mon répertoire est-elle conservée ?

Quand vous préparez un disque dur pour une tâche d’importation, la destination est spécifiée par le champ DstBlobPathOrPrefix du fichier CSV du jeu de données. Il s’agit du conteneur cible dans le compte de stockage dans lequel les données du disque dur sont copiées. Dans ce conteneur cible, des répertoires virtuels sont créés pour les dossiers du disque dur et des objets blob sont créés pour les fichiers.

### <a name="if-a-drive-has-files-that-already-exist-in-my-storage-account-does-the-service-overwrite-existing-blobs-or-files"></a>Si un disque contient des fichiers qui existent déjà dans mon compte de stockage, le service remplace-t-il les objets blob ou les fichiers existants ?

Cela dépend. Lors de la préparation du disque, vous pouvez indiquer si les fichiers cibles doivent être remplacés ou ignorés à l’aide du champ Disposition:< rename|no-overwrite|overwrite> du fichier CSV du jeu de données. Par défaut, le service renomme les nouveaux fichiers au lieu de remplacer les objets blob ou les fichiers existants.

### <a name="is-the-waimportexport-tool-compatible-with-32-bit-operating-systems"></a>L’outil WAImportExport est-il compatible avec les systèmes d’exploitation 32 bits ?

Non. L’outil WAImportExport est compatible uniquement avec les systèmes d’exploitation Windows 64 bits. Pour obtenir une liste complète des systèmes d’exploitation pris en charge, accédez à [Systèmes d’exploitation pris en charge](./storage-import-export-requirements.md).

### <a name="what-is-the-maximum-block-blob-and-page-blob-size-supported-by-azure-importexport"></a>Quelle est la taille maximale des objets blob de blocs et des objets blob de pages prise en charge par Azure Import/Export ?

- La taille maximale des objets blob de blocs est d’environ 4,768 To ou 5 000 000 Mo.
- La taille maximale des objets blob de pages est de 8 To.

### <a name="does-azure-importexport-support-aes-256-encryption"></a>Azure Import/Export prend-il en charge le chiffrement AES-256 ?

Oui. Le service Azure Import/Export utilise le chiffrement BitLocker AES-256.

## <a name="next-steps"></a>Étapes suivantes

* [Qu’est-ce qu’Azure Import/Export ?](storage-import-export-service.md)