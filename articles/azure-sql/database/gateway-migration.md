---
title: Avis de migration du trafic des passerelles
description: Cet article informe les utilisateurs sur la migration des adresses IP des passerelles Azure SQL Database
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: sqldbrb=1
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: vanto
ms.date: 07/01/2019
ms.openlocfilehash: 7fadbecc2c00a739afb2f94dd1d049805915cfa5
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92427099"
---
# <a name="azure-sql-database-traffic-migration-to-newer-gateways"></a>Migration du trafic des bases de données Azure SQL Database vers des passerelles plus récentes
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Comme l’infrastructure Azure évolue, Microsoft actualise régulièrement le matériel, afin de proposer la meilleure expérience client possible. Dans les prochains mois, nous prévoyons d’ajouter des passerelles basées sur des générations de matériel plus récentes, de migrer le trafic vers ces passerelles et de mettre hors service les passerelles basées sur du matériel plus ancien dans certaines régions.  

Les clients seront informés très tôt de toute modification apportée aux passerelles disponibles dans chaque région, par e-mail et via le Portail Microsoft Azure. Les informations les plus récentes seront conservées dans la table [Adresses IP des passerelles Azure SQL Database](connectivity-architecture.md#gateway-ip-addresses).

## <a name="status-updates"></a>Mises à jour d’état

# <a name="in-progress"></a>[En cours](#tab/in-progress-ip)
### <a name="october-2020"></a>Octobre 2020

De nouvelles passerelles SQL sont ajoutées aux régions suivantes :

- Allemagne Centre-Ouest : 51.116.240.0, 51.116.248.0

Ces passerelles SQL devraient commencer à accepter le trafic client le 12 octobre 2020. 

### <a name="september-2020"></a>Septembre 2020
De nouvelles passerelles SQL sont ajoutées aux régions suivantes : Ces passerelles SQL devraient commencer à accepter le trafic client le **15 septembre 2020** :

- Australie Sud-Est : 13.77.48.10
- Canada Est : 40.86.226.166, 52.242.30.154
- Royaume-Uni Sud : 51.140.184.11, 51.105.64.0

Les passerelles SQL existantes commenceront à accepter le trafic dans les régions suivantes : Ces passerelles SQL devraient commencer à accepter le trafic client le **15 septembre 2020** :

- Australie Sud-Est : 191.239.192.109 et 13.73.109.251
- USA Centre : 13.67.215.62, 52.182.137.15, 23.99.160.139, 104.208.16.96 et 104.208.21.1
- Asie Est : 191.234.2.139, 52.175.33.150 et 13.75.32.4
- USA Est : 40.121.158.30, 40.79.153.12, 191.238.6.43 et 40.78.225.32
- USA Est 2 : 40.79.84.180, 52.177.185.181, 52.167.104.0, 191.239.224.107 et 104.208.150.3
- France Centre : 40.79.137.0 et 40.79.129.1
- Japon Ouest : 104.214.148.156, 40.74.100.192, 191.238.68.11 et 40.74.97.10
- USA Centre Nord : 23.96.178.199, 23.98.55.75 et 52.162.104.33
- Asie Sud-Est : 104.43.15.0, 23.100.117.95 et 40.78.232.3
- USA Ouest : 104.42.238.205, 23.99.34.75 et 13.86.216.196

De nouvelles passerelles SQL sont ajoutées aux régions suivantes : Ces passerelles SQL devraient commencer à accepter le trafic client le **10 septembre 2020** :

- USA Centre-Ouest : 13.78.248.43 
- Afrique du Sud Nord 102.133.120.2  

De nouvelles passerelles SQL sont ajoutées aux régions suivantes : Ces passerelles SQL devraient commencer à accepter le trafic client le **1er septembre 2020** :

- Europe Nord : 13.74.104.113 
- USA Ouest 2 : 40.78.248.10 
- Europe Ouest : 52.236.184.163 
- USA Centre Sud : 20.45.121.1, 20.49.88.1 

Les passerelles SQL existantes commenceront à accepter le trafic dans les régions suivantes : Ces passerelles SQL devraient commencer à accepter le trafic client le **1er septembre 2020** :
- Japon Est : 40.79.184.8, 40.79.192.5

# <a name="completed"></a>[Terminé](#tab/completed-ip)

Les migrations de passerelle suivantes sont terminées : 

### <a name="august-2020"></a>Août 2020

De nouvelles passerelles SQL sont ajoutées aux régions suivantes :

- Australie Est : 13.70.112.9
- Canada Centre : 52.246.152.0, 20.38.144.1 
- USA Ouest 2 : 40.78.240.8

Ces passerelles SQL devraient commencer à accepter le trafic client le 10 août 2020. 

### <a name="october-2019"></a>2 octobre 2019
- Brésil Sud
- USA Ouest
- Europe Ouest
- USA Est
- USA Centre
- Asie Sud-Est
- États-Unis - partie centrale méridionale
- Europe Nord
- Centre-Nord des États-Unis
- OuJapon Est
- Japon Est
- USA Est 2
- Asie Est

---

## <a name="impact-of-this-change"></a>Portée de ce changement

La migration du trafic peut changer l’adresse IP publique résolue par DNS pour votre base de données dans Azure SQL Database.
Vous risquez d’être impacté si :

- Vous avez codé en dur l’adresse IP d’une passerelle particulière dans votre pare-feu local
- Vous avez des sous-réseaux qui utilisent Microsoft.SQL comme point de terminaison de service, mais vous ne pouvez pas communiquer avec les adresses IP de passerelle
- Vous utilisez la [configuration de zone redondante pour le niveau Usage général](high-availability-sla.md#general-purpose-service-tier-zone-redundant-availability-preview)
- Utilisez la [configuration de zone redondante pour les niveaux Premium et Critiques pour l’entreprise](high-availability-sla.md#premium-and-business-critical-service-tier-zone-redundant-availability)

Vous n’êtes pas impacté dans les cas suivants :
 
- Vous utilisez la redirection comme stratégie de connexion.
- Vous avez des connexions à SQL Database à partir d’Azure et vous utilisez des balises de service.
- Il en va de même pour les connexions effectuées à l’aide des versions prises en charge de Microsoft JDBC Driver pour SQL Server. Pour les versions prises en charge du pilote JDBC, voir [Télécharger Microsoft JDBC Driver pour SQL Server](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server).

## <a name="what-to-do-you-do-if-youre-affected"></a>Que faire si vous êtes concerné

Nous vous recommandons d'autoriser le trafic sortant vers les adresses IP pour l'ensemble des [adresses IP de passerelles](connectivity-architecture.md#gateway-ip-addresses) de la région sur le port TCP 1433, et sur la plage de ports 11000-11999. Cette recommandation s'applique aux clients qui se connectent localement et à ceux qui se connectent via des points de terminaison de service. Pour en savoir plus sur les plages de ports, voir [Stratégie de connexion](connectivity-architecture.md#connection-policy).

La validation des certificats risque d’échouer pour les connexions établies à partir d’applications à l’aide d’une version de Microsoft JDBC Driver antérieure à 4.0. Les versions inférieures de Microsoft JDBC s’appuient sur un nom courant (CN) dans le champ d’objet du certificat. L’atténuation consiste à s’assurer que la propriété hostNameInCertificate est définie sur *.database.windows.net. Pour savoir comment définir la propriété hostNameInCertificate, voir [Connexion avec chiffrement](/sql/connect/jdbc/connecting-with-ssl-encryption).

Si l'atténuation ci-dessus ne fonctionne pas, envoyez une demande de support concernant SQL Database ou SQL Managed Instance à l'aide de l'URL suivante : https://aka.ms/getazuresupport

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [l’architecture de la connectivité Azure SQL](connectivity-architecture.md)
