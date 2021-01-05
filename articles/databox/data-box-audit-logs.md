---
title: Journal d'audit pour les événements Azure Data Box, Azure Data Box Heavy | Microsoft Docs
description: Décrit les journaux d’audit complets pour Data Box collectés aux différentes étapes de votre commande d’Azure Data Box et Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/10/2020
ms.author: alkohli
ms.openlocfilehash: 50dbe9ab649a708fb36b1c9e4fe89bccadc7ea90
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86208992"
---
# <a name="audit-logs-for-your-azure-data-box-and-azure-data-box-heavy"></a>Journal d'audit de votre Azure Data Box et Azure Data Box Heavy

Les journaux sont des enregistrements immuables, horodatés, d'événements discrets qui se sont produits au fil du temps. Les journaux contiennent des informations de diagnostic, d’audit et de sécurité de votre appareil.  

Une commande Data Box ou Data Box Heavy passe par les étapes suivante : commande, configuration, copie des données, retour, chargement sur Azure et vérification, puis effacement des données. Pour chacune de ces étapes, tous les événements sont audités et journalisés.

Cet article contient des informations sur les journaux d’audit Data Box, notamment les types de journaux et les informations collectées, ainsi que l’emplacement des journaux. 

Les informations de cet article s’appliquent à Data Box et à Data Box Heavy. Dans les sections suivantes, toutes les références à Data Box s’appliquent également à Data Box Heavy. Les journaux collectés à partir du service Data Box s’exécutant dans Azure ne sont pas traités dans cet article. 


## <a name="about-audit-logs"></a>À propos des journaux d’audit 

Sur votre Data Box, les journaux suivants sont collectés :

- **Les journaux système** Data Box étant un appareil Windows, tout le matériel, les logiciels et les événements système sont journalisés. Un ensemble de ces événements est collecté et signalé dans les journaux d’audit du système. 

- **Sécurité** : Data Box étant un appareil Windows, tous les événements de sécurité sont consignés. Un ensemble de ces événements est collecté et signalé dans les journaux d’audit du système. 

- **Application** - Ces journaux sont spécifiques à Data Box uniquement. Ces journaux contiennent tous les événements générés sur l’appareil en réponse aux services de Data Box qui s’exécutent.

Chacun de ces journaux est abordé dans la section suivante.

### <a name="system-logs"></a>Journaux système

Les ID d’événements de journal système suivants sont collectés en tant que journaux d’audit système sur votre Data Box :

|Nom du fournisseur d'événements     |ID de l’événement collecté   |Description de l'événement   |
|-------------------|----------|----------------|
|Microsoft-Windows-Kernel-General|12  |Heure UTC de redémarrage du système d’exploitation.   |
|                                |13  |Heure UTC de l’arrêt du système d’exploitation. |
|    |                              |
|Microsoft-Windows-Kernel-Power  |41  |Le système a redémarré sans arrêt normal.| 
|    |                              |
|Microsoft-Windows-BitLocker-Driver|Tous|    |

### <a name="security-logs"></a>Journaux d’activité de sécurité

Les ID d’événements de journal de sécurité suivants sont collectés en tant que journaux d’audit système sur votre Data Box :

|Nom du fournisseur d'événements                   |ID de l’événement collecté    |Description de l'événement       |
|--------------------------------------|------------|----------|
|Microsoft-Windows-Security-Auditing   |4624        |Réussite de l'ouverture de session. |
|                                      |4625        |Une ouverture de session de compte a échoué. Nom d'utilisateur inconnu ou mot de passe incorrect. |
|                                     

### <a name="application-logs"></a>Journaux d’activité d’application

Les ID d’événements de journal de sécurité suivants sont collectés en tant que partie des journaux d’audit en package sur votre Data Box.     

- **Microsoft-Azure-DataBox-OOBE-Auditing**-contient les événements qui se produisent dans l’interface utilisateur locale. 
- **Microsoft-Azure-DataBox-reprovision-audit**-contient les événements liés au réapprovisionnement de l’appareil Data Box. La reconfiguration de la Data Box se produit lorsque l’appareil est réinitialisé via l’interface utilisateur locale. Choisissez cette option lorsque vous voulez effacer les données que vous avez copiées en supprimant les partages existants et en recréant les partages dans le cadre du réapprovisionnement ou de la réinitialisation de l’appareil.
- **Microsoft-Azure-DataBox-HcsMgmt-audit** - contient les événements liés uniquement à l’étape **Préparer l’expédition** avant que l’appareil ne soit renvoyé au centre de données Azure. 
- **Microsoft-Azure-DataBox-IfxAudit** - contient les messages consignés par différentes entités du produit sur les travaux, les journaux qui indiquent plus d’informations sur ce qui se passe dans certains flux.

Voici un tableau résumant les différents fournisseurs d’événements et les ID d’événements correspondants qui sont collectés dans chaque cas.

|Nom du fournisseur d’événements    |ID de l’événement    | Notes |
|-----------------|-----------------|-------------------|
|Microsoft-Azure-DataBox-OOBE-Auditing |4624        |Réussite de l'ouverture de session.|
|                                      |4625        |Une ouverture de session de compte a échoué. Nom d'utilisateur inconnu ou mot de passe incorrect.|
|                                     |4634        |Déconnectez l’événement.|
|                                   |  | |
|Microsoft-Azure-DataBox-Reprovision-Audit    |65 001       |Événement de remise en service réussi.|
|                                                  |65002       |Événement de remise en service réussi.|
|                                                  |                 |         |
|Microsoft-Azure-DataBox-HcsMgmt-Audit        |65003       |Événement d’état préparation de l’expédition      NotStarted,     InProgress,     Failed,     Canceled,     Succeeded,     ScanCompletedWithIssues,     SucceededWithWarnings          |
|                                                  |                 |     |
|Microsoft-Azure-DataBox-IfxAudit    |Tous |Tous les événements sont journalisés avec l’API du journal d’audit dans le code |

Voici un exemple de journal d’audit de l’infrastructure d’instrumentation (IFX) :

|     Tâche/Travail/API                              |     Événements enregistrés                                                                                                              | 
|-----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
|     Nettoyage                                   |     Les événements liés au démarrage, à l’achèvement ou à l’échec d’un travail de nettoyage sont journalisés. |                                              
|     Préparer l’appareil pour l’expédition client    |     Les événements liés au démarrage, à l’achèvement ou à l’échec de la tâche de préparation de l’appareil pour l’expédition sont consignés. |
|     Approvisionner                                 |     Les événements liés au démarrage, à l’achèvement ou à l’échec d’un travail provisionnement des appareils sont journalisés.|
|     Audit - travail du package                       |     Les événements liés au démarrage, à l’achèvement ou à l’échec d’un travail de package d’audit qui crée une chaîne de journaux de garde sont journalisés.|
|     Disque   remplacer                          |     L’échec de remplacement du disque est consigné.|
|     Activation   ou désactivation de PowerShell à distance     |     Les événements liés à l’activation ou à la désactivation de PowerShell à distance sur l’appareil sont journalisés. |
|     Get   détails de la phase d’installation               |     Les événements liés à l’installation des logiciels sur l’appareil en plusieurs phases sont consignés dans le centre de données Azure.|
|     Déverrouiller ou verrouiller le volume BitLocker           |     Les événements sont consignés pour indiquer l’état BitLocker de *basevolume* et *hcsdata* volume.|
|     Assainir le disque                              |     Les événements liés à la défaillance des disques physiques qui n'ont pas pu être effacés, et les événements lorsque tous les disques physiques de l'appareil sont effacés avec succès, sont consignés. |
|     Activer ou désactiver l’utilisateur local               |     Les événements liés à l’activation ou à la désactivation de comptes d’utilisateurs locaux pour StorSimpleAdmin et PodSupportAdminUser sont consignés.| 
|     Mot de passe   réinitialisation                          |     Les événements liés à la réussite ou à l’échec de la réinitialisation du mot de passe pour l’utilisateur local StorSimpleAdmin sont enregistrés. |


Outre les journaux d’audit IFX, les journaux d’audit de la chaîne de conservation sont également collectés pour Data Box. Ces journaux ne peuvent pas être affichés en temps réel, mais uniquement après la fin du travail et les données sont effacées des disques Data Box. Ces journaux contiennent un sous-ensemble des informations contenues dans les journaux d’audit IFX.

Pour plus d’informations sur la chaîne des journaux d’audit de conservation, consultez [Obtenir une chaîne de journaux de garde après l’effacement des données](data-box-logs.md#get-chain-of-custody-logs-after-data-erasure).

<!-- write a few lines about order history and link out to the detailed section on order history-->

## <a name="access-audit-logs"></a>Accéder aux journaux d’audit

Ces journaux sont stockés dans Azure et ne sont pas accessibles directement. Si vous avez besoin d’accéder à ces journaux, remplissez un ticket de support. Pour plus d'informations, consultez [Contacter le support technique de Microsoft](data-box-disk-contact-microsoft-support.md). 

Une fois le ticket de support rempli, Microsoft télécharge et vous transmet l’accès à ces journaux.


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [résoudre les problèmes sur votre Data Box et votre Data Box Heavy](data-box-troubleshoot.md).
