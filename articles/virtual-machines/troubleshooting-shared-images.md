---
title: Résoudre les problèmes liés aux images partagées dans Azure
description: Découvrez comment résoudre les problèmes des galeries d’images partagées.
author: olayemio
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: troubleshooting
ms.workload: infrastructure
ms.date: 10/27/2020
ms.author: olayemio
ms.reviewer: cynthn
ms.openlocfilehash: 74e1dc12be9bfec57d76357a335b46a55912f6df
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500272"
---
# <a name="troubleshoot-shared-image-galleries-in-azure"></a>Résoudre les problèmes liés aux galeries d’images partagées dans Azure

Si vous rencontrez des problèmes en exécutant des opérations sur les galeries d’images partagées, les définitions d’image et les versions d’image, réexécutez la commande ayant échoué en mode débogage. Vous activez le mode débogage à l’aide du commutateur `--debug` dans l’interface CLI et du commutateur `-Debug` dans PowerShell. Après avoir localisé l’erreur, suivez cet article pour la corriger.


## <a name="creating-or-modifying-a-gallery"></a>Création ou modification d’une galerie ##

*Le nom de la galerie n’est pas valide. Les caractères valides sont les caractères alphanumériques anglais, les traits de soulignement et les points autorisés au milieu, jusqu’à 80 caractères au total. Tous les autres caractères, y compris les tirets, sont interdits.*  
**Cause** : le nom de la galerie n’est pas conforme aux exigences d’affectation de noms.  
**Solution de contournement** : choisissez un nom remplissant les conditions suivantes : 
- 80 caractères maximum
- Contient uniquement des lettres latines, des chiffres, des traits de soulignement et des points
- Commence et se termine par des lettres latines ou des chiffres

*Le nom d’entité « galleryName » n’est pas valide selon sa règle de validation : ^[^\_\W][\w-.\_]{0,79}(?<![-.])$.*  
**Cause** : le nom de la galerie n’est pas conforme aux exigences d’affectation de noms.  
**Solution de contournement** : choisissez pour la galerie un nom remplissant les conditions suivantes : 
- 80 caractères maximum
- Contient uniquement des lettres latines, des chiffres, des traits de soulignement et des points
- Commence et se termine par des lettres latines ou des chiffres

*Le nom de ressource fourni, <galleryName\>, contient les caractères de fin suivants : <character\>. Le nom ne peut pas se terminer par les caractères : <character\>*  
**Cause** : le nom de la galerie se termine par un point ou un trait de soulignement.  
**Solution de contournement** : choisissez pour la galerie un nom remplissant les conditions suivantes : 
- 80 caractères maximum
- Contient uniquement des lettres latines, des chiffres, des traits de soulignement et des points
- Commence et se termine par des lettres latines ou des chiffres

*L’emplacement fourni, <region\>, n’est pas disponible pour le type de ressource « Microsoft.Compute/galeries ». La liste des régions disponibles pour le type de ressource est ...*  
**Cause** : la région spécifiée pour la galerie est incorrecte ou requiert une demande d’accès.  
**Solution de contournement** : vérifiez que le nom de la région est correctement orthographié. Vous pouvez exécuter cette commande pour voir les régions auxquelles vous avez accès. Si la région n’est pas dans la liste, envoyez [une demande d’accès](/troubleshoot/azure/general/region-access-request-process).

*Impossible de supprimer une ressource avant la suppression des ressources imbriquées.*  
**Cause** : vous avez tenté de supprimer une galerie contenant au moins une définition d’image existante. Une galerie doit être vide pour qu’il soit possible de la supprimer.  
**Solution de contournement** : supprimez toutes les définitions d’images que contient la galerie, puis supprimez la galerie. Si la définition d’image contient des versions d’images, vous devez supprimer celles-ci avant de supprimer les définitions d’images.

*La ressource <galleryName\> existe déjà à l’emplacement <region\_1\> dans le groupe de ressources <resourceGroup\>. Vous ne pouvez pas créer une ressource du même nom dans l’emplacement <region\_2\>. Sélectionner un nouveau nom de ressource.*  
**Cause** : le groupe de ressources contient une galerie de ce nom, et vous avez tenté d’en créer une autre dans une région différente.  
**Solution de contournement** : utilisez une galerie ou un groupe de ressources différents.

## <a name="creating-or-modifying-image-definitions"></a>Création ou modification de définitions d’images ##

*La modification de la propriété « galleryImage.properties.<property\> » n’est pas autorisée.*  
**Cause** : vous avez tenté de modifier le type de système d’exploitation, l’état du système d’exploitation, la génération Hyper-V, l’offre, l’éditeur ou la référence SKU. La modification de ces propriétés n’est pas autorisée.  
**Solution de contournement** : créez plutôt une nouvelle définition d’image.

*La ressource <galleryName/imageDefinitionName\> existe déjà à l’emplacement <region\_1\> dans le groupe de ressources <resourceGroup\>. Vous ne pouvez pas créer une ressource du même nom dans l’emplacement <region\_2\>. Sélectionner un nouveau nom de ressource.*  
**Cause** : Une définition d’image existante dans la même galerie et dans le même groupe de ressources porte déjà ce nom. Vous avez tenté de créer une autre définition d’image avec le même nom et dans la même galerie, mais dans une autre région.  
**Solution de contournement** : utilisez un nom différent pour la définition d’image ou placez celle-ci dans une galerie ou un groupe de ressources différents.

*Le nom de ressource fourni <galleryName\>/<imageDefinitionName\> contient les caractères de fin suivants : <character\>. Le nom ne peut pas se terminer par les caractères : <character\>*  
**Cause** : le nom <imageDefinitionName\> se termine par un point ou un trait de soulignement.  
**Solution de contournement** : choisissez pour la définition d’image un nom remplissant les conditions suivantes : 
- 80 caractères maximum
- Contient uniquement des lettres latines, des chiffres, des traits de soulignement, des traits d’union et des points
- Commence et se termine par des lettres latines ou des chiffres.

*Le nom d’entité <imageDefinitionName\> n’est pas valide selon sa règle de validation : ^[^\_\\W][\\w-.\_]{0,79}(?<![-.])$"*  
**Cause** : le nom <imageDefinitionName\> se termine par un point ou un trait de soulignement.  
**Solution de contournement** : choisissez pour la définition d’image un nom remplissant les conditions suivantes : 
- 80 caractères maximum
- Contient uniquement des lettres latines, des chiffres, des traits de soulignement, des traits d’union et des points
- Commence et se termine par des lettres latines ou des chiffres

Le *nom de ressource galleryImage.properties.identifier.<property\> n’est pas valide. Il ne peut pas être vide. Les caractères autorisés sont les lettres majuscules ou minuscules, les chiffres, les traits d’union (-), les points (.) et les traits de soulignement (\_). Les noms ne peuvent pas se terminer par un point (.). Leur longueur ne peut pas dépasser le nombre de caractères <number\>.*  
**Cause** : la valeur d’éditeur, d’offre ou de référence SKU n’est pas conforme aux exigences d’affectation des noms.  
**Solution de contournement** : choisissez une valeur remplissant les conditions suivantes : 
- Limitée à 128 caractères pour l’éditeur, ou à 64 caractères pour l’offre et la référence SKU
- Contient uniquement des lettres latines, des chiffres, des traits d’union, des traits de soulignement et des points
- Ne se termine pas par un point

*Impossible d’effectuer l’opération demandée sur la ressource imbriquée. Ressource parente <galleryName\> introuvable.*  
**Cause** : l’abonnement et le groupe de ressources actuels ne contiennent aucune galerie nommée <galleryName\>.  
**Solution de contournement** : vérifiez que les noms de la galerie, de l’abonnement et du groupe de ressources sont corrects. Sinon, créez une galerie nommée <galleryName\>.

*L’emplacement fourni, <region\>, n’est pas disponible pour le type de ressource « Microsoft.Compute/galeries ». La liste des régions disponibles pour le type de ressource est ...*  
**Cause** : le nom <region\> est incorrect ou requiert une demande d’accès.  
**Solution de contournement** : vérifiez que le nom de la région est correctement orthographié. Vous pouvez exécuter cette commande pour voir les régions auxquelles vous avez accès. Si la région n’est pas dans la liste, envoyez [une demande d’accès](/troubleshoot/azure/general/region-access-request-process).

*Impossible de sérialiser la valeur <value\> en tant que type : « iso-8601 »., ISO8601Error : L’indicateur de temps ISO 8601 « T » est manquant. Impossible d’analyser la chaîne DateHeure <value\>* .  
**Cause** : la valeur attribuée à la propriété n’a pas une mise en forme de date correcte.  
**Solution de contournement** : spécifiez une date au format valide aaaa-MM-jj, aaaa-MM-jj'T'HH:mm:sszzz ou [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).

*Impossible de convertir la chaîne en DateTimeOffset : <value\>. Chemin « properties.<property\> »*  
**Cause** : la valeur attribuée à la propriété n’a pas une mise en forme de date correcte.  
**Solution de contournement** : spécifiez une date au format valide aaaa-MM-jj, aaaa-MM-jj'T'HH:mm:sszzz ou [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).

*EndOfLifeDate doit être une date future.*  
**Cause** : la propriété de date de fin de vie n’a pas une mise en forme correcte de date postérieure à la date du jour.  
**Solution de contournement** : spécifiez une date au format valide aaaa-MM-jj, aaaa-MM-jj'T'HH:mm:sszzz ou [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).

*argument --<property\> : valeur int non valide : <value\>* .  
**Cause**: la propriété <property\> accepte uniquement des valeurs entières et <value\> n’est pas un entier.  
**Solution de contournement** : choisissez une valeur entière.

*La valeur minimale de <property\> ne peut pas être supérieure à la valeur maximale de <property\>.*  
**Cause** : La valeur minimale attribuée à <property\> est supérieure à la valeur maximale attribuée à <property\>.  
**Solution de contournement** : modifiez les valeurs de manière à ce que la valeur minimale soit inférieure ou égale à la valeur maximale.

*L’image de galerie : <imageDefinitionName\> identifiée par (éditeur : <Publisher\>, offre : <Offer\>, référence SKU : <SKU\>) existe déjà. Choisissez une autre combinaison d’éditeur, d’offre et de référence SKU.*  
**Cause** : vous avez tenté de créer une définition d’image avec le même triplet éditeur, offre et références SKU qu’une définition d’image existante dans la même galerie.  
**Solution de contournement** : dans une galerie, toutes les définitions d’images doivent avoir une combinaison unique d’éditeur, d’offre et de référence SKU. Choisissez une combinaison unique ou une nouvelle galerie, puis recréez la définition d’image.

*Impossible de supprimer une ressource avant la suppression des ressources imbriquées.*  
**Cause** : vous avez tenté de supprimer une définition d’image contenant des versions d’image. Une définition d’image doit être vide pour qu’il soit possible de la supprimer.  
**Solution de contournement** : supprimez toutes les versions d’image que contient la définition d’image, puis supprimez la définition d’image.

*Impossible de lier le paramètre <property\>. Impossible de convertir la valeur <value\> en type <propertyType\>. Impossible de faire correspondre le nom d’identificateur <value\> à un nom d’énumérateur valide. Spécifiez l’un des noms d’énumérateur suivants, puis réessayez : <choice1\>, <choice2\>, …*  
**Cause** : la propriété a une liste restreinte de valeurs possibles et <value\> n’est pas l’une d’elles.  
**Solution de contournement** : choisissez l’une des valeurs <choice\> possibles.

*Impossible de lier le paramètre <property\>. Impossible de convertir la valeur <value\> en type &quot;System.DateTime&quot;* .  
**Cause** : la valeur attribuée à la propriété n’a pas une mise en forme de date correcte.  
**Solution de contournement** : spécifiez une date au format valide aaaa-MM-jj, aaaa-MM-jj'T'HH:mm:sszzz ou [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).

*Impossible de lier le paramètre <property\>. Impossible de convertir la valeur <value\> en type &quot;System.Int32&quot;* .  
**Cause**: la propriété <property\> accepte uniquement des valeurs entières et <value\> n’est pas un entier.  
**Solution de contournement** : choisissez une valeur entière.

*Le type de compte ZRS n’est pas pris en charge dans cette région.*  
**Cause** : vous avez choisi un stockage redondant de zone standard (ZRS) dans une région qui ne le prend pas encore en charge.  
**Solution de contournement** : modifiez le type de compte de stockage en **Premium\_LRS** ou **Standard\_LRS**. Consultez notre documentation pour obtenir la dernière [liste des régions](../storage/common/storage-redundancy.md#zone-redundant-storage) avec la préversion ZRS activée.

## <a name="creating-or-updating-image-versions"></a>Création ou mise à jour de versions d’images ##

*L’emplacement fourni, <region\>, n’est pas disponible pour le type de ressource « Microsoft.Compute/galeries ». La liste des régions disponibles pour le type de ressource est ...*  
**Cause** : le nom <region\> est incorrect ou requiert une demande d’accès.  
**Solution de contournement** : vérifiez que le nom de la région est correctement orthographié. Vous pouvez exécuter cette commande pour voir les régions auxquelles vous avez accès. Si la région n’est pas dans la liste, envoyez [une demande d’accès](/troubleshoot/azure/general/region-access-request-process).

*Impossible d’effectuer l’opération demandée sur la ressource imbriquée. La ressource parent <galleryName/imageDefinitionName\> est introuvable.*  
**Cause** : l’abonnement et le groupe de ressources actuels ne contiennent aucune galerie nommée <galleryName/imageDefinitionName\>.  
**Solution de contournement** : vérifiez que les noms de la galerie, de l’abonnement et du groupe de ressources sont corrects. Sinon, créez une galerie nommée <galleryName\> et/ou une définition d’image nommée <imageDefinitionName\> dans le groupe de ressources indiqué.

*Impossible de lier le paramètre <property\>. Impossible de convertir la valeur <value\> en type &quot;System.DateTime&quot;* .  
**Cause** : la valeur attribuée à la propriété n’a pas une mise en forme de date correcte.  
**Solution de contournement** : spécifiez une date au format valide aaaa-MM-jj, aaaa-MM-jj'T'HH:mm:sszzz ou [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).

*Impossible de lier le paramètre <property\>. Impossible de convertir la valeur <value\> en type &quot;System.Int32&quot;* .  
**Cause**: la propriété <property\> accepte uniquement des valeurs entières et <value\> n’est pas un entier.  
**Solution de contournement** : choisissez une valeur entière.

*Les régions de profil de publication de version d’image de galerie <publishingRegions\> doivent contenir l’emplacement de version d’image <sourceRegion\>* .  
**Cause** : l’emplacement de l’image source (<sourceRegion\>) doit figurer dans la liste <publishingRegions\>.  
**Solution de contournement** : Incluez <sourceRegion\> dans la liste <publishingRegions\>.

*La valeur <value\> du paramètre <property\> hors limites. La valeur doit être comprise entre les valeurs <minValue\> et <maxValue\> incluses.*  
**Cause** : la valeur <value\> est en dehors de la plage de valeurs possibles pour <property\>.  
**Solution de contournement** : choisissez une valeur comprise dans la plage de valeurs <minValue\> à <maxValue\> incluses.

*La source <resourceID\> est introuvable. Vérifiez que la source existe et qu’elle se trouve dans la même région que la version d’image de galerie en cours de création.*  
**Cause** : il n’existe aucune source située dans <resourceID\>, ou la source dans <resourceID\> n’est pas dans la même région que l’image de galerie en cours de création.  
**Solution de contournement** : vérifiez que la valeur <resourceID\> est correcte et que la région source de la version d’image de galerie est la même que la valeur <resourceID\>.

*La modification de la propriété « galleryImageVersion.properties.storageProfile.<diskImage\>.source.id » n’est pas autorisée.*  
**Cause** : l’ID de source d’une version d’image de galerie ne peut pas être modifié après sa création.  
**Solution de contournement** : assurez-vous que l’ID source est identique à l’ID source existant, modifiez le numéro de la version d’image, ou supprimez la version d’image actuelle, puis réessayez.

*Des numéros d’unité logique (lun) en double ont été détectés dans les disques de données d’entrée. Le numéro de lun doit être unique pour chaque disque de données.*  
**Cause** : lors de la création d’une version d’image à l’aide d’une liste de disques et/ou de captures instantanées de disques, au moins deux disques ou captures instantanées de disque ont le même numéro d’unité logique.  
**Solution de contournement** : supprimez ou modifiez les numéros d’unité logique en double.

*Les ID de source en double se trouvent dans les disques d’entrée. L’ID de source doit être unique pour chaque disque.*  
**Cause** : lors de la création d’une version d’image à l’aide d’une liste de disques et/ou de captures instantanées de disques, au moins deux disques ou captures instantanées de disque ont le même ID de ressource.  
**Solution de contournement** : supprimez ou modifiez tout ID de source de disque en double.

*L’ID de propriété <resourceID\> dans le chemin « properties.storageProfile.<diskImages\>.source.id » n’est pas valide. L’ID de ressource complet attendu commence par « /subscriptions/{subscriptionId} » ou « /providers/{resourceProviderNamespace}/ ».*  
**Cause** : le format de la valeur <resourceID\> n’est pas correct.  
**Solution de contournement** : vérifiez que l’ID de la ressource est correct.

*L’ID de source : <resourceID\> doit être une image managée, une machine virtuelle ou une autre version d’image de galerie*.  
**Cause** : le format de la valeur <resourceID\> n’est pas correct.  
**Solution de contournement** : Si vous utilisez une machine virtuelle, une image managée ou une version d’image de galerie en tant qu’image source, vérifiez que l’ID de ressource de la machine virtuelle, de l’image managée ou de la version d’image de galerie est correct.

*L’ID de source : <resourceID\> doit être un disque ou une capture instantanée managés.*  
**Cause** : le format de la valeur <resourceID\> n’est pas correct.  
**Solution de contournement** : si vous utilisez des disques et/ou des captures instantanées de disque en tant que sources pour la version d’image, vérifiez que les ID de ressource des disques et/ou des captures instantanées de disque sont corrects.

*Impossible de créer la version d’image de galerie à partir de : <resourceID\> parce que l’état du système d’exploitation dans l’image de galerie parente (<OsState\_1\>) n’est pas <OsState\_2\>.*  
**Cause** : l’état de système d’exploitation (généralisé ou spécialisé) ne correspond pas à celui spécifié dans la définition d’image.  
**Solution de contournement** : choisissez une source basée sur une machine virtuelle avec l’état de système d’exploitation <OsState\_1\>, ou créez une définition d’image pour machines virtuelles basée sur <OsState\_2\>.

*La ressource avec l’ID « <resourceID\>» a une génération d’hyperviseur ['<V#\_1\>'] différente de la génération d’hyperviseur d’image de galerie parente ['<V#\_2\>']* .  
**Cause** : la génération d’hyperviseur de la version d’image ne correspond pas à la génération d’hyperviseur spécifiée dans la définition d’image. Le système d’exploitation de la définition d’image est <V#\_1\> et le système d’exploitation de la version d’image est <V#\_2\>.  
**Solution de contournement** : choisissez une source avec la même génération d’hyperviseur que la définition d’image, ou créez/choisissez une nouvelle définition d’image ayant la même génération d’hyperviseur que la version d’image.

*La ressource avec l’ID « <resourceID\>» a un type de système d’exploitation ['<OsType\_1\>'] différent de la génération du type de système d’exploitation de la galerie parente ['<OsType \_2\>']* .  
**Cause** : la génération d’hyperviseur de la version d’image ne correspond pas à la génération d’hyperviseur spécifiée dans la définition d’image. Le système d’exploitation de la définition d’image est <OsType\_1\>, et le système d’exploitation de la version d’image est <OsType\_2\>.  
**Solution de contournement** : choisissez une source avec le même système d’exploitation (Linux/Windows) que la définition d’image, ou créez/choisissez une nouvelle définition d’image ayant la même génération de système d’exploitation que la version d’image.

*La machine virtuelle source <resourceID\> ne peut pas contenir de disque de système d’exploitation éphémère.*  
**Cause** : la source dans <resourceID\> contient un disque de système d’exploitation éphémère. Le service Shared Image Gallery ne prend pas en charge actuellement les disques de systèmes d’exploitation éphémères.  
**Solution de contournement** : choisissez une autre source basée sur une machine virtuelle qui n’utilise pas de disque de système d’exploitation éphémère.

*La machine virtuellle source <resourceID\> ne peut pas contenir de disque ['<diskID\>'] stocké dans un type de compte SSD Ultra.*  
**Cause** : Le disque <diskID\> est un disque SSD Ultra. Le service Shared Image Gallery ne prend pas en charge actuellement les disques SSD Ultra.  
**Solution de contournement** : utilisez une source contenant uniquement des disques managés SSD Premium, des SSD Standard et/ou HDD Standard.

*La machine virtuelle source <resourceID\> doit être créée à partir de disques managés.*  
**Cause** : la machine virtuelle dans <resourceID\> utilise des disques non managés.  
**Solution de contournement** : utilisez une source basée sur une machine virtuelle contenant uniquement des disques managés SSD Premium, des SSD Standard et/ou HDD Standard.

*Trop de demandes sur la source « <resourceID\> ». Réduisez le nombre de requêtes sur la source ou patientez un peu avant de réessayer.*  
**Cause** : la source de cette version d’image est actuellement limitée en raison d’un trop grand nombre de requêtes.  
**Solution de contournement** : essayez de créer la version de l’image ultérieurement.

*Le jeu de chiffrement de disque « <diskEncryptionSetID\>» doit se trouver dans le même abonnement, « <subscriptionID\>», que la ressource de galerie.*  
**Cause** : des jeux de chiffrement de disque ne peuvent être utilisés que dans l’abonnement et la région dans lesquels ils ont été créés.  
**Solution de contournement** : créez ou utilisez un jeu de chiffrement dans le même abonnement et la même région que la version d’image.

*La source chiffrée : « <resourceID\>» se trouve dans un ID d’abonnement différent de celui de l’abonnement actuel de la version d’image de galerie « <subscriptionID\_1\>». Réessayez avec une ou plusieurs sources non chiffrées, ou utilisez l’abonnement de la source, « <subcriptionID\_2\>», pour créer la version d’image de galerie.*  
**Cause** : le service Shared Image Gallery ne prend pas en charge actuellement la création de versions d’images dans un autre abonnement d’une autre image source si l’image source est chiffrée.  
**Solution de contournement** : utilisez une source non chiffrée, ou créez la version d’image dans le même abonnement que la source.

*Le jeu de chiffrement de disque <diskEncryptionSetID\> est introuvable.*  
**Cause** : le chiffrement de disque est peut-être incorrect.  
**Solution de contournement** : vérifiez que l’ID de ressource du jeu de chiffrement de disque est correct.

*Le nom de version d’image n’est pas valide. Le nom de version d’image doit respecter le format Major(int).Minor(int).Patch(int). Par exemple : 1.0.0, 2018.12.1, etc.*  
**Cause** : le format valide pour une version d’image est trois entiers séparés par un point. Le format du nom de version d’image n’est pas valide.  
**Solution de contournement** : utilisez un nom de version d’image respectant le format Major(int).Minor(int).Patch(int). Exemple : 1.0.0. ou 2018.12.1.

*La valeur du paramètre galleryArtifactVersion.properties.publishingProfile.targetRegions.encryption.dataDiskImages.diskEncryptionSetId n’est pas valide*.  
**Cause** : l’ID de ressource du jeu de chiffrement de disque utilisé sur une image de disque de données utilise un format non valide.  
**Solution de contournement** : assurez-vous que l’ID de ressource du jeu de chiffrement de disque respecte le format /subscriptions/<subscriptionID\>/resourceGroups/<resourceGroupName\>/providers/Microsoft.Compute/<diskEncryptionSetName\>.

*La valeur du paramètre galleryArtifactVersion.properties.publishingProfile.targetRegions.encryption.osDiskImage.diskEncryptionSetId n’est pas valide.*  
**Cause** : l’ID de ressource du jeu de chiffrement de disque utilisé sur l’image de disque de système d’exploitation utilise un format non valide.  
**Solution de contournement** : assurez-vous que l’ID de ressource du jeu de chiffrement de disque respecte le format /subscriptions/<subscriptionID\>/resourceGroups/<resourceGroupName\>/providers/Microsoft.Compute/<diskEncryptionSetName\>.

*Impossible de spécifier un nouveau lun de chiffrement d’image de disque de données [<number\>] avec un jeu de chiffrement de disque dans la région [<region\>] pour la demande de version d’image de galerie de mise à jour. Pour mettre à jour cette version, supprimez le nouveau lun. Si vous avez besoin de modifier les paramètres de chiffrement d’image de disque de données, vous devez créer une version d’image de galerie avec les paramètres corrects.*  
**Cause** : vous avez ajouté un chiffrement au disque de données d’une version d’image existante. Vous ne pouvez pas ajouter un chiffrement à une version d’image existante.  
**Solution de contournement** : créez une nouvelle version d’image de galerie ou supprimez les paramètres de chiffrement ajoutés.

*La source de la version d’artefact de galerie ne peut être spécifiée que directement sous storageProfile ou dans un système d’exploitation ou des disques de données individuels. Vous ne pouvez fournir qu’un seul type de source (image utilisateur, capture instantanée, disque, machine virtuelle).*  
**Cause** : l’ID de source est manquant.  
**Solution de contournement** : Assurez-vous que l’ID de la source est présent.

*La source est introuvable : <resourceID\>. Vérifiez que la source existe.*  
**Cause** : l’ID de la ressource de la source est peut-être incorrect.  
**Solution de contournement** : assurez-vous que l’ID de la ressource de la source est correct.

*Un jeu de chiffrement de disque est requis pour le disque « galleryArtifactVersion.properties.publishingProfile.targetRegions.encryption.osDiskImage.diskEncryptionSetId » dans la région cible « <region\_1\> », car le jeu de chiffrement de disque « <diskEncryptionSetID\> » est utilisé pour le disque correspondant dans la région « <region\_2\> »* .  
**Cause** : un chiffrement a été utilisé sur le disque de système d’exploitation dans la région <region\_2\>, mais pas dans la région <region\_1\>.  
**Solution de contournement** : si vous utilisez un chiffrement sur le disque de système d’exploitation, utilisez-le dans toutes les régions.

*Un jeu de chiffrement de disque est requis pour le « LUN <number\> » du disque dans la région cible « <Region\_1\> », car le jeu de chiffrement de disque « <diskEncryptionSetID\> » est utilisé pour le disque correspondant dans la région « <Region\_2\> »* .  
**Cause** : un chiffrement a été utilisé sur le disque de données dans le LUN <number\> dans la région <Region\_2\>, mais pas dans la région <Region\_1\>.  
**Solution de contournement** : si vous utilisez un chiffrement sur un disque de données, utilisez-le dans toutes les régions.

*Un lun [<number\>] non valide a été spécifié dans Encryption.dataDiskImages. Le lun doit avoir l’une des valeurs suivantes ['0,9'].*  
**Cause** : le numéro d’unité logique spécifié pour le chiffrement ne correspond à aucun numéro d’unité logique de disque attaché à la machine virtuelle.  
**Solution de contournement** : remplacez le numéro d’unité logique dans le chiffrement par le numéro d’unité logique d’un disque de données présent dans la machine virtuelle.

*Des numéros de lun « number<\> » en double ont été spécifiés dans les encryption.dataDiskImages de la région cible « <region\>  ».*  
**Cause** : les paramètres de chiffrement utilisés dans la région <region\> spécifient un numéro d’unité logique au moins deux fois.  
**Solution de contournement** : modifiez le numéro d’unité logique dans la région <region\> pour vous assurer que tous les numéros d’unité logique sont uniques dans la région <region\>.

*OSDiskImage et DataDiskImage ne peuvent pas pointer vers le même blob <sourceID\>* .  
**Cause** : les sources du disque de système d’exploitation et au moins un disque de données ne sont pas uniques.  
**Solution de contournement** : modifiez la source du disque de système d’exploitation et/ou des disques de données pour vous assurer que le disque de système d’exploitation ainsi que chaque disque de données sont uniques.

*Les régions en double ne sont pas autorisées dans les régions de publication cibles.*  
**Cause** : une région est référencée plus d’une fois dans les régions de publication.  
**Solution de contournement** : supprimez la région en double.

*L’ajout de nouveaux disques de données ou la modification du lun d’un disque de données dans une image existante ne sont pas autorisés.*  
**Cause** : un appel de mise à jour à la version d’image contient un nouveau disque de données ou un nouveau numéro d’unité logique pour un disque.  
**Solution de contournement** : utilisez les numéros d’unité logique et les disques de données de la version d’image existante.

*Le jeu de chiffrement de disque <diskEncryptionSetID\> doit se trouver dans le même abonnement <subscriptionID\> que la ressource de galerie.*  
**Cause** : le service Shared Image Gallery ne prend pas en charge actuellement l’utilisation d’un jeu de chiffrement de disque dans un autre abonnement.  
**Solution de contournement** : créez la version d’image et le jeu de chiffrement de disque dans le même abonnement.

## <a name="creating-or-updating-a-vm-or-scale-sets-from-an-image-version"></a>Création ou la mise à jour d’une machine virtuelle ou de groupes identiques à partir d’une version d’image ##

*Le client a l’autorisation d’effectuer l’action « Microsoft.Compute/galleries/images/versions/read » sur l’étendue <resourceID\>, mais le locataire actuel, <tenantId1\>, n’est pas autorisé à accéder à l’abonnement lié <subscriptionID2\>.*  
**Cause** : la machine virtuelle ou le groupe identique ont été créés à l’aide d’une image SIG dans un autre locataire. Vous avez tenté d’apporter une modification à la machine virtuelle ou au groupe identique, mais vous n’avez pas accès à l’abonnement possédant l’image.  
**Solution de contournement** : contactez le propriétaire de l’abonnement de la version d’image pour accorder un accès en lecture à la version d’image.

*L’image de galerie <resourceID\> n’est pas disponible dans la région <region\>. Contactez le propriétaire de l’image à répliquer dans cette région, ou modifiez la région demandée.*  
**Cause** : la machine virtuelle est en cours de création dans une région ne figurant pas dans la liste des régions publiées pour l’image de galerie.  
**Solution de contournement** : répliquez l’image dans la région ou créez une machine virtuelle dans l’une des régions de publication de l’image de galerie.

*Le paramètre « osProfile » n’est pas autorisé.*  
**Cause** : le nom d’utilisateur, le mot de passe ou les clés SSH d’administrateur ont été fournis pour une machine virtuelle qui a été créée à partir d’une version d’image spécialisée.  
**Solution de contournement** : n’incluez pas le nom d’utilisateur administrateur, le mot de passe ou les clés SSH d’administrateur si vous envisagez de créer une machine virtuelle à partir de cette image. Sinon, utilisez une version d’image généralisée et fournissez le nom d’utilisateur administrateur, le mot de passe ou les clés SSH d’administrateur.

*Le paramètre obligatoire « osProfile » est manquant (null).*  
**Cause** : la machine virtuelle est créée à partir d’une image généralisée et il lui manque le nom d’utilisateur administrateur, le mot de passe ou les clés SSH d’administrateur. Comme les images généralisées ne conservent pas le nom d’utilisateur, le mot de passe ou les clés SSH d’administrateur, ces champs doivent être spécifiés lors de la création d’une machine virtuelle ou d’un groupe identique.  
**Solution de contournement** : spécifiez le nom d’utilisateur administrateur, le mot de passe ou les clés SSH d’administrateur, ou utilisez une version d’image spécialisée.

*Impossible de créer une version d’image de galerie à partir de <resourceID\>, car l’état du système d’exploitation dans l’image de galerie parente (« Specialized ») n’est pas « Generalized ».*  
**Cause** : la version d’image est créée à partir d’une source généralisée, mais sa définition parente est spécialisée.  
**Solution de contournement** : créez la version d’image à l’aide d’une source spécialisée, ou utilisez une définition parente généralisée.

*Impossible de mettre à jour le groupe de machines virtuelles identiques <vmssName\>, car l’état actuel du système d’exploitation du groupe de machines virtuelles identiques est Generalized, contrairement à l’état du système d’exploitation d’image de galerie mise à jour qui est Specialized.*  
**Cause** : l’image source actuelle pour le groupe identique est généralisée, mais elle est mise à jour avec une image source spécialisée. L’image source actuelle et la nouvelle image source pour un groupe identique doivent avoir le même état.  
**Solution de contournement** : pour mettre à jour le groupe identique, utilisez une version d’image généralisée.

*Le jeu de chiffrement de disque <diskEncryptionSetId\> dans le service Shared Image Gallery <versionId\> appartient à un abonnement <subscriptionID1\>, et ne peut pas être utilisé avec une ressource «» dans un abonnement <subscriptionID2\>* .  
**Cause** : le jeu de chiffrement de disque utilisé pour chiffrer la version d’image réside dans un abonnement différent de l’abonnement hébergeant la version d’image.  
**Solution de contournement** : utilisez le même abonnement pour la version d’image et le jeu de chiffrement du disque.

*La création de la machine virtuelle ou du groupe de machines virtuelles identiques prend beaucoup de temps.*  
**Solution de contournement** : Vérifiez que la valeur **OSType** de la version d’image à partir de laquelle vous tentez de créer la machine virtuelle ou le groupe de machines virtuelles identiques est identique à la valeur **OSType** de la source utilisée pour créer la version d’image. 

## <a name="creating-a-disk-from-an-image-version"></a>Création d’un disque à partir d’une version d’image ##

*La valeur du paramètre imageReference n’est pas valide.*  
**Cause** : vous avez tenté d’exporter à partir d’une version d’image SIG vers un disque, mais vous avez utilisé une position de numéro d'unité logique qui n’existe pas sur l’image.    
**Solution de contournement** : vérifiez la version d’image pour connaître les positions de lun utilisées.

## <a name="sharing-resources"></a>Partage des ressources

Le partage des ressources de galerie d’images, de définition d’image et de version d’image entre abonnements est activé à l’aide du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/rbac-and-directory-admin-roles.md). 

## <a name="replication-speed"></a>Vitesse de réplication

Utilisez l’indicateur **--expand ReplicationStatus** pour vérifier si la réplication sur toutes les régions cibles spécifiées a été achevée. Si ce n’est pas le cas, attendez six heures maximum jusqu’à la fin de la tâche. En cas d’échec, redéclenchez la commande pour créer et répliquer la version d’image. Si la version d’image est répliquée sur un grand nombre de régions cibles, effectuez la réplication par phases.

## <a name="azure-limits-and-quotas"></a>Limites et quotas Azure 

[Les limites et quotas Azure](../azure-resource-manager/management/azure-subscription-service-limits.md) s’appliquent à toutes les ressources de galerie d’images partagées, définition d’image et version d’image. Vérifiez que vous êtes dans les limites de vos abonnements. 


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [ galeries d’images partagées](./linux/shared-image-galleries.md)