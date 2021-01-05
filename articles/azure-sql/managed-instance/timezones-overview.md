---
title: Fuseaux horaires d'Azure SQL Managed Instance
description: En savoir plus sur les spécificités des fuseaux horaires d'Azure SQL Managed Instance
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: reference
author: MladjoA
ms.author: mlandzic
ms.reviewer: ''
ms.date: 10/12/2020
ms.openlocfilehash: bf0cfd70c9850cc6a5ff4482b494d68700022ad8
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790625"
---
# <a name="time-zones-in-azure-sql-managed-instance"></a>Fuseaux horaires d'Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Le temps universel coordonné (UTC) est le fuseau horaire recommandé pour le niveau des données de solutions dans le cloud. Azure SQL Managed Instance offre également un choix de fuseaux horaires pour répondre aux besoins des applications existantes qui stockent les valeurs de date et d'heure et les fonctions d'appel de date et d'heure avec le contexte implicite d'un fuseau horaire spécifique.

Les fonctions T-SQL comme [GETDATE()](/sql/t-sql/functions/getdate-transact-sql) ou le code CLR utilisent le fuseau horaire défini au niveau de l'instance. Les tâches SQL Server Agent suivent également des planifications en fonction du fuseau horaire de l'instance.

  > [!NOTE]
  > Azure SQL Database ne prend pas en charge les paramètres de fuseau horaire ; il suit toujours le temps universel coordonné (UTC). Utilisez [AT TIME ZONE](/sql/t-sql/queries/at-time-zone-transact-sql) dans SQL Database si vous devez interpréter des informations de date et d'heure dans un fuseau horaire autre qu'UTC.

## <a name="supported-time-zones"></a>Fuseaux horaires pris en charge

Un ensemble de fuseaux horaires pris en charge est hérité du système d'exploitation sous-jacent de l'instance gérée. Il est régulièrement mis à jour pour obtenir de nouvelles définitions de fuseaux horaires et refléter les changements apportés aux fuseaux horaires existants.

[La stratégie Heure d’été/modifications de fuseau horaire](https://aka.ms/time) garantit la précision de l’historique à partir de 2010.

Une liste avec les noms des fuseaux horaires pris en charge est affichée dans la vue système [sys.time_zone_info](/sql/relational-databases/system-catalog-views/sys-time-zone-info-transact-sql).

## <a name="set-a-time-zone"></a>Régler un fuseau horaire

Le fuseau horaire d'une instance gérée ne peut être réglé que lors de la création de l'instance. Le fuseau horaire par défaut est UTC.

  >[!NOTE]
  > Le fuseau horaire d'une instance gérée existante ne peut pas être modifié.

### <a name="set-the-time-zone-through-the-azure-portal"></a>Régler le fuseau horaire via le portail Azure

Lorsque vous saisissez les paramètres d’une nouvelle instance, sélectionnez un fuseau horaire dans la liste des fuseaux horaires pris en charge.
  
![Réglage d'un fuseau horaire lors de la création d'une instance](./media/timezones-overview/01-setting_timezone-during-instance-creation.png)

### <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager

Spécifiez la propriété timezoneId dans votre [modèle Resource Manager](./scripts/create-powershell-azure-resource-manager-template.md) pour régler le fuseau horaire lors de la création de l'instance.

```json
"properties": {
                "administratorLogin": "[parameters('user')]",
                "administratorLoginPassword": "[parameters('pwd')]",
                "subnetId": "[parameters('subnetId')]",
                "storageSizeInGB": 256,
                "vCores": 8,
                "licenseType": "LicenseIncluded",
                "hardwareFamily": "Gen5",
                "collation": "Serbian_Cyrillic_100_CS_AS",
                "timezoneId": "Central European Standard Time"
            },

```

Vous trouverez à la fin de cet article une liste des valeurs prises en charge pour la propriété timezoneId.

Si aucun fuseau horaire n’est spécifié, le fuseau horaire est réglé sur UTC.

## <a name="check-the-time-zone-of-an-instance"></a>Vérifier le fuseau horaire d'une instance

La fonction [CURRENT_TIMEZONE](/sql/t-sql/functions/current-timezone-transact-sql) retourne un nom d'affichage du fuseau horaire de l'instance.

## <a name="cross-feature-considerations"></a>Considérations multifonctionnalités

### <a name="restore-and-import"></a>Restaurer et importer

Vous pouvez restaurer un fichier de sauvegarde ou importer des données vers une instance gérée à partir d'une instance ou d'un serveur avec différents paramètres de fuseau horaire. Procédez avec précaution ! Analysez le comportement de l'application et les résultats des requêtes et des rapports, comme lorsque vous transférez des données entre deux instances SQL Server avec différents paramètres de fuseau horaire.

### <a name="point-in-time-restore"></a>Restauration dans le temps

Lorsque vous effectuez une restauration dans le temps, l'heure de la restauration est interprétée comme une heure UTC. Toute ambiguïté liée à l'heure d'été et à ses éventuelles modifications est ainsi évitée.

### <a name="auto-failover-groups"></a>Groupes de basculement automatique

L'utilisation du même fuseau horaire dans des instances principale et secondaire d'un groupe de basculement n'est pas imposée, mais nous le recommandons fortement.

  >[!WARNING]
  > Nous vous conseillons vivement d'utiliser le même fuseau horaire pour les instances principale et secondaire d’un groupe de basculement. En raison de certains cas d'usage rares, le même fuseau horaire n'est pas conservé entre les instances principale et secondaire. Il est important de comprendre qu'en cas de basculement manuel ou automatique, l'instance secondaire conservera son fuseau horaire original.

## <a name="limitations"></a>Limites

- Le fuseau horaire de l’instance gérée existante ne peut pas être modifié. En guise de solution de contournement, créez une nouvelle instance gérée avec le fuseau horaire approprié, puis soit une sauvegarde et une restauration manuelles, soit, ce que nous recommandons, une [restauration à un moment donné d’une autre instance](./point-in-time-restore.md?tabs=azure-portal#restore-an-existing-database).
- Les processus externes lancés à partir des travaux SQL Server Agent ne suivent pas le fuseau horaire de l'instance.

## <a name="list-of-supported-time-zones"></a>Liste des fuseaux horaires pris en charge

| **ID de fuseau horaire** | **Nom d'affichage du fuseau horaire** |
| --- | --- |
| Ligne de changement de date internationale | (UTC-12:00) Ligne de changement de date ouest |
| UTC-11 | (UTC-11:00) Temps universel coordonné-11 |
| Heure standard des îles Aléoutiennes | (UTC-10:00) Îles Aléoutiennes |
| Heure standard d’Hawaï | (UTC-10:00) Hawaï |
| Heure standard des Marquises | (UTC-09:30) Îles Marquises |
| Heure standard de l'Alaska | (UTC-09:00) Alaska |
| UTC-09 | (UTC-09:00) Temps universel coordonné-09 |
| Heure standard du Pacifique (Mexique) | (UTC-08:00) Baja California |
| UTC-08 | (UTC-08:00) Temps universel coordonné-08 |
| Heure standard du Pacifique | (UTC-08:00) Heure du Pacifique (États-Unis et Canada) |
| Heure standard des Rocheuses (États-Unis) | (UTC-07:00) Arizona |
| Heure standard des Rocheuses (Mexique) | (UTC-07:00) Chihuahua, La Paz, Mazatlan |
| Heure standard des Rocheuses | (UTC-07:00) Heure des Rocheuses (États-Unis et Canada) |
| Heure standard du Centre (Amérique) | (UTC-06:00) Amérique centrale |
| Heure standard du Centre | (UTC-06:00) Heure du Centre (États-Unis et Canada) |
| Heure standard de l’Île de Pâques | (UTC-06:00) Île de Pâques |
| Heure standard du Centre (Mexique) | (UTC-06:00) Guadalajara, Mexico City, Monterrey |
| Heure standard du Centre (Canada) | (UTC-06:00) Saskatchewan |
| Heure standard du Pacifique (Amérique du Sud) | (UTC-05:00) Bogota, Lima, Quito, Rio Branco |
| Heure standard de l'Est (Mexique) | (UTC-05:00) Chetumal |
| Heure standard de l'Est | (UTC-05:00) Heure de l’Est (États-Unis et Canada) |
| Heure standard de Haïti | (UTC-05:00) Haïti |
| Heure standard de Cuba | (UTC-05:00) La Havane |
| Heure standard de l'Est (États-Unis) | (UTC-05:00) Indiana (Est) |
| Heure standard des Îles Turques-et-Caïques | (UTC-05:00) Îles Turques-et-Caïques |
| Heure standard du Paraguay | (UTC-04:00) Asuncion |
| Heure standard de l’Atlantique | (UTC-04:00) Heure de l’Atlantique (Canada) |
| Heure standard du Venezuela | (UTC-04:00) Caracas |
| Heure standard du Brésil (Centre) | (UTC-04:00) Cuiaba |
| Heure standard de l’Amérique du Sud (Ouest) | (UTC-04:00) Georgetown, La Paz, Manaus, San Juan |
| Heure standard de l’Amérique du Sud (Pacifique) | (UTC-04:00) Santiago |
| Heure standard de Terre-Neuve | (UTC-03:30) Terre-Neuve |
| Heure standard de Tocantins | (UTC-03:00) Araguaina |
| E. Heure standard de l’Amérique du Sud | (UTC-03:00) Brasilia |
| Heure standard de l’Amérique du Sud (Est) | (UTC-03:00) Cayenne, Fortaleza |
| Heure standard de l’Argentine | (UTC-03:00) Buenos Aires |
| Heure standard du Groenland | (UTC-03:00) Groenland |
| Heure standard de Montevideo | (UTC-03:00) Montevideo |
| Heure standard de Magallanes | (UTC-03:00) Punta Arenas |
| Heure standard de Saint-Pierre-et-Miquelon | (UTC-03:00) Saint-Pierre-et-Miquelon |
| Heure standard de Bahia | (UTC-03:00) Salvador |
| UTC-02 | (UTC-02:00) Temps universel coordonné-02 |
| Heure standard de Centre-Atlantique | (UTC-02:00) Centre-Atlantique - Ancien |
| Heure standard des Açores | (UTC-01:00) Açores |
| Heure standard des Îles du Cabo Verde | (UTC-01:00) Îles du Cabo Verde |
| UTC | (UTC) Temps universel coordonné |
| Heure standard GMT | (UTC+00:00) Dublin, Édimbourg, Lisbonne, Londres |
| Heure standard de Greenwich | (UTC+00:00) Monrovia, Reykjavik |
| Heure standard d’Europe de l’Ouest | (UTC+01:00) Amsterdam, Berlin, Berne, Rome, Stockholm, Vienne |
| Heure standard d’Europe centrale | (UTC+01:00) Belgrade, Bratislava, Budapest, Ljubljana, Prague |
| Romance Standard Time (RST) | (UTC+01:00) Bruxelles, Copenhague, Madrid, Paris |
| Heure standard du Maroc | (UTC+01:00) Casablanca |
| Heure standard de Sao Tomé-et-Principe | (UTC+01:00) Sao Tomé |
| Heure standard d’Europe centrale | (UTC+01:00) Sarajevo, Skopje, Varsovie, Zagreb |
| Heure standard d’Afrique Centrale de l’Ouest | (UTC+01:00) Afrique Centrale de l’Ouest |
| Heure standard de Jordanie | (UTC+02:00) Amman |
| Heure standard GTB | (UTC+02:00) Athènes, Bucarest |
| Heure standard du Moyen-Orient | (UTC+02:00) Beyrouth |
| Heure standard d’Égypte | (UTC+02:00) Le Caire |
| E. d’Europe de l’Ouest | (UTC+02:00) Chisinau |
| Heure standard de Syrie | (UTC+02:00) Damas |
| Heure standard de Cisjordanie | (UTC+02:00) Gaza, Hébron |
| Heure standard d’Afrique du Sud | (UTC+02:00) Harare, Pretoria |
| Heure standard FLE | (UTC+02:00) Helsinki, Kiev, Riga, Sofia, Tallinn, Vilnius |
| Heure standard d’Israël | (UTC+02:00) Jérusalem |
| Heure standard de Kaliningrad | (UTC+02:00) Kaliningrad |
| Heure standard du Soudan | (UTC+02:00) Khartoum |
| Heure standard de Lybie | (UTC+02:00) Tripoli |
| Heure standard de Namibie | (UTC+02:00) Windhoek |
| Heure standard d’Arabie | (UTC+03:00) Bagdad |
| Heure standard de Turquie | (UTC+03:00) Istanbul |
| Heure standard arabe | (UTC+03:00) Koweït, Riyad |
| Heure standard du Bélarus | (UTC+03:00) Minsk |
| Heure standard de Russie | (UTC+03:00) Moscou, Saint-Pétersbourg, Volgograd |
| E. Heure standard d’Afrique de l’Est | (UTC+03:00) Nairobi |
| Heure standard d’Iran | (UTC+03:30) Téhéran |
| Heure standard arabe | (UTC+04:00) Abu Dhabi, Mascate |
| Heure standard d’Astrakhan | (UTC+04:00) Astrakhan, Oulianovsk |
| Heure standard d’Azerbaïdjan | (UTC+04:00) Bakou |
| Fuseau horaire de Russie 3 | (UTC+04:00) Izhevsk, Samara |
| Heure standard de Maurice | (UTC+04:00) Port Louis |
| Heure standard de Saratov | (UTC+04:00) Saratov |
| Heure standard de Géorgie | (UTC+04:00) Tbilissi |
| Heure standard de Volgograd | (UTC+04:00) Volgograd |
| Heure standard du Caucase | (UTC+04:00) Erevan |
| Heure standard d’Afghanistan | (UTC+04:30) Kaboul |
| Heure standard d’Asie de l’Ouest | (UTC+05:00) Achgabat, Tachkent |
| Heure standard d’Iekaterinbourg | (UTC+05:00) Iekaterinbourg |
| Heure standard du Pakistan | (UTC+05:00) Islamabad, Karachi |
| Heure standard de l’Inde | (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi |
| Heure standard du Sri Lanka | (UTC+05:30) Sri Jayawardenepura |
| Heure standard du Népal | (UTC+05:45) Katmandou |
| Heure standard d’Asie centrale (Nord) | (UTC+06:00) Astana |
| Heure standard du Bangladesh | (UTC+06:00) Dacca |
| Heure standard d’Omsk | (UTC+06:00) Omsk |
| Heure standard du Myanmar | (UTC+06:30) Yangon (Rangoon) |
| Heure standard d’Asie du Sud-Est | (UTC+07:00) Bangkok, Hanoï, Jakarta |
| Heure standard de l’Altaï | (UTC+07:00) Barnaul, Gorno-Altaysk |
| Heure standard de Mongolie occidentale | (UTC+07:00) Hovd |
| Heure standard d’Asie du Nord | (UTC+07:00) Krasnoïarsk |
| N. Heure standard d’Asie centrale (Nord) | (UTC+07:00) Novossibirsk |
| Heure standard de Tomsk | (UTC+07:00) Tomsk |
| Heure standard de Chine | (UTC+08:00) Beijing, Chongqing, Hong Kong, Urumqi |
| Heure standard d’Asie de l’Est (Nord) | (UTC+08:00) Irkoutsk |
| Heure standard de Singapour | (UTC+08:00) Kuala Lumpur, Singapour |
| Heure standard Heure standard d’Australie de l’Est | (UTC+08:00) Perth |
| Heure standard de Taipei | (UTC+08:00) Taipei |
| Heure standard d’Oulan-Bator | (UTC+08:00) Oulan-Bator |
| Heure standard d’Australie occidentale (Centre) | (UTC+08:45) Eucla |
| Heure standard de Transbaïkalie | (UTC+09:00) Chita |
| Heure standard de Tokyo | (UTC+09:00) Osaka, Sapporo, Tokyo |
| Heure standard de Corée du Nord | (UTC+09:00) Pyongyang |
| Heure standard de Corée du Sud | (UTC+09:00) Séoul |
| Heure standard d’Iakoutsk | (UTC+09:00) Iakoutsk |
| Heure standard Heure standard d’Australie de l’Est | (UTC+09:30) Adelaide |
| Heure standard d’Australie Centre) | (UTC+09:30) Darwin |
| E. Heure standard d’Australie de l’Est | (UTC+10:00) Brisbane |
| Heure standard d’Australie de l’Est | (UTC+10:00) Canberra, Melbourne, Sydney |
| Heure standard du Pacifique (Ouest) | (UTC+10:00) Guam, Port Moresby |
| Heure standard de Tasmanie | (UTC+10:00) Hobart |
| Heure standard de Vladivostok | (UTC+10:00) Vladivostok |
| Heure standard de l’Île Lord Howe | (UTC+10:30) Île Lord Howe |
| Heure standard de l’Île Bougainville | (UTC+11:00) Île Bougainville |
| Fuseau horaire de Russie 10 | (UTC+11:00) Chokurdakh |
| Heure standard de Magadan | (UTC+11:00) Magadan |
| Heure standard de l’Île Norfolk | (UTC+11:00) Île Norfolk |
| Heure standard de Sakhaline | (UTC+11:00) Sakhaline |
| Heure standard du Pacifique (Centre) | (UTC+11:00) Îles Salomon, Nouvelle-Calédonie |
| Fuseau horaire de Russie 11 | (UTC+12:00) Anadyr, Petropavlovsk-Kamchatsky |
| Heure standard de Nouvelle-Zélande | (UTC+12:00) Auckland, Wellington |
| UTC+12 | (UTC+12:00) Temps universel coordonné+12 |
| Heure standard de Fidji | (UTC+12:00) Fidji |
| Heure standard du Kamchatka | (UTC+12:00) Petropavlovsk-Kamchatsky - Ancien |
| Heure standard des Îles Chatham | (UTC+12:45) Îles Chatham |
| UTC+13 | (UTC+13:00) Temps universel coordonné+13 |
| Heure standard Tonga | (UTC+13:00) Nuku’alofa |
| Heure standard de Samoa | (UTC+13:00) Samoa |
| Heure standard des Îles de la Ligne | (UTC+14:00) Île de Kiritimati |

## <a name="see-also"></a>Voir aussi 

- [CURRENT_TIMEZONE (Transact-SQL)](/sql/t-sql/functions/current-timezone-transact-sql)
- [CURRENT_TIMEZONE_ID (Transact-SQL)](/sql/t-sql/functions/current-timezone-id-transact-sql)
- [AT TIME ZONE (Transact-SQL)](/sql/t-sql/queries/at-time-zone-transact-sql)
- [sys.time_zone_info (Transact-SQL)](/sql/relational-databases/system-catalog-views/sys-time-zone-info-transact-sql)