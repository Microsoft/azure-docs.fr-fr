---
title: Notes de publication de StorSimple série 8000 Update 3
description: Décrit les nouvelles fonctionnalités, les problèmes et les solutions de contournement associés à StorSimple série 8000 Update 3.
author: alkohli
ms.assetid: 2158aa7a-4ac3-42ba-8796-610d1adb984d
ms.service: storsimple
ms.topic: conceptual
ms.date: 01/09/2018
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e081acc357c1b8872dd3a25c552d4cbe9785e212
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96007067"
---
# <a name="update-3-release-notes-for-your-storsimple-8000-series-device"></a>Notes de publication Update 3 pour votre appareil StorSimple série 8000

## <a name="overview"></a>Vue d’ensemble
Les notes de publication suivantes décrivent les nouvelles fonctionnalités et identifient les problèmes critiques non résolus relatifs à StorSimple série 8000 Update 3. Elles contiennent également une liste des mises à jour du logiciel StorSimple incluses dans cette version. 

Update 3 peut être appliquée à n’importe quel appareil StorSimple exécutant la version Release (GA) ou toute version entre Update 0.1 et Update 2.2. La version d’appareil associée à Update 3 est 6.3.9600.17759.

Lisez les informations contenues dans les notes de publication avant de déployer la mise à jour dans votre solution StorSimple.

> [!IMPORTANT]
> * Update 3 comprend les mises à jour du logiciel de l’appareil, du microprogramme et du pilote LSI et de Storport et Spaceport. L’installation de cette mise à jour prend entre 1 h 30 et 2 h. 
> * Pour les nouvelles versions, vous ne voyez pas immédiatement les mises à jour, car nous effectuons un déploiement échelonné des mises à jour. Revérifiez les mises à jour dans quelques jours, car elles seront bientôt disponibles.
> 
> 

## <a name="whats-new-in-update-3"></a>Nouveautés d’Update 3
Update 3 comporte les principaux correctifs de bogues et améliorations suivants.

* **Modifications de la récupération d’espace automatisée** : à partir d’Update 3, les algorithmes de récupération d’espace s’exécutent sur le contrôleur de secours du système, ce qui accélère l’exécution. Pour plus d’informations sur les ports requis pour utiliser la récupération d’espace, reportez-vous aux [Exigences de mise en réseau StorSimple](storsimple-8000-system-requirements.md#networking-requirements-for-your-storsimple-device).
* **Amélioration des performances** : Update 3 a amélioré les performances de lecture-écriture dans le cloud.
* **Améliorations liées à la migration** : dans cette version, plusieurs correctifs de bogues et améliorations ont été apportés à la fonctionnalité de migration d’appareils de la série 5000/7000 vers des appareils de la série 8000. Pour plus d’informations sur l’utilisation de la fonctionnalité de migration, consultez [Migration à partir d’appareils de la série 5000/7000 vers des appareils de la série 8000](https://gallery.technet.microsoft.com/Azure-StorSimple-50007000-c1a0460b). 
* **Correctifs liés à la surveillance** : dans cette version, les bogues liés aux graphiques d’analyse, au tableau de bord du service et au tableau de bord de l’appareil ont été résolus.

## <a name="issues-fixed-in-update-3"></a>Problèmes résolus dans Update 3
Les tableaux suivants récapitulent les problèmes qui ont été résolus dans Update 3.    

| Non | Fonctionnalité | Problème | S’applique à un appareil physique | S’applique à un appareil virtuel |
| --- | --- | --- | --- | --- |
| 1 |Migration des données côté hôte |Dans la version antérieure, l’équipement cloud StorSimple se déconnectait pendant la migration des données côté hôte. Ce problème a été résolu dans cette version. |Non |Oui |
| 2 |Volumes épinglés localement |Dans la version précédente, il existait des problèmes liés aux échecs d’E/S, de conversion de volumes et de chemins de données pour les volumes épinglés localement. La cause racine de ces problèmes a été identifiée et ils ont été résolus dans cette version. |Oui |Non |
| 3 |Surveillance |Il y avait plusieurs problèmes liés aux unités de compte rendu et à la surveillance, ainsi que des graphiques du tableau de bord de l’appareil qui affichaient des informations incorrectes pour les volumes localement épinglés. Ces problèmes ont été résolus dans cette version. |Oui |Non |
| 4 |E/S à haut volume d’écritures |En utilisant StorSimple pour des charges de travail impliquant des volumes élevés d’écritures, l’utilisateur rencontrait un bogue peu fréquent qui entraînait la hiérarchisation de la plage de travail dans le cloud. Ce bogue est résolu dans cette version. |Oui |Oui |
| 5 |Backup |Dans certains cas rares, dans les versions précédentes du logiciel, lorsque l’utilisateur effectuait une sauvegarde d’un clone à distance, il rencontrait des erreurs de cloud et l’opération provoquait une erreur. Dans cette version, le problème est résolu et l’opération réussit. |Oui |Oui |
| 6 |Stratégie de sauvegarde |Dans certains cas rares, dans les versions antérieures du logiciel, il y avait un bogue lié à la suppression d’une stratégie de sauvegarde. Ce problème a été résolu dans cette version. |Oui |Oui |

## <a name="known-issues-in-update-3"></a>Problèmes connus dans Update 3
Le tableau suivant récapitule les problèmes connus dans cette version.

| Non. | Fonctionnalité | Problème | Commentaires/Solution de contournement | S’applique à un appareil physique | S’applique à un appareil virtuel |
| --- | --- | --- | --- | --- | --- |
| 1 |Disque quorum |Dans de rares cas, si la majorité des disques du boîtier EBOD d’un appareil 8600 sont déconnectés, ce qui signifie qu’il n’y a pas de disque quorum, le pool de stockage est hors connexion. Il reste hors connexion même si les disques sont reconnectés. |Vous devez redémarrer l’appareil. Si le problème persiste, contactez le support technique Microsoft. |Oui |Non |
| 2 |ID de contrôleur incorrect |Lorsqu’un contrôleur est remplacé, le contrôleur 0 peut apparaître comme contrôleur 1. Pendant le remplacement du contrôleur, lorsque l’image est chargée à partir du nœud homologue, l’ID du contrôleur peut s’afficher initialement comme l’ID du contrôleur homologue. Dans de rares cas, ce comportement peut également se produire après un redémarrage du système. |Aucune action de l'utilisateur n'est requise. Cette situation se résout automatiquement une fois le contrôleur remplacé. |Oui |Non |
| 3 |Comptes de stockage |La suppression du compte de stockage à l’aide du service de stockage n’est pas prise en charge. En effet, cette opération donnerait lieu à une situation dans laquelle il serait impossible de récupérer les données utilisateur. | |Oui |Oui |
| 4 |Basculement de l’appareil |Le basculement multiple d’un conteneur de volumes d’un même appareil source vers différents appareils cibles n’est pas pris en charge. Si un appareil inactif est basculé vers plusieurs appareils, les conteneurs de volumes du premier appareil basculé perdent la propriété des données. Après un basculement de ce type, les conteneurs de volumes concernés apparaissent ou se comportent différemment lorsque vous les affichez dans le portail Azure Classic. | |Oui |Non |
| 5 |Installation |Lors de l’installation de l’adaptateur StorSimple pour SharePoint, vous devez fournir une adresse IP d’appareil pour que l’installation s’effectue correctement. | |Oui |Non |
| 6 |Proxy web |Si HTTPS est défini comme protocole dans la configuration du proxy web, la communication appareil-service est altérée et l’appareil se met hors connexion. Des packages de prise en charge sont également générés, ce qui consomme de nombreuses ressources de l’appareil. |Vérifiez que le protocole défini pour l’URL du proxy web est bien HTTP. Pour plus d’informations, consultez la section [Configuration du proxy web pour votre appareil](storsimple-8000-configure-web-proxy.md). |Oui |Non |
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
| 22 |Mises à jour |Lorsque vous appliquez Update 3, la page **Maintenance** dans le portail Azure Classic affiche le message suivant lié à Update 2 : « La série StorSimple 8000 Update 2 permet à Microsoft de collecter de façon proactive les informations des journaux de votre appareil quand nous détectons des problèmes potentiels ». Cela est trompeur car cela indique que l’appareil est mis à jour vers Update 2. Une fois l’appareil mis à jour vers Update 3, le message disparaît. |Ce problème sera résolu dans une prochaine version. |Oui |Non |

## <a name="controller-and-firmware-updates-in-update-3"></a>Mises à jour du contrôleur et du microprogramme dans Update 3
Cette version contient des mises à jour du microprogramme et du pilote LSI. Pour plus d’informations sur la procédure d’installation du microprogramme et du pilote LSI, consultez [Installer Update 3 sur votre appareil StorSimple](./index.yml) .

## <a name="virtual-device-updates-in-update-3"></a>Mises à jour des appareils virtuels dans Update 3
Cette mise à jour ne peut pas s’appliquer à l’équipement cloud StorSimple (également connu sous le nom d’appareil virtuel). De nouveaux appareils virtuels devront être créés. 

## <a name="next-step"></a>Étape suivante
Découvrez comment [installer Update 3](./index.yml) sur votre appareil StorSimple.