---
title: Chiffrement - Outil Microsoft de modélisation des menaces - Azure | Microsoft Docs
description: En savoir plus sur l’atténuation des menaces exposées dans l’outil Threat Modeling Tool, possible grâce au chiffrement. Voir les informations sur l’atténuation et les exemples de code.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.openlocfilehash: 7af115d8f1244253e461f796c5665609d3b84b21
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517442"
---
# <a name="security-frame-cryptography--mitigations"></a>Infrastructure de sécurité : chiffrement | Atténuation des risques 

| Produit/Service | Article |
| --------------- | ------- |
| **Application Web** | <ul><li>[Utiliser uniquement les longueurs de clé et les chiffrements par bloc symétriques approuvés](#cipher-length)</li><li>[Utiliser les modes de chiffrement par bloc et les vecteurs d’initialisation pour les chiffrements symétriques approuvés](#vector-ciphers)</li><li>[Utiliser les algorithmes asymétriques, les longueurs de clé et le remplissage approuvés](#padding)</li><li>[Utiliser les générateurs de nombres aléatoires approuvés](#numgen)</li><li>[Ne pas utiliser les chiffrements de flux symétriques](#stream-ciphers)</li><li>[Utiliser les algorithmes de hachage MAC/HMAC/indexé approuvés](#mac-hash)</li><li>[Utiliser uniquement les fonctions de hachage de chiffrement approuvées](#hash-functions)</li></ul> |
| **Sauvegarde de la base de données** | <ul><li>[Utiliser des algorithmes de chiffrement fort pour chiffrer les données dans la base de données](#strong-db)</li><li>[Les packages SSIS doivent être chiffrés et signés numériquement](#ssis-signed)</li><li>[Ajouter une signature numérique aux éléments sécurisables de bases de données critiques](#securables-db)</li><li>[Utiliser EKM SQL Server pour protéger les clés de chiffrement](#ekm-keys)</li><li>[Utiliser la fonction AlwaysEncrypted si les clés de chiffrement ne doivent pas être révélées au moteur de base de données](#keys-engine)</li></ul> |
| **Appareil IoT** | <ul><li>[Stocker les clés de chiffrement façon sécurisée sur IoT Device](#keys-iot)</li></ul> | 
| **Passerelle de cloud IoT** | <ul><li>[Générer une clé symétrique aléatoire de longueur suffisante pour l’authentification vers IoT Hub](#random-hub)</li></ul> | 
| **Client mobile Dynamics CRM** | <ul><li>[Garantir qu’une stratégie de gestion des appareils, demandant un code confidentiel utilisateur et permettant la réinitialisation à distance, est en place](#pin-remote)</li></ul> | 
| **Client Outlook Dynamics CRM** | <ul><li>[Garantir qu’une stratégie de gestion des appareils, demandant un code confidentiel/mot de passe/verrouillage auto et chiffrant toutes les données (p. ex. BitLocker), est en place](#bitlocker)</li></ul> | 
| **Serveur d’identité** | <ul><li>[Garantir que les clés de signature sont annulées lors de l’utilisation d’IdentityServer](#rolled-server)</li><li>[Garantir qu’un ID de client et une clé secrète client forts en termes de chiffrement sont utilisés dans IdentityServer](#client-server)</li></ul> | 

## <a name="use-only-approved-symmetric-block-ciphers-and-key-lengths"></a><a id="cipher-length"></a>Utiliser uniquement les longueurs de clé et les chiffrements par bloc symétriques approuvés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Les produits doivent uniquement utiliser les chiffrements par bloc symétriques et les longueurs de clé associées qui ont été explicitement approuvés par le conseiller en matière de chiffrement de votre organisation. Les algorithmes symétriques approuvés par Microsoft incluent les chiffrements par bloc suivants :</p><ul><li>Pour le nouveau code, AES-128, AES-192 et AES-256 sont acceptables.</li><li>Pour la compatibilité descendante avec le code existant, 3DES avec trois clés est acceptable.</li><li>Pour les produits utilisant les chiffrements par bloc symétriques :<ul><li>Advanced Encryption Standard (AES) est requis pour le nouveau code.</li><li>Triple Data Encryption Standard (3DES) avec trois clés est autorisé dans le code existant pour la compatibilité descendante.</li><li>Tous les autres chiffrements par bloc, notamment RC2, DES, 3DES avec 2 clés, DESX et Skipjack, peuvent uniquement être utilisés pour le déchiffrement des données anciennes et doivent être remplacés en cas d’utilisation pour le chiffrement.</li></ul></li><li>Pour les algorithmes de chiffrement par bloc symétrique, une longueur de clé minimale de 128 bits est requise. Le seul algorithme de chiffrement par bloc recommandé pour le nouveau code est AES (AES-128, AES-192 et AES-256 sont tous acceptables).</li><li>3DES avec trois clés est actuellement acceptable s’il est déjà utilisé dans le code existant ; la transition vers AES est recommandée. DES, DESX, RC2 et Skipjack ne sont plus considérés comme sûrs. Ces algorithmes peuvent uniquement être utilisés pour le déchiffrement des données existantes pour des raisons de compatibilité descendante, et les données doivent être chiffrées de nouveau à l’aide d’un chiffrement par bloc recommandé.</li></ul><p>Notez que tous les chiffrements par bloc symétriques doivent être utilisés avec un mode de chiffrement approuvé, ce qui nécessite l’utilisation d’un vecteur d’initialisation approprié. Un vecteur d’initialisation approprié est généralement un nombre aléatoire et jamais une valeur constante.</p><p>L’utilisation d’algorithmes de chiffrement hérités ou d’autres algorithmes de chiffrement non approuvés et de longueurs de clé plus petites pour la lecture des données existantes (par opposition à l’écriture de nouvelles données) peut être autorisée après examen du comité responsable du chiffrement de votre organisation. Toutefois, vous devez demander une exception à cette exigence. En outre, dans les déploiements d’entreprise, les produits doivent envisager d’avertir les administrateurs lorsqu’un chiffrement faible est utilisé pour lire des données. Ces avertissements doivent être explicatifs et exploitables. Dans certains cas, il peut être approprié d’avoir une stratégie de groupe pour contrôler l’utilisation d’un chiffrement faible.</p><p>Algorithmes .NET autorisés pour l’agilité de chiffrement géré (par ordre de préférence)</p><ul><li>AesCng (compatible FIPS)</li><li>AuthenticatedAesCng (compatible FIPS)</li><li>AESCryptoServiceProvider (compatible FIPS)</li><li>AESManaged (non compatible FIPS)</li></ul><p>Notez qu’aucun de ces algorithmes ne peut être spécifié via les méthodes `SymmetricAlgorithm.Create` ou `CryptoConfig.CreateFromName` sans apporter de modifications au fichier machine.config. En outre, notez qu’AES dans les versions de .NET avant .NET 3.5 est nommé `RijndaelManaged`, et `AesCng` et `AuthenticatedAesCng` sont > disponibles via CodePlex et nécessitent CNG dans le système d’exploitation sous-jacent.</p>

## <a name="use-approved-block-cipher-modes-and-initialization-vectors-for-symmetric-ciphers"></a><a id="vector-ciphers"></a>Utiliser les modes de chiffrement par bloc et les vecteurs d’initialisation pour les chiffrements symétriques approuvés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Tous les chiffrements par bloc symétriques doivent être utilisés avec un mode de chiffrement symétrique approuvé. Les seuls modes approuvés sont CBC et CTS. En particulier, le mode ECB d’opération doit être évité ; l’utilisation d’ECB requiert l’examen du comité responsable du chiffrement de votre organisation. Toute utilisation d’OFB, de CFB, de CTR, de CCM et de GCM ou de tout autre mode de chiffrement doit être examiné par le comité responsable du chiffrement de votre organisation. Réutiliser le même vecteur d’initialisation avec les chiffrements par bloc dans les « modes de chiffrements de flux », tels que CTR, peut entraîner la divulgation des données chiffrées. Tous les chiffrements par bloc symétriques doivent également être utilisés avec un vecteur d’initialisation approprié. Un vecteur d’initialisation approprié est généralement un nombre aléatoire fort et jamais une valeur constante. |

## <a name="use-approved-asymmetric-algorithms-key-lengths-and-padding"></a><a id="padding"></a>Utiliser les algorithmes asymétriques, les longueurs de clé et le remplissage approuvés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>L’utilisation des algorithmes de chiffrement interdits introduit un risque significatif pour la sécurité des produits et doit être évitée. Les produits doivent utiliser uniquement ces algorithmes de chiffrement ainsi que les longueurs de clé et le remplissage associés qui ont été approuvés explicitement par le comité responsable du chiffrement de votre organisation.</p><ul><li>**RSA-** peut être utilisé pour le chiffrement, la signature et l’échange de clés. Le chiffrement RSA doit utiliser uniquement les modes de remplissage OAEP ou RSA-KEM. Le code existant peut utiliser le mode de remplissage PKCS #1 v1.5 à des fins de compatibilité uniquement. L’utilisation du remplissage nul est explicitement interdite. Des clés supérieures ou égales à 2 048 bits sont requises pour le nouveau code. Le code existant peut prendre en charge des clés inférieures à 2 048 bits uniquement à des fins de compatibilité descendante après un examen par le comité responsable du chiffrement de votre organisation. Des clés inférieures à 1 024 bits peuvent uniquement être utilisées pour le déchiffrement/la vérification d’anciennes données et doivent être remplacées si elles sont utilisées pour des opérations de chiffrement ou de signature.</li><li>**ECDSA-** peut être utilisé pour la signature uniquement. ECDSA avec des clés supérieures ou égales à 256 bits est requis pour le nouveau code. Les signatures ECDSA doivent utiliser l’une des trois courbes approuvées par NIST (P-256, P-384 ou P521). Les courbes qui ont été rigoureusement analysées peuvent être utilisées uniquement après un examen du comité responsable du chiffrement de votre organisation.</li><li>**ECDH-** peut être utilisé pour l’échange de clés uniquement. ECDH avec des clés supérieures ou égales à 256 bits est requis pour le nouveau code. Les échanges de clé ECDH doivent utiliser l’une des trois courbes approuvées par NIST (P-256, P-384 ou P521). Les courbes qui ont été rigoureusement analysées peuvent être utilisées uniquement après un examen du comité responsable du chiffrement de votre organisation.</li><li>**DSA-** peut être acceptable après examen et approbation du comité responsable du chiffrement de votre organisation. Contactez votre conseiller en matière de sécurité pour planifier l’examen par le comité responsable du chiffrement de votre organisation. Si votre utilisation de DSA est approuvée, notez que vous devrez interdire l’utilisation des clés d’une longueur inférieure à 2 048 bits. CNG prend en charge des longueurs de clé de 2 048 bits et supérieures à compter de Windows 8.</li><li>**Diffie-Hellman-** peut être utilisé pour la gestion des clés de session uniquement. Des longueurs de clé supérieures ou égales à 2 048 bits sont requises pour le nouveau code. Le code existant peut prendre en charge des longueurs de clé inférieures à 2048 bits uniquement à des fins de compatibilité descendante après un examen par le comité responsable du chiffrement de votre organisation. Des clés inférieures à 1 024 bits ne peuvent pas être utilisées.</li><ul>

## <a name="use-approved-random-number-generators"></a><a id="numgen"></a>Utiliser les générateurs de nombres aléatoires approuvés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Les produits doivent utiliser les générateurs de nombres aléatoires approuvés. Les fonctions pseudoaléatoires telles que C runtime function rand, .NET Framework class System.Random ou les fonctions système comme GetTickCount doivent par conséquent ne jamais être utilisé dans du code de ce type. L’utilisation de l’algorithme du générateur de nombres aléatoires à courbe elliptique double (DUAL_EC_DRBG) est interdite.</p><ul><li>**CNG-** BCryptGenRandom (utilisation de l’indicateur BCRYPT_USE_SYSTEM_PREFERRED_RNG recommandé, sauf si l’appelant peut s’exécuter sur n’importe quel IRQL supérieur à 0 [c.-à-d. PASSIVE_LEVEL]).</li><li>**CAPI-** cryptGenRandom.</li><li>**Win32/64-** RtlGenRandom (les nouvelles implémentations doivent utiliser BCryptGenRandom ou CryptGenRandom) * rand_s * SystemPrng (pour le mode noyau).</li><li>**.NET-** RNGCryptoServiceProvider ou RNGCng.</li><li>**Applications du Windows Store-** Windows.Security.Cryptography.CryptographicBuffer.GenerateRandom ou .GenerateRandomNumber.</li><li>**Apple OS X (10.7+)/iOS(2.0+)-** int SecRandomCopyBytes (SecRandomRef random, size_t count, uint8_t \*bytes )</li><li>**Apple OS X (<10.7)-** Utilisez /dev/random pour récupérer des nombres aléatoires</li><li>**Java (y compris le code Java Google Android)-** java.security.SecureRandom class. Notez que pour Android 4.3 (Jelly Bean), les développeurs doivent suivre la solution de contournement Android recommandée et mettre à jour leurs applications pour initialiser explicitement le PRNG avec entropie à partir de /dev/urandom ou/dev/random.</li></ul>|

## <a name="do-not-use-symmetric-stream-ciphers"></a><a id="stream-ciphers"></a>Ne pas utiliser les chiffrements de flux symétriques

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Les chiffrements de flux symétriques, tels que RC4, ne doivent pas être utilisés. Au lieu des chiffrements de flux symétriques, les produits doivent utiliser un chiffrement par bloc, en particulier AES avec une longueur de clé d’au moins 128 bits. |

## <a name="use-approved-machmackeyed-hash-algorithms"></a><a id="mac-hash"></a>Utiliser les algorithmes de hachage MAC/HMAC/indexé approuvés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Les produits doivent utiliser uniquement le code d’authentification de message (MAC) approuvé ou des algorithmes de code d’authentification de message basé sur le hachage (HMAC).</p><p>Un code d’authentification de message (MAC) est un élément d’information associé à un message qui permet à son destinataire de vérifier à la fois l’authenticité de l’expéditeur et l’intégrité du message à l’aide d’une clé secrète. L’utilisation d’un MAC basé sur le hachage ([HMAC](https://csrc.nist.gov/publications/nistpubs/800-107-rev1/sp800-107-rev1.pdf)) ou d’un [MAC basé sur le chiffrement par bloc](https://csrc.nist.gov/publications/nistpubs/800-38B/SP_800-38B.pdf) est autorisée tant que tous les algorithmes de chiffrement symétrique ou de hachage sous-jacents sont également approuvés pour une utilisation ; actuellement, cela inclut les fonctions HMAC-SHA2 (HMAC-SHA256, HMAC-SHA384 et HMAC-SHA512) et les MAC basés sur le chiffrement par bloc CMAC/OMAC1 et OMAC2 (ces derniers sont basés sur AES).</p><p>L’utilisation de HMAC-SHA1 peut être autorisée pour la compatibilité de la plateforme, mais vous devrez demander une exception pour cette procédure et la soumettre à un examen du comité responsable du chiffrement de votre organisation. La troncation des HMAC à moins de 128 bits n’est pas autorisée. L’utilisation des méthodes du client pour hacher une clé et des données n’est pas approuvée et doit être soumise à un examen du comité responsable du chiffrement de votre organisation avant toute mise en œuvre.</p>|

## <a name="use-only-approved-cryptographic-hash-functions"></a><a id="hash-functions"></a>Utiliser uniquement les fonctions de hachage de chiffrement approuvées

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Les produits doivent utiliser la famille d’algorithmes de hachage SHA-2 (SHA256, SHA384 et SHA512). Si un hachage plus court est nécessaire, par exemple une longueur de sortie de 128 bits pour s’adapter à une structure de données conçue avec le hachage MD5 plus court à l’esprit, les équipes produit peuvent tronquer l’un des hachages SHA2 (généralement SHA256). Notez que SHA384 est une version tronquée de SHA512. La troncation des hachages de chiffrement pour des raisons de sécurité à moins de 128 bits n’est pas autorisée. Le nouveau code ne doit pas utiliser les algorithmes de hachage MD2, MD4, MD5, SHA-0, SHA-1 ou RIPEMD. Les conflits de hachage sont possibles en termes de calculs pour ces algorithmes, qui les brisent efficacement.</p><p>Algorithmes de hachage .NET autorisés pour l’agilité de chiffrement géré (par ordre de préférence) :</p><ul><li>SHA512Cng (compatible FIPS)</li><li>SHA384Cng (compatible FIPS)</li><li>SHA256Cng (compatible FIPS)</li><li>SHA512Managed (non compatible FIPS) (utilisez SHA512 en tant que nom d’algorithme dans les appels à HashAlgorithm.Create ou CryptoConfig.CreateFromName)</li><li>SHA384Managed (non compatible FIPS) (utilisez SHA384 en tant que nom d’algorithme dans les appels à HashAlgorithm.Create ou CryptoConfig.CreateFromName)</li><li>SHA256Managed (non compatible FIPS) (utilisez SHA256 en tant que nom d’algorithme dans les appels à HashAlgorithm.Create ou CryptoConfig.CreateFromName)</li><li>SHA512CryptoServiceProvider (compatible FIPS)</li><li>SHA256CryptoServiceProvider (compatible FIPS)</li><li>SHA384CryptoServiceProvider (compatible FIPS)</li></ul>| 

## <a name="use-strong-encryption-algorithms-to-encrypt-data-in-the-database"></a><a id="strong-db"></a>Utiliser des algorithmes de chiffrement fort pour chiffrer les données dans la base de données

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Choisir un algorithme de chiffrement](/sql/relational-databases/security/encryption/choose-an-encryption-algorithm) |
| **Étapes** | Les algorithmes de chiffrement définissent les transformations de données qui ne peuvent pas être facilement inversées par les utilisateurs non autorisés. SQL Server permet aux administrateurs et aux développeurs de choisir parmi plusieurs algorithmes, notamment DES, Triple DES, TRIPLE_DES_3KEY, RC2, RC4, RC4 128 bits, DESX, AES 128 bits, AES 192 bits et AES 256 bits. |

## <a name="ssis-packages-should-be-encrypted-and-digitally-signed"></a><a id="ssis-signed"></a>Les packages SSIS doivent être chiffrés et signés numériquement

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Identifier la source de packages à l’aide de signatures numériques](/sql/integration-services/security/identify-the-source-of-packages-with-digital-signatures), [Limitation des menaces et des risques de vulnérabilité (Integration Services)](/sql/integration-services/security/security-overview-integration-services) |
| **Étapes** | La source d'un package correspond à l'individu ou à l'organisation qui a créé le package. L'exécution d'un package à partir d'une source inconnue ou non approuvée peut être risquée. Pour empêcher la falsification non autorisée des packages SSIS, les signatures numériques doivent être utilisées. En outre, pour garantir la confidentialité des packages lors du stockage/transit, les packages SSIS doivent être chiffrés. |

## <a name="add-digital-signature-to-critical-database-securables"></a><a id="securables-db"></a>Ajouter une signature numérique aux éléments sécurisables de bases de données critiques

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [ADD SIGNATURE (Transact-SQL)](/sql/t-sql/statements/add-signature-transact-sql) |
| **Étapes** | Dans les cas où l’intégrité d’un élément sécurisable de base de données critique doit être vérifiée, les signatures numériques doivent être utilisées. Les éléments sécurisables de base de données tels qu’une procédure stockée, une fonction, un assembly ou un déclencheur sont signés numériquement. Vous trouverez ci-dessous un exemple pour lequel cela peut être utile : supposons qu’un éditeur de logiciels indépendant a fourni la prise en charge pour un logiciel fourni à l’un de ses clients. Avant de fournir la prise en charge, l’éditeur de logiciels indépendant souhaitera garantir qu’un élément sécurisable de base de données inclus dans le logiciel n’a pas été falsifié par inadvertance ou par une tentative malveillante. Si l’élément sécurisable est signé numériquement, l’éditeur de logiciels indépendant peut vérifier sa signature numérique et valider son intégrité.| 

## <a name="use-sql-server-ekm-to-protect-encryption-keys"></a><a id="ekm-keys"></a>Utiliser EKM SQL Server pour protéger les clés de chiffrement

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Gestion de clés extensible (EKM)](/sql/relational-databases/security/encryption/extensible-key-management-ekm), [Gestion de clés extensible à l’aide d’Azure Key Vault (SQL Server)](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server) |
| **Étapes** | La gestion de clés extensible SQL Server active les clés de chiffrement qui protègent les fichiers de base de données à stocker dans un appareil distant comme une carte à puce, un périphérique USB ou un module EKM/HSM. Elle permet aussi la protection des données pour les administrateurs de base de données (sauf les membres du groupe sysadmin). Les données peuvent être chiffrées à l'aide des clés de chiffrement auxquelles seul l'utilisateur de base de données peut accéder sur le module EKM/HSM externe. |

## <a name="use-alwaysencrypted-feature-if-encryption-keys-should-not-be-revealed-to-database-engine"></a><a id="keys-engine"></a>Utiliser la fonction AlwaysEncrypted si les clés de chiffrement ne doivent pas être révélées au moteur de base de données

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | SQL Azure, OnPrem |
| **Attributs**              | Version SQL - V12, MsSQL2016 |
| **Informations de référence**              | [Always Encrypted (moteur de base de données)](/sql/relational-databases/security/encryption/always-encrypted-database-engine) |
| **Étapes** | Always Encrypted est une fonction conçue pour protéger les données sensibles, comme les numéros de carte bancaire ou les numéros d’identification (par exemple les numéros de sécurité sociale), stockées dans Azure SQL Database ou les bases de données SQL Server. Always Encrypted permet aux clients de chiffrer les données sensibles des applications clientes et de ne jamais divulguer les clés de chiffrement au moteur de base de données (SQL Database ou SQL Server). Always Encrypted sépare ainsi les utilisateurs qui sont propriétaires des données (et peuvent les afficher) des utilisateurs qui gèrent les données (mais n’y ont pas accès). |

## <a name="store-cryptographic-keys-securely-on-iot-device"></a><a id="keys-iot"></a>Stocker les clés de chiffrement de façon sécurisée sur IoT Device

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Appareil IoT | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | Système d’exploitation d’appareil - Windows IoT Standard, connectivité des appareils - Azure IoT device SDK |
| **Informations de référence**              | [TPM on Windows IoT Core](/windows/iot-core/secure-your-device/TPM) (Module de plateforme sécurisée sur Windows IoT Standard), [TPM on Windows IoT Core](/windows/iot-core/secure-your-device/setuptpm) (Module de plateforme sécurisée sur Windows IoT Standard), [Azure IoT Device SDK TPM](https://github.com/Azure/azure-iot-hub-vs-cs/wiki/Device-Provisioning-with-TPM) (Module de plateforme sécurisée Azure IoT Device SDK) |
| **Étapes** | Clés privées de certificat ou symétriques dans un stockage matériel protégé tel qu’un module de plateforme sécurisée ou des cartes à puce. Windows 10 IoT Standard prend en charge l’utilisateur d’un module de plateforme sécurisée, dont plusieurs types compatibles peuvent être utilisés : https://docs.microsoft.com/windows/iot-core/secure-your-device/tpm#discrete-tpm-dtpm. Il est recommandé d’utiliser un module de plateforme sécurisée de type discret ou micrologiciel. Un module de plateforme sécurisée de type logiciel doit uniquement être utilisé à des fins de développement et de test. Une fois qu’un module de plateforme sécurisée est disponible et que les clés sont approvisionnées dans celui-ci, le code qui génère le jeton doit être écrit sans coder en dur les informations sensibles qu’il contient. | 

### <a name="example"></a>Exemple
```
TpmDevice myDevice = new TpmDevice(0);
// Use logical device 0 on the TPM 
string hubUri = myDevice.GetHostName(); 
string deviceId = myDevice.GetDeviceId(); 
string sasToken = myDevice.GetSASToken(); 

var deviceClient = DeviceClient.Create( hubUri, AuthenticationMethodFactory. CreateAuthenticationWithToken(deviceId, sasToken), TransportType.Amqp); 
```
Comme vous pouvez le constater, la clé primaire de l’appareil n’est pas présente dans le code. Au lieu de cela, elle est stockée dans le module de plateforme sécurisée à l’emplacement 0. L’appareil de module de plateforme sécurisée génère un jeton SAP temporaire qui est ensuite utilisé pour se connecter à IoT Hub. 

## <a name="generate-a-random-symmetric-key-of-sufficient-length-for-authentication-to-iot-hub"></a><a id="random-hub"></a>Générer une clé symétrique aléatoire de longueur suffisante pour l’authentification vers IoT Hub

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle cloud IoT | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | Choix de passerelle - Azure IoT Hub |
| **Informations de référence**              | N/A  |
| **Étapes** | IoT Hub contient un registre des identités de l’appareil et, lors de l’approvisionnement d’un appareil, il génère automatiquement une clé symétrique aléatoire. Il est recommandé d’utiliser cette fonctionnalité du registre des identités Azure IoT Hub pour générer la clé utilisée pour l’authentification. IoT Hub permet également de spécifier une clé lors de la création de l’appareil. Si une clé est générée en dehors d’IoT Hub pendant l’approvisionnement de l’appareil, il est recommandé de créer une clé symétrique aléatoire ou d’au moins 256 bits. |

## <a name="ensure-a-device-management-policy-is-in-place-that-requires-a-use-pin-and-allows-remote-wiping"></a><a id="pin-remote"></a>Garantir qu’une stratégie de gestion des appareils, demandant un code confidentiel utilisateur et permettant la réinitialisation à distance, est en place

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Client mobile Dynamics CRM | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Garantissez qu’une stratégie de gestion des appareils, demandant un code confidentiel utilisateur et permettant la réinitialisation à distance, est en place. |

## <a name="ensure-a-device-management-policy-is-in-place-that-requires-a-pinpasswordauto-lock-and-encrypts-all-data-eg-bitlocker"></a><a id="bitlocker"></a>Garantir qu’une stratégie de gestion des appareils, demandant un code confidentiel/mot de passe/verrouillage auto et chiffrant toutes les données (p. ex. BitLocker), est en place

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Client Outlook Dynamics CRM | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Garantir qu’une stratégie de gestion des appareils, demandant un code confidentiel/mot de passe/verrouillage auto et chiffrant toutes les données (p. ex. BitLocker), est en place |

## <a name="ensure-that-signing-keys-are-rolled-over-when-using-identity-server"></a><a id="rolled-server"></a>Garantir que les clés de signature sont annulées lors de l’utilisation d’IdentityServer

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | IdentityServer | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [IdentityServer - Clés, signatures et chiffrement](https://identityserver.github.io/Documentation/docsv2/configuration/crypto.html) |
| **Étapes** | Garantissez que les clés de signature sont annulées lors de l’utilisation d’IdentityServer. Le lien dans la section Références explique comment cela doit être planifié sans entraîner de pannes pour les applications reposant sur IdentityServer. |

## <a name="ensure-that-cryptographically-strong-client-id-client-secret-are-used-in-identity-server"></a><a id="client-server"></a>Garantir qu’un ID de client et une clé secrète client forts en termes de chiffrement sont utilisés dans IdentityServer

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | IdentityServer | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Garantir qu’un ID de client et une clé secrète client forts en termes de chiffrement sont utilisés dans IdentityServer. Les instructions ci-après doivent être suivies lors de la génération d’un ID de client et d’une clé secrète :</p><ul><li>Générez un GUID aléatoire comme ID de client.</li><li>Générez une clé de 256 bits aléatoire comme clé secrète.</li></ul>|