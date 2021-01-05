---
title: Codes d’erreur de machine virtuelle courants dans Azure | Microsoft Docs
description: Présentation de certains des codes d’erreur courants rencontrés lors du provisionnement et de la gestion des machines virtuelles dans Azure
services: virtual-machines
documentationcenter: ''
author: xujing-ms
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure
ms.date: 5/22/2017
ms.author: xujing
ms.openlocfilehash: 52508a6820ce0cbbbe3a0341a99894f8b92b1645
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87831224"
---
# <a name="understand-common-error-messages-when-you-manage-virtual-machines-in-azure"></a>Présentation des messages d’erreur courants lors de la gestion des machines virtuelles dans Azure

Cet article décrit certains des messages et des codes d’erreur les plus courants que vous pouvez rencontrer quand vous créez ou gérez des machines virtuelles dans Azure.

>[!NOTE]
> Vous pouvez laisser des commentaires sur cette page, ou via [Azure feedback](https://feedback.azure.com/forums/216843-virtual-machines) (commentaires Azure) avec la balise #azerrormessage.

## <a name="error-response-format"></a>Format de réponse d’erreur 
Les machines virtuelles Azure utilisent le format JSON suivant pour les réponses d’erreur :

```json
{
  "status": "status code",
  "error": {
    "code":"Top level error code",
    "message":"Top level error message",
    "details":[
     {
      "code":"Inner level error code",
      "message":"Inner level error message"
     }
    ]
   }
}
```

Une réponse d’erreur inclut systématiquement un code d’état et un objet d’erreur. Chaque objet d’erreur contient toujours un code d’erreur et un message. Si la machine virtuelle est créée selon un modèle, l’objet d’erreur contient également une section de détails, qui possède un niveau interne de codes et de messages d’erreur. Normalement, le niveau le plus interne des messages d’erreur est l’échec principal.


## <a name="common-virtual-machine-management-errors"></a>Erreurs courantes de gestion des machines virtuelles

Cette section répertorie les messages d’erreur pouvant apparaître lors de la gestion des machines virtuelles :

|  Code d'erreur  |  Message d’erreur  |  
|  :------| :-------------|  
|  AcquireDiskLeaseFailed  |  Échec de l’acquisition du bail lors de la création du disque « {0} » en utilisant un objet blob avec l’URI {1}. L’objet blob est déjà utilisé.  |  
|  AllocationFailed  |  L’allocation a échoué. Essayez de réduire la taille ou le nombre de machines virtuelles, recommencez plus tard ou essayez d’effectuer un déploiement vers un autre groupe à haute disponibilité ou un autre emplacement Azure.  |  
|  AllocationFailed  |  L’allocation de machine virtuelle a échoué en raison d’une erreur interne. Réessayez ultérieurement ou essayez d’effectuer le déploiement vers un autre emplacement.  |
|  ArtifactNotFound  |  L'extension de machine virtuelle avec le serveur de publication « {0} » et le type « {1} » est introuvable à l'emplacement « {2} ».  |
|  ArtifactNotFound  |  L'extension avec le serveur de publication « {0} », le type « {1} » et la version du gestionnaire de types « {2} » est introuvable dans le référentiel des extensions.  |
|  ArtifactVersionNotFound  |  Aucune des versions trouvées dans le référentiel d’artefacts ne correspond à la version demandée « {0} ».  |
|  ArtifactVersionNotFound  |  Aucune des versions trouvées dans le référentiel d'artefacts ne correspond à la version demandée « {0} » pour l'extension de machine virtuelle avec le serveur de publication « {1} » et le type « {2} ».  |
|  AttachDiskWhileBeingDetached  |  Impossible d’attacher le disque de données « {0} » à la machine virtuelle « {1} », car il est actuellement en cours de détachement. Veuillez attendre que le disque soit complètement détaché, puis réessayez.  |
|  BadRequest  |  Les groupes à haute disponibilité alignés ne sont pas encore pris en charge dans cette région.  |
|  BadRequest  |  L’ajout d’une machine virtuelle comportant des disques managés à un groupe à haute disponibilité non géré ou l’ajout d’une machine virtuelle comportant des disques basés sur des objets blob à un groupe à haute disponibilité géré n’est pas pris en charge. Créez un groupe à haute disponibilité avec la propriété « managed » définie afin de lui ajouter une machine virtuelle comportant des disques managés.  |
|  BadRequest  |  Les disques managés ne sont pas pris en charge dans cette région.  |
|  BadRequest  |  Plusieurs VMExtensions par gestionnaire non prises en charge pour le type de système d’exploitation « {0} ». VMExtension « {1} » avec le gestionnaire « {2} » déjà ajoutée ou spécifiée dans l’entrée.  |
|  BadRequest  |  L'opération « {0} » n'est pas prise en charge sur la ressource « {1} » avec des disques managés.  |
|  CertificateImproperlyFormatted  |  La représentation JSON du secret récupérée auprès de {0} a un champ de données qui n’est pas un fichier PFX correctement mis en forme ou bien le mot de passe fourni ne décode pas correctement le fichier PFX.  |
|  CertificateImproperlyFormatted  |  Les données récupérées auprès de {0} ne sont pas désérialisables dans JSON.  |
|  Conflit  |  Un redimensionnement de disque est autorisé uniquement durant la création ou la libération d’une machine virtuelle.  |
|  ConflictingUserInput  |  Impossible d’attacher le disque « {0} », car il est déjà détenu par la machine virtuelle « {1} ».  |
|  ConflictingUserInput  |  Les groupes de ressources source et de destination sont identiques.  |
|  ConflictingUserInput  |  Les comptes de stockage source et de destination pour le disque {0} sont différents.  |
|  ContainerAlreadyOnLease  |  Il existe déjà un bail dans le conteneur de stockage hébergeant l’objet blob avec l’URI {0}.  |
|  CrossSubscriptionMoveWithKeyVaultResources  |  La demande de déplacement de ressources contient des ressources KeyVault qui sont référencées par un ou plusieurs {0} dans la demande. Pour le moment, cela n’est pas pris en charge dans le déplacement entre les abonnements. Dans les détails de l’erreur, recherchez les ID de ressource KeyVault.  |
|  DiagnosticsOperationInternalError  |  Une erreur interne s’est produite pendant le traitement du profil de diagnostic de la machine virtuelle {0}.  |
|  DiskBlobAlreadyInUseByAnotherDisk  |  L’objet blob {0} est déjà utilisé par un autre disque appartenant à la machine virtuelle « {1} ». Vous pouvez examiner les métadonnées de l’objet blob pour y rechercher les informations de référence du disque.  |
|  DiskBlobNotFound  |  Impossible de trouver l’objet blob VHD avec l’URI {0} pour le disque « {1} ».  |
|  DiskBlobNotFound  |  Impossible de trouver l’objet blob VHD avec l’URI {0}.  |
|  DiskEncryptionKeySecretMissingTags  |  Le secret {0} ne contient pas les balises {1}. Mettez à jour la version du secret, ajoutez les balises nécessaires et réessayez.  |
|  DiskEncryptionKeySecretUnwrapFailed  |  Le déchiffrement de la valeur {0} du secret à l'aide de la clé {1} a échoué.  |
|  DiskImageNotReady  |  L’image de disque {0} a l’état {1}. Réessayez lorsque l’image est prête.  |
|  DiskPreparationError  |  Une ou plusieurs erreurs se sont produites lors de la préparation des disques de machine virtuelle. Pour plus d’informations, consultez la vue d’instance de disque.  |
|  DiskProcessingError  |  Le traitement de disque s’est interrompu, car la machine virtuelle contient d’autres disques dans les disques défaillants.  |
|  ImageBlobNotFound  |  Impossible de trouver l’objet blob VHD avec l’URI {0} pour le disque « {1} ».  |
|  ImageBlobNotFound  |  Impossible de trouver l’objet blob VHD avec l’URI {0}.  |
|  IncorrectDiskBlobType  |  Les objets blob de disques ne peuvent être que du type objet blob de pages. L’objet blob {0} du disque « {1} » est du type objet blob de blocs.  |
|  IncorrectDiskBlobType  |  Les objets blob de disques ne peuvent être que du type objet blob de pages. L’objet blob {0} est du type « {1} ».  |
|  IncorrectImageBlobType  |  Les objets blob de disques ne peuvent être que du type objet blob de pages. L’objet blob {0} du disque « {1} » est du type objet blob de blocs.  |
|  IncorrectImageBlobType  |  Les objets blob de disques ne peuvent être que du type objet blob de pages. L’objet blob {0} est du type « {1} ».  |
|  InternalOperationError  |  Impossible de résoudre le compte de stockage {0}. Vérifiez qu’il a été créé via le fournisseur de ressources de stockage au même emplacement que la ressource de calcul.  |
|  InternalOperationError  |  Échec des tâches de recherche de valeur cible {0}.  |
|  InternalOperationError  |  Une erreur s’est produite lors de la validation du profil réseau de la machine virtuelle « {0} ».  |
|  InvalidAccountType  |  L’élément AccountType {0} n’est pas valide.  |
|  InvalidParameter  |  La valeur du paramètre {0} n'est pas valide.  |
|  InvalidParameter  |  Le mot de passe spécifié pour l’administrateur n’est pas autorisé.  |
|  InvalidParameter  |  Le mot de passe fourni doit comprendre entre {0} et {1} caractères et doit satisfaire au moins {2} des exigences suivantes en termes de complexité de mot de passe : <ol><li> Contient une majuscule.</li><li>Contient une minuscule.</li><li>Contient un chiffre.</li><li>Contient un caractère spécial.</li></ol>  |
|  InvalidParameter  |  Le nom d’utilisateur spécifié pour l’administrateur n’est pas autorisé.  |
|  InvalidParameter  |  Impossible d’attacher un disque de système d’exploitation existant si la machine virtuelle est créée à partir d’une image de plateforme ou d’utilisateur.  |
|  InvalidParameter  |  Le nom de conteneur {0} n’est pas valide. Les noms de conteneur doivent comporter 3 à 63 caractères, lesquels ne peuvent être que des caractères alphanumériques minuscules et un tiret. Le tiret doit être précédé et suivi d’un caractère alphanumérique.  |
|  InvalidParameter  |  Le nom de conteneur {0} dans l’URL {1} n’est pas valide. Les noms de conteneur doivent comporter 3 à 63 caractères, lesquels ne peuvent être que des caractères alphanumériques minuscules et un tiret. Le tiret doit être précédé et suivi d’un caractère alphanumérique.  |
|  InvalidParameter  |  Le nom de l’objet blob indiqué dans l’URL {0} contient une barre oblique. Pour le moment, ce n’est pas pris en charge pour les disques.  |
|  InvalidParameter  |  L’URI {0} ne semble pas être un URI d’objet blob correct.  |
|  InvalidParameter  |  Un disque nommé « {0} » utilise déjà le même LUN : {1}.  |
|  InvalidParameter  |  Un disque nommé « {0} » existe déjà.  |
|  InvalidParameter  |  Impossible de spécifier des remplacements d’image d’utilisateur pour un disque déjà défini dans la référence d’image spécifiée.  |
|  InvalidParameter  |  Un disque nommé « {0} » utilise déjà la même URL de VHD {1}.  |
|  InvalidParameter  |  Le nombre de domaines d'erreur spécifié {0} doit être compris entre {1} et {2}.  |
|  InvalidParameter  |  Le type de licence {0} n’est pas valide. Les types de licence valides sont les suivants : Windows_Client ou Windows_Server, avec respect de la casse.  |
|  InvalidParameter  |  Le nom d’hôte Linux ne peut pas dépasser {0} caractères, ni contenir les caractères suivants : {1}.  |
|  InvalidParameter  |  Le chemin de destination des clés publiques SSH est actuellement limité à sa valeur par défaut {0} en raison d’un problème connu dans l’agent d’approvisionnement Linux.  |
|  InvalidParameter  |  Il existe déjà un disque sur le numéro d’unité logique {0}.  |
|  InvalidParameter  |  L'abonnement {0} de la demande doit correspondre à l'abonnement {1} contenu dans l'ID de disque managé.  |
|  InvalidParameter  |  Les données personnalisées dans le profil OSProfile doivent être encodées en Base64, et leur longueur maximale doit être de {0} caractères.  |
|  InvalidParameter  |  Le nom de l'objet blob dans l'URL {0} doit se terminer par l'extension « {1} ».  |
|  InvalidParameter  |  {0} n’est pas un préfixe de nom d’objet blob VHD capturé valide. Un préfixe valide correspond à l’expression régulière « {1} ».  |
|  InvalidParameter  |  Impossible d’ajouter des certificats à votre machine virtuelle si l’agent de machine virtuelle n’est pas approvisionné.  |
|  InvalidParameter  |  Il existe déjà un disque sur le numéro d’unité logique {0}.  |
|  InvalidParameter  |  Impossible de créer la machine virtuelle, car la taille demandée {0} n’est pas disponible dans le cluster dans lequel le groupe à haute disponibilité est actuellement alloué. Les tailles disponibles sont les suivantes : {1}. En savoir plus sur la stratégie de redimensionnement de machine virtuelle sur https://aka.ms/azure-resizevm.  |
|  InvalidParameter  |  La taille de machine virtuelle demandée {0} n’est pas disponible dans la région actuelle. Les tailles disponibles dans la région actuelle sont les suivantes : {1}. Pour en savoir plus sur les tailles de machines virtuelles disponibles dans chaque région, consultez https://aka.ms/azure-regions.  |
|  InvalidParameter  |  La taille de machine virtuelle demandée {0} n’est pas disponible dans la région actuelle. Pour en savoir plus sur les tailles de machines virtuelles disponibles dans chaque région, consultez https://aka.ms/azure-regions.  |
|  InvalidParameter  |  Le nom d’utilisateur de l’administrateur Windows ne peut pas dépasser {0} caractères, ni se terminer par un point (.), ni contenir les caractères suivants : {1}.  |
|  InvalidParameter  |  Le nom d’ordinateur Windows ne peut pas comporter plus de {0} caractères, ni être entièrement constitué de chiffres, ni contenir les caractères suivants : {1}.  |
|  MissingMoveDependentResources  |  La demande de déplacement de ressources ne contient pas toutes les ressources dépendantes. Dans les détails de l’erreur, recherchez les ID des ressources manquantes.  |
|  MoveResourcesHaveInvalidState  |  La demande de déplacement de ressources contient des machines virtuelles qui sont associées à des comptes de stockage non valides. Dans les détails de l’erreur, recherchez les ID de ces ressources ainsi que les noms des comptes de stockage référencés.  |
|  MoveResourcesHavePendingOperations  |  La demande de déplacement de ressources contient des ressources pour lesquelles une opération est en attente. Dans les détails de l’erreur, recherchez les ID de ces ressources. Réessayez l’opération dès que les opérations en attente seront terminées.  |
|  MoveResourcesNotFound  |  La demande de déplacement de ressources contient des ressources qui sont introuvables. Dans les détails de l’erreur, recherchez les ID de ces ressources.  |
|  NetworkingInternalOperationError  |  Erreur d’allocation réseau inconnue.  |
|  NetworkingInternalOperationError  |  Erreur d’allocation réseau inconnue  |
|  NetworkingInternalOperationError  |  Une erreur interne s’est produite lors du traitement du profil réseau de la machine virtuelle.  |
|  NotFound  |  Le groupe à haute disponibilité {0} est introuvable.  |
|  NotFound  |  La machine virtuelle source « {0} » spécifiée dans la demande n’existe pas à cet emplacement Azure.  |
|  NotFound  |  Le client avec l’ID {0} est introuvable.  |
|  NotFound  |  L’image {0} est introuvable.  |
|  NotSupported  |  Le type de licence est {0} alors que l'objet blob d'image {1} n'est pas en local.  |
|  OperationNotAllowed  |  Impossible de supprimer le groupe à haute disponibilité {0}. Avant de supprimer un groupe à haute disponibilité, vérifiez qu’il ne contient aucune machine virtuelle.  |
|  OperationNotAllowed  |  Il est interdit de modifier la référence du groupe à haute disponibilité « Aligné » et de la remplacer par « Classique ».  |
|  OperationNotAllowed  |  Impossible de modifier des extensions dans la machine virtuelle quand elle n’est pas en cours d’exécution.  |
|  OperationNotAllowed  |  L’action Capturer est uniquement prise en charge sur une machine virtuelle comportant des disques basés sur des objets blob. Utilisez les API de ressource « Image » pour créer une image à partir d’une machine virtuelle gérée.  |
|  OperationNotAllowed  |  Impossible de créer la ressource {0} à partir de l'image {1} tant que l'image n'a pas été créée.  |
|  OperationNotAllowed  |  Les mises à jour d’encryptionSettings ne sont pas autorisées quand la machine virtuelle est allouée. Réessayez une fois la machine virtuelle libérée.  |
|  OperationNotAllowed  |  L’ajout d’un disque géré à une machine virtuelle comportant des disques basés sur des objets blob n’est pas pris en charge.  |
|  OperationNotAllowed  |  Le nombre maximal de disques de données autorisés à être attachés à une machine virtuelle de cette taille est de {0}.  |
|  OperationNotAllowed  |  L’ajout d’un disque basé sur un objet blob à une machine virtuelle comportant des disques managés n’est pas pris en charge.  |
|  OperationNotAllowed  |  L’opération « {0} » n’est pas autorisée sur l’image « {1} », car l’image est marquée pour suppression. Vous pouvez uniquement réessayer l’opération de suppression (ou attendre la fin de l’opération en cours).  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée sur la machine virtuelle « {1} », car cette dernière est généralisée.  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée car la collection de points de restauration « {1} » est marquée pour suppression.  |
|  OperationNotAllowed  |  L’opération « {0} » n’est pas autorisée sur l’extension de machine virtuelle « {1} », car cette dernière est marquée pour suppression. Vous pouvez uniquement réessayer l’opération de suppression (ou attendre la fin de l’opération en cours).  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée car les machines virtuelles « {1} » sont actuellement provisionnées avec l'image « {2} ».  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée, car le groupe de machines virtuelles identiques « {1} » utilise actuellement l'image « {2} ».  |
|  OperationNotAllowed  |  L’opération « {0} » n’est pas autorisée sur la machine virtuelle « {1} », car cette dernière est marquée pour suppression. Vous pouvez uniquement réessayer l’opération de suppression (ou attendre la fin de l’opération en cours).  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée sur la machine virtuelle « {1} », car celle-ci est désallouée ou marquée comme désallouée.  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée sur la machine virtuelle « {1} », car cette dernière est en cours d’exécution. Mettez explicitement la machine virtuelle hors tension si vous l’avez arrêtée à partir du système d’exploitation invité.  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée sur la machine virtuelle « {1} » car celle-ci n'est pas désallouée.  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée sur la machine virtuelle « {1} » car cette machine virtuelle a l'extension « {2} » dans un état d'échec.  |
|  OperationNotAllowed  |  L'opération « {0} » n'est pas autorisée sur la machine virtuelle « {1} », car une autre opération est en cours.  |
|  OperationNotAllowed  |  L'opération « {0} » nécessite la généralisation de la machine virtuelle « {1} ».  |
|  OperationNotAllowed  |  L’opération nécessite que la machine virtuelle soit en cours d’exécution (ou définie pour l’exécution).  |
|  OperationNotAllowed  |  Le disque de {0} Go, dont la taille est inférieure à la taille du disque correspondant ({1} Go) dans l'image, n'est pas autorisé.  |
|  OperationNotAllowed  |  Les extensions de groupe de machines virtuelles identiques du gestionnaire « {0} » ne peuvent être ajoutées que lors de la création du groupe de machines virtuelles identiques.  |
|  OperationNotAllowed  |  Les extensions de groupe de machines virtuelles identiques du gestionnaire « {0} » ne peuvent être supprimées que lors de la suppression du groupe de machines virtuelles identiques.  |
|  OperationNotAllowed  |  La machine virtuelle « {0} » utilise déjà des disques managés.  |
|  OperationNotAllowed  |  La machine virtuelle « {0} » appartient au groupe à haute disponibilité « Classique » « {1} ». Mettez à jour le groupe à haute disponibilité pour utiliser la référence « Aligné », puis réessayez la conversion.  |
|  OperationNotAllowed  |  La machine virtuelle créée à partir d’une image ne peut pas posséder de disques basés sur des objets blob. Tous les disques doivent être managés.  |
|  OperationNotAllowed  |  L’opération de capture ne peut pas être effectuée, car la machine virtuelle n’est pas généralisée.  |
|  OperationNotAllowed  |  Les opérations de gestion sur la machine virtuelle « {0} » ne sont pas autorisées, car les disques de machine virtuelle sont convertis en disques managés.  |
|  OperationNotAllowed  |  Une opération en cours change l’état d’alimentation de la machine virtuelle {0} et le définit sur {1}. Après un certain temps, effectuez l’opération {2}.  |
|  OperationNotAllowed  |  Impossible d’ajouter ou de mettre à jour la machine virtuelle. La taille de machine virtuelle {0} demandée n’est peut-être pas disponible dans l’unité d’allocation existante. En savoir plus sur la stratégie de redimensionnement de machine virtuelle sur https://aka.ms/azure-resizevm.  |
|  OperationNotAllowed  |  Impossible de redimensionner la machine virtuelle, car la taille demandée {0} n’est pas disponible dans le cluster dans lequel le groupe à haute disponibilité est actuellement alloué. Les tailles disponibles sont les suivantes : {1}. En savoir plus sur la stratégie de redimensionnement de machine virtuelle sur https://aka.ms/azure-resizevm.  |
|  OperationNotAllowed  |  Impossible de redimensionner la machine virtuelle, car la taille demandée {0} n’est pas disponible dans le cluster dans lequel la machine virtuelle est actuellement allouée. Pour redimensionner votre machine virtuelle sur {1}, libérez (il s’agit de l’opération d’arrêt dans le portail Azure) et réessayez l’opération de redimensionnement. En savoir plus sur la stratégie de redimensionnement de machine virtuelle sur https://aka.ms/azure-resizevm.  |
|  OSProvisioningClientError  |  Échec du provisionnement du système d’exploitation pour la machine virtuelle « {0} », car le système d’exploitation invité est actuellement en cours de provisionnement.  |
|  OSProvisioningClientError  |  Échec du provisionnement du système d’exploitation pour la machine virtuelle « {0} ». Détails de l’erreur : {1} Vérifiez que l’image a été correctement préparée (généralisée). <ul><li>Instructions pour Windows : https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/  </li></ul> |
|  OSProvisioningClientError  |  Échec de la génération de clé hôte SSH. Détails de l’erreur : {0}. Pour résoudre ce problème, vérifiez si l’agent Linux est configuré correctement. <ul><li>Vous pouvez vérifier les instructions sur : https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux/ </li></ul> |
|  OSProvisioningClientError  |  Le nom d’utilisateur spécifié pour la machine virtuelle n’est pas valide pour cette distribution Linux. Détails de l’erreur : {0}.  |
|  OSProvisioningInternalError  |  Échec du provisionnement du système d’exploitation pour la machine virtuelle « {0} » en raison d’une erreur interne.  |
|  OSProvisioningTimedOut  |  Le provisionnement du système d’exploitation de la machine virtuelle « {0} » ne s’est pas terminé dans le délai imparti. La machine virtuelle peut cependant le terminer avec succès. Vérifiez l’état d’approvisionnement ultérieurement.  |
|  OSProvisioningTimedOut  |  Le provisionnement du système d’exploitation de la machine virtuelle « {0} » ne s’est pas terminé dans le délai imparti. La machine virtuelle peut cependant le terminer avec succès. Vérifiez l’état d’approvisionnement ultérieurement. Vérifiez également que l’image a été correctement préparée (généralisée).   <ul><li>Instructions pour [Windows]( ../windows/upload-generalized-managed.md)</li><li> Instructions pour [Linux](../linux/capture-image.md)</li></ul>  |
|  OSProvisioningTimedOut  |  Le provisionnement du système d’exploitation de la machine virtuelle « {0} » ne s’est pas terminé dans le délai imparti. Toutefois, l’agent invité de la machine virtuelle a été détecté en cours d’exécution. Cela signifie que le système d’exploitation invité n’a pas été correctement préparé pour être utilisé en tant qu’image de machine virtuelle (avec CreateOption=FromImage). Pour résoudre ce problème, utilisez le disque VHD tel quel avec CreateOption=Attach ou préparez-le correctement pour l’utiliser en tant qu’image :   <ul><li>Instructions pour Windows : https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/ </li><li> Instructions pour Linux : https://azure.microsoft.com/documentation/articles/virtual-machines-linux-capture-image/</li></ul>  |
|  OverConstrainedAllocationRequest  |  La taille de machine virtuelle requise n’est pas disponible à l’emplacement sélectionné.  |
|  ResourceUpdateBlockedOnPlatformUpdate  |  Impossible de mettre à jour la ressource pour le moment en raison d’une mise à jour en cours de la plateforme. Veuillez réessayer plus tard.  |
|  StorageAccountLimitation  |  Le compte de stockage « {0} » ne prend pas en charge les objets blob de page qui sont requis pour créer des disques.  |
|  StorageAccountLimitation  |  Le compte de stockage « {0} » a dépassé son quota alloué.  |
|  StorageAccountLocationMismatch  |  Impossible de résoudre le compte de stockage {0}. Vérifiez qu’il a été créé via le fournisseur de ressources de stockage au même emplacement que la ressource de calcul.  |
|  StorageAccountNotFound  |  Compte de stockage {0} introuvable. Vérifiez que le compte de stockage n’est pas supprimé et qu’il appartient au même emplacement Azure que celui de la machine virtuelle.  |
|  StorageAccountNotRecognized  |  Utilisez un compte de stockage géré par le fournisseur de ressources de stockage. L’utilisation de {0} n’est pas prise en charge.  |
|  StorageAccountOperationInternalError  |  Une erreur interne s’est produite lors de l’accès au compte de stockage {0}.  |
|  StorageAccountSubscriptionMismatch  |  Le compte de stockage {0} n’appartient pas à l’abonnement {1}.  |
|  StorageAccountTooBusy  |  Le compte de stockage « {0} » est actuellement trop occupé. Pensez à utiliser un autre compte.  |
|  StorageAccountTypeNotSupported  |  Le disque {0} utilise {1} qui est un compte de stockage blob. Réessayez avec un compte de stockage à usage général.  |
|  StorageAccountTypeNotSupported  |  Le compte de stockage {0} est de type {1}. La fonctionnalité Diagnostics de démarrage prend en charge {2} types de compte de stockage.  <ul><li>Cette erreur se produit si vous utilisez le compte de stockage Premium pour les diagnostics de démarrage. Pour plus d’informations, consultez [Comment utiliser les diagnostics de démarrage](boot-diagnostics.md). </li></ul> |
|  SubscriptionNotAuthorizedForImage  |  L’abonnement n’est pas autorisé.  |
|  TargetDiskBlobAlreadyExists  |  Le blob {0} existe déjà. Spécifiez un autre URI d’objet blob pour créer un disque de données vide « {1} ».  |
|  TargetDiskBlobAlreadyExists  |  L’opération de capture ne peut pas continuer, car l’objet blob d’image cible {0} existe déjà et l’indicateur pour remplacer les objets blob VHD n’est pas défini. Supprimez l’objet blob ou définissez l’indicateur pour remplacer les objets blob VHD, puis réessayez.  |
|  TargetDiskBlobAlreadyExists  |  L’opération de capture ne peut pas continuer, car un bail est actif sur l’objet blob d’image cible {0}.   |
|  TargetDiskBlobAlreadyExists  |  Le blob {0} existe déjà. Spécifiez un autre URI d’objet blob comme cible pour le disque « {1} ».  |
|  TooManyVMRedeploymentRequests  |  Les demandes de redéploiement reçues pour la machine virtuelle « {0} » ou les machines virtuelles du même groupe à haute disponibilité sont trop nombreuses. Veuillez réessayer ultérieurement.  |
|  VHDSizeInvalid  |  La valeur de la taille {0} spécifiée pour le disque « {1} » avec l’objet blob {2} n’est pas valide. La taille de disque doit être comprise entre {3} et {4}.  |
|  VMAgentStatusCommunicationError  |  La machine virtuelle « {0} » n’a pas signalé d’état pour l’agent ou les extensions de machine virtuelle. Vérifiez qu’un agent de machine virtuelle s’exécute sur la machine virtuelle et que cette dernière peut établir des connexions sortantes vers le stockage Azure.  |
|  VMArtifactRepositoryInternalError  |  Une erreur s’est produite lors de la communication avec le référentiel d’artefacts pour récupérer les détails de l’artefact de machine virtuelle.  |
|  VMArtifactRepositoryInternalError  |  Une erreur interne s’est produite lors de la récupération des données d’artefact de machine virtuelle dans le référentiel d’artefacts.  |
|  VMExtensionHandlerNonTransientError  |  Le gestionnaire « {0} » a signalé un problème au niveau de l'extension de machine virtuelle « {1} » avec le code d'erreur de terminal « {2} » et le message d'erreur : « {3} »  |
|  VMExtensionManagementInternalError  |  Une erreur interne s’est produite lors du traitement de l’extension de machine virtuelle « {0} ».  |
|  VMExtensionManagementInternalError  |  Plusieurs erreurs se sont produites lors de la préparation des extensions de machine virtuelle. Pour plus d’informations, consultez la vue d’instance de l’extension de machine virtuelle.  |
|  VMExtensionProvisioningError  |  La machine virtuelle a signalé un échec lors du traitement de l’extension « {0} ». Message d’erreur : « {1} ».  |
|  VMExtensionProvisioningError  |  Échec de l’approvisionnement de plusieurs extensions de machine virtuelle sur la machine virtuelle. Pour plus d’informations, consultez la vue d’instance de l’extension de machine virtuelle.  |
|  VMExtensionProvisioningTimeout  |  Le délai d’attente du provisionnement de l’extension de machine virtuelle « {0} » est arrivé à expiration. L’installation de l’extension prend peut-être trop de temps ou l’état de l’extension n’a pas pu être obtenu.  |
|  VMMarketplaceInvalidInput  |  La création d’une machine virtuelle à partir d’une image autre que Place de marché ne nécessite pas d’informations de plan. Supprimez les informations de plan dans la demande. Le nom du disque du système d’exploitation est {0}.  |
|  VMMarketplaceInvalidInput  |  Les informations d’achat ne correspondent pas. Impossible de déployer à partir de l’image Place de marché. Le nom du disque du système d’exploitation est {0}.  |
|  VMMarketplaceInvalidInput  |  La création d’une machine virtuelle à partir d’une image Place de marché nécessite des informations de plan dans la demande. Le nom du disque du système d’exploitation est {0}.  |
|  VMNotFound  |  La machine virtuelle « {0} » est introuvable.  |
|  VMRedeploymentFailed  |  Échec du redéploiement de la machine virtuelle « {0} » en raison d’une erreur interne. Veuillez réessayer ultérieurement.  |
|  VMRedeploymentTimedOut  |  Le redéploiement de la machine virtuelle « {0} » ne s’est pas terminé dans le délai imparti. Il peut se terminer correctement dans un certain temps. Sinon, vous pouvez renouveler la demande.  |
|  VMStartTimedOut  |  La machine virtuelle « {0} » n’a pas démarré dans le délai imparti. La machine virtuelle peut toujours démarrer avec succès. Vérifiez l’état d’alimentation plus tard.  |


## <a name="next-steps"></a>Étapes suivantes
Si vous avez besoin d’aide supplémentaire, vous avez la possibilité de contacter les experts Azure sur [les forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.