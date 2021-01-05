---
title: Notes de publication de StorSimple série 8000 Update 2.2 | Microsoft Docs
description: Décrit les nouvelles fonctionnalités, les problèmes et les solutions de contournement associés à StorSimple série 8000 Update 2.2.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 5cf03ea8-2a0f-4552-b6dc-7ea517783d7b
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: alkohli
ms.openlocfilehash: 73b9ecd03875b60ed2d9b9d4c8e8a3a0c8de3cfa
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94956600"
---
# <a name="storsimple-8000-series-update-22-release-notes"></a>Notes de publication de StorSimple série 8000 Update 2.2

## <a name="overview"></a>Vue d’ensemble
Les notes de publication suivantes décrivent les nouvelles fonctionnalités et identifient les problèmes critiques non résolus relatifs à StorSimple série 8000 Update 2.2. Elles contiennent également une liste des mises à jour du logiciel StorSimple incluses dans cette version.

La solution Update 2.2 peut être appliquée à n’importe quel appareil StorSimple exécutant le logiciel Release (GA), des versions Update 0.1 à Update 2.1. La version d’appareil associée à Update 2.2 est 6.3.9600.17708.

Lisez les informations contenues dans les notes de publication avant de déployer la mise à jour dans votre solution StorSimple.

> [!IMPORTANT]
> * Update 2.2 contient uniquement des mises à jour logicielles. L’installation de cette mise à jour prend entre 1 h 30 et 2 h. 
> * Si vous exécutez Update 2.1, nous vous recommandons d’appliquer Update 2.2 dès que possible.
> * Pour les nouvelles versions, vous ne voyez pas immédiatement les mises à jour, car nous effectuons un déploiement échelonné des mises à jour. Revérifiez les mises à jour dans quelques jours, car elles seront bientôt disponibles.
> 
> 

## <a name="whats-new-in-update-22"></a>Nouveautés d’Update 2.2
Update 2.2 comporte les principales améliorations suivantes.

* **Optimisation automatique de la récupération d’espace** : lorsque des données sont supprimées dans des volumes alloués de façon dynamique, les blocs de stockage inutilisés doivent être récupérés. Cette version améliore le processus de récupération d’espace à partir du cloud de manière à accélérer la disponibilité de l’espace inutilisé par rapport aux versions précédentes.
* **Améliorations des performances de capture d’instantanés** : Update 2.2 améliore le temps de traitement d’un instantané cloud dans certains scénarios où de grands volumes sont utilisés et où les données évoluent peu voire n’évoluent pas du tout. Cette amélioration peut être utile pour les volumes d’archives, par exemple.
* **Renforcement de la collecte du package Support** : cette version améliore le mode de collecte et de chargement du package Support. 
* **Amélioration de fiabilité des mises à jour** : cette version comporte des correctifs de bogues qui améliorent la fiabilité d’Update.

## <a name="issues-fixed-in-update-22"></a>Problèmes résolus dans Update 2.2
Le tableau suivant récapitule les problèmes qui ont été résolus dans Update 2.2 et Update 2.1.    

| Non | Fonctionnalité | Problème | S’applique à un appareil physique | S’applique à un appareil virtuel |
| --- | --- | --- | --- | --- |
| 1 |Performances de l’hôte |Dans la version précédente, nous avons constaté des problèmes de performances côté hôte lors de la création d’un volume épinglé localement et lors de la conversion d’un volume à plusieurs niveaux en volume épinglé localement. Ces problèmes ont été résolus dans cette version afin d’améliorer les performances de l’hôte pendant les procédures de création et de conversion de volumes. |Oui |Non |
| 2 |Volumes épinglés localement |Dans de rares cas, le système plantait lors de la création d’un volume épinglé localement. Ce bogue a été résolu dans cette version. |Oui |Non |
| 3 |Hiérarchisation |Des incidents sporadiques se produisaient auparavant lorsque les métadonnées des appliances cloud StorSimple (8010 et 8020) étaient hiérarchisées dans le cloud. Ce problème a été résolu dans cette version. |Non |Oui |
| 4 |Création d’instantanés |Des problèmes ont été constatés lors de la création d’instantanés incrémentiels dans des scénarios impliquant d’importants volumes et une évolution des données réduite voire nulle. Ces problèmes ont été résolus dans cette version. |Oui |Oui |
| 5 |Authentification OpenStack |Lorsque vous utilisez Openstack comme fournisseur de services cloud, l’utilisateur rencontrait parfois un bogue d’authentification qui entraînait un plantage de l’analyseur JSON. Ce bogue est résolu dans cette version. |Oui |Non |
| 6 |Copie côté hôte |Dans les versions précédentes du logiciel, un bogue lié à l’horodatage ODX survenait parfois lors de la copie des données d’un volume vers un autre. Ce problème entraînait un basculement du contrôleur, avec le risque de voir le système passer en mode de récupération. Ce bogue est résolu dans cette version. |Oui |Non |
| 7 |Windows Management Instrumentation (WMI) |Dans les versions précédentes du logiciel, plusieurs cas d’échec du proxy web ont été observés avec l’exception «\<ManagementException> Échec du chargement du fournisseur ». Ce bogue a été attribué à une fuite de mémoire WMI et est maintenant résolu. |Oui |Non |
| 8 |Update |Dans de rares cas, l’utilisateur des versions précédentes du logiciel recevait un message « CisPowershellHcsscripterror » lorsqu’il tentait d’effectuer une analyse ou d’installer des mises à jour. Ce problème a été résolu dans cette version. |Oui |Oui |
| 9 |Package de prise en charge |Cette version améliore le mode de collecte et de chargement du package Support. |Oui |Oui |

## <a name="known-issues-in-update-22"></a>Problèmes connus dans Update 2.2
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
| 20 |Volumes épinglés localement |Si vous tentez de convertir un volume à plusieurs niveaux (créé et cloné avec Update 1.2 ou une version antérieure) sur un volume épinglé localement et si la capacité de votre appareil est insuffisante ou que le cloud est indisponible, alors le ou les clones peuvent être endommagés. |Ce problème se produit uniquement avec les volumes qui ont été créés et clonés avec un logiciel antérieur à Update 2.1. Il s'agit d'un scénario peu fréquent. | | |
| 21 |Conversion des volumes |Ne mettez pas à jour les enregistrements de contrôle d'accès liés à un volume durant la conversion d'un volume (de plusieurs niveaux vers épinglé localement ou vice versa). La mise à jour des enregistrements de contrôle d'accès pourrait entraîner une corruption des données. |Si nécessaire, mettez à jour les enregistrements de contrôle d'accès avant la conversion du volume et n'effectuez aucune autre mise à jour des enregistrements de contrôle d'accès pendant la conversion. | | |

## <a name="controller-and-firmware-updates-in-update-22"></a>Mises à jour du contrôleur et du microprogramme dans Update 2.2
Cette version comporte uniquement des mises à jour logicielles. Toutefois, si vous effectuez la mise à jour à partir d’une version antérieure à Update 2, vous devrez installer sur votre appareil les mises à jour du pilote, de Storport, de Spaceport et, dans certains cas,du microprogramme de disque.

Pour plus d’informations sur la procédure d’installation du pilote, de Storport, de Spaceport et du microprogramme de disque, consultez [Installer Update 2.2 sur votre appareil StorSimple](./storsimple-8000-install-update-5.md) .

## <a name="virtual-device-updates-in-update-22"></a>Mises à jour des appareils virtuels dans Update 2.2
Cette mise à jour ne peut pas être appliquée à l’appareil virtuel. De nouveaux appareils virtuels devront être créés. 

## <a name="next-step"></a>Étape suivante
Découvrez comment [installer Update 2.2](./storsimple-8000-install-update-5.md) sur votre appareil StorSimple.