---
title: Référence de schéma de normalisation des données Azure Sentinel | Microsoft Docs
description: Cet article présente le schéma de normalisation des données Azure Sentinel.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 09/08/2020
ms.author: yelevin
ms.openlocfilehash: eb1752ea66f2cbebf6a653705b5a760e8e268240
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929905"
---
# <a name="azure-sentinel-data-normalization-schema-reference"></a>Référence de schéma de normalisation des données Azure Sentinel

## <a name="terminology"></a>Terminologie

La terminologie suivante est utilisée dans les schémas de Sentinel :

| Terme | Définition |
| ---- | ---------- |
| Périphérique de création de rapport | Système envoyant les enregistrements à Azure Sentinel. Il peut ne pas s’agir du système de sujet de l’enregistrement. |
| Enregistrement | Unité de données envoyée à partir du périphérique de création de rapport. Il s’agit souvent d’un « journal », d’un « événement » ou d’une « alerte », mais ce ne doit pas nécessairement être l’un de ces éléments. |
|

## <a name="data-types-and-formats"></a>Types de données et formats

Les valeurs doivent être normalisées conformément aux instructions ci-dessous. Cela est obligatoire pour les champs normalisés et recommandé pour d’autres champs. 

| Type de données | Type physique | Format et valeur |
| --------- | ------------- | ---------------- |
| **Date/Heure** | Selon la capacité de la méthode d’ingestion, utilisez dans l’ordre descendant de priorité :<ul><li>Type de DateHeure intégré de Log Analytics</li><li>Champ de type entier utilisant une représentation numérique de DateHeure de Log Analytics</li><li>Champ de type chaîne utilisant une représentation numérique de DateHeure de Log Analytics</li></ul> | Représentation de DateHeure de Log Analytics. <br></br>La représentation de DateHeure de Log Analytics est de nature similaire, mais différente de celle d’Unix. Consultez les instructions de conversion. <br></br>La date et l’heure doivent être ajustées au fuseau horaire. |
| **Adresse MAC** | String | Notation hexadécimale à deux-points |
| **Adresse IP** | Adresse IP | Le schéma n’a pas d’adresses IPv4 et IPv6 distinctes. Tout champ d’adresse IP peut inclure une adresse IPv4 ou IPv6 :<ul><li>IPv4 dans une notation décimale à points</li><li>IPv6 en notation 8 hextets, permettant les formes abrégées décrites ici.</li></ul> |
| **Utilisateur** | String | Les 3 champs utilisateurs suivants sont disponibles :<ul><li>Nom d’utilisateur</li><li>UPN de l’utilisateur</li><li>Domaine de l’utilisateur</li></ul> |
| **ID d'utilisateur** | String | Les deux ID d’utilisateur suivants sont actuellement pris en charge :<ul><li>SID de l’utilisateur</li><li>ID Azure Active Directory</li></ul> |
| **Appareil** | String | Les 3 colonnes d’appareils/hôtes suivantes sont prises en charge :<ul><li>id</li><li>Nom</li><li>Nom de domaine complet (FQDN)</li></ul> |
| **Pays** | String | Chaîne utilisant ISO 3166-1 conformément à cette priorité :<ul><li>Codes alpha-2 (par exemple, US pour les États-Unis)</li><li>Codes alpha-3 (par exemple, USA pour les États-Unis)</li><li>Nom court</li></ul> |
| **Région** | String | Nom de subdivision du pays selon la norme ISO 3166-2 |
| **Ville** | String | |
| **Longitude** | Double | Représentation de coordonnée ISO 6709 (décimale signée) |
| **Latitude** | Double | Représentation de coordonnée ISO 6709 (décimale signée) |
| **Algorithme de hachage** | String | Les 4 colonnes de hachage suivantes sont prises en charge :<ul><li>MD5</li><li>SHA1</li><li>SHA256</li><li>SHA512</li></ul> |
| **Type de fichier** | String | Type du type de fichier :<ul><li>Extension</li><li>Classe</li><li>NamedType</li></ul> |
| 

## <a name="network-sessions-table-schema"></a>Schéma de table de sessions réseau

Voici le schéma de la table de sessions réseau, version 1.0.0

| Nom du champ | Type de valeur | Exemple | Description | Entités OSSEM associées |
|-|-|-|-|-|
| Type d’événement | String | Trafic | Type d’événement collecté | événement |
| EventSubType | String | Authentification | Description supplémentaire du type, le cas échéant | événement |
| EventCount | Integer  | 10 | Nombre d’événements agrégés, le cas échéant. | événement |
| EventEndTime | Date/Heure | Consultez « types de données » | Heure de fin de l’événement | événement |
| EventMessage | string |  accès refusé | Message ou description généraux, inclus dans l’enregistrement ou générés à partir de celui-ci | événement |
| DvcIpAddr | Adresse IP |  23.21.23.34 | Adresse IP de l’appareil générant l’enregistrement | périphérique,<br>IP |
| DvcMacAddr | String | 06:10:9f:eb:8f:14 | Adresse MAC de l’interface réseau du périphérique de création de rapport à partir duquel l’événement a été envoyé. | périphérique,<br>Mac |
| DvcHostname | Nom du nœud (chaîne) | syslogserver1.contoso.com | Nom d’appareil du périphérique générant le message. | Appareil |
| EventProduct | String | OfficeSharepoint | Produit générant l’événement. | événement |
| EventProductVersion | string | 9.0 |  Version du produit générant l’événement. | événement |
| EventResourceId | ID de l’appareil (chaîne) | /subscriptions/3c1bb38c-82e3-4f8d-a115-a7110ba70d05 /resourcegroups/contoso77/providers /microsoft.compute/virtualmachines /syslogserver1 | ID de ressource de l’appareil générant le message. | événement |
| EventReportUrl | String | https://192.168.1.1/repoerts/ae3-56.htm | Lien vers le rapport complet créé par le périphérique de création de rapport | événement |
| EventVendor | String | Microsoft | Fournisseur du produit générant l’événement. | événement |
| EventResult | Valeurs multiples : Réussite, Partiel, Échec, [Vide] (chaîne) | Succès | Résultat signalé pour l’activité. Valeur vide quand non applicable. | événement |
| EventResultDetails | String | Mot de passe incorrect | Raison ou détails du résultat rapporté dans EventResult | événement |
| EventSchemaVersion | Real | 0,1 | Version du schéma Azure Sentinel. Actuellement 0.1. | événement |
| EventSeverity | String | Faible | Si l’activité rapportée a un impact sur la sécurité, indique la gravité de l’impact. | événement |
| EventOriginalUid | String | af6ae8fe-ff43-4a4c-b537-8635976a2b51 | ID d’enregistrement du périphérique de création de rapport. | événement |
| EventStartTime | Date/Heure | Consultez « types de données » | Heure à laquelle l’événement a été déclaré | événement |
| TimeGenerated | Date/Heure | Consultez « types de données » | Heure à laquelle l’événement s’est produit, telle de rapportée par la source de création de rapport. | Champ personnalisé |
| EventTimeIngested | Date/Heure | Consultez « types de données » | Heure à laquelle l’événement a été ingéré dans Azure Sentinel. Sera ajoutée par Azure Sentinel. | événement |
| EventUid | Guid (chaîne) | 516a64e3-8360-4f1e-a67c-d96b3d52df54 | Identificateur unique utilisé par Sentinel pour marquer une ligne. | événement |
| NetworkApplicationProtocol | String | HTTPS | Protocole de couche Application utilisé par la connexion ou la session. | Réseau |
| DstBytes | int | 32455 | Nombre d’octets envoyés de la destination à la source pour la connexion ou la session. | Destination |
| SrcBytes | int | 46536 | Nombre d’octets envoyés de la source à la destination pour la connexion ou la session. | Source |
| NetworkBytes | int | 78991 | Nombre d’octets envoyés dans les deux sens. Si les valeurs BytesReceived et BytesSent existent, la valeur BytesTotal doit être égale à leur somme. | Réseau |
| NetworkDirection | Valeurs multiples : Entrants, Sortants (chaîne) | Entrant | Direction de la connexion ou de la session, vers l’intérieur ou l’extérieur de l’organisation. | Réseau |
| DstGeoCity | String | Burlington | Ville associée à l’adresse IP de destination | Destination,<br>Zone géographique |
| DstGeoCountry | Pays (chaîne) | États-Unis | Pays associé à l’adresse IP source | Destination,<br>Zone géographique |
| DstDvcHostname | Nom de l’appareil (chaîne) |  victim_pc | Nom de l’appareil de destination | Destination<br>Appareil |
| DstDvcFqdn | String | victim_pc.contoso.local | Nom de domaine complet de l’hôte sur lequel le journal a été créé | Destination,<br>Appareil |
| DstDomainHostname | string | CONTOSO | Domaine de la destination, domaine de l’hôte de destination (site web, nom de domaine, etc.), par exemple, pour les recherches de DNS ou de NS | Destination |
| DstInterfaceName | string | Carte réseau Microsoft Hyper-V | Interface réseau que l’appareil de destination utilise pour la connexion ou la session. | Destination |
| DstInterfaceGuid | string | 2BB33827-6BB6-48DB-8DE6-DB9E0B9F9C9B | GUID de l’interface réseau utilisée pour la demande d’authentification  | Destination |
| DstIpAddr | Adresse IP | 2001:db8::ff00:42:8329 | Adresse IP de la destination de la connexion ou de la session, plus communément appelée adresse IP de destination dans le paquet réseau | Destination,<br>IP |
| DstDvcIpAddr | Adresse IP | 75.22.12.2 | Adresse IP de destination d’un appareil qui n’est pas directement associé au paquet réseau | Destination,<br>périphérique,<br>IP
| DstGeoLatitude | Latitude (double) | 44.475833 | Latitude de la coordonnée géographique associée à l’adresse IP de destination | Destination,<br>Zone géographique |
| DstMacAddr | String | 06:10:9f:eb:8f:14 | Adresse MAC de l’interface réseau à laquelle la connexion ou la session s’est terminée, le plus souvent appelée MAC de destination dans le paquet réseau | Destination,<br>MAC |
| DstDvcMacAddr | String | 06:10:9f:eb:8f:14 | Adresse MAC de destination d’un appareil qui n’est pas directement associé au paquet réseau. | Destination,<br>périphérique,<br>MAC |
| DstDvcDomain | String | CONTOSO | Domaine de l’appareil de destination. | Destination,<br>Appareil |
| DstPortNumber | Integer | 443 | Port IP de destination. | Destination,<br>Port |
| DstGeoRegion | Région (chaîne) | Vermont | Région au sein d’un pays associé à l’adresse IP de destination | Destination,<br>Zone géographique |
| DstResourceId | ID de l’appareil (chaîne) |  /subscriptions/3c1bb38c-82e3-4f8d-a115-a7110ba70d05 /resourcegroups/contoso77/providers /microsoft.compute/virtualmachines /victim | ID de ressource de l’appareil de destination. | Destination |
| DstNatIpAddr | Adresse IP | 2::1 | Si elle est signalée par un périphérique NAT intermédiaire tel qu’un pare-feu, adresse IP utilisée par le périphérique NAT pour la communication avec la source. | NAT de destination,<br>IP |
| DstNatPortNumber | int | 443 | S’il est signalé par un périphérique NAT intermédiaire tel qu’un pare-feu, port utilisé par le périphérique NAT pour la communication avec la source. | NAT de destination,<br>Port |
| DstUserSid | SID de l’utilisateur |  S-12-1445 | ID d’utilisateur de l’identité associée à la destination de la session. Généralement, identité utilisée pour authentifier un serveur. Pour plus d’informations, consultez « types de données ». | Destination,<br>Utilisateur |
| DstUserAadId | Chaîne (guid) | ae92b0b4-cfba-4b42-85a0-fbd862f4df54 | ID d’objet de compte Azure AD de l’utilisateur à la terminaison de destination de la session | Destination,<br>Utilisateur |
| DstUserName | Nom d’utilisateur (chaîne) | johnd | Nom d’utilisateur de l’identité associée à la destination de la session.  | Destination,<br>Utilisateur |
| DstUserUpn | string | johnd@anon.com | UPN de l’identité associée à la destination de la session. | Destination,<br>Utilisateur |
| DstUserDomain | string | GROUPE DE TRAVAIL | Nom de domaine ou d’ordinateur du compte à la destination de la session | Destination,<br>Utilisateur |
| DstZone | String | Dmz | Zone réseau de la destination, telle que définie par le périphérique de création de rapport. | Destination |
| DstGeoLongitude | Longitude (double) | -73.211944 | Longitude de la coordonnée géographique associée à l’adresse IP de destination | Destination,<br>Zone géographique |
| DvcAction | Valeurs multiples : Autoriser, Refuser, Supprimer (chaîne) | Autoriser | En cas de signalement par un périphérique intermédiaire tel qu’un pare-feu, action effectuée par celui-ci. | Appareil |
| DvcInboundInterface | String | eth0 | En cas de signalement par un périphérique intermédiaire tel qu’un pare-feu, interface réseau utilisée par celui-ci pour la connexion à l’appareil source. | Appareil |
| DvcOutboundInterface | String  | Carte Ethernet 4 | En cas de signalement par un périphérique intermédiaire tel qu’un pare-feu, interface réseau utilisée par celui-ci pour la connexion au périphérique de destination. | Appareil |
| NetworkDuration | Integer | 1500 | Durée, en millisecondes, de la session ou de la connexion réseau | Réseau |
| NetworkIcmpCode | Integer | 34 | Pour un message ICMP, valeur numérique du type de message ICMP (RFC 2780 ou RFC 4443). | Réseau |
| NetworkIcmpType | String | Destination inaccessible | Pour un message ICMP, représentation textuelle du type de message ICMP (RFC 2780 ou RFC 4443). | Réseau |
| DstPackets | int  | 446 | Nombre de paquets envoyés de la destination à la source pour la connexion ou la session. La signification d’un paquet est définie par le périphérique de création de rapport. | Destination |
| SrcPackets | int  | 6478 | Nombre de paquets envoyés de la source à la destination pour la connexion ou la session. La signification d’un paquet est définie par le périphérique de création de rapport. | Source |
| NetworkPackets | int  | 0 | Nombre de paquets envoyés dans les deux sens. Si les valeurs PacketsReceived et PacketsSent existent, la valeur BytesTotal doit être égale à leur somme. | Réseau |
| HttpRequestTime | Integer | 700 | Durée de l’envoi de la demande au serveur, le cas échéant. | Http |
| HttpResponseTime | Integer | 800 | Durée nécessaire à la réception d’une réponse sur le serveur, le cas échéant. | Http |
| NetworkRuleName | String | AnyAnyDrop | Nom ou ID de la règle en vertu de laquelle DeviceAction a été décidé | Réseau |
| NetworkRuleNumber | int |  23 | Numéro de règle correspondant  | Réseau |
| NetworkSessionId | string | 172_12_53_32_4322__123_64_207_1_80 | Identificateur de session signalé par le périphérique de création de rapport. Par exemple, identificateur de session L7 pour des applications spécifiques suite à une authentification | Réseau |
| SrcGeoCity | String | Burlington | Ville associée à l’adresse IP source | Source,<br>Zone géographique |
| SrcGeoCountry | Pays (chaîne) | États-Unis | Pays associé à l’adresse IP source | Source,<br>Zone géographique |
| SrcDvcHostname | Nom de l’appareil (chaîne) |  villain | Nom de l’appareil source | Source,<br>Appareil |
| SrcDvcFqdn | string | Villain.malicious.com | Nom de domaine complet de l’hôte sur lequel le journal a été créé | Source,<br>Appareil |
| SrcDvcDomain | string | EVILORG | Domaine de l’appareil à partir duquel la session a été lancée | Source,<br>Appareil |
| SrcDvcOs | String | iOS | Système d’exploitation de l’appareil source | Source,<br>Appareil |
| SrcDvcModelName | String | Samsung Galaxy Note | Nom de modèle de l’appareil source | Source,<br>Appareil |
| SrcDvcModelNumber | String | 10.0 | Numéro de modèle de l’appareil source | Source,<br>Appareil |
| SrcDvcType | String | Mobile | Type de l’appareil source | Source,<br> Appareil |
| SrcIntefaceName | String | eth01 | Interface réseau que l’appareil source utilise pour la connexion ou la session. | Source |
| SrcInterfaceGuid | String | 46ad544b-eaf0-47ef-827c-266030f545a6 | GUID de l’interface réseau utilisée | Source |
| SrcIpAddr | Adresse IP | 77.138.103.108 | Adresse IP d’origine de la connexion ou de la session. | Source,<br>IP |
| SrcDvcIpAddr | Adresse IP | 77.138.103.108 | L’adresse IP source d’un appareil n’est pas directement associée au paquet réseau (collectée par un fournisseur ou calculée explicitement). | Source,<br>périphérique,<br>IP |
| SrcGeoLatitude | Latitude (double) | 44.475833 | Latitude de la coordonnée géographique associée à l’adresse IP source | Source,<br>Zone géographique |
| SrcGeoLongitude | Longitude (double) | -73.211944 | Longitude de la coordonnée géographique associée à l’adresse IP source | Source,<br>Zone géographique |
| SrcMacAddr | String | 06:10:9f:eb:8f:14 | Adresse MAC de l’interface réseau d’où provient la connexion ou la session. | Source,<br>Mac |
| SrcDvcMacAddr | String | 06:10:9f:eb:8f:14 | Adresse MAC source d’un appareil qui n’est pas directement associé au paquet réseau. | Source,<br>périphérique,<br>Mac |
| SrcPortNumber | Integer | 2335 | Port IP d’où provient la connexion. Peut ne pas être pertinent pour une session comprenant plusieurs connexions. | Source,<br>Port |
| SrcGeoRegion | Région (chaîne) | Vermont | Région au sein d’un pays associé à l’adresse IP source | Source,<br>Zone géographique |
| SrcResourceId | String | /subscriptions/3c1bb38c-82e3-4f8d-a115-a7110ba70d05 /resourcegroups/contoso77/providers /microsoft.compute/virtualmachines /syslogserver1 | ID de ressource de l’appareil générant le message. | Source |
| SrcNatIpAddr | Adresse IP | 4.3.2.1 | Si elle est signalée par un périphérique NAT intermédiaire tel qu’un pare-feu, adresse IP utilisée par le périphérique NAT pour la communication avec la destination. | NAT source,<br>IP |
| SrcNatPortNumber | Integer | 345 | S’il est signalé par un périphérique NAT intermédiaire tel qu’un pare-feu, port utilisé par le périphérique NAT pour la communication avec la destination. | NAT source,<br>Port |
| SrcUserSid | ID d’utilisateur (chaîne) | S-15-1445 | ID d’utilisateur de l’identité associée à la source de la session. En général, l’utilisateur effectuant une action sur le client. Pour plus d’informations, consultez « types de données ». | Source,<br>Utilisateur |
| SrcUserAadId | Chaîne (guid) | 16c8752c-7dd2-4cad-9e03-fb5d1cee5477 | ID d’objet de compte Azure AD de l’utilisateur à la terminaison source de la session | Source,<br>Utilisateur |
| SrcUserName | Nom d’utilisateur (chaîne) | bob | Nom d’utilisateur de l’identité associée à la source de la session. En général, l’utilisateur effectuant une action sur le client. Pour plus d’informations, consultez « types de données ». | Source<br>Utilisateur |
| SrcUserUpn | string | bob@alice.com | UPN du compte à l’origine de la session | Source,<br>Utilisateur |
| SrcUserDomain | string | BUREAU | Domaine du compte à l’origine de la session | Source,<br>Utilisateur |
| SrcZone | String | Taper | Zone réseau de la source, telle que définie par le périphérique de création de rapport. | Source |
| NetworkProtocol | String | TCP | Protocole IP utilisé par la connexion ou la session. En règle générale, TCP, UDP ou ICMP | Réseau |
| CloudAppName | String | Facebook | Nom de l’application de destination pour une application HTTP, tel qu’identifié par un proxy. | Cloud |
| CloudAppId | String | 124 | ID de l’application de destination pour une application HTTP, tel qu’identifié par un proxy. Cette valeur est généralement spécifique du proxy utilisé. | Cloud |
| CloudAppOperation | String | DeleteFile | Opération effectuée par l’utilisateur dans le contexte de l’application de destination pour une application HTTP, telle qu’identifiée par un proxy. Cette valeur est généralement spécifique du proxy utilisé. | Cloud |
| CloudAppRiskLevel | String | 3 | Niveau de risque associé à une application HTTP, tel qu’identifié par un proxy. Cette valeur est généralement spécifique du proxy utilisé. | Cloud |
| FileName | String | ImNotMalicious.exe | Nom de fichier transmis via les connexions réseau pour des protocoles tels que FTP et HTTP qui fournissent les informations de nom de fichier. | File |
| FilePath | String | C:\Malicious\ImNotMalicious.exe | Chemin d’accès complet, incluant nom du fichier | File |
| FileHashMd5 | String | 51BC68715FC7C109DCEA406B42D9D78F | Valeur de hachage MD5 du fichier transmis sur les connexions réseau pour les protocoles. | File |
| FileHashSha1 | String | 491AE3…C299821476F4 | Valeur de hachage SHA1 du fichier transmis sur les connexions réseau pour les protocoles. | File |
| FileHashSha256 | String | 9B8F8EDB…C129976F03 | Valeur de hachage SHA256 du fichier transmis sur les connexions réseau pour les protocoles. | File |
| FileHashSha512 | String | 5E127D…F69F73F01F361 | Valeur de hachage SHA512 du fichier transmis sur les connexions réseau pour les protocoles. | File |
| FileExtension |  String | exe | Type du fichier transmis sur les connexions réseau pour des protocoles tels que FTP et HTTP. | File
| FileMimeType | String | application/msword | Type MIME du fichier transmis sur les connexions réseau pour des protocoles tels que FTP et HTTP | File |
| FileSize | Integer | 23500 | Taille en octets du fichier transmis sur les connexions réseau pour les protocoles. | File |
| HttpVersion | String | 2.0 | Version de requête HTTP pour les connexions réseau HTTP/HTTPS. | Http |
| HttpRequestMethod | String | GET | Méthode HTTP pour les sessions réseau HTTP/HTTPS. | Http |
| HttpStatusCode | String | 404 | Code d’état HTTP pour les sessions réseau HTTP/HTTPS. | Http |
| HttpContentType | String | multipart/form-data; boundary=something | En-tête de type de contenu de réponse HTTP pour les sessions réseau HTTP/HTTPS. | Http |
| HttpReferrerOriginal | String | https://developer.mozilla.org/en-US/docs/Web/JavaScript | En-tête de référent HTTP pour les sessions réseau HTTP/HTTPS. | Http |
| HttpUserAgentOriginal | String | Mozilla/5.0 (Windows NT 10.0 ; WOW64) AppleWebKit/537.36 (KHTML, comme Gecko) Chrome/83.0.4103.97 Safari/537.36 | En-tête de l’agent utilisateur HTTP pour les sessions réseau HTTP/HTTPS. | Http |
| HttpRequestXff | String | 120.12.41.1 | En-tête HTTP X-Forwarded-for pour les sessions réseau HTTP/HTTPS. | Http |
| UrlCategory | String | Moteurs de recherche | Regroupement défini d’une URL (pourrait aussi être simplement basé sur le domaine dans l’URL) lié à son contenu (par exemple, adultes, actualités, publicité, domaines parqués, etc.) | url |
| UrlOriginal | String | https:// contoso.com/fo/?k=v&q=u#f | URL de la requête HTTP pour les sessions réseau HTTP/HTTPS. | Url |
| UrlHostname | String | contoso.com | Partie domaine d’une URL de requête HTTP pour les sessions réseau HTTP/HTTPS. | Url |
| ThreatCategory | String | Trojan | Catégorie d’une menace identifiée par un système de sécurité tel que la passerelle de sécurité web d’une IPS associée à cette session réseau. | Menace |
| ThreatId | String | Tr.124 | ID d’une menace identifiée par un système de sécurité tel que la passerelle de sécurité web d’une IPS associée à cette session réseau. | Menace |
| ThreatName | String | Fichier de test EICAR | Nom de la menace ou du programme malveillant identifiés | Menace |
| AdditionalFields | Dynamique (conteneur JSON) | {<br>Property1: “val1”,<br>Property2: “val2”<br>} | Quand aucune colonne correspondante dans le schéma ne correspond, des champs supplémentaires peuvent être stockés dans un conteneur JSON.<br>Pour l’analyse au moment de la requête, il est recommandé de ne pas utiliser cette méthode, car l’empaquetage de données dans un fichier JSON aura pour effet de dégrader les performances de requête. Au lieu de cela, il est recommandé de promouvoir des colonnes supplémentaires.<br>Pour les futurs scénarios d’analyse au moment de l’ingestion, des données supplémentaires seront évidemment collectées dans cette colonne de conteneur JSON. | Champ personnalisé |
| 
