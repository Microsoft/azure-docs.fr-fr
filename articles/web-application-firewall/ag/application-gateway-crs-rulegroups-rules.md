---
title: Règles et groupes de règles CRS
titleSuffix: Azure Web Application Firewall
description: Cette page fournit des informations sur les règles et groupes CRS de pare-feu d’applications web.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 11/14/2019
ms.author: victorh
ms.topic: conceptual
ms.openlocfilehash: 890e2b972818cf9805623d94709ce5631b50aaf5
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608603"
---
# <a name="web-application-firewall-crs-rule-groups-and-rules"></a>Règles et groupes de règles CRS de pare-feu d’applications web

Le pare-feu d’applications web Application Gateway protège les applications web contre les vulnérabilités et failles de sécurité. Cette protection s’effectue par le biais de règles définies à partir des ensembles de règles de base OWASP 3.1, 3.0 ou 2.2.9. Ces règles peuvent être désactivées une par une. Cet article contient les règles et ensembles de règles actuellement proposées.

> [!NOTE]
> Cet article contient des références au terme *liste noire*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

## <a name="core-rule-sets"></a>Ensembles de règles de base

Le pare-feu d’applications web d’Application Gateway est préconfiguré avec CRS 3.0 par défaut. Mais vous pouvez choisir d’utiliser CRS 3.1 ou CRS 2.2.9 à la place. CRS 3.1 offre de nouveaux ensembles de règles qui assurent une protection contre les infections Java, un ensemble initial de contrôles des chargements de fichiers, des faux positifs corrigés, etc. CRS 3.0 permet une réduction des faux positifs par rapport à la version 2.2.9. Vous pouvez également [personnaliser les règles en fonction de vos besoins](application-gateway-customize-waf-rules-portal.md).

> [!div class="mx-imgBorder"]
> ![Gestion des règles](../media/application-gateway-crs-rulegroups-rules/managed-rules-01.png)

Le WAF protège contre les vulnérabilités web suivantes :

- Attaques par injection de code SQL
- Attaques par exécution de scripts de site à site
- Autres attaques courantes comme l’injection de commande, les dissimulations de requêtes HTTP, la séparation de réponse HTTP et les attaques RFI (Remote File Inclusion)
- Violations de protocole HTTP
- Anomalies de protocole HTTP comme un agent utilisateur hôte manquant et les en-têtes Accept
- Bots, robots de recherche et scanneurs
- Erreurs de configuration d’application courantes (par exemple, Apache et IIS)

### <a name="owasp-crs-31"></a>OWASP CRS 3.1

CRS 3.1 inclut 13 groupes de règles, comme illustré dans le tableau suivant. Chaque groupe contient plusieurs règles qui peuvent être désactivées.

> [!NOTE]
> CRS 3.1 est uniquement disponible sur la référence SKU WAF_v2.

|Groupe de règles|Description|
|---|---|
|**[Généralités](#general-31)**|Groupe général|
|**[REQUEST-911-METHOD-ENFORCEMENT](#crs911-31)**|Verrouiller les méthodes (PUT, PATCH)|
|**[REQUEST-913-SCANNER-DETECTION](#crs913-31)**|Protéger contre les scanneurs de port et d’environnement.|
|**[REQUEST-920-PROTOCOL-ENFORCEMENT](application-gateway-crs-rulegroups-rules.md#crs920-31)**|Protéger contre les problèmes de protocole et d’encodage.|
|**[REQUEST-921-PROTOCOL-ATTACK](#crs921-31)**|Protéger contre les attaques par injection d’en-tête, dissimulation de requête et fractionnement de réponse.|
|**[REQUEST-930-APPLICATION-ATTACK-LFI](#crs930-31)**|Protéger contre les attaques par fichier et chemin d’accès.|
|**[REQUEST-931-APPLICATION-ATTACK-RFI](#crs931-31)**|Protéger contre les attaques par inclusion de fichier distant (RFI)|
|**[REQUEST-932-APPLICATION-ATTACK-RCE](#crs932-31)**|Protéger contre les attaques par exécution de code à distance.|
|**[REQUEST-933-APPLICATION-ATTACK-PHP](#crs933-31)**|Protéger contre les attaques par injection de code PHP.|
|**[REQUEST-941-APPLICATION-ATTACK-XSS](#crs941-31)**|Protéger contre les attaques par exécution de scripts de site à site.|
|**[REQUEST-942-APPLICATION-ATTACK-SQLI](#crs942-31)**|Protéger contre les attaques par injection de code SQL.|
|**[REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION](#crs943-31)**|Protéger contre les attaques par fixation de session.|
|**[REQUEST-944-APPLICATION-ATTACK-SESSION-JAVA](#crs944-31)**|Protéger contre les attaques JAVA|

### <a name="owasp-crs-30"></a>OWASP CRS 3.0

CRS 3.0 inclut 12 groupes de règles, comme illustré dans le tableau suivant. Chaque groupe contient plusieurs règles qui peuvent être désactivées.

|Groupe de règles|Description|
|---|---|
|**[Généralités](#general-30)**|Groupe général|
|**[REQUEST-911-METHOD-ENFORCEMENT](#crs911-30)**|Verrouiller les méthodes (PUT, PATCH)|
|**[REQUEST-913-SCANNER-DETECTION](#crs913-30)**|Protéger contre les scanneurs de port et d’environnement.|
|**[REQUEST-920-PROTOCOL-ENFORCEMENT](application-gateway-crs-rulegroups-rules.md#crs920-30)**|Protéger contre les problèmes de protocole et d’encodage.|
|**[REQUEST-921-PROTOCOL-ATTACK](#crs921-30)**|Protéger contre les attaques par injection d’en-tête, dissimulation de requête et fractionnement de réponse.|
|**[REQUEST-930-APPLICATION-ATTACK-LFI](#crs930-30)**|Protéger contre les attaques par fichier et chemin d’accès.|
|**[REQUEST-931-APPLICATION-ATTACK-RFI](#crs931-30)**|Protéger contre les attaques par inclusion de fichier distant (RFI)|
|**[REQUEST-932-APPLICATION-ATTACK-RCE](#crs932-30)**|Protéger contre les attaques par exécution de code à distance.|
|**[REQUEST-933-APPLICATION-ATTACK-PHP](#crs933-30)**|Protéger contre les attaques par injection de code PHP.|
|**[REQUEST-941-APPLICATION-ATTACK-XSS](#crs941-30)**|Protéger contre les attaques par exécution de scripts de site à site.|
|**[REQUEST-942-APPLICATION-ATTACK-SQLI](#crs942-30)**|Protéger contre les attaques par injection de code SQL.|
|**[REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION](#crs943-30)**|Protéger contre les attaques par fixation de session.|

### <a name="owasp-crs-229"></a>OWASP CRS 2.2.9

CRS 2.2.9 inclut 10 groupes de règles, comme illustré dans le tableau suivant. Chaque groupe contient plusieurs règles qui peuvent être désactivées.

|Groupe de règles|Description|
|---|---|
|**[crs_20_protocol_violations](#crs20)**|Protéger contre les violations de protocole (par exemple, caractères non valides, GET avec un corps de demande).|
|**[crs_21_protocol_anomalies](#crs21)**|Protéger contre des informations d’en-tête incorrectes.|
|**[crs_23_request_limits](#crs23)**|Protéger contre les arguments ou fichiers qui dépassent les limites.|
|**[crs_30_http_policy](#crs30)**|Protéger contre les types de fichiers, les en-têtes et les méthodes restreints.|
|**[crs_35_bad_robots](#crs35)**|Protéger contre les analyseurs et les scanneurs de sites web.|
|**[crs_40_generic_attacks](#crs40)**|Protéger contre les attaques génériques (par fixation de session, inclusion de fichier distant et injection de code PHP).|
|**[crs_41_sql_injection_attacks](#crs41sql)**|Protéger contre les attaques par injection de code SQL.|
|**[crs_41_xss_attacks](#crs41xss)**|Protéger contre les attaques par exécution de scripts de site à site.|
|**[crs_42_tight_security](#crs42)**|Protéger contre les attaques par traversée de chemin.|
|**[crs_45_trojans](#crs45)**|Protéger contre les chevaux de Troie.|

Les règles et groupes de règles suivants sont disponibles durant l’utilisation du pare-feu d’applications web sur Application Gateway.

# <a name="owasp-31"></a>[OWASP 3.1](#tab/owasp31)

## <a name="rule-sets"></a><a name="owasp31"></a> Ensembles de règles

### <a name="p-x-ms-format-detectionnonegeneralp"></a><a name="general-31"></a> <p x-ms-format-detection="none">Général</p>

|ID de la règle|Description|
|---|---|
|200004|Possible limite multipart sans correspondance.|

### <a name="p-x-ms-format-detectionnonerequest-911-method-enforcementp"></a><a name="crs911-31"></a> <p x-ms-format-detection="none">REQUEST-911-METHOD-ENFORCEMENT</p>

|ID de la règle|Description|
|---|---|
|911100|Méthode non autorisée par la stratégie|


### <a name="p-x-ms-format-detectionnonerequest-913-scanner-detectionp"></a><a name="crs913-31"></a> <p x-ms-format-detection="none">REQUEST-913-SCANNER-DETECTION</p>

|ID de la règle|Description|
|---|---|
|913100|Détection d’agent utilisateur associé au scanner de sécurité|
|913101|Détection d’agent utilisateur associé aux scripts/client HTTP générique|
|913102|Détection d’agent utilisateur associé à l’analyseur web/robot|
|913110|Détection d’en-tête de requête associé au scanner de sécurité|
|913120|Détection de nom de fichier/argument associé au scanner de sécurité|


### <a name="p-x-ms-format-detectionnonerequest-920-protocol-enforcementp"></a><a name="crs920-31"></a> <p x-ms-format-detection="none">REQUEST-920-PROTOCOL-ENFORCEMENT</p>

|ID de la règle|Description|
|---|---|
|920100|Ligne de requête HTTP non valide|
|920120|Tentative de contournement de données en plusieurs parties/de formulaire|
|920121|Tentative de contournement de données en plusieurs parties/de formulaire|
|920130|Échec de l’analyse du corps de la requête.|
|920140|Échec de la validation stricte du corps de la requête à parties multiples|
|920160|L’en-tête HTTP Content-Length n’est pas numérique.|
|920170|Requête GET ou HEAD avec contenu du corps.|
|920171|Requête GET ou HEAD avec Transfer-Encoding.|
|920180|En-tête Content-Length manquant dans la requête POST.|
|920190|Plage = dernière valeur d’octet non valide.|
|920200|Plage = Champs trop nombreux (6 ou plus)|
|920201|Plage = Champs trop nombreux pour la requête PDF (35 ou plus)|
|920202|Plage = Champs trop nombreux pour la requête PDF (6 ou plus)|
|920210|Détection de données d’en-tête de connexion en conflit/multiples.|
|920220|Tentative d’attaque abusive d’encodage d’URL|
|920230|Détection d’encodage de plusieurs URL|
|920240|Tentative d’attaque abusive d’encodage d’URL|
|920250|Tentative d’attaque abusive d’encodage UTF8|
|920260|Tentative d’attaque abusive Pleine/Moyenne largeur Unicode|
|920270|Caractère non valide dans la requête (caractère null)|
|920271|Caractère non valide dans la requête (caractères non imprimables)|
|920272|Caractère non valide dans la requête (en dehors des caractères imprimables avant ascii 127)|
|920273|Caractère non valide dans la requête (en dehors de l’ensemble très strict)|
|920274|Caractère non valide dans les en-têtes de requête (en dehors de l’ensemble très strict)|
|920280|En-tête d’hôte manquant dans la requête|
|920290|En-tête d’hôte vide|
|920300|En-tête Accept manquant dans la requête|
|920310|Requête contenant un en-tête Accept vide|
|920311|Requête contenant un en-tête Accept vide|
|920320|En-tête User-Agent manquant|
|920330|En-tête User-Agent vide|
|920340|Requête contenant du contenu mais dont l’en-tête Content-Type est manquant|
|920341|Requête dont le contenu nécessite l’en-tête Content-Type|
|920350|L’en-tête d’hôte est une adresse IP numérique|
|920360|Nom d’argument trop long|
|920370|Valeur d’argument trop longue|
|920380|Trop d’arguments dans la requête|
|920390|Taille totale d’arguments dépassée|
|920400|Taille du fichier chargé trop volumineuse|
|920410|Taille totale des fichiers chargés trop volumineuse|
|920420|Type de contenu de requête non autorisé par la stratégie|
|920430|Version du protocole HTTP non autorisée par la stratégie|
|920440|Extension de fichier URL limitée par la stratégie|
|920450|En-tête HTTP limité par la stratégie (%@{MATCHED_VAR})|
|920460|Caractères d’échappement anormaux|
|920470|En-tête Content-type non conforme|
|920480|Restriction du paramètre charset dans l’en-tête Content-type|

### <a name="p-x-ms-format-detectionnonerequest-921-protocol-attackp"></a><a name="crs921-31"></a> <p x-ms-format-detection="none">REQUEST-921-PROTOCOL-ATTACK</p>

|ID de la règle|Description|
|---|---|
|921110|Attaque par dissimulation de requête HTTP|
|921120|Attaque par fractionnement de réponse HTTP|
|921130|Attaque par fractionnement de réponse HTTP|
|921140|Attaque par injection d’en-tête HTTP via les en-têtes|
|921150|Attaque par injection d’en-tête HTTP via la charge utile (CR/LF détecté)|
|921151|Attaque par injection d’en-tête HTTP via la charge utile (CR/LF détecté)|
|921160|Attaque par injection d’en-tête HTTP via la charge utile (CR/LF et nom d’en-tête détecté)|
|921170|Pollution du paramètre HTTP|
|921180|Pollution du paramètre HTTP (%{TX.1})|

### <a name="p-x-ms-format-detectionnonerequest-930-application-attack-lfip"></a><a name="crs930-31"></a> <p x-ms-format-detection="none">REQUEST-930-APPLICATION-ATTACK-LFI</p>

|ID de la règle|Description|
|---|---|
|930100|Attaque par traversée de chemin (/../)|
|930110|Attaque par traversée de chemin (/../)|
|930120|Tentative d’accès au fichier du système d’exploitation|
|930130|Tentative d’accès au fichier restreint|

### <a name="p-x-ms-format-detectionnonerequest-931-application-attack-rfip"></a><a name="crs931-31"></a> <p x-ms-format-detection="none">REQUEST-931-APPLICATION-ATTACK-RFI</p>

|ID de la règle|Description|
|---|---|
|931100|Possible attaque par inclusion de fichier distant = Paramètre d’URL utilisant l’adresse IP|
|931110|Possible attaque par inclusion de fichier distant = Nom de paramètre vulnérable RFI commun utilisé avec la charge utile URL|
|931120|Possible attaque par inclusion de fichier distant = Charge utile URL utilisée avec caractère point d’interrogation de fin (?)|
|931130|Attaque possible par inclusion de fichier distant = Référence/Lien hors domaine|

### <a name="p-x-ms-format-detectionnonerequest-932-application-attack-rcep"></a><a name="crs932-31"></a> <p x-ms-format-detection="none">REQUEST-932-APPLICATION-ATTACK-RCE</p>

|ID de la règle|Description|
|---|---|
|932100|Exécution de commande à distance : Injection de commande Unix|
|932105|Exécution de commande à distance : Injection de commande Unix|
|932106|Exécution de commande à distance : Injection de commande Unix|
|932110|Exécution de commande à distance : Injection de commande Windows|
|932115|Exécution de commande à distance : Injection de commande Windows|
|932120|Exécution de la commande à distance = Commande Windows PowerShell détectée|
|932130|Exécution de la commande à distance = Expression Shell Unix détectée|
|932140|Exécution de la commande à distance = Commande Windows FOR/IF détectée|
|932150|Exécution de commande à distance : Exécution de commande UNIX directe|
|932160|Exécution de la commande à distance = Code Shell Unix détecté|
|932170|Exécution de la commande à distance = Shellshock (CVE-2014-6271)|
|932171|Exécution de la commande à distance = Shellshock (CVE-2014-6271)|
|932180|Tentative de chargement de fichier limitée|
|932190|Exécution de commande à distance : Tentative de technique de contournement générique|

### <a name="p-x-ms-format-detectionnonerequest-933-application-attack-phpp"></a><a name="crs933-31"></a> <p x-ms-format-detection="none">REQUEST-933-APPLICATION-ATTACK-PHP</p>

|ID de la règle|Description|
|---|---|
|933100|Attaque par injection de code PHP = Balise d’ouverture/fermeture détectée|
|933110|Attaque par injection de code PHP = Chargement de fichiers de script PHP détecté|
|933111|Attaque par injection de code PHP : Chargement de fichier de script PHP détecté|
|933120|Attaque par injection de code PHP = Directive de configuration détectée|
|933130|Attaque par injection de code PHP = Variables détectées|
|933131|Attaque par injection de code PHP : Variables détectées|
|933140|Attaque par injection de code PHP : Flux d’E/S détecté|
|933150|Attaque par injection de code PHP = Nom de fonction PHP à risque élevé détecté|
|933151|Attaque par injection de code PHP : Nom de fonction PHP à risque moyen détecté|
|933160|Attaque par injectionde code PHP = Nom d’appel de fonction PHP à risque élevé détecté|
|933161|Attaque par injection de code PHP : Appel de fonction PHP à valeur faible détecté|
|933170|Attaque par injection de code PHP : Injection d’objet sérialisé|
|933180|Attaque par injection de code PHP = Appel de fonction variable détecté|
|933190|Attaque par injection de code PHP : Balise de fermeture PHP détectée|

### <a name="p-x-ms-format-detectionnonerequest-941-application-attack-xssp"></a><a name="crs941-31"></a> <p x-ms-format-detection="none">REQUEST-941-APPLICATION-ATTACK-XSS</p>

|ID de la règle|Description|
|---|---|
|941100|Attaque XSS détectée via libinjection|
|941101|Attaque XSS détectée via libinjection|
|941110|Filtre XSS - Catégorie 1 = vecteur de balise de script|
|941130|Filtre XSS - Catégorie 3 = vecteur d’attribut|
|941140|Filtre XSS - Catégorie 4 = vecteur URI Javascript|
|941150|Filtre XSS - Catégorie 5 = attributs HTML non autorisés|
|941160|NoScript XSS InjectionChecker : Injection de code HTML|
|941170|NoScript XSS InjectionChecker : Injection d’attribut|
|941180|Mots clés de la liste rouge du validateur de nœuds|
|941190|XSS utilisant des feuilles de style|
|941200|XSS utilisant des frames VML|
|941210|XSS utilisant du JavaScript obfusqué|
|941220|XSS utilisant du VB Script obfusqué|
|941230|XSS utilisant la balise « embed »|
|941240|XSS utilisant l’attribut « import » ou « implementation »|
|941250|Filtre XSS IE - Attaque détectée|
|941260|XSS utilisant la balise « meta »|
|941270|XSS utilisant le href « link »|
|941280|XSS utilisant la balise « base »|
|941290|XSS utilisant la balise « applet »|
|941300|XSS utilisant la balise « object »|
|941310|Filtre XSS d’encodage incorrectement formé US-ASCII - Attaque détectée.|
|941320|Possible attaque XSS détectée - Gestionnaire de balise HTML|
|941330|Filtre XSS IE - Attaque détectée.|
|941340|Filtre XSS IE - Attaque détectée.|
|941350|XSS IE d’encodage UTF-7 - Attaque détectée.|


### <a name="p-x-ms-format-detectionnonerequest-942-application-attack-sqlip"></a><a name="crs942-31"></a> <p x-ms-format-detection="none">REQUEST-942-APPLICATION-ATTACK-SQLI</p>

|ID de la règle|Description|
|---|---|
|942100|Attaque par injection de code SQL détectée via libinjection|
|942110|Attaque par injection de code SQL : Test d’injection commune détecté|
|942120|Attaque par injection de code SQL : Opérateur SQL détecté|
|942130|Attaque par injection de code SQL : Tautologie SQL détectée.|
|942140|Attaque par injection de code SQL = Noms de base de données courants détectés|
|942150|Attaque par injection de code SQL|
|942160|Détecte des tests sqli à l’aveugle à l’aide de la méthode sleep() ou benchmarch().|
|942170|Détecte les tentatives d’injection par les méthodes benchmark (test d’évaluation) et sleep (mise en veille), y compris les requêtes conditionnelles|
|942180|Détecte les tentatives de contournement d’authentification SQL de base 1/3|
|942190|Détecte les tentatives de collecte d’informations et de d’exécution de code MSSQL|
|942200|Détecte les injections MySQL de type comment-/space-obfuscated et d’accent grave|
|942210|Détecte les tentatives d’injection de code SQL chaîné 1/2|
|942220|Recherche des attaques par dépassement de capacité d’entiers, extraites de Skipfish, à l’exception de 3.0.00738585072|
|942230|Détecte des tentatives d’injection de code SQL conditionnel|
|942240|Détecte les tentatives de basculement de charset MySQL et d’attaque DoS MSSQL|
|942250|Détecte les injections de code MATCH AGAINST, MERGE et EXECUTE IMMEDIATE|
|942251|Détecte des injections de code HAVING|
|942260|Détecte les tentatives de contournement d’authentification SQL de base 2/3|
|942270|Recherche d’injection de code SQL de base. Chaîne d’attaque courante pour mysql oracle entre autres|
|942280|Détecte l’injection de Postgres pg_sleep, les attaques par délai d’attente et les tentatives d’arrêt de base de données|
|942290|Recherche de tentatives d’injection de code SQL MongoDB de base|
|942300|Détecte les commentaires MySQL, les conditions et les injections de ch(a)r|
|942310|Détecte des tentatives d’injection de code SQL chaîné 2/2|
|942320|Détecte des injections de fonctions/procédures stockées MySQL et PostgresSQL|
|942330|Détecte les sondes d’injection SQL classiques 1/2|
|942340|Détecte les tentatives de contournement d’authentification SQL de base 3/3|
|942350|Détecte l’injection de code UDF MySQL et autres tentatives de manipulation de données/structures|
|942360|Détecte l’injection SQL de base concaténée et les tentatives SQLLFI|
|942361|Détecte l’injection de code SQL de base par le biais du mot clé alter ou union|
|942370|Détecte les sondes d’injection SQL classiques 2/2|
|942380|Attaque par injection de code SQL|
|942390|Attaque par injection de code SQL|
|942400|Attaque par injection de code SQL|
|942410|Attaque par injection de code SQL|
|942420|Détection restreinte d’anomalies de caractères SQL (cookies) : nombre de caractères spéciaux dépassés (8)|
|942421|Détection restreinte d’anomalies de caractères SQL (cookies) : nombre de caractères spéciaux dépassés (3)|
|942430|Détection restreinte d’anomalies de caractères SQL (args) : nombre de caractères spéciaux dépassés (12)|
|942431|Détection restreinte d’anomalies de caractères SQL (args) : nombre de caractères spéciaux dépassés (6)|
|942432|Détection restreinte d’anomalies de caractères SQL (args) : nombre de caractères spéciaux dépassés (2)|
|942440|Séquence de commentaire SQL détectée.|
|942450|Encodage hexadécimal SQL identifié|
|942460|Alerte de détection d’anomalies de métacaractères - Caractères non textuels répétitifs|
|942470|Attaque par injection de code SQL|
|942480|Attaque par injection de code SQL|
|942490|Détecte les sondes d’injection SQL classiques 3/3|

### <a name="p-x-ms-format-detectionnonerequest-943-application-attack-session-fixationp"></a><a name="crs943-31"></a> <p x-ms-format-detection="none">REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION</p>

|ID de la règle|Description|
|---|---|
|943100|Possible attaque par fixation de session = Définition de valeurs de cookies en HTML|
|943110|Possible attaque par fixation de session = Nom du paramètre SessionID avec référent hors domaine|
|943120|Possible attaque par fixation de session = Nom du paramètre SessionID sans référent|

### <a name="p-x-ms-format-detectionnonerequest-944-application-attack-session-javap"></a><a name="crs944-31"></a> <p x-ms-format-detection="none">REQUEST-944-APPLICATION-ATTACK-SESSION-JAVA</p>

|ID de la règle|Description|
|---|---|
|944120|Exécution de charge utile et de commande à distance possible|
|944130|Classes Java suspectes|
|944200|Exploitation de la désérialisation Java Apache Commons|

# <a name="owasp-30"></a>[OWASP 3.0](#tab/owasp30)

## <a name="rule-sets"></a><a name="owasp30"></a> Ensembles de règles

### <a name="p-x-ms-format-detectionnonegeneralp"></a><a name="general-30"></a> <p x-ms-format-detection="none">Général</p>

|ID de la règle|Description|
|---|---|
|200004|Possible limite multipart sans correspondance.|

### <a name="p-x-ms-format-detectionnonerequest-911-method-enforcementp"></a><a name="crs911-30"></a> <p x-ms-format-detection="none">REQUEST-911-METHOD-ENFORCEMENT</p>

|ID de la règle|Description|
|---|---|
|911100|Méthode non autorisée par la stratégie|


### <a name="p-x-ms-format-detectionnonerequest-913-scanner-detectionp"></a><a name="crs913-30"></a> <p x-ms-format-detection="none">REQUEST-913-SCANNER-DETECTION</p>

|ID de la règle|Description|
|---|---|
|913100|Détection d’agent utilisateur associé au scanner de sécurité|
|913110|Détection d’en-tête de requête associé au scanner de sécurité|
|913120|Détection de nom de fichier/argument associé au scanner de sécurité|
|913101|Détection d’agent utilisateur associé aux scripts/client HTTP générique|
|913102|Détection d’agent utilisateur associé à l’analyseur web/robot|

### <a name="p-x-ms-format-detectionnonerequest-920-protocol-enforcementp"></a><a name="crs920-30"></a> <p x-ms-format-detection="none">REQUEST-920-PROTOCOL-ENFORCEMENT</p>

|ID de la règle|Description|
|---|---|
|920100|Ligne de requête HTTP non valide|
|920130|Échec de l’analyse du corps de la requête.|
|920140|Échec de la validation stricte du corps de la requête à parties multiples|
|920160|L’en-tête HTTP Content-Length n’est pas numérique.|
|920170|Requête GET ou HEAD avec contenu du corps.|
|920180|En-tête Content-Length manquant dans la requête POST.|
|920190|Plage = dernière valeur d’octet non valide.|
|920210|Détection de données d’en-tête de connexion en conflit/multiples.|
|920220|Tentative d’attaque abusive d’encodage d’URL|
|920240|Tentative d’attaque abusive d’encodage d’URL|
|920250|Tentative d’attaque abusive d’encodage UTF8|
|920260|Tentative d’attaque abusive Pleine/Moyenne largeur Unicode|
|920270|Caractère non valide dans la requête (caractère null)|
|920280|En-tête d’hôte manquant dans la requête|
|920290|En-tête d’hôte vide|
|920310|Requête contenant un en-tête Accept vide|
|920311|Requête contenant un en-tête Accept vide|
|920330|En-tête User-Agent vide|
|920340|Requête contenant du contenu mais dont l’en-tête Content-Type est manquant|
|920350|L’en-tête d’hôte est une adresse IP numérique|
|920380|Trop d’arguments dans la requête|
|920360|Nom d’argument trop long|
|920370|Valeur d’argument trop longue|
|920390|Taille totale d’arguments dépassée|
|920400|Taille du fichier chargé trop volumineuse|
|920410|Taille totale des fichiers chargés trop volumineuse|
|920420|Type de contenu de requête non autorisé par la stratégie|
|920430|Version du protocole HTTP non autorisée par la stratégie|
|920440|Extension de fichier URL limitée par la stratégie|
|920450|En-tête HTTP limité par la stratégie (%@{MATCHED_VAR})|
|920200|Plage = Champs trop nombreux (6 ou plus)|
|920201|Plage = Champs trop nombreux pour la requête PDF (35 ou plus)|
|920230|Détection d’encodage de plusieurs URL|
|920300|En-tête Accept manquant dans la requête|
|920271|Caractère non valide dans la requête (caractères non imprimables)|
|920320|En-tête User-Agent manquant|
|920272|Caractère non valide dans la requête (en dehors des caractères imprimables avant ascii 127)|
|920202|Plage = Champs trop nombreux pour la requête PDF (6 ou plus)|
|920273|Caractère non valide dans la requête (en dehors de l’ensemble très strict)|
|920274|Caractère non valide dans les en-têtes de requête (en dehors de l’ensemble très strict)|
|920460|Caractères d’échappement anormaux|

### <a name="p-x-ms-format-detectionnonerequest-921-protocol-attackp"></a><a name="crs921-30"></a> <p x-ms-format-detection="none">REQUEST-921-PROTOCOL-ATTACK</p>

|ID de la règle|Description|
|---|---|
|921100|Attaque par dissimulation de requête HTTP.|
|921110|Attaque par dissimulation de requête HTTP|
|921120|Attaque par fractionnement de réponse HTTP|
|921130|Attaque par fractionnement de réponse HTTP|
|921140|Attaque par injection d’en-tête HTTP via les en-têtes|
|921150|Attaque par injection d’en-tête HTTP via la charge utile (CR/LF détecté)|
|921160|Attaque par injection d’en-tête HTTP via la charge utile (CR/LF et nom d’en-tête détecté)|
|921151|Attaque par injection d’en-tête HTTP via la charge utile (CR/LF détecté)|
|921170|Pollution du paramètre HTTP|
|921180|Pollution du paramètre HTTP (% @{TX.1})|

### <a name="p-x-ms-format-detectionnonerequest-930-application-attack-lfip"></a><a name="crs930-30"></a> <p x-ms-format-detection="none">REQUEST-930-APPLICATION-ATTACK-LFI</p>

|ID de la règle|Description|
|---|---|
|930100|Attaque par traversée de chemin (/../)|
|930110|Attaque par traversée de chemin (/../)|
|930120|Tentative d’accès au fichier du système d’exploitation|
|930130|Tentative d’accès au fichier restreint|

### <a name="p-x-ms-format-detectionnonerequest-931-application-attack-rfip"></a><a name="crs931-30"></a> <p x-ms-format-detection="none">REQUEST-931-APPLICATION-ATTACK-RFI</p>

|ID de la règle|Description|
|---|---|
|931100|Possible attaque par inclusion de fichier distant = Paramètre d’URL utilisant l’adresse IP|
|931110|Possible attaque par inclusion de fichier distant = Nom de paramètre vulnérable RFI commun utilisé avec la charge utile URL|
|931120|Possible attaque par inclusion de fichier distant = Charge utile URL utilisée avec caractère point d’interrogation de fin (?)|
|931130|Attaque possible par inclusion de fichier distant = Référence/Lien hors domaine|

### <a name="p-x-ms-format-detectionnonerequest-932-application-attack-rcep"></a><a name="crs932-30"></a> <p x-ms-format-detection="none">REQUEST-932-APPLICATION-ATTACK-RCE</p>

|ID de la règle|Description|
|---|---|
|932120|Exécution de la commande à distance = Commande Windows PowerShell détectée|
|932130|Exécution de la commande à distance = Expression Shell Unix détectée|
|932140|Exécution de la commande à distance = Commande Windows FOR/IF détectée|
|932160|Exécution de la commande à distance = Code Shell Unix détecté|
|932170|Exécution de la commande à distance = Shellshock (CVE-2014-6271)|
|932171|Exécution de la commande à distance = Shellshock (CVE-2014-6271)|

### <a name="p-x-ms-format-detectionnonerequest-933-application-attack-phpp"></a><a name="crs933-30"></a> <p x-ms-format-detection="none">REQUEST-933-APPLICATION-ATTACK-PHP</p>

|ID de la règle|Description|
|---|---|
|933100|Attaque par injection de code PHP = Balise d’ouverture/fermeture détectée|
|933110|Attaque par injection de code PHP = Chargement de fichiers de script PHP détecté|
|933120|Attaque par injection de code PHP = Directive de configuration détectée|
|933130|Attaque par injection de code PHP = Variables détectées|
|933150|Attaque par injection de code PHP = Nom de fonction PHP à risque élevé détecté|
|933160|Attaque par injectionde code PHP = Nom d’appel de fonction PHP à risque élevé détecté|
|933180|Attaque par injection de code PHP = Appel de fonction variable détecté|
|933151|Attaque par injection de code PHP = Nom de fonction PHP à risque modéré détecté|
|933131|Attaque par injection de code PHP = Variables détectées|
|933161|Attaque par injection de code PHP = Appel de fonction PHP à risque faible détecté|
|933111|Attaque par injection de code PHP = Chargement de fichiers de script PHP détecté|

### <a name="p-x-ms-format-detectionnonerequest-941-application-attack-xssp"></a><a name="crs941-30"></a> <p x-ms-format-detection="none">REQUEST-941-APPLICATION-ATTACK-XSS</p>

|ID de la règle|Description|
|---|---|
|941100|Attaque XSS détectée via libinjection|
|941110|Filtre XSS - Catégorie 1 = vecteur de balise de script|
|941130|Filtre XSS - Catégorie 3 = vecteur d’attribut|
|941140|Filtre XSS - Catégorie 4 = vecteur URI Javascript|
|941150|Filtre XSS - Catégorie 5 = attributs HTML non autorisés|
|941180|Mots clés de la liste rouge du validateur de nœuds|
|941190|XSS utilisant des feuilles de style|
|941200|XSS utilisant des frames VML|
|941210|XSS utilisant du JavaScript obfusqué|
|941220|XSS utilisant du VB Script obfusqué|
|941230|XSS utilisant la balise « embed »|
|941240|XSS utilisant l’attribut « import » ou « implementation »|
|941260|XSS utilisant la balise « meta »|
|941270|XSS utilisant le href « link »|
|941280|XSS utilisant la balise « base »|
|941290|XSS utilisant la balise « applet »|
|941300|XSS utilisant la balise « object »|
|941310|Filtre XSS d’encodage incorrectement formé US-ASCII - Attaque détectée.|
|941330|Filtre XSS IE - Attaque détectée.|
|941340|Filtre XSS IE - Attaque détectée.|
|941350|XSS IE d’encodage UTF-7 - Attaque détectée.|
|941320|Possible attaque XSS détectée - Gestionnaire de balise HTML|

### <a name="p-x-ms-format-detectionnonerequest-942-application-attack-sqlip"></a><a name="crs942-30"></a> <p x-ms-format-detection="none">REQUEST-942-APPLICATION-ATTACK-SQLI</p>

|ID de la règle|Description|
|---|---|
|942100|Attaque par injection de code SQL détectée via libinjection|
|942110|Attaque par injection de code SQL : Test d’injection commune détecté|
|942130|Attaque par injection de code SQL : Tautologie SQL détectée.|
|942140|Attaque par injection de code SQL = Noms de base de données courants détectés|
|942160|Détecte des tests sqli à l’aveugle à l’aide de la méthode sleep() ou benchmarch().|
|942170|Détecte les tentatives d’injection par les méthodes benchmark (test d’évaluation) et sleep (mise en veille), y compris les requêtes conditionnelles|
|942190|Détecte les tentatives de collecte d’informations et de d’exécution de code MSSQL|
|942200|Détecte les injections MySQL de type comment-/space-obfuscated et d’accent grave|
|942230|Détecte des tentatives d’injection de code SQL conditionnel|
|942260|Détecte les tentatives de contournement d’authentification SQL de base 2/3|
|942270|Recherche d’injection de code SQL de base. Chaîne d’attaque courante pour mysql oracle entre autres.|
|942290|Recherche de tentatives d’injection de code SQL MongoDB de base|
|942300|Détecte les commentaires MySQL, les conditions et les injections de ch(a)r|
|942310|Détecte des tentatives d’injection de code SQL chaîné 2/2|
|942320|Détecte des injections de fonctions/procédures stockées MySQL et PostgresSQL|
|942330|Détecte les sondes d’injection SQL classiques 1/2|
|942340|Détecte les tentatives de contournement d’authentification SQL de base 3/3|
|942350|Détecte l’injection de code UDF MySQL et autres tentatives de manipulation de données/structures|
|942360|Détecte l’injection SQL de base concaténée et les tentatives SQLLFI|
|942370|Détecte les sondes d’injection SQL classiques 2/2|
|942150|Attaque par injection de code SQL|
|942410|Attaque par injection de code SQL|
|942430|Détection restreinte d’anomalies de caractères SQL (args) : nombre de caractères spéciaux dépassés (12)|
|942440|Séquence de commentaire SQL détectée.|
|942450|Encodage hexadécimal SQL identifié|
|942251|Détecte des injections de code HAVING|
|942460|Alerte de détection d’anomalies de métacaractères - Caractères non textuels répétitifs|

### <a name="p-x-ms-format-detectionnonerequest-943-application-attack-session-fixationp"></a><a name="crs943-30"></a> <p x-ms-format-detection="none">REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION</p>

|ID de la règle|Description|
|---|---|
|943100|Possible attaque par fixation de session = Définition de valeurs de cookies en HTML|
|943110|Possible attaque par fixation de session = Nom du paramètre SessionID avec référent hors domaine|
|943120|Possible attaque par fixation de session = Nom du paramètre SessionID sans référent|

# <a name="owasp-229"></a>[OWASP 2.2.9](#tab/owasp2)

## <a name="rule-sets"></a><a name="owasp229"></a> Ensembles de règles

### <a name="crs_20_protocol_violations"></a><a name="crs20"></a> crs_20_protocol_violations

|ID de la règle|Description|
|---|---|
|960911|Ligne de requête HTTP non valide|
|981227|Erreur Apache = URI non valide dans la requête.|
|960912|Échec de l’analyse du corps de la requête.|
|960914|Échec de la validation stricte du corps de la requête à parties multiples|
|960915|L’analyseur multipart a détecté une possible limite sans correspondance.|
|960016|L’en-tête HTTP Content-Length n’est pas numérique.|
|960011|Requête GET ou HEAD avec contenu du corps.|
|960012|En-tête Content-Length manquant dans la requête POST.|
|960902|Utilisation incorrecte de l’encodage d’identités.|
|960022|En-tête attendu non autorisé pour HTTP 1.0.|
|960020|L’en-tête Pragma requiert un en-tête Cache-Control pour les requêtes HTTP/1.1.|
|958291|Plage = Le champ existe et il commence par 0.|
|958230|Plage = dernière valeur d’octet non valide.|
|958295|Détection de données d’en-tête de connexion en conflit/multiples.|
|950107|Tentative d’attaque abusive d’encodage d’URL|
|950109|Détection d’encodage de plusieurs URL|
|950108|Tentative d’attaque abusive d’encodage d’URL|
|950801|Tentative d’attaque abusive d’encodage UTF8|
|950116|Tentative d’attaque abusive Pleine/Moyenne largeur Unicode|
|960901|Caractère non valide dans la requête|
|960018|Caractère non valide dans la requête|

### <a name="crs_21_protocol_anomalies"></a><a name="crs21"></a> crs_21_protocol_anomalies

|ID de la règle|Description|
|---|---|
|960008|En-tête d’hôte manquant dans la requête|
|960007|En-tête d’hôte vide|
|960015|En-tête Accept manquant dans la requête|
|960021|Requête contenant un en-tête Accept vide|
|960009|En-tête User-Agent manquant dans la requête|
|960006|En-tête User-Agent vide|
|960904|Requête contenant du contenu mais dont l’en-tête Content-Type est manquant|
|960017|L’en-tête d’hôte est une adresse IP numérique|

### <a name="crs_23_request_limits"></a><a name="crs23"></a> crs_23_request_limits

|ID de la règle|Description|
|---|---|
|960209|Nom d’argument trop long|
|960208|Valeur d’argument trop longue|
|960335|Trop d’arguments dans la requête|
|960341|Taille totale d’arguments dépassée|
|960342|Taille du fichier chargé trop volumineuse|
|960343|Taille totale des fichiers chargés trop volumineuse|

### <a name="crs_30_http_policy"></a><a name="crs30"></a> crs_30_http_policy

|ID de la règle|Description|
|---|---|
|960032|Méthode non autorisée par la stratégie|
|960010|Type de contenu de requête non autorisé par la stratégie|
|960034|Version du protocole HTTP non autorisée par la stratégie|
|960035|Extension de fichier URL limitée par la stratégie|
|960038|En-tête HTTP limité par la stratégie|

### <a name="crs_35_bad_robots"></a><a name="crs35"></a> crs_35_bad_robots

|ID de la règle|Description|
|---|---|
|990002|La requête indique qu’un système d’analyse de la sécurité a analysé le site|
|990901|La requête indique qu’un système d’analyse de la sécurité a analysé le site|
|990902|La requête indique qu’un système d’analyse de la sécurité a analysé le site|
|990012|Analyseur de sites web non fiables|

### <a name="crs_40_generic_attacks"></a><a name="crs40"></a> crs_40_generic_attacks

|ID de la règle|Description|
|---|---|
|960024|Alerte de détection d’anomalies de métacaractères - Caractères non textuels répétitifs|
|950008|Injection de balises ColdFusion non documentées|
|950010|Attaque par injection de code LDAP|
|950011|Attaque par injection de code SSI|
|950018|URL XSS Universal PDF détectée.|
|950019|Attaque par injection d’e-mail|
|950012|Attaque par dissimulation de requête HTTP.|
|950910|Attaque par fractionnement de réponse HTTP|
|950911|Attaque par fractionnement de réponse HTTP|
|950117|Attaque par inclusion de fichier distant|
|950118|Attaque par inclusion de fichier distant|
|950119|Attaque par inclusion de fichier distant|
|950120|Attaque possible par inclusion de fichier distant = Référence/Lien hors domaine|
|981133|Règle 981133|
|981134|Règle 981134|
|950009|Attaque par fixation de session|
|950003|Fixation de session|
|950000|Fixation de session|
|950005|Tentative d’accès au fichier distant|
|950002|Accès aux commandes système|
|950006|Injection de commandes système|
|959151|Attaque par injection de code PHP|
|958976|Attaque par injection de code PHP|
|958977|Attaque par injection de code PHP|

### <a name="crs_41_sql_injection_attacks"></a><a name="crs41sql"></a> crs_41_sql_injection_attacks

|ID de la règle|Description|
|---|---|
|981231|Séquence de commentaire SQL détectée.|
|981260|Encodage hexadécimal SQL identifié|
|981320|Attaque par injection de code SQL = Noms de base de données courants détectés|
|981300|Règle 981300|
|981301|Règle 981301|
|981302|Règle 981302|
|981303|Règle 981303|
|981304|Règle 981304|
|981305|Règle 981305|
|981306|Règle 981306|
|981307|Règle 981307|
|981308|Règle 981308|
|981309|Règle 981309|
|981310|Règle 981310|
|981311|Règle 981311|
|981312|Règle 981312|
|981313|Règle 981313|
|981314|Règle 981314|
|981315|Règle 981315|
|981316|Règle 981316|
|981317|Alerte de détection d’anomalies d’instructions SQL SELECT|
|950007|Attaque par injection de code SQL aveugle|
|950001|Attaque par injection de code SQL|
|950908|Attaque par injection de code SQL.|
|959073|Attaque par injection de code SQL|
|981272|Détecte des tests sqli à l’aveugle à l’aide de la méthode sleep() ou benchmarch().|
|981250|Détecte les tentatives d’injection par les méthodes benchmark (test d’évaluation) et sleep (mise en veille), y compris les requêtes conditionnelles|
|981241|Détecte des tentatives d’injection de code SQL conditionnel|
|981276|Recherche d’injection de code SQL de base. Chaîne d’attaque courante pour mysql oracle entre autres.|
|981270|Recherche de tentatives d’injection de code SQL MongoDB de base|
|981253|Détecte des injections de fonctions/procédures stockées MySQL et PostgresSQL|
|981251|Détecte l’injection de code UDF MySQL et autres tentatives de manipulation de données/structures|

### <a name="crs_41_xss_attacks"></a><a name="crs41xss"></a> crs_41_xss_attacks

|ID de la règle|Description|
|---|---|
|973336|Filtre XSS - Catégorie 1 = vecteur de balise de script|
|973338|Filtre XSS - Catégorie 3 = Vecteur URI Javascript|
|981136|Règle 981136|
|981018|Règle 981018|
|958016|Attaque par scripts intersites (XSS)|
|958414|Attaque par scripts intersites (XSS)|
|958032|Attaque par scripts intersites (XSS)|
|958026|Attaque par scripts intersites (XSS)|
|958027|Attaque par scripts intersites (XSS)|
|958054|Attaque par scripts intersites (XSS)|
|958418|Attaque par scripts intersites (XSS)|
|958034|Attaque par scripts intersites (XSS)|
|958019|Attaque par scripts intersites (XSS)|
|958013|Attaque par scripts intersites (XSS)|
|958408|Attaque par scripts intersites (XSS)|
|958012|Attaque par scripts intersites (XSS)|
|958423|Attaque par scripts intersites (XSS)|
|958002|Attaque par scripts intersites (XSS)|
|958017|Attaque par scripts intersites (XSS)|
|958007|Attaque par scripts intersites (XSS)|
|958047|Attaque par scripts intersites (XSS)|
|958410|Attaque par scripts intersites (XSS)|
|958415|Attaque par scripts intersites (XSS)|
|958022|Attaque par scripts intersites (XSS)|
|958405|Attaque par scripts intersites (XSS)|
|958419|Attaque par scripts intersites (XSS)|
|958028|Attaque par scripts intersites (XSS)|
|958057|Attaque par scripts intersites (XSS)|
|958031|Attaque par scripts intersites (XSS)|
|958006|Attaque par scripts intersites (XSS)|
|958033|Attaque par scripts intersites (XSS)|
|958038|Attaque par scripts intersites (XSS)|
|958409|Attaque par scripts intersites (XSS)|
|958001|Attaque par scripts intersites (XSS)|
|958005|Attaque par scripts intersites (XSS)|
|958404|Attaque par scripts intersites (XSS)|
|958023|Attaque par scripts intersites (XSS)|
|958010|Attaque par scripts intersites (XSS)|
|958411|Attaque par scripts intersites (XSS)|
|958422|Attaque par scripts intersites (XSS)|
|958036|Attaque par scripts intersites (XSS)|
|958000|Attaque par scripts intersites (XSS)|
|958018|Attaque par scripts intersites (XSS)|
|958406|Attaque par scripts intersites (XSS)|
|958040|Attaque par scripts intersites (XSS)|
|958052|Attaque par scripts intersites (XSS)|
|958037|Attaque par scripts intersites (XSS)|
|958049|Attaque par scripts intersites (XSS)|
|958030|Attaque par scripts intersites (XSS)|
|958041|Attaque par scripts intersites (XSS)|
|958416|Attaque par scripts intersites (XSS)|
|958024|Attaque par scripts intersites (XSS)|
|958059|Attaque par scripts intersites (XSS)|
|958417|Attaque par scripts intersites (XSS)|
|958020|Attaque par scripts intersites (XSS)|
|958045|Attaque par scripts intersites (XSS)|
|958004|Attaque par scripts intersites (XSS)|
|958421|Attaque par scripts intersites (XSS)|
|958009|Attaque par scripts intersites (XSS)|
|958025|Attaque par scripts intersites (XSS)|
|958413|Attaque par scripts intersites (XSS)|
|958051|Attaque par scripts intersites (XSS)|
|958420|Attaque par scripts intersites (XSS)|
|958407|Attaque par scripts intersites (XSS)|
|958056|Attaque par scripts intersites (XSS)|
|958011|Attaque par scripts intersites (XSS)|
|958412|Attaque par scripts intersites (XSS)|
|958008|Attaque par scripts intersites (XSS)|
|958046|Attaque par scripts intersites (XSS)|
|958039|Attaque par scripts intersites (XSS)|
|958003|Attaque par scripts intersites (XSS)|
|973300|Possible attaque XSS détectée - Gestionnaire de balise HTML|
|973301|Attaque XSS détectée|
|973302|Attaque XSS détectée|
|973303|Attaque XSS détectée|
|973304|Attaque XSS détectée|
|973305|Attaque XSS détectée|
|973306|Attaque XSS détectée|
|973307|Attaque XSS détectée|
|973308|Attaque XSS détectée|
|973309|Attaque XSS détectée|
|973311|Attaque XSS détectée|
|973313|Attaque XSS détectée|
|973314|Attaque XSS détectée|
|973331|Filtre XSS IE - Attaque détectée.|
|973315|Filtre XSS IE - Attaque détectée.|
|973330|Filtre XSS IE - Attaque détectée.|
|973327|Filtre XSS IE - Attaque détectée.|
|973326|Filtre XSS IE - Attaque détectée.|
|973346|Filtre XSS IE - Attaque détectée.|
|973345|Filtre XSS IE - Attaque détectée.|
|973324|Filtre XSS IE - Attaque détectée.|
|973323|Filtre XSS IE - Attaque détectée.|
|973348|Filtre XSS IE - Attaque détectée.|
|973321|Filtre XSS IE - Attaque détectée.|
|973320|Filtre XSS IE - Attaque détectée.|
|973318|Filtre XSS IE - Attaque détectée.|
|973317|Filtre XSS IE - Attaque détectée.|
|973329|Filtre XSS IE - Attaque détectée.|
|973328|Filtre XSS IE - Attaque détectée.|

### <a name="crs_42_tight_security"></a><a name="crs42"></a> crs_42_tight_security

|ID de la règle|Description|
|---|---|
|950103|Attaque par traversée de chemin|

### <a name="crs_45_trojans"></a><a name="crs45"></a> crs_45_trojans

|ID de la règle|Description|
|---|---|
|950110|Accès par porte dérobée|
|950921|Accès par porte dérobée|
|950922|Accès par porte dérobée|

---

## <a name="next-steps"></a>Étapes suivantes

- [Personnaliser les règles du pare-feu d’applications web à l’aide du portail Azure](application-gateway-customize-waf-rules-portal.md)
