---
title: Corriger les erreurs du fournisseur de ressources Azure NetApp Files | Microsoft Docs
description: Cet article décrit les causes, solutions et contournements à employer pour résoudre les erreurs courantes du fournisseur de ressources Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
tags: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 10/18/2019
ms.author: b-juche
ms.openlocfilehash: 870caffe2bd286c2eec3390915bc5e64e0103a07
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85483463"
---
# <a name="troubleshoot-azure-netapp-files-resource-provider-errors"></a>Corriger les erreurs du fournisseur de ressources Azure NetApp Files 

Cet article décrit les erreurs courantes du fournisseur de ressources Azure NetApp Files, ainsi que leurs causes. Il présente des solutions et divers contournements possibles pour résoudre ces erreurs (le cas échant).

## <a name="common-azure-netapp-files-resource-provider-errors"></a>Erreurs courantes du fournisseur de ressources Azure NetApp Files

***Impossible de modifier BareMetalTenantId.***  

Cette erreur se produit quand vous essayez de mettre à jour ou de corriger un volume et que la valeur de la propriété `BaremetalTenantId` a été modifiée.

* Cause :   
Vous essayez de mettre à jour un volume et la propriété `BaremetalTenantId` a une valeur différente de celle stockée dans Azure.
* Solution :   
N’incluez pas `BaremetalTenantId` dans la requête (put) de correctif et de mise à jour. Vous pouvez également vérifier que `BaremetalTenantId` est identique dans la requête.

***Impossible de modifier ServiceLevel.***  

Cette erreur se produit quand vous essayez de mettre à jour ou de corriger un pool de capacité avec un niveau de service différent quand le pool de capacité contient déjà des volumes.

* Cause :   
Vous essayez de mettre à jour un niveau de service de pool de capacité quand le pool contient des volumes.
* Solution :   
Supprimez tous les volumes du pool de capacité, puis modifiez le niveau de service.
* Solution de contournement :   
Créez un autre pool de capacité, puis recréez les volumes dans le nouveau pool de capacité.

***Impossible de modifier PoolId***  

Cette erreur se produit quand vous essayez de mettre à jour ou de corriger un pool de capacité avec une propriété `PoolId` modifiée.

* Cause :   
Vous essayez de mettre à jour la propriété `PoolId` d’un pool de capacité. La propriété `PoolId` est en lecture seule et ne peut pas être modifiée.
* Solution :   
N’incluez pas `PoolId` dans la requête (put) de correctif et de mise à jour.  Vous pouvez également vérifier que `PoolId` est identique dans la requête.

***Impossible de modifier CreationToken.***

Cette erreur se produit quand vous essayez de modifier le chemin de fichier (`CreationToken`) après la création du volume. Le chemin de fichier (`CreationToken`) doit être défini lors de la création du volume et ne peut pas être modifié ultérieurement.

* Cause :   
Vous essayez de modifier le chemin de fichier (`CreationToken`) après la création du volume. Cette opération n’est pas prise en charge. 
* Solution :   
Si vous n’avez pas besoin de modifier le chemin de fichier, supprimez le paramètre de la requête pour ignorer le message d’erreur.
* Solution de contournement :   
Si vous devez modifier le chemin de fichier (`CreationToken`), vous pouvez créer un volume avec un nouveau chemin de fichier, puis migrer les données vers ce nouveau volume.

***CreationToken doit comporter au moins 16 caractères.***

Cette erreur se produit quand le chemin de fichier (`CreationToken`) ne répond pas à la longueur requise. La longueur du chemin de fichier doit être d’au moins un caractère.

* Cause :   
Le chemin de fichier est vide.  Lorsque vous créez un volume à l’aide de l’API, un jeton de création est requis. Si vous utilisez le portail Azure, le chemin de fichier est généré automatiquement.
* Solution :   
Entrez au moins un caractère en tant que chemin de fichier (`CreationToken`).

***Impossible de modifier le nom de domaine.***

Cette erreur se produit quand vous essayez de modifier le nom de domaine dans Active Directory.

* Cause :   
Vous essayez de mettre à jour la propriété de nom de domaine.
* Solution :    
Aucun. Vous ne pouvez pas modifier le nom de domaine.
* Solution de contournement :   
Supprimez tous les volumes à l’aide de la configuration Active Directory. Ensuite, supprimez la configuration Active Directory et recréez les volumes.

***Erreur de valeur dupliquée pour l’objet ExportPolicy.Rules[resourceTypeName].***

Cette erreur se produit lorsque la stratégie d’exportation n’est pas définie avec un index unique. Lorsque vous définissez des stratégies d’exportation, toutes les règles de stratégie d’exportation doivent avoir un index unique compris entre 1 et 5.

* Cause :   
La stratégie d’exportation définie ne respecte pas la configuration requise pour les règles de stratégie d’exportation. Vous devez avoir au moins une règle de stratégie d’exportation et jusqu’à cinq règles de stratégie d’exportation au maximum.
* Solution :   
Assurez-vous que l’index n’est pas déjà utilisé et qu’il se trouve dans la plage de 1 à 5.
* Solution de contournement :   
Utilisez un autre index pour la règle que vous essayez de définir.

***Erreur {action} {resourceTypeName}***

Cette erreur s’affiche quand d’autres opérations de gestion des erreurs n’ont pas réussi à gérer l’erreur lors de l’exécution d’une action sur une ressource.   Elle comprend le texte « Erreur ». `{action}` peut être `getting`, `creating`, `updating` ou `deleting`.  `{resourceTypeName}` est le `resourceTypeName` (par exemple, `netAppAccount`, `capacityPool`, `volume`, et ainsi de suite).

* Cause :   
Cette erreur est une exception non gérée dont la cause n’est pas connue.
* Solution :   
Contactez le centre d’aide et de support Azure pour signaler la raison détaillée mentionnée dans les journaux.
* Solution de contournement :   
Aucun.

***Le nom du chemin du fichier peut contenir seulement des lettres, des chiffres et des traits d’union (« - »).***

Cette erreur se produit quand le chemin du fichier contient des caractères qui ne sont pas pris en charge. Il peut s’agir d’un point (« . »), d’une virgule (« , »), d’un trait de soulignement (« _ »), ou du signe « dollar » (« $ »).

* Cause :   
Le chemin du fichier contient des caractères qui ne sont pas pris en charge. Il peut s’agir d’un point (« . »), d’une virgule (« , »), d’un trait de soulignement (« _ »), ou du signe « dollar » (« $ »).
* Solution :   
Supprimez du chemin d’accès du fichier les caractères qui ne sont pas des caractères alphabétiques, des nombres ou des traits d’union («- »).
* Solution de contournement :   
Vous pouvez remplacer un trait de soulignement par un trait d’union ou utiliser des majuscules plutôt que des espaces pour indiquer le début de nouveaux mots.  Par exemple, utilisez « NouveauVolume » au lieu de « nouveau volume ».

***Impossible de modifier FileSystemId.***

Cette erreur se produit quand vous essayez de modifier `FileSystemId`.  La modification de `FileSystemdId` n’est pas une opération prise en charge. 

* Cause :   
L’ID du système de fichiers est défini lors de la création du volume. `FileSystemId` ne peut pas être modifié par la suite.
* Solution :   
N’incluez pas `FileSystemId` dans une requête (put) de correctif et de mise à jour.  Vous pouvez également vérifier que `FileSystemId` est identique dans la requête.

***ActiveDirectory avec ID : « {string} » n’existe pas.***

La partie `{string}` est la valeur que vous avez entrée dans la propriété `ActiveDirectoryId` pour la connexion Active Directory.

* Cause :   
Quand vous avez créé un compte avec la configuration Active Directory, vous avez entré une valeur pour `ActiveDirectoryId` qui est censée être vide.
* Solution :   
N’incluez pas `ActiveDirectoryId` dans la requête (put) de création.

***api-version non valide.***

La version de l’API n’est pas envoyée ou contient une valeur non valide.

* Cause :   
La valeur du paramètre de requête `api-version` contient une valeur non valide.
* Solution :   
Utilisez la valeur de version d’API correcte.  Le fournisseur de ressources prend en charge de nombreuses versions d’API. La valeur est au format aaaa-mm-jj.

***Une valeur « {value} » non valide a été reçue pour {1}.***

Ce message indique une erreur dans les champs pour `RuleIndex`, `AllowedClients`, `UnixReadOnly`, `UnixReadWrite`, `Nfsv3` et `Nfsv4`.

* Cause :   
La requête de validation d’entrée a échoué pour au moins un des champs suivants : `RuleIndex`, `AllowedClients`, `UnixReadOnly`, `UnixReadWrite`, `Nfsv`3 et `Nfsv4`.
* Solution :   
Veillez à définir tous les paramètres obligatoires et non conflictuels sur la ligne de commande. Par exemple, vous ne pouvez pas définir les paramètres `UnixReadOnly` et `UnixReadWrite` en même temps.
* Solution de contournement :   
Consultez la solution ci-dessus.

***La plage IP {0} à {1} pour le réseau local virtuel {2} est déjà utilisée***

Cette erreur se produit car les enregistrements internes des plages d’adresses IP utilisées sont en conflit avec l’adresse IP qui vient d’être attribuée.

* Cause :   
L’adresse IP affectée à la création du volume est déjà inscrite.
La raison peut être l’échec antérieur de création d’un volume.
* Solution :   
Contactez le centre de support Azure.

***Valeur manquante pour « {property} ».***

Cette erreur indique qu’une propriété requise est absente de la requête. La chaîne {property} contient le nom de la propriété manquante.

* Cause :   
La requête de validation d’entrée a échoué pour au moins l’une des propriétés.
* Solution :   
Veillez à définir toutes les propriétés obligatoires et non conflictuelles dans la requête, en particulier la propriété mentionnée dans le message d’erreur.

***Impossible de modifier MountTargets.***

Cette erreur se produit quand un utilisateur tente de mettre à jour ou de corriger la propriété MountTargets d’un volume.

* Cause :   
Vous essayez de mettre à jour la propriété `MountTargets` du volume. La modification de cette propriété n’est pas prise en charge.
* Solution :   
N’incluez pas `MountTargets` dans une requête (put) de correctif et de mise à jour.  Vous pouvez également vérifier que `MountTargets` est identique dans la requête.

***Nom déjà utilisé.***

Cette erreur indique que le nom de la ressource est déjà utilisé.

* Cause :   
Vous essayez de créer une ressource avec un nom qui est utilisé pour une ressource existante.
* Solution :   
Utilisez un nom unique lors de la création de la ressource.

***Chemin de fichier déjà utilisé.***

Cette erreur indique que le chemin de fichier du volume est déjà utilisé.

* Cause :   
Vous essayez de créer un volume avec un chemin de fichier identique à celui d’un volume existant.
* Solution :   
Utilisez un chemin de fichier unique lors de la création du volume.

***Nom trop long.***

Cette erreur indique que le nom de la ressource ne répond pas à la longueur maximale requise.

* Cause :   
Le nom de la ressource est trop long.
* Solution :   
Utilisez un nom plus court pour la ressource.

***Chemin de fichier trop long.***

Cette erreur indique que le chemin de fichier du volume ne répond pas à la longueur maximale requise.

* Cause :   
Le chemin de fichier du volume est trop long.
* Solution :   
Utilisez un chemin de fichier plus court.

***Nom trop court.***

Cette erreur indique que le nom de la ressource ne répond pas à la longueur minimale requise.

* Cause :   
Le nom de la ressource est trop court.
* Solution :   
Utilisez un nom plus long pour la ressource.

***Chemin de fichier trop court.***

Cette erreur indique que le chemin de fichier du volume ne répond pas à la longueur minimale requise.

* Cause :   
Le chemin de fichier du volume est trop court.
* Solution :   
Augmentez la longueur du chemin de fichier de volume.

***API Azure NetApp Files inaccessible.***

L’API Azure s’appuie sur l’API Azure NetApp Files pour gérer les volumes. Cette erreur indique un problème avec la connexion d’API.

* Cause :   
L’API sous-jacente ne répond pas, ce qui se traduit par une erreur interne. Cette erreur est généralement temporaire.
* Solution :   
Ce problème est généralement temporaire. La requête devrait réussir après un certain temps.
* Solution de contournement :   
Aucun. L’API sous-jacente est essentielle pour la gestion des volumes.

***Aucun ID de résultat d’opération trouvé pour « {0} ».***

Cette erreur indique qu’une erreur interne empêche l’opération de se terminer.

* Cause :   
Une erreur interne s’est produite et a empêché l’opération de se terminer.
* Solution :   
Cette erreur est généralement temporaire. Patientez quelques minutes, puis réessayez. Si le problème persiste, créez un ticket de support technique pour étudier le problème.
* Solution de contournement :   
Patientez quelques minutes et vérifiez si le problème persiste.

***Vous n’êtes pas autorisé à combiner les types de protocole CIFS et NFS***

Cette erreur se produit quand vous essayez de créer un volume et que les propriétés du volume contiennent à la fois les types de protocole CIFS (SMB) et NFS.

* Cause :   
Les types de protocole CIFS (SMB) et NFS sont tous les deux utilisés dans les propriétés du volume.
* Solution :   
Supprimez l’un des types de protocole.
* Solution de contournement :   
Laissez la propriété de type de protocole vide ou Null.

***Nombre d’éléments : {value} pour l’objet : ExportPolicy.Rules[RuleIndex] est en dehors de la plage min-max.***

Cette erreur se produit lorsque les règles de stratégie d’exportation ne répondent pas à la spécification d’une plage minimum ou maximum. Si vous définissez la stratégie d’exportation, elle doit avoir au moins une règle de stratégie d’exportation et jusqu’à cinq règles de stratégie d’exportation au maximum.

* Cause :   
Vous avez défini une stratégie d’exportation qui ne respecte pas la plage requise.
* Solution :   
Assurez-vous que l’index n’est pas déjà utilisé et qu’il se trouve dans la plage de 1 à 5.
* Solution de contournement :   
Il n’est pas obligatoire d’utiliser la stratégie d’exportation sur les volumes. Vous pouvez omettre la totalité de la stratégie d’exportation si vous n’avez pas besoin d’utiliser des règles de stratégie d’exportation.

***Un seul répertoire Active Directory est autorisé***

Cette erreur se produit quand vous essayez de créer une configuration Active Directory et qu’il en existe déjà une pour l’abonnement dans la région. Elle peut aussi se produire quand vous essayez de créer plusieurs configurations Active Directory.

* Cause :   
Vous essayez de créer (et non de mettre à jour) un répertoire Active Directory, mais il en existe déjà un.
* Solution :   
Si la configuration Active Directory n’est pas utilisée, vous pouvez d’abord supprimer la configuration existante, puis réessayer l’opération de création.
* Solution de contournement :   
Aucun. Un seul répertoire Active Directory est autorisé.

***Opération « {operation} » non prise en charge.***

Cette erreur indique que l’opération n’est pas disponible pour l’abonnement ou la ressource que vous utilisez.

* Cause :   
L’opération n’est pas disponible pour l’abonnement ou la ressource.
* Solution :   
Vérifiez que l’opération est correcte et disponible pour la ressource et l’abonnement que vous utilisez.

***Impossible de modifier OwnerId***

Cette erreur se produit quand vous essayez de modifier la propriété OwnerId du volume. La modification d’OwnerId n’est pas une opération prise en charge. 

* Cause :   
La propriété `OwnerId` est définie lors de la création du volume. Vous ne pouvez pas la modifier par la suite.
* Solution :   
N’incluez pas `OwnerId` dans une requête (put) de correctif et de mise à jour. Vous pouvez également vérifier que `OwnerId` est identique dans la requête.

***Pool parent introuvable***

Cette erreur se produit quand vous essayez de créer un volume et que le pool de capacité dans lequel vous créez le volume est introuvable.

* Cause :   
Le pool de capacité dans lequel le volume est créé est introuvable.
* Solution :   
Le pool n’a probablement pas été entièrement créé ou a déjà été supprimé au moment de la création du volume.

***L’opération de correction n’est pas prise en charge pour ce type de ressource.***

Cette erreur se produit lorsque vous essayez de modifier la cible de montage ou la capture instantanée.

* Cause :   
La cible de montage est définie lors de sa création. Vous ne pouvez pas la modifier par la suite.
Les instantanés ne contiennent pas de propriétés modifiables.
* Solution :   
Aucun. Ces ressources n’ont pas de propriétés modifiables.

***Taille du pool trop petite pour la taille totale du volume.***

Cette erreur se produit quand vous mettez à jour la taille du pool de capacité et que la taille est inférieure à la valeur `usedBytes` totale de tous les volumes dans ce pool de capacité.  Elle peut également se produire quand vous créez un volume ou redimensionnez un volume existant, et que la nouvelle taille du volume dépasse l’espace libre dans le pool de capacité.

* Cause :   
Vous essayez de mettre à jour le pool de capacité à une taille inférieure à usedBytes dans tous les volumes du pool de capacité.  Ou vous essayez de créer un volume qui est plus grand que l’espace libre dans le pool de capacité.  Ou encore, vous essayez de redimensionner un volume et la nouvelle taille dépasse l’espace libre dans le pool de capacité.
* Solution :   
Affectez à la taille du pool de capacité une valeur supérieure, ou créez un volume plus petit.
* Solution de contournement :   
Supprimez suffisamment de volumes pour que le pool de capacité puisse être mise à jour à cette taille.

***La propriété Emplacement de l’instantané doit être la même que Volume***

Cette erreur se produit quand vous créez un instantané avec un emplacement autre que le volume propriétaire de l’instantané.

* Cause :   
Valeur non valide dans la propriété Emplacement de l’instantané.
* Solution :   
Définissez une chaîne valide dans la propriété Emplacement.

***Le nom {resourceType} doit être identique au nom de l’identificateur de ressource.***

Cette erreur se produit quand vous créez une ressource et que vous affectez à la propriété de nom une valeur autre que la propriété de nom de `resourceId`.

* Cause :   
Valeur non valide dans la propriété de nom lors de la création d’une ressource.
* Solution :   
Laissez la propriété de nom vide ou laissez-la utiliser la même valeur que la propriété de nom (entre la dernière barre oblique inverse « / » et le point d’interrogation « ? ») dans `resourceId`.

***Type de protocole {value} inconnu***

Cette erreur se produit quand vous créez un volume avec un type de protocole inconnu.  Les valeurs valides sont « NFSv3 », « NFSv4 » et « CIFS ».

* Cause :   
Vous essayez de définir une valeur non valide dans la propriété `protocolType` du volume.
* Solution :   
Définissez une chaîne valide dans `protocolType`.
* Solution de contournement :   
Affectez la valeur Null à `protocolType`.

***Les types de protocole ne peuvent pas être modifiés***

Cette erreur se produit quand vous essayez de mettre à jour ou de corriger `ProtocolType` pour un volume.  La modification de ProtocolType n’est pas une opération prise en charge.

* Cause :   
La propriété `ProtocolType` est définie lors de la création du volume.  Vous ne pouvez pas la mettre à jour.
* Solution :   
Aucun.
* Solution de contournement :   
Créez un autre volume avec de nouveaux types de protocole.

***La création de la ressource de type {resourceType} dépasserait le quota de ressources {quota} de type {resourceType} par {parentResourceType}. Le nombre de ressources actuel est {currentCount}. Veuillez supprimer certaines ressources de ce type avant d’en créer une nouvelle.***

Cette erreur se produit quand vous essayez de créer une ressource (`NetAppAccount`, `CapacityPool`, `Volume` ou `Snapshot`), mais que votre quota a atteint sa limite.

* Cause :   
Vous essayez de créer une ressource, mais la limite de quota est atteinte (par exemple : `NetAppAccounts` par abonnement ou `CapacityPools` par `NetAppAccount`).
* Solution :   
Augmentez la limite de quota.
* Solution de contournement :   
Supprimez les ressources inutilisées du même type et recréez-les.

***À reçu une valeur pour la propriété en lecture seule « {propertyName} ».***

Cette erreur se produit lorsque vous définissez une valeur pour une propriété qui ne peut pas être modifiée. Par exemple, vous ne pouvez pas modifier l’ID du volume.

* Cause :   
Vous essayez de modifier un paramètre (par exemple l’ID de volume) qui ne peut pas être modifié.
* Solution :   
Ne modifiez pas la valeur de la propriété.

***La {resource} demandée est introuvable.***

Cette erreur se produit lorsque vous tentez de référencer une ressource qui n’existe pas, par exemple, un volume ou un instantané. La ressource a peut-être été supprimée ou son nom n’est pas correctement saisi.

* Cause :   
Vous tentez de référencer une ressource qui n’existe pas (par exemple un volume ou un instantané), qui a déjà été supprimée ou dont le nom n’a pas été correctement entré.
* Solution :   
Vérifiez si le nom de la ressource est correctement saisi dans la requête.
* Solution de contournement :   
Veuillez consulter la section Solution ci-dessus.

***Le niveau de service « {volumeServiceLevel} » est supérieur au « {poolServiceLevel} » parent***

Cette erreur se produit quand vous créez ou mettez à jour un volume, et que vous avez défini le niveau de service à un niveau supérieur à celui du pool de capacité qui le contient.

* Cause :   
Vous essayez de créer ou de mettre à jour un volume avec un niveau de service plus élevé que celui du pool de capacité parent.
* Solution :   
Définissez le niveau de service à un rang identique ou inférieur à celui du pool de capacité parent.
* Solution de contournement :   
Créez le volume dans un autre pool de capacité avec un niveau de service correct. Vous pouvez également supprimer tous les volumes du pool de capacité, et affectez un rang supérieur au niveau de service du pool de capacité.

***Le nom du serveur SMB ne doit pas comporter plus de 10 caractères.***

Cette erreur se produit quand vous créez ou mettez à jour une configuration Active Directory pour un compte.

* Cause :   
La longueur du nom du serveur SMB dépasse 10 caractères.
* Solution :   
Utilisez un nom de serveur plus court. La longueur maximale est de 10 caractères.
* Solution de contournement :   
Aucun.  Consultez la solution ci-dessus. 

***Impossible de modifier SubnetId***

Cette erreur se produit quand vous essayez de modifier le `subnetId` après la création du volume.  `SubnetId` doit être défini lors de la création du volume. Il ne peut pas être modifié ultérieurement.

* Cause :   
Vous essayez de modifier le `subnetId` après la création du volume. Cette opération n’est pas prise en charge. 
* Solution :   
Si vous n’avez pas besoin de modifier le `subnetId`, supprimez le paramètre de la requête pour ignorer le message d’erreur.
* Solution de contournement :   
Si vous devez modifier le `subnetId`, vous pouvez créer un volume avec un nouveau `subnetId`, puis migrer les données vers ce nouveau volume.

***Le format de SubnetId n’est pas valide.***

Cette erreur se produit quand vous essayez de créer un nouveau volume, mais que le `subnetId` n’est pas un `resourceId` pour un sous-réseau.

* Cause :   
Cette erreur se produit quand vous essayez de créer un nouveau volume, mais que le `subnetId` n’est pas un `resourceId` pour un sous-réseau. 
* Solution :   
Vérifiez la valeur de `subnetId` pour vous assurer qu’elle contient un `resourceId` pour le sous-réseau utilisé.
* Solution de contournement :   
Aucun. Consultez la solution ci-dessus. 

***Le sous-réseau doit avoir une délégation « Microsoft.NetApp/volumes ».***

Cette erreur se produit quand vous créez un volume et que le sous-réseau sélectionné n’est pas délégué à `Microsoft.NetApp/volumes`.

* Cause :   
Vous avez essayé de créer un volume et vous avez sélectionné un sous-réseau qui n’est pas délégué à `Microsoft.NetApp/volumes`.
* Solution :   
Sélectionnez un autre sous-réseau délégué à `Microsoft.NetApp/volumes`.
* Solution de contournement :   
Ajoutez une délégation correcte au sous-réseau.

***Le type de ressource spécifié est inconnu/non applicable.***

Cette erreur se produit quand une vérification du nom a été demandée sur un type de ressource non applicable ou pour un type de ressource inconnu.

* Cause :   
Une vérification du nom a été demandée pour un type de ressource inconnu ou non pris en charge.
* Solution :   
Vérifiez que la ressource pour laquelle vous effectuez la requête est prise en charge ou ne contient aucune faute d’orthographe.
* Solution de contournement :   
Consultez la solution ci-dessus.

***Erreur Azure NetApp Files inconnue.***

L’API Azure s’appuie sur l’API Azure NetApp Files pour gérer les volumes. Cette erreur indique un problème de communication avec l’API.

* Cause :   
L’API sous-jacente envoie une erreur inconnue. Cette erreur est généralement temporaire.
* Solution :   
Ce problème est généralement temporaire. La requête devrait réussir après un certain temps. Si le problème persiste, créez un ticket de support technique pour étudier le problème.
* Solution de contournement :   
Aucun. L’API sous-jacente est essentielle pour la gestion des volumes.

***La valeur reçue « {propertyName} » correspond à une propriété inconnue.***

Cette erreur se produit lorsque des propriétés qui n’existent pas sont fournies pour une ressource, telles qu’un nom de volume, d’instantané ou de cible de montage.

* Cause :   
Une requête est constituée d’un ensemble de propriétés qui peuvent être utilisées avec chaque ressource. Vous ne pouvez pas inclure dans une requête des propriétés qui n’existent pas.
* Solution :   
Vérifiez que tous les noms de propriétés sont correctement orthographiés et que les propriétés sont disponibles dans l’abonnement et la ressource.
* Solution de contournement :   
Réduisez le nombre de propriétés définies dans la requête pour éliminer la propriété ayant provoqué l’erreur.

***L’opération de mise à jour n’est pas prise en charge pour ce type de ressource.***

Seuls les volumes peuvent être mis à jour. Cette erreur se produit lorsque vous essayez d’effectuer une opération de mise à jour non prise en charge, par exemple, en mettant à jour un instantané.

* Cause :   
La ressource que vous voulez mettre à jour ne prend pas en charge l’opération de mise à jour. Seules les propriétés des volumes peuvent être modifiées.
* Solution :   
Aucun. La ressource que vous essayez de mettre à jour ne prend pas en charge l’opération de mise à jour. Il n’est donc pas possible de la modifier.
* Solution de contournement :   
Pour un volume : créez une ressource avec la mise à jour en place, puis migrez les données.

***Le volume ne peut pas être créé dans un pool dont l’état n’est pas Réussite.***

Cette erreur se produit quand vous essayez de créer un volume dans un pool qui n’est pas à l’état Réussite. L’opération de création du pool de capacité a probablement échoué pour une raison quelconque.

* Cause :   
Le pool de capacité contenant le nouveau volume est dans un état d’échec.
* Solution :   
Vérifiez que le pool de capacité est correctement créé et qu’il n’est pas dans un état d’échec.
* Solution de contournement :   
Créez un nouveau pool de capacités et créez le volume dans le nouveau pool.

***Le volume est en cours de création et ne peut pas être supprimé pour le moment.***

Cette erreur se produit quand vous essayez de supprimer un volume qui est toujours en cours de création.

* Cause :   
Un volume est toujours en cours de création au moment où vous essayez de le supprimer.
* Solution :   
Attendez la fin de la création du volume, puis réessayez de le supprimer.
* Solution de contournement :   
Consultez la solution ci-dessus.

***Le volume est en cours de suppression et ne peut pas être supprimé pour le moment.***

Cette erreur se produit quand vous essayez de supprimer un volume alors qu’il est déjà en cours de suppression.

* Cause :   
Un volume est déjà en cours de suppression au moment où vous essayez de le supprimer.
* Solution :   
Attendez que l’opération de suppression en cours soit terminée.
* Solution de contournement :   
Consultez la solution ci-dessus.

***Le volume est en cours de mise à jour et ne peut pas être supprimé pour le moment.***

Cette erreur se produit quand vous essayez de supprimer un volume en cours de mise à jour.

* Cause :   
Un volume est en cours de mise à jour au moment où vous essayez de le supprimer.
* Solution :   
Attendez la fin de l’opération de mise à jour, puis réessayez la suppression.
* Solution de contournement :   
Consultez la solution ci-dessus.

***Le volume est introuvable ou n’a pas été créé correctement.***

Cette erreur se produit quand la création du volume a échoué et que vous essayez de modifier le volume ou de créer un instantané pour le volume.

* Cause :   
Le volume n’existe pas ou la création a échoué.
* Solution :   
Vérifiez que vous modifiez le volume approprié et que la création du volume a réussi. Ou vérifiez que le volume pour lequel vous créez un instantané existe.
* Solution de contournement :   
Aucun.  Consultez la solution ci-dessus. 

***Le jeton de création spécifié existe déjà***

Cette erreur se produit quand vous essayez de créer un volume et que vous spécifiez un jeton de création (chemin d’exportation) pour lequel il existe déjà un volume.

* Cause :   
Le jeton de création (chemin d’exportation) que vous avez spécifié lors de la création du volume est déjà associé à un autre volume. 
* Solution :   
Choisissez un autre jeton de création.  Vous pouvez également supprimer l’autre volume.

***Le jeton de création spécifié est réservé***

Cette erreur se produit quand vous essayez de créer un volume et que vous spécifiez « par défaut » ou « aucun » comme chemin de fichier (jeton de création).

* Cause :    
Vous essayez de créer un volume et que vous spécifiez « par défaut » ou « aucun » comme chemin de fichier (jeton de création).
* Solution :   
Choisissez un autre chemin de fichier (jeton de création).
 
***Les informations d’identification Active Directory sont en cours d’utilisation***

Cette erreur se produit quand vous essayez de supprimer la configuration Active Directory d’un compte où il existe encore au moins un volume SMB.  Le volume SMB a été créé à l’aide de la configuration Active Directory que vous essayez de supprimer.

* Cause :   
Vous essayez de supprimer la configuration Active Directory d’un compte, mais il existe encore au moins un volume SMB qui a été initialement créé à l’aide de la configuration Active Directory. 
* Solution :   
Tout d’abord, supprimez tous les volumes SMB créés à l’aide de la configuration Active Directory.  Ensuite, réessayez de supprimer la configuration.

***Impossible de modifier l’affectation de l’unité d’organisation si les informations d’identification sont utilisées***

Cette erreur se produit quand vous essayez de modifier l’unité d’organisation d’une configuration Active Directory, mais qu’il existe encore au moins un volume SMB.  Le volume SMB a été créé à l’aide de la configuration Active Directory que vous essayez de supprimer.

* Cause :   
Vous essayez de modifier l’unité d’organisation d’une configuration Active Directory,  mais il existe encore au moins un volume SMB qui a été initialement créé à l’aide de la configuration Active Directory.
* Solution :   
 Tout d’abord, supprimez tous les volumes SMB créés à l’aide de la configuration Active Directory.  Ensuite, réessayez de supprimer la configuration. 

***Mise à jour Active Directory déjà en cours***

Cette erreur se produit quand vous essayez de modifier une configuration Active Directory pour laquelle une opération de modification est déjà en cours.

* Cause :   
Vous essayez de modifier une configuration Active Directory, mais une autre opération de modification est déjà en cours.
* Solution :   
Attendez la fin de l’opération de modification en cours d’exécution.

***Supprimez d’abord tous les volumes utilisant les informations d’identification sélectionnées***

Cette erreur se produit quand vous essayez de supprimer une configuration Active Directory alors qu’il existe encore au moins un volume SMB.  Le volume SMB a été créé à l’aide de la configuration Active Directory que vous essayez de supprimer.

* Cause :   
Vous essayez de supprimer une configuration Active Directory, mais il existe encore au moins un volume SMB qui a été initialement créé à l’aide de la configuration Active Directory.
* Solution :   
Tout d’abord, supprimez tous les volumes SMB créés à l’aide de la configuration Active Directory.  Ensuite, réessayez de supprimer la configuration. 

***Aucune information d’identification Active Directory trouvée dans la région***

Cette erreur se produit quand vous essayez de créer un volume SMB mais qu’aucune configuration Active Directory n’a été ajoutée au compte pour la région.

* Cause :   
Vous essayez de créer un volume SMB, mais aucune configuration Active Directory n’a été ajoutée au compte. 
* Solution :   
Ajoutez une configuration Active Directory au compte avant de créer un volume SMB.

***Impossible d’interroger le serveur DNS. Vérifiez que la configuration réseau est correcte et que les serveurs DNS sont disponibles.***

Cette erreur se produit quand vous essayez de créer un volume SMB, mais qu’un serveur DNS (spécifié dans votre configuration Active Directory) est inaccessible. 

* Cause :   
Vous essayez de créer un volume SMB, mais un serveur DNS (spécifié dans votre configuration Active Directory) est inaccessible.
* Solution :   
Passez en revue votre configuration Active Directory et vérifiez que les adresses IP du serveur DNS sont correctes et accessibles.
S’il n’y a aucun problème avec les adresses IP du serveur DNS, vérifiez qu’aucun pare-feu ne bloque l’accès.

***Trop de travaux simultanés***

Cette erreur se produit quand vous essayez de créer un instantané quand trois autres opérations de création d’instantané sont déjà en cours pour l’abonnement.

* Cause :   
Vous essayez de créer un instantané quand trois autres opérations de création d’instantané sont déjà en cours pour l’abonnement. 
* Solution :   
Les travaux de création d’instantané prennent quelques secondes au plus.  Patientez quelques secondes et recommencez l’opération de création de l’instantané.

***Impossible de générer des travaux supplémentaires. Veuillez patienter jusqu’à la fin des travaux en cours, puis réessayez***

Cette erreur peut se produire quand vous essayez de créer ou de supprimer un volume dans des circonstances spécifiques.

* Cause :   
Vous essayez de créer ou de supprimer un volume dans des circonstances spécifiques.
* Solution :   
Attendez environ une minute, puis recommencez l’opération.

***Le volume est déjà en cours de transition d’un état à un autre***

Cette erreur peut se produire quand vous essayez de supprimer un volume qui se trouve actuellement dans un état de transition (c’est-à-dire en cours de création, de mise à jour ou de suppression).

* Cause :   
Vous essayez de supprimer un volume qui est actuellement dans un état de transition.
* Solution :   
Attendez que l’opération en cours d’exécution (transition d’un état à un autre) soit terminée, puis recommencez l’opération.

***Échec du fractionnement du nouveau volume à partir de l’instantané du volume source***

 Cette erreur peut se produire quand vous essayez de créer un volume à partir d’un instantané.  

* Cause :   
Vous essayez de créer un volume à partir d’un instantané et le volume se retrouve dans un état d’erreur.
* Solution :   
Supprimez le volume, puis recommencez l’opération de création du volume à partir de l’instantané.

 
## <a name="next-steps"></a>Étapes suivantes

* [Développer pour Azure NetApp Files avec l’API REST](azure-netapp-files-develop-with-rest-api.md)
