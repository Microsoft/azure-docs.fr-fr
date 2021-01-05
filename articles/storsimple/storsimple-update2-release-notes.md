---
title: Notes de publication de StorSimple série 8000 Update 2 | Microsoft Docs
description: Décrit les nouvelles fonctionnalités, les problèmes et les solutions de contournement associés à StorSimple série 8000 Update 2.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: e2c8bffd-7fc5-4b77-b632-a4f59edacc3a
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: alkohli
ms.openlocfilehash: b68e338c2675a3aeefda390d12ab2d82ea1c7a9f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94956651"
---
# <a name="storsimple-8000-series-update-2-release-notes"></a>Notes de publication de StorSimple série 8000 Update 2

## <a name="overview"></a>Vue d’ensemble
Les notes de publication suivantes décrivent les nouvelles fonctionnalités et identifient les problèmes critiques non résolus relatifs à StorSimple série 8000 Update 2. Elles contiennent également une liste des mises à jour du logiciel, du pilote et du microprogramme de disque StorSimple incluses dans cette version. 

La solution Update 2 peut être appliquée à n'importe quel appareil StorSimple exécutant le logiciel Release (GA), des versions Update 0.1 à Update 1.2. La version d’appareil associée à Update 2 est 6.3.9600.17673.

Lisez les informations contenues dans les notes de publication avant de déployer la mise à jour dans votre solution StorSimple.

> [!IMPORTANT]
> * L’installation de cette mise à jour dure entre 4 et 7 heures (en comptant les mises à jour Windows). 
> * La solution Update 2 contient des mises à jour du logiciel, du microprogramme, du pilote LSI et du microprogramme SSD.
> * Pour les nouvelles versions, vous ne voyez pas immédiatement les mises à jour, car nous effectuons un déploiement échelonné des mises à jour. Revérifiez les mises à jour dans quelques jours, car elles seront bientôt disponibles.
> 
> 

## <a name="whats-new-in-update-2"></a>Nouveautés d'Update 2
La solution Update 2 présente les nouvelles fonctionnalités suivantes :

* **Volumes épinglés localement** : dans les versions précédentes du StorSimple série 8000, les blocs de données étaient hiérarchisées vers le cloud selon leur utilisation. Il n'y avait aucun moyen de garantir que ces blocs resteraient locaux. Dans la solution Update 2, lorsque vous créez un volume, vous pouvez désigner un volume comme étant épinglé localement. Les données primaires de ce volume ne seront alors pas hiérarchisées vers le cloud. Les snapshots des volumes épinglés localement sont toujours copiés vers le cloud pour une sauvegarde ; ainsi, le cloud peut être utilisé à des fins de récupération d'urgence et de mobilité des données. En outre, vous pouvez modifier le type de volume (autrement dit, convertir des volumes hiérarchisés en volumes épinglés localement, et inversement). 
* **Améliorations des appareils virtuels StorSimple** : auparavant, les appareils StorSimple série 8000 faisaient de l’appareil virtuel une solution de développement/test ou de récupération d'urgence. Il n’existait alors qu’un seul modèle d’appareil virtuel (modèle 1100). La solution Update 2 propose deux modèles d’appareil virtuel : 
  
  * Le modèle 8010 (anciennement le modèle 1100) : il ne présente aucune modification, dispose d’une capacité de 30 To et utilise le stockage standard d’Azure.
  * Le modèle 8020 : il dispose d’une capacité de 64 To et utilise le stockage Premium d’Azure pour de meilleures performances.
    
    Il existe un seul disque dur virtuel pour les deux modèles d’appareil virtuel (8010/8020). Lorsque vous démarrez l’appareil virtuel, il détecte les paramètres de plateforme et applique la version de modèle approprié.
* **Améliorations réseau** : la solution Update 2 contient les améliorations de mise en réseau suivantes :
  
  * Plusieurs cartes réseau peuvent être activées pour le cloud afin que le basculement se produise si une carte réseau échoue.
  * Améliorations du routage, avec des mesures fixes pour les blocs compatibles avec le cloud.
  * Tentatives en ligne des ressources en cas d’échec avant un basculement.
  * Nouvelles alertes pour les échecs de service.
* **Améliorations des mises à jour** : dans Update 1.2 et les versions antérieures, les appareils StorSimple série 8000 étaient mis à jour via deux canaux : Windows Update pour le clustering, iSCSI et autres, et Microsoft Update pour les fichiers binaires et les microprogrammes.
    Update 2 utilise Microsoft Update pour tous les packages de mise à jour. Cette solution devrait permettre de réduire le temps des mises à jour correctives ou des basculements à effectuer. 
* **Mises à jour de microprogramme** : les mises à jour de microprogramme suivantes sont incluses :
  
  * LSI: lsi_sas2.sys Version du produit 2.00.72.10
  * Disque SSD uniquement (aucune mise à jour du disque dur) : XMGG, XGEG, KZ50, F6C2 et VR08
* **Prise en charge proactive** : Update 2 permet à Microsoft d'extraire des informations de diagnostic supplémentaires de l’appareil. Lorsque notre équipe en charge des opérations identifie des appareils qui ont un problème, nous sommes plus à même de collecter des informations à partir de ces appareils et de diagnostiquer les problèmes. **En acceptant d’installer Update 2, vous nous permettez de fournir cette assistance proactive**.    

## <a name="issues-fixed-in-update-2"></a>Problèmes résolus dans Update 2
Le tableau suivant récapitule les problèmes qui ont été résolus dans Updates 2.    

| Non. | Fonctionnalité | Problème | S’applique à un appareil physique | S’applique à un appareil virtuel |
| --- | --- | --- | --- | --- |
| 1 |Interfaces réseau |Après une mise à niveau vers Update 1, le service StorSimple Manager a signalé que les ports Data2 et Data3 ont échoué sur un contrôleur. Ce problème est à présent résolu. |Oui |Non |
| 2 |Mises à jour |Après une mise à niveau vers Update 1, des alertes d'alarme sonore se sont produites dans le portail Azure Classic sur plusieurs appareils. Ce problème est à présent résolu. |Oui |Non |
| 3 |Authentification OpenStack |Lorsque vous utilisiez OpenStack comme fournisseur de services cloud, vous pouviez recevoir une erreur indiquant que la chaîne d'authentification cloud était trop longue. Ce problème a été résolu. |Oui |Non |

## <a name="known-issues-in-update-2"></a>Problèmes connus dans Update 2
Le tableau suivant récapitule les problèmes connus dans cette version.

| Non. | Fonctionnalité | Problème | Commentaires/Solution de contournement | S’applique à un appareil physique | S’applique à un appareil virtuel |
| --- | --- | --- | --- | --- | --- |
| 1 |Disque quorum |Dans de rares cas, si la majorité des disques du boîtier EBOD d’un appareil 8600 sont déconnectés, ce qui signifie qu’il n’y a pas de disque quorum, le pool de stockage est hors connexion. Il reste hors connexion même si les disques sont reconnectés. |Vous devez redémarrer l’appareil. Si le problème persiste, contactez le support technique Microsoft. |Oui |Non |
| 2 |ID de contrôleur incorrect |Lorsqu’un contrôleur est remplacé, le contrôleur 0 peut apparaître comme contrôleur 1. Pendant le remplacement du contrôleur, lorsque l’image est chargée à partir du nœud homologue, l’ID du contrôleur peut s’afficher initialement comme l’ID du contrôleur homologue. Dans de rares cas, ce comportement peut également se produire après un redémarrage du système. |Aucune action de l'utilisateur n'est requise. Cette situation se résout automatiquement une fois le contrôleur remplacé. |Oui |Non |
| 3 |Comptes de stockage |La suppression du compte de stockage à l’aide du service de stockage n’est pas prise en charge. En effet, cette opération donnerait lieu à une situation dans laquelle il serait impossible de récupérer les données utilisateur. | |Oui |Oui |
| 4 |Basculement de l’appareil |Le basculement multiple d’un conteneur de volumes d’un même appareil source vers différents appareils cibles n’est pas pris en charge. Si un appareil inactif est basculé vers plusieurs appareils, les conteneurs de volumes du premier appareil basculé perdent la propriété des données. Après un basculement de ce type, les conteneurs de volumes concernés apparaissent ou se comportent différemment lorsque vous les affichez dans le portail Azure Classic. | |Oui |Non |
| 5 |Installation |Lors de l’installation de l’adaptateur StorSimple pour SharePoint, vous devez fournir une adresse IP d’appareil pour que l’installation s’effectue correctement. | |Oui |Non |
| 6 |Proxy web |Si HTTPS est défini comme protocole dans la configuration du proxy web, la communication appareil-service est altérée et l’appareil se met hors connexion. Des packages de prise en charge sont également générés, ce qui consomme de nombreuses ressources de l’appareil. |Vérifiez que le protocole défini pour l’URL du proxy web est bien HTTP. Pour plus d’informations, consultez la section [Configuration du proxy web pour votre appareil](./storsimple-8000-configure-web-proxy.md). |Oui |Non |
| 7 |Proxy web |Si vous configurez et activez le proxy web sur un appareil inscrit, vous devez redémarrer le contrôleur actif sur votre appareil. | |Oui |Non |
| 8 |Latence de cloud élevée et charge de travail d’E/S élevée |Lorsque l’appareil StorSimple doit gérer à la fois des latences de cloud très élevées (de l’ordre de quelques secondes) et une charge de travail d’E/S élevée, ses volumes se détériorent et une défaillance peut se produire au niveau des E/S, avec l’erreur « appareil non prêt ». |Vous devez redémarrer les contrôleurs de l’appareil manuellement ou effectuer un basculement d’appareil pour résoudre ce problème. |Oui |Non |
| 9 |Azure PowerShell |Lorsque vous utilisez l’applet de commande StorSimple **Get-AzureStorSimpleStorageAccountCredential &#124; Select-Object -First 1 -Wait** pour sélectionner le premier objet afin de créer un objet **contrôleur de volumes**, l’applet de commande renvoie l’ensemble des objets. |Encapsulez l’applet de commande entre parenthèses comme suit : **(Get-Azure-StorSimpleStorageAccountCredential) &#124; Select-Object -First 1 -Wait** |Oui |Oui |
| 10 |Migration |Lorsque plusieurs conteneurs de volumes sont transmis pour migration, l’heure prévue de la dernière sauvegarde est exacte uniquement pour le premier conteneur de volumes. Par ailleurs, la migration parallèle démarre après la migration des quatre premières sauvegardes du premier conteneur de volumes. |Nous vous recommandons de migrer un seul conteneur de volumes à la fois. |Oui |Non |
| 11 |Migration |Après la restauration, les volumes ne sont pas ajoutés à la stratégie de sauvegarde ni au groupe de disques virtuels. |Pour créer les sauvegardes, il vous faudra ajouter ces volumes à une stratégie de sauvegarde. |Oui |Oui |
| 12 |Migration |Une fois la migration terminée, l’appareil de série 5000/7000 ne doit pas accéder aux conteneurs de données migrées. |Nous vous recommandons de supprimer les conteneurs de données migrées une fois que la migration est terminée et validée. |Oui |Non |
| 13 |Clonage et récupération d’urgence |Un appareil StorSimple exécutant Update 1 ne peut pas cloner un appareil exécutant une version logicielle antérieure, ni exécuter de récupération d'urgence sur ce dernier. |Pour être en mesure d’effectuer ces opérations, vous devrez mettre à jour l’appareil cible vers Update 1. |Oui |Oui |
| 14 |Migration |La sauvegarde de la configuration pour la migration peut être mise en échec sur un appareil de série 5000-7000 lorsqu’aucun volume n’est associé à certains groupes de volumes. |Supprimez l’ensemble des groupes de volumes vides ne présentant aucun volume associé, puis réessayez la sauvegarde de configuration. |Oui |Non |
| 15 |Applets de commande Azure PowerShell et volumes épinglés localement |Vous ne pouvez pas créer un volume épinglé localement via les applets de commande Azure PowerShell. (Tous les volumes que vous créez via Azure PowerShell sont hiérarchisés.) |Utilisez toujours le service StorSimple Manager pour configurer les volumes épinglés localement. |Oui |Non |
| 16 |Espace disponible pour les volumes épinglés localement |Si vous supprimez un volume épinglé localement, l'espace disponible pour les nouveaux volumes ne sera peut-être pas mis à jour immédiatement. Le service StorSimple Manager met à jour l'espace local disponible environ toutes les heures. |Patientez une heure avant d'essayer de créer le nouveau volume. |Oui |Non |
| 17 |Volumes épinglés localement |Votre travail de restauration expose la sauvegarde d'instantanés temporaires dans le catalogue de sauvegarde, mais uniquement pour la durée du travail de restauration. En outre, il expose un groupe de disques virtuels avec le préfixe **tmpCollection** sur la page **Stratégies de sauvegarde**, mais uniquement pour la durée du travail de restauration. |Ce comportement peut se produire si votre travail de restauration possède des volumes épinglés localement ou une combinaison de volumes épinglés localement et de volumes hiérarchisés. Si le travail de restauration comporte uniquement des volumes hiérarchisés, cette situation ne se produira pas. Aucune intervention de l’utilisateur n’est nécessaire. |Oui |Non |
| 18 |Volumes épinglés localement |Si vous annulez un travail de restauration et qu’un basculement de contrôleur se produit immédiatement après, le travail de restauration indique **Échec** et non pas **Annulé**. Si un travail de restauration échoue et qu’un basculement de contrôleur se produit immédiatement après, le travail de restauration indique **Annulé** et non pas **Échec**. |Ce comportement peut se produire si votre travail de restauration possède des volumes épinglés localement ou une combinaison de volumes épinglés localement et de volumes hiérarchisés. Si le travail de restauration comporte uniquement des volumes hiérarchisés, cette situation ne se produira pas. Aucune intervention de l’utilisateur n’est nécessaire. |Oui |Non |
| 19 |Volumes épinglés localement |Si vous annulez un travail de restauration ou si une restauration échoue et qu’un basculement de contrôleur a lieu, un travail de restauration supplémentaire s’affiche dans la page **Travaux** . |Ce comportement peut se produire si votre travail de restauration possède des volumes épinglés localement ou une combinaison de volumes épinglés localement et de volumes hiérarchisés. Si le travail de restauration comporte uniquement des volumes hiérarchisés, cette situation ne se produira pas. Aucune intervention de l’utilisateur n’est nécessaire. |Oui |Non |
| 20 |Volumes épinglés localement |Si vous tentez de convertir un volume à plusieurs niveaux (créé et cloné avec Update 1.2 ou une version antérieure) sur un volume épinglé localement et si la capacité de votre appareil est insuffisante ou que le cloud est indisponible, alors le ou les clones peuvent être endommagés. |Ce problème se produit uniquement avec les volumes qui ont été créés et clonés avec un logiciel antérieur à Update 2. Il s'agit d'un scénario peu fréquent. | | |
| 21 |Conversion des volumes |Ne mettez pas à jour les enregistrements de contrôle d'accès liés à un volume durant la conversion d'un volume (de plusieurs niveaux vers épinglé localement ou vice versa). La mise à jour des enregistrements de contrôle d'accès pourrait entraîner une corruption des données. |Si nécessaire, mettez à jour les enregistrements de contrôle d'accès avant la conversion du volume et n'effectuez aucune autre mise à jour des enregistrements de contrôle d'accès pendant la conversion. | | |

## <a name="controller-and-firmware-updates-in-update-2"></a>Mises à jour du contrôleur et du microprogramme dans Update 2
Cette version met à jour le pilote et le microprogramme du disque sur votre appareil.

* Pour plus d'informations sur la mise à jour du microprogramme LSI, consultez l'article 3121900 de la base de connaissances Microsoft. 
* Pour plus d'informations sur la mise à jour du microprogramme du disque, consultez l'article 3121899 de la base de connaissances Microsoft.

## <a name="virtual-device-updates-in-update-2"></a>Mises à jour des appareils virtuels dans Update 2
Cette mise à jour ne peut pas être appliquée à l’appareil virtuel. De nouveaux appareils virtuels devront être créés. 

## <a name="next-step"></a>Étape suivante
Découvrez comment [installer Update 2](./storsimple-8000-install-update-5.md) sur votre appareil StorSimple.