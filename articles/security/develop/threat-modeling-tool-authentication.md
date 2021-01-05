---
title: Authentification - Outil Microsoft de modélisation des menaces - Azure | Microsoft Docs
description: En savoir plus sur l’atténuation de l’authentification dans Threat Modeling Tool. Voir les informations sur l’atténuation et les exemples de code.
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
ms.custom: has-adal-ref, devx-track-js, devx-track-csharp
ms.openlocfilehash: ee4dd70faab9ed44b1aa6ca8ca0ec517c7746f66
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94832528"
---
# <a name="security-frame-authentication--mitigations"></a>Infrastructure de sécurité : Authentification | Mesures de correction

| Produit/Service | Article |
| --------------- | ------- |
| **Application Web**    | <ul><li>[Envisager d’utiliser un mécanisme d’authentification standard pour l’application web](#standard-authn-web-app)</li><li>[Les applications doivent gérer les scénarios d’authentification ayant échoué en toute sécurité ](#handle-failed-authn)</li><li>[Activer l’authentification adaptative ou avancée](#step-up-adaptive-authn)</li><li>[Garantir que les interfaces d’administration sont correctement verrouillées](#admin-interface-lockdown)</li><li>[Implémenter des fonctionnalités de mot de passe oublié en toute sécurité](#forgot-pword-fxn)</li><li>[Garantir que la stratégie de compte et de mot de passe est implémentée](#pword-account-policy)</li><li>[Implémenter des contrôles pour empêcher l’énumération de nom d’utilisateur](#controls-username-enum)</li></ul> |
| **Sauvegarde de la base de données** | <ul><li>[Si possible, utiliser l’authentification Windows pour la connexion à SQL Server](#win-authn-sql)</li><li>[Si possible, utiliser l’authentification Azure Active Directory pour la connexion à SQL Database](#aad-authn-sql)</li><li>[Lorsque le mode d’authentification SQL est utilisé, garantir que la stratégie de compte et de mot de passe est appliquée sur SQL Server](#authn-account-pword)</li><li>[Ne pas utiliser l’authentification SQL dans des bases de données autonomes](#autn-contained-db)</li></ul> |
| **Azure Event Hub** | <ul><li>[Utiliser les informations d’authentification par appareil à l’aide des jetons SAP](#authn-sas-tokens)</li></ul> |
| **Délimitation d’approbation Azure** | <ul><li>[Activer Azure AD Multi-Factor Authentication pour les administrateurs Azure](#multi-factor-azure-admin)</li></ul> |
| **Délimitation d’approbation Service Fabric** | <ul><li>[Restreindre l’accès anonyme au cluster Service Fabric](#anon-access-cluster)</li><li>[Garantir que le certificat client à nœud Service Fabric est différent du certificat nœud à nœud](#fabric-cn-nn)</li><li>[Utiliser AAD pour authentifier les clients sur les clusters Service Fabric](#aad-client-fabric)</li><li>[Garantir que les certificats Service Fabric proviennent d’une autorité de certification approuvée](#fabric-cert-ca)</li></ul> |
| **Serveur d’identité** | <ul><li>[Utiliser des scénarios d’authentification standard pris en charge par IdentityServer](#standard-authn-id)</li><li>[Remplacer le cache de jetons IdentityServer par défaut par une solution évolutive](#override-token)</li></ul> |
| **Délimitation d’approbation machine** | <ul><li>[Garantir que les fichiers binaires de l’application déployée sont signés numériquement](#binaries-signed)</li></ul> |
| **WCF** | <ul><li>[Activer l’authentification lors de la connexion aux files d’attente MSMQ dans WCF](#msmq-queues)</li><li>[WCF - Ne pas définir clientCredentialType du message sur la valeur « aucun »](#message-none)</li><li>[WCF - Ne pas définir clientCredentialType du transport sur la valeur « aucun »](#transport-none)</li></ul> |
| **API Web** | <ul><li>[Garantir que les techniques d’authentification standard sont utilisées pour sécuriser les API Web](#authn-secure-api)</li></ul> |
| **Azure AD** | <ul><li>[Utiliser des scénarios d’authentification standard pris en charge par Azure Active Directory](#authn-aad)</li><li>[Remplacer le cache de jetons ADAL par défaut par une solution évolutive](#adal-scalable)</li><li>[Garantir que TokenReplayCache est utilisé pour empêcher la relecture de jetons d’authentification ADAL](#tokenreplaycache-adal)</li><li>[Utiliser les bibliothèques ADAL pour gérer les demandes de jeton des clients OAuth2 vers AAD (ou Active Directory local)](#adal-oauth2)</li></ul> |
| **Passerelle de champ IoT** | <ul><li>[Authentifier les appareils se connectant à la passerelle de champ](#authn-devices-field)</li></ul> |
| **Passerelle de cloud IoT** | <ul><li>[Garantir que les appareils se connectant à la passerelle de cloud sont authentifiés](#authn-devices-cloud)</li><li>[Utiliser les informations d’authentification par appareil](#authn-cred)</li></ul> |
| **Stockage Azure** | <ul><li>[Garantir que seuls les conteneurs et blobs requis disposent d’un accès en lecture anonyme](#req-containers-anon)</li><li>[Accorder un accès limité aux objets dans le stockage Azure à l’aide de SAS ou SAP](#limited-access-sas)</li></ul> |

## <a name="consider-using-a-standard-authentication-mechanism-to-authenticate-to-web-application"></a><a id="standard-authn-web-app"></a>Envisager d’utiliser un mécanisme d’authentification standard pour l’application web

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| Détails | <p>L’authentification désigne le processus dans lequel une entité prouve son identité, généralement par le biais d’informations d’identification, comme le nom d’utilisateur et le mot de passe. Il existe plusieurs protocoles d’authentification disponibles qui peuvent vous être utiles. Certains d’entre eux sont répertoriés ci-dessous :</p><ul><li>Certificats clients</li><li>Basé sur Windows</li><li>Basé sur des formulaires</li><li>Fédération - ADFS</li><li>Fédération - Azure AD</li><li>Fédération - IdentityServer</li></ul><p>Envisagez d’utiliser un mécanisme d’authentification standard pour identifier le processus source.</p>|

## <a name="applications-must-handle-failed-authentication-scenarios-securely"></a><a id="handle-failed-authn"></a>Les applications doivent gérer les scénarios d’authentification ayant échoué en toute sécurité

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| Détails | <p>Les applications qui authentifient explicitement les utilisateurs doivent gérer des scénarios d’authentification ayant échoué en toute sécurité. Le mécanisme d’authentification doit :</p><ul><li>Refuser l’accès aux ressources avec des privilèges lorsque l’authentification échoue.</li><li>Afficher un message d’erreur générique après l’échec d’authentification et le refus de l’accès.</li></ul><p>Test des éléments suivants :</p><ul><li>Protection des ressources avec des privilèges après les échecs de connexion.</li><li>Un message d’erreur générique est affiché après les événements d’échec d’authentification et de refus de l’accès.</li><li>Les comptes sont désactivés après un nombre excessif d’échecs.</li><ul>|

## <a name="enable-step-up-or-adaptive-authentication"></a><a id="step-up-adaptive-authn"></a>Activer l’authentification adaptative ou avancée

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| Détails | <p>Vérifiez que l’application dispose d’une autorisation supplémentaire (telle qu’une authentification adaptative ou avancée via l’authentification multifacteur comme l’envoi d’OTP par SMS, e-mail, etc. ou une demande de nouvelle authentification) pour que l’utilisateur soit interrogé avant d’obtenir l’accès aux informations sensibles. Cette règle s’applique également pour les modifications critiques d’une action ou d’un compte.</p><p>Cela signifie également que l’adaptation de l’authentification doit être implémentée de manière que l’application applique correctement l’autorisation sensible au contexte pour interdire les manipulations non autorisées via la falsification de paramètres par exemple.</p>|

## <a name="ensure-that-administrative-interfaces-are-appropriately-locked-down"></a><a id="admin-interface-lockdown"></a>Garantir que les interfaces d’administration sont correctement verrouillées

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| Détails | La première solution consiste à accorder l’accès uniquement à partir d’une certaine plage d’adresses IP source à l’interface d’administration. Si cette solution n’est pas envisageable, il est toujours recommandé d’appliquer une authentification adaptative ou avancée pour la connexion à l’interface d’administration. |

## <a name="implement-forgot-password-functionalities-securely"></a><a id="forgot-pword-fxn"></a>Implémenter des fonctionnalités de mot de passe oublié en toute sécurité

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| Détails | <p>La première chose consiste à vérifier que les chemins de mot de passe oublié ou autres chemins de récupération envoient un lien incluant un jeton d’activation à durée limitée plutôt que le mot de passe lui-même. Une authentification supplémentaire basée sur les jetons logiciels (par exemple, jeton SMS, applications mobiles natives, etc.) peut être utilisée avant l’envoi du lien. Ensuite, vous ne devez pas verrouiller les comptes des utilisateurs tant que le processus d’obtention d’un nouveau mot de passe est en cours.</p><p>Cela peut entraîner une attaque par déni de service lorsqu’un pirate décide de verrouiller intentionnellement les utilisateurs avec une attaque automatisée. Puis, lorsque la demande de nouveau mot de passe est en cours, le message que vous affichez doit être généralisé afin d’éviter l’énumération de nom d’utilisateur. Enfin, interdisez toujours l’utilisation des anciens mots de passe et implémentez une stratégie de mot de passe forte.</p> |

## <a name="ensure-that-password-and-account-policy-are-implemented"></a><a id="pword-account-policy"></a>Garantir que la stratégie de compte et de mot de passe est implémentée

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| Détails | <p>Une stratégie de compte et de mot de passe conforme à la stratégie et aux bonnes pratiques de l’organisation doit être implémentée.</p><p>Pour se défendre contre les attaques en force brute et les vols basés sur le dictionnaire : une stratégie de mot de passe forte doit être implémentée pour garantir que les utilisateurs créent des mots de passe complexes (par exemple, longueur minimale de 12 caractères, caractères spéciaux et alphanumériques).</p><p>Les stratégies de verrouillage de compte peuvent être implémentées de la manière suivante :</p><ul><li>**Verrouillage léger :** il peut s’agir d’une bonne option pour protéger vos utilisateurs des attaques en force brute. Par exemple, lorsque l’utilisateur entre un mot de passe incorrect trois fois de suite, l’application peut verrouiller le compte pendant une minute pour ralentir le processus d’attaque en force brute de son mot de passe, réduisant le champ d’action de l’attaquant. Si vous souhaitez implémenter des contre-mesures de verrouillage fort pour cet exemple, vous obtenez une « attaque DoS » en verrouillant les comptes de manière permanente. Une application peut également générer un OTP (mot de passe à usage unique) et l’envoyer hors bande (par e-mail, sms, etc.) à l’utilisateur. Une autre approche peut consister à implémenter CAPTCHA après qu’un nombre maximum de tentatives est atteint.</li><li>**Verrouillage fort :** ce type de verrouillage doit être appliqué lorsque vous détectez un utilisateur qui attaque votre application. Il est contré au moyen d’un verrouillage permanent de son compte jusqu’à ce qu’une équipe de réponse ait le temps de traiter le problème. Après ce processus, vous pouvez décider de rendre son compte à l’utilisateur ou d’entamer des poursuites contre lui. Ce type d’approche empêche le pirate de pénétrer davantage dans votre application et infrastructure.</li></ul><p>Pour se défendre contre les attaques sur les comptes par défaut et prévisibles, vérifiez que toutes les clés et que tous les mots de passe sont remplaçables et qu’ils sont générés ou remplacés après l’installation.</p><p>Si l’application doit générer automatiquement des mots de passe, assurez-vous que les mots de passe générés sont aléatoires et ont une entropie élevée.</p>|

## <a name="implement-controls-to-prevent-username-enumeration"></a><a id="controls-username-enum"></a>Implémenter des contrôles pour empêcher l’énumération de nom d’utilisateur

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Tous les messages d’erreur doivent être généralisés afin d’éviter l’énumération de nom d’utilisateur. En outre, parfois vous ne pouvez pas éviter une fuite d’informations dans des fonctionnalités comme une page d’enregistrement. Ici, vous devez utiliser des méthodes de limite de débit comme CAPTCHA pour empêcher une attaque automatisée par un pirate. |

## <a name="when-possible-use-windows-authentication-for-connecting-to-sql-server"></a><a id="win-authn-sql"></a>Si possible, utiliser l’authentification Windows pour la connexion à SQL Server

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données |
| **Phase SDL**               | Build |
| **Technologies applicables** | Local |
| **Attributs**              | Version SQL - Toutes |
| **Informations de référence**              | [SQL Server - Choisir un mode d’authentification](/sql/relational-databases/security/choose-an-authentication-mode) |
| **Étapes** | L’authentification Windows utilise le protocole de sécurité Kerberos, met en œuvre des stratégies de mot de passe portant sur la validation de la complexité des mots de passe forts et prend en charge le verrouillage des comptes et l’expiration des mots de passe.|

## <a name="when-possible-use-azure-active-directory-authentication-for-connecting-to-sql-database"></a><a id="aad-authn-sql"></a>Si possible, utiliser l’authentification Azure Active Directory pour la connexion à SQL Database

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données |
| **Phase SDL**               | Build |
| **Technologies applicables** | SQL Azure |
| **Attributs**              | Version SQL - V12 |
| **Informations de référence**              | [Connexion à SQL Database avec l’authentification Azure Active Directory](../../azure-sql/database/authentication-aad-overview.md) |
| **Étapes** | **Version minimum :** Azure SQL Database V12 est nécessaire pour autoriser Azure SQL Database à utiliser l’authentification AAD par rapport à Microsoft Directory |

## <a name="when-sql-authentication-mode-is-used-ensure-that-account-and-password-policy-are-enforced-on-sql-server"></a><a id="authn-account-pword"></a>Lorsque le mode d’authentification SQL est utilisé, garantir que la stratégie de compte et de mot de passe est appliquée sur SQL Server

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Stratégie de mot de passe SQL Server](/previous-versions/sql/sql-server-2012/ms161959(v=sql.110)) |
| **Étapes** | Lorsque vous utilisez l’authentification SQL Server, les connexions sont créées dans SQL Server et ne sont pas basées sur les comptes d’utilisateur Windows. Le nom d’utilisateur et le mot de passe sont créés à l’aide de SQL Server et stockés dans SQL Server. SQL Server peut utiliser des mécanismes de stratégie de mot de passe Windows. Il peut appliquer la même complexité et les mêmes stratégies d’expiration utilisées dans Windows pour les mots de passe utilisés dans SQL Server. |

## <a name="do-not-use-sql-authentication-in-contained-databases"></a><a id="autn-contained-db"></a>Ne pas utiliser l’authentification SQL dans des bases de données à relation contenant-contenu

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données |
| **Phase SDL**               | Build |
| **Technologies applicables** | Local, SQL Azure |
| **Attributs**              | Version SQL - MSSQL2012, Version SQL - V12 |
| **Informations de référence**              | [Meilleures pratiques de sécurité recommandées avec les bases de données autonomes](/sql/relational-databases/databases/security-best-practices-with-contained-databases) |
| **Étapes** | L’absence d’une stratégie de mot de passe appliquée peut augmenter la probabilité d’une information d’identification faible établie dans une base de données autonome. Utilisez l’authentification Windows. |

## <a name="use-per-device-authentication-credentials-using-sas-tokens"></a><a id="authn-sas-tokens"></a>Utiliser les informations d’authentification par appareil à l’aide des jetons SAP

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure Event Hub |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Présentation du modèle de sécurité et de l’authentification Event Hubs](../../event-hubs/authenticate-shared-access-signature.md) |
| **Étapes** | <p>Le modèle de sécurité Event Hubs est basé sur une combinaison de jetons de signature d’accès partagé (SAP) et d’éditeurs d’événements. Le nom de l’éditeur représente l’ID de l’appareil qui reçoit le jeton. Cela permet d’associer les jetons générés aux appareils respectifs.</p><p>Tous les messages sont marqués avec le donneur d’ordre côté service autorisant la détection de tentatives d’usurpation d’origine en charge utile. Lors de l’authentification des appareils, générez un jeton SAP par appareil limité à un éditeur unique.</p>|

## <a name="enable-azure-ad-multi-factor-authentication-for-azure-administrators"></a><a id="multi-factor-azure-admin"></a>Activer Azure AD Multi-Factor Authentication pour les administrateurs Azure

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Délimitation d’approbation Azure |
| **Phase SDL**               | Déploiement |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Présentation d’Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md) |
| **Étapes** | <p>L’authentification multi-facteur est une méthode d’authentification qui nécessite plusieurs méthodes de vérification et ajoute une deuxième couche critique de sécurité aux connexions et transactions des utilisateurs. Cette authentification fonctionne en nécessitant au minimum deux des méthodes de vérification suivantes :</p><ul><li>Un élément que vous connaissez (généralement un mot de passe)</li><li>Un élément que vous possédez (un appareil de confiance qui n'est pas facilement dupliqué, comme un téléphone)</li><li>Un élément vous concernant particulièrement (biométrie)</li><ul>|

## <a name="restrict-anonymous-access-to-service-fabric-cluster"></a><a id="anon-access-cluster"></a>Restreindre l’accès anonyme au cluster Service Fabric

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Délimitation d’approbation Service Fabric |
| **Phase SDL**               | Déploiement |
| **Technologies applicables** | Générique |
| **Attributs**              | Environnement - Azure  |
| **Informations de référence**              | [Scénarios de sécurité d’un cluster Service Fabric](../../service-fabric/service-fabric-cluster-security.md) |
| **Étapes** | <p>Les clusters doivent toujours être sécurisés pour empêcher les utilisateurs non autorisés de se connecter à votre cluster, surtout quand des charges de travail sont en cours d’exécution sur ce dernier.</p><p>Lors de la création d’un cluster Service Fabric, assurez-vous que le mode de sécurité est défini sur la valeur « sécurisé » et configurez le certificat de serveur X.509 requis. La création d’un cluster « non sécurisé » permettra à tout utilisateur anonyme de s’y connecter si les points de terminaison de gestion sont exposés sur l’Internet public.</p>|

## <a name="ensure-that-service-fabric-client-to-node-certificate-is-different-from-node-to-node-certificate"></a><a id="fabric-cn-nn"></a>Garantir que le certificat client à nœud Service Fabric est différent du certificat nœud à nœud

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Délimitation d’approbation Service Fabric |
| **Phase SDL**               | Déploiement |
| **Technologies applicables** | Générique |
| **Attributs**              | Environnement - Azure, environnement - Autonome |
| **Informations de référence**              | [Sécurité par certificat de client à nœud](../../service-fabric/service-fabric-cluster-security.md#client-to-node-certificate-security), [Se connecter à un cluster sécurisé à l’aide d’un certificat client](../../service-fabric/service-fabric-connect-to-secure-cluster.md) |
| **Étapes** | <p>La configuration de la sécurité par certificat de client à nœud s’effectue lors de la création du cluster (par le biais du Portail Azure, des modèles Resource Manager ou d’un modèle JSON autonome) en spécifiant un certificat client d’administration et/ou un certificat de client utilisateur.</p><p>Les certificats client d’administration et client utilisateur que vous spécifiez doivent être différents des certificats primaires et secondaires que vous spécifiez pour la sécurité de nœud à nœud.</p>|

## <a name="use-aad-to-authenticate-clients-to-service-fabric-clusters"></a><a id="aad-client-fabric"></a>Utiliser AAD pour authentifier les clients sur les clusters Service Fabric

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Délimitation d’approbation Service Fabric |
| **Phase SDL**               | Déploiement |
| **Technologies applicables** | Générique |
| **Attributs**              | Environnement - Azure |
| **Informations de référence**              | [Scénarios de sécurité d’un cluster - Recommandations en matière de sécurité](../../service-fabric/service-fabric-cluster-security.md#security-recommendations) |
| **Étapes** | Les clusters s’exécutant sur Azure peuvent également sécuriser l’accès aux points de terminaison de gestion à l’aide d’Azure Active Directory (AAD), en plus des certificats client. Pour les clusters Azure, nous vous recommandons d’utiliser la sécurité AAD pour authentifier les clients et les certificats pour la sécurité de nœud à nœud.|

## <a name="ensure-that-service-fabric-certificates-are-obtained-from-an-approved-certificate-authority-ca"></a><a id="fabric-cert-ca"></a>Garantir que les certificats Service Fabric proviennent d’une autorité de certification approuvée

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Délimitation d’approbation Service Fabric |
| **Phase SDL**               | Déploiement |
| **Technologies applicables** | Générique |
| **Attributs**              | Environnement - Azure |
| **Informations de référence**              | [Certificats X.509 et Service Fabric](../../service-fabric/service-fabric-cluster-security.md#x509-certificates-and-service-fabric) |
| **Étapes** | <p>Service Fabric utilise des certificats de serveur X.509 pour l’authentification des clients et des nœuds.</p><p>Voici quelques éléments importants à prendre en compte lors de l’utilisation de certificats dans Service Fabric :</p><ul><li>Les certificats utilisés dans les clusters qui exécutent des charges de travail de production doivent être créés en utilisant un service de certificats Windows Server correctement configuré ou obtenus auprès d’une autorité de certification approuvée. L’autorité de certification peut être une autorité de certification externe approuvée ou une infrastructure de clé publique interne correctement gérée.</li><li>En production, n’utilisez jamais de certificats temporaires ou de test créés avec des outils comme MakeCert.exe.</li><li>Vous pouvez utiliser un certificat auto-signé, mais seulement pour les clusters de test et non en production.</li></ul>|

## <a name="use-standard-authentication-scenarios-supported-by-identity-server"></a><a id="standard-authn-id"></a>Utiliser des scénarios d’authentification standard pris en charge par IdentityServer

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | IdentityServer |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [IdentityServer3 - The Big Picture](https://identityserver.github.io/Documentation/docsv2/overview/bigPicture.html) (IdentityServer3 - Vue d’ensemble) |
| **Étapes** | <p>Vous trouverez ci-dessous les interactions classiques prises en charge par IdentityServer :</p><ul><li>Les navigateurs communiquent avec les applications web.</li><li>Les applications web communiquent avec les API web (parfois pour elles-mêmes, parfois pour le compte d’un utilisateur).</li><li>Les applications basées sur un navigateur communiquent avec les API web.</li><li>Les applications natives communiquent avec les API web.</li><li>Les applications basées sur un serveur communiquent avec les API web.</li><li>Les API web communiquent avec les API web (parfois pour elles-mêmes, parfois pour le compte d’un utilisateur).</li></ul>|

## <a name="override-the-default-identity-server-token-cache-with-a-scalable-alternative"></a><a id="override-token"></a>Remplacer le cache de jetons IdentityServer par défaut par une solution évolutive

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | IdentityServer |
| **Phase SDL**               | Déploiement |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Identity Server Deployment - Caching](https://identityserver.github.io/Documentation/docsv2/advanced/deployment.html) (Déploiement d’IdentityServer - Mise en cache) |
| **Étapes** | <p>IdentityServer a un cache en mémoire intégré simple. Bien que cette fonction soit utile pour les applications natives à petite échelle, elle n’est pas adaptée pour les applications de serveur principal et de niveau intermédiaire pour les raisons suivantes :</p><ul><li>Ces applications sont accessibles par plusieurs utilisateurs simultanément. L’enregistrement de tous les jetons d’accès dans le même magasin crée des problèmes d’isolation et présente des défis lors du fonctionnement à l’échelle : beaucoup d’utilisateurs, chacun avec autant de jetons que de ressources auxquelles l’application accède pour leur compte, peut entraîner un nombre très élevé d’opérations de recherche très coûteuses.</li><li>Ces applications sont généralement déployées sur des topologies distribuées, où plusieurs nœuds doivent accéder au même cache.</li><li>Les jetons mis en cache doivent survivre aux désactivations et recyclages de processus.</li><li>Pour toutes les raisons susmentionnées, lors de l’implémentation des applications web, il est recommandé de remplacer le cache de jeton d’IdentityServer par défaut avec une solution évolutive telle que le Cache Azure pour Redis</li></ul>|

## <a name="ensure-that-deployed-applications-binaries-are-digitally-signed"></a><a id="binaries-signed"></a>Garantir que les fichiers binaires de l’application déployée sont signés numériquement

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Délimitation d’approbation machine |
| **Phase SDL**               | Déploiement |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Assurez-vous que les fichiers binaires de l’application déployée sont signés numériquement afin que l’intégrité des fichiers binaires puisse être vérifiée.|

## <a name="enable-authentication-when-connecting-to-msmq-queues-in-wcf"></a><a id="msmq-queues"></a>Activer l’authentification lors de la connexion aux files d’attente MSMQ dans WCF

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique, NET Framework 3 |
| **Attributs**              | N/A |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)) |
| **Étapes** | Le programme ne parvient pas à activer l’authentification lors de la connexion aux files d’attente MSMQ, un pirate peut envoyer anonymement des messages à la file d’attente pour traitement. Si l’authentification n’est pas utilisée pour se connecter à une file d’attente MSMQ pour remettre un message à un autre programme, un pirate pourrait envoyer un message malveillant anonyme.|

### <a name="example"></a>Exemple
L’élément `<netMsmqBinding/>` du fichier de configuration WCF ci-dessous indique à WCF de désactiver l’authentification lors de la connexion à une file d’attente MSMQ pour la remise du message.
```
<bindings>
    <netMsmqBinding>
        <binding>
            <security>
                <transport msmqAuthenticationMode=""None"" />
            </security>
        </binding>
    </netMsmqBinding>
</bindings>
```
Configurez MSMQ pour exiger l’authentification de certificat ou de domaine Windows à tout moment pour les messages entrants ou sortants.

### <a name="example"></a>Exemple
L’élément `<netMsmqBinding/>` du fichier de configuration WCF ci-dessous indique à WCF d’activer l’authentification de certificat lors de la connexion à une file d’attente MSMQ. Le client est authentifié à l’aide de certificats X.509. Le certificat client doit être présent dans le magasin de certificats du serveur.
```
<bindings>
    <netMsmqBinding>
        <binding>
            <security>
                <transport msmqAuthenticationMode=""Certificate"" />
            </security>
        </binding>
    </netMsmqBinding>
</bindings>
```

## <a name="wcf-do-not-set-message-clientcredentialtype-to-none"></a><a id="message-none"></a>WCF - Ne pas définir clientCredentialType du message sur la valeur « aucun »

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF |
| **Phase SDL**               | Build |
| **Technologies applicables** | .NET Framework 3 |
| **Attributs**              | Type d’informations d’identification client - Aucun |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify](https://community.microfocus.com/t5/UFT-Discussions/UFT-API-Test-with-WCF-wsHttpBinding/m-p/600927) |
| **Étapes** | L’absence d’authentification signifie que tout le monde peut accéder à ce service. Un service qui n’authentifie pas ses clients autorise l’accès à tous les utilisateurs. Configurez l’application pour une authentification en fonction des informations d’identification du client. Cela est possible en définissant le clientCredentialType du message sur Windows ou Certificat. |

### <a name="example"></a>Exemple
```
<message clientCredentialType=""Certificate""/>
```

## <a name="wcf-do-not-set-transport-clientcredentialtype-to-none"></a><a id="transport-none"></a>WCF - Ne pas définir clientCredentialType du transport sur la valeur « aucun »

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique, .NET Framework 3 |
| **Attributs**              | Type d’informations d’identification client - Aucun |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify](https://community.microfocus.com/t5/UFT-Discussions/UFT-API-Test-with-WCF-wsHttpBinding/m-p/600927) |
| **Étapes** | L’absence d’authentification signifie que tout le monde peut accéder à ce service. Un service qui n’authentifie pas ses clients autorise tous les utilisateurs à accéder à ses fonctionnalités. Configurez l’application pour une authentification en fonction des informations d’identification du client. Cela est possible en définissant le clientCredentialType du transport sur Windows ou Certificat. |

### <a name="example"></a>Exemple
```
<transport clientCredentialType=""Certificate""/>
```

## <a name="ensure-that-standard-authentication-techniques-are-used-to-secure-web-apis"></a><a id="authn-secure-api"></a>Garantir que les techniques d’authentification standard sont utilisées pour sécuriser les API Web

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Authentication and Authorization in ASP.NET Web API](https://www.asp.net/web-api/overview/security/authentication-and-authorization-in-aspnet-web-api) (Authentification et autorisation dans l’API web ASP.NET), [External Authentication Services with ASP.NET Web API (C#)](https://www.asp.net/web-api/overview/security/external-authentication-services) (Services d’authentification externe avec l’API web ASP.NET (C#)) |
| **Étapes** | <p>L’authentification désigne le processus dans lequel une entité prouve son identité, généralement par le biais d’informations d’identification, comme le nom d’utilisateur et le mot de passe. Il existe plusieurs protocoles d’authentification disponibles qui peuvent vous être utiles. Certains d’entre eux sont répertoriés ci-dessous :</p><ul><li>Certificats clients</li><li>Basé sur Windows</li><li>Basé sur des formulaires</li><li>Fédération - ADFS</li><li>Fédération - Azure AD</li><li>Fédération - IdentityServer</li></ul><p>Les liens dans la section Références fournissent des informations de bas niveau sur la façon dont les schémas d’authentification peuvent être implémentés pour sécuriser une API web.</p>|

## <a name="use-standard-authentication-scenarios-supported-by-azure-active-directory"></a><a id="authn-aad"></a>Utiliser des scénarios d’authentification standard pris en charge par Azure Active Directory

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure AD |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Scénarios d’authentification pour Azure AD](../../active-directory/develop/authentication-vs-authorization.md), [Exemples de code Azure Active Directory](../../active-directory/azuread-dev/sample-v1-code.md), [Guide du développeur Azure Active Directory](../../active-directory/develop/index.yml) |
| **Étapes** | <p>Azure Active Directory (Azure AD) simplifie l’authentification pour les développeurs en fournissant l’identité en tant que service, avec la prise en charge des protocoles standard tels que OAuth 2.0 et OpenID Connect. Vous trouverez ci-dessous les cinq principaux scénarios d’application pris en charge par Azure AD :</p><ul><li>Navigateur web vers application web : un utilisateur doit se connecter à une application web sécurisée par Azure AD</li><li>Application à page unique (SPA) : un utilisateur doit se connecter à une application à page unique sécurisée par Azure AD</li><li>Application native vers API web : une application native qui s’exécute sur un téléphone, une tablette ou un PC doit authentifier un utilisateur pour obtenir des ressources d’une API web sécurisée par Azure AD</li><li>Application web vers API web : une application web doit obtenir des ressources d’une API web sécurisée par Azure AD</li><li>Application démon ou serveur vers API web : une application démon ou une application serveur sans interface utilisateur web doit obtenir des ressources d’une API web sécurisée par Azure AD</li></ul><p>Consultez les liens dans la section Références pour en savoir plus sur l’implémentation de bas niveau.</p>|

## <a name="override-the-default-adal-token-cache-with-a-scalable-alternative"></a><a id="adal-scalable"></a>Remplacer le cache de jetons ADAL par défaut par une solution évolutive

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure AD |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Modern Authentication with Azure Active Directory for Web Applications](/archive/blogs/microsoft_press/new-book-modern-authentication-with-azure-active-directory-for-web-applications) (Authentification moderne avec Azure Active Directory pour les applications web), [Using Redis as ADAL token cache](https://blogs.msdn.microsoft.com/mrochon/2016/09/19/using-redis-as-adal-token-cache/) (Utilisation de Redis en tant que cache de jeton ADAL)  |
| **Étapes** | <p>Le cache par défaut utilisé par ADAL (bibliothèque d’authentification Active Directory) est un cache en mémoire qui s’appuie sur un magasin statique, disponible au niveau du processus. Bien que cela fonctionne pour les applications natives, cela n’est pas adapté pour les applications de serveur principal et de niveau intermédiaire pour les raisons suivantes :</p><ul><li>Ces applications sont accessibles par plusieurs utilisateurs simultanément. L’enregistrement de tous les jetons d’accès dans le même magasin crée des problèmes d’isolation et présente des défis lors du fonctionnement à l’échelle : beaucoup d’utilisateurs, chacun avec autant de jetons que de ressources auxquelles l’application accède pour leur compte, peut entraîner un nombre très élevé d’opérations de recherche très coûteuses.</li><li>Ces applications sont généralement déployées sur des topologies distribuées, où plusieurs nœuds doivent accéder au même cache.</li><li>Les jetons mis en cache doivent survivre aux désactivations et recyclages de processus.</li></ul><p>Pour toutes les raisons susmentionnées, lors de l’implémentation des applications web, il est recommandé de remplacer le cache de jeton ADAL par défaut avec une solution évolutive telle que le Cache Azure pour Redis.</p>|

## <a name="ensure-that-tokenreplaycache-is-used-to-prevent-the-replay-of-adal-authentication-tokens"></a><a id="tokenreplaycache-adal"></a>Garantir que TokenReplayCache est utilisé pour empêcher la relecture de jetons d’authentification ADAL

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure AD |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Modern Authentication with Azure Active Directory for Web Applications](/archive/blogs/microsoft_press/new-book-modern-authentication-with-azure-active-directory-for-web-applications) (Authentification moderne avec Azure Active Directory pour les applications web) |
| **Étapes** | <p>La propriété TokenReplayCache permet aux développeurs de définir un cache de relecture de jetons, un magasin qui peut être utilisé pour enregistrer les jetons afin de vérifier qu’aucun jeton ne peut être utilisé plusieurs fois.</p><p>Il s’agit d’une mesure contre une attaque courante, l’attaque judicieusement appelée relecture de jetons : un pirate interceptant le jeton envoyé à la connexion peut tenter de l’envoyer à nouveau à l’application (le « relire ») pour établir une nouvelle session. Par exemple, dans le flux d’octroi de code d’OIDC, après une authentification utilisateur réussie, une demande vers le point de terminaison « /signin-oidc » de la partie de confiance est réalisée avec les paramètres « id_token », « code » et « état ».</p><p>La partie de confiance valide cette demande et établit une nouvelle session. Si un adversaire capture cette demande et la relit, il peut réussir à établir une session et tromper l’utilisateur. La présence de la valeur à usage unique dans OpenID Connect peut limiter mais pas complètement éliminer les circonstances dans lesquelles l’attaque peut être mise en œuvre correctement. Pour protéger leurs applications, les développeurs peuvent fournir une implémentation de ITokenReplayCache et assigner une instance à TokenReplayCache.</p>|

### <a name="example"></a>Exemple
```csharp
// ITokenReplayCache defined in ADAL
public interface ITokenReplayCache
{
bool TryAdd(string securityToken, DateTime expiresOn);
bool TryFind(string securityToken);
}
```

### <a name="example"></a>Exemple
Voici un exemple d’implémentation de l’interface ITokenReplayCache. (Veuillez personnaliser et mettre en œuvre votre infrastructure de mise en cache spécifique au projet)
```csharp
public class TokenReplayCache : ITokenReplayCache
{
    private readonly ICacheProvider cache; // Your project-specific cache provider
    public TokenReplayCache(ICacheProvider cache)
    {
        this.cache = cache;
    }
    public bool TryAdd(string securityToken, DateTime expiresOn)
    {
        if (this.cache.Get<string>(securityToken) == null)
        {
            this.cache.Set(securityToken, securityToken);
            return true;
        }
        return false;
    }
    public bool TryFind(string securityToken)
    {
        return this.cache.Get<string>(securityToken) != null;
    }
}
```
Le cache implémenté doit être référencé dans les options OIDC via la propriété « TokenValidationParameters » comme suit.
```csharp
OpenIdConnectOptions openIdConnectOptions = new OpenIdConnectOptions
{
    AutomaticAuthenticate = true,
    ... // other configuration properties follow..
    TokenValidationParameters = new TokenValidationParameters
    {
        TokenReplayCache = new TokenReplayCache(/*Inject your cache provider*/);
    }
}
```

Notez que pour tester l’efficacité de cette configuration, vous devez vous connecter à votre application protégée par OIDC locale et capturer la demande vers le point de terminaison `"/signin-oidc"` dans fiddler. Lorsque la protection n’est pas en place, la relecture de cette demande dans fiddler définira un nouveau cookie de session. Lors de la relecture de la demande après avoir ajouté la protection TokenReplayCache, l’application lève une exception comme suit :`SecurityTokenReplayDetectedException: IDX10228: The securityToken has previously been validated, securityToken: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSIsImtpZCI6Ik1uQ1......`

## <a name="use-adal-libraries-to-manage-token-requests-from-oauth2-clients-to-aad-or-on-premises-ad"></a><a id="adal-oauth2"></a>Utiliser les bibliothèques ADAL pour gérer les demandes de jeton des clients OAuth2 vers AAD (ou Active Directory local)

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure AD |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [ADAL](../../active-directory/azuread-dev/active-directory-authentication-libraries.md) |
| **Étapes** | <p>La bibliothèque d'authentification Azure AD (ADAL) permet aux développeurs d’applications clientes d’authentifier facilement les utilisateurs sur Active Directory (AD) en local ou sur le cloud, puis d'obtenir des jetons d'accès pour sécuriser les appels d'API.</p><p>ADAL possède de nombreuses fonctionnalités qui facilitent l'authentification aux développeurs, notamment la prise en charge asynchrone, un cache de jeton configurable qui stocke les jetons d'accès et les jetons d'actualisation, l'actualisation automatique des jetons lorsqu'un jeton d'accès expire et qu’un jeton d'actualisation est disponible, et bien plus encore.</p><p>En gérant l’essentiel de la complexité, la bibliothèque ADAL peut aider les développeurs à se concentrer sur la logique métier dans leur application et à sécuriser facilement les ressources sans être un expert de la sécurité. Des bibliothèques distinctes sont disponibles pour .NET, JavaScript (client et Node.js), Python; iOS, Android et Java.</p>|

## <a name="authenticate-devices-connecting-to-the-field-gateway"></a><a id="authn-devices-field"></a>Authentifier les appareils se connectant à la passerelle de champ

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle de champ IoT |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Assurez-vous que chaque appareil est authentifié par la passerelle de champ avant d’accepter des données provenant de ceux-ci et de faciliter les communications en amont avec la passerelle de cloud. En outre, assurez-vous que les appareils se connectent avec des informations d’identification par appareil, afin que chaque appareil puisse être identifié de manière unique.|

## <a name="ensure-that-devices-connecting-to-cloud-gateway-are-authenticated"></a><a id="authn-devices-cloud"></a>Garantir que les appareils se connectant à la passerelle de cloud sont authentifiés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle cloud IoT |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique, C#, Node.JS  |
| **Attributs**              | N/A, choix de passerelle - Azure IoT Hub |
| **Informations de référence**              | N/A, [Mise en route d’Azure IoT Hub (.NET)](../../iot-hub/quickstart-send-telemetry-dotnet.md), [Mise en route d’Azure IoT Hub (Node)](../../iot-hub/quickstart-send-telemetry-node.md), [Contrôler l’accès à IoT Hub](../../iot-hub/iot-hub-devguide-security.md), [référentiel Git](https://github.com/Azure/azure-iot-sdks/) |
| **Étapes** | <ul><li>**Générique :** authentifiez l’appareil à l’aide du protocole TLS (Transport Layer Security) ou IPSec. L’infrastructure doit prendre en charge l’utilisation d’une clé prépartagée (PSK) sur les périphériques qui ne peuvent pas gérer le chiffrement asymétrique complet. Utilisez Azure AD, OAuth.</li><li>**C# :** par défaut, la méthode de création Create crée une instance DeviceClient qui utilise le protocole AMQP pour communiquer avec IoT Hub. Pour utiliser le protocole HTTPS, remplacez la méthode Create afin de pouvoir spécifier le protocole. Si vous utilisez le protocole HTTPS, vous devez également ajouter le package NuGet `Microsoft.AspNet.WebApi.Client` à votre projet de manière à inclure l’espace de noms `System.Net.Http.Formatting`.</li></ul>|

### <a name="example"></a>Exemple
```csharp
static DeviceClient deviceClient;

static string deviceKey = "{device key}";
static string iotHubUri = "{iot hub hostname}";

var messageString = "{message in string format}";
var message = new Message(Encoding.ASCII.GetBytes(messageString));

deviceClient = DeviceClient.Create(iotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey("myFirstDevice", deviceKey));

await deviceClient.SendEventAsync(message);
```

### <a name="example"></a>Exemple
**Node.JS : Authentification**
#### <a name="symmetric-key"></a>Clé symétrique
* Créer un hub IoT sur Azure
* Créez une entrée dans le registre des identités de l’appareil.
    ```javascript
    var device = new iothub.Device(null);
    device.deviceId = <DeviceId >
    registry.create(device, function(err, deviceInfo, res) {})
    ```
* Créez un appareil simulé.
    ```javascript
    var clientFromConnectionString = require('azure-iot-device-amqp').clientFromConnectionString;
    var Message = require('azure-iot-device').Message;
    var connectionString = 'HostName=<HostName>DeviceId=<DeviceId>SharedAccessKey=<SharedAccessKey>';
    var client = clientFromConnectionString(connectionString);
    ```
  #### <a name="sas-token"></a>Jeton SAP
* Généré en interne lors de l’utilisation de la clé symétrique, mais nous pouvons le générer et l’utiliser explicitement également.
* Définissez un protocole : `var Http = require('azure-iot-device-http').Http;`.
* Créez un jeton SAP :
    ```javascript
    resourceUri = encodeURIComponent(resourceUri.toLowerCase()).toLowerCase();
    var deviceName = "<deviceName >";
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    var toSign = resourceUri + '\n' + expires;
    // using crypto
    var decodedPassword = new Buffer(signingKey, 'base64').toString('binary');
    const hmac = crypto.createHmac('sha256', decodedPassword);
    hmac.update(toSign);
    var base64signature = hmac.digest('base64');
    var base64UriEncoded = encodeURIComponent(base64signature);
    // construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "%2fdevices%2f"+deviceName+"&sig="
  + base64UriEncoded + "&se=" + expires;
    if (policyName) token += "&skn="+policyName;
    return token;
    ```
* Connectez-vous à l’aide du jeton SAP :
    ```javascript
    Client.fromSharedAccessSignature(sas, Http);
    ```
  #### <a name="certificates"></a>Certificats
* Générez un certificat X509 auto-signé à l’aide d’un outil tel qu’OpenSSL pour créer des fichiers .cert et .key pour stocker respectivement le certificat et la clé.
* Approvisionnez un appareil qui accepte une connexion sécurisée à l’aide de certificats.
    ```javascript
    var connectionString = '<connectionString>';
    var registry = iothub.Registry.fromConnectionString(connectionString);
    var deviceJSON = {deviceId:"<deviceId>",
    authentication: {
        x509Thumbprint: {
        primaryThumbprint: "<PrimaryThumbprint>",
        secondaryThumbprint: "<SecondaryThumbprint>"
        }
    }}
    var device = deviceJSON;
    registry.create(device, function (err) {});
    ```
* Connectez un appareil à l’aide d’un certificat.
    ```javascript
    var Protocol = require('azure-iot-device-http').Http;
    var Client = require('azure-iot-device').Client;
    var connectionString = 'HostName=<HostName>DeviceId=<DeviceId>x509=true';
    var client = Client.fromConnectionString(connectionString, Protocol);
    var options = {
        key: fs.readFileSync('./key.pem', 'utf8'),
        cert: fs.readFileSync('./server.crt', 'utf8')
    };
    // Calling setOptions with the x509 certificate and key (and optionally, passphrase) will configure the client //transport to use x509 when connecting to IoT Hub
    client.setOptions(options);
    //call fn to execute after the connection is set up
    client.open(fn);
    ```

## <a name="use-per-device-authentication-credentials"></a><a id="authn-cred"></a>Utiliser les informations d’authentification par appareil

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle cloud IoT  |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | Choix de passerelle - Azure IoT Hub |
| **Informations de référence**              | [Contrôler l’accès à IoT Hub](../../iot-hub/iot-hub-devguide-security.md) |
| **Étapes** | Utilisez des informations d’authentification par appareil à l’aide des jetons SAP en fonction de la clé d’appareil ou du certificat de client plutôt que des stratégies d’accès partagé au niveau d’IoT Hub. Cela empêche la réutilisation des jetons d’authentification d’une passerelle de champ ou d’un appareil par un autre. |

## <a name="ensure-that-only-the-required-containers-and-blobs-are-given-anonymous-read-access"></a><a id="req-containers-anon"></a>Garantir que seuls les conteneurs et blobs requis disposent d’un accès en lecture anonyme

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Stockage Azure |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | StorageType - Blob |
| **Informations de référence**              | [Gestion de l’accès en lecture anonyme aux conteneurs et aux objets blob](../../storage/blobs/anonymous-read-access-configure.md), [Signatures d’accès partagé (SAP), partie 1 : Présentation du modèle SAP](../../storage/common/storage-sas-overview.md) |
| **Étapes** | <p>Par défaut, un conteneur et tous les objets blob qu'il contient sont accessibles uniquement par le propriétaire du compte de stockage. Pour fournir aux utilisateurs anonymes des autorisations de lecture sur un conteneur et ses blobs, il est possible de configurer les autorisations du conteneur afin d’autoriser l’accès public. Les utilisateurs anonymes peuvent lire les objets blob d’un conteneur accessible publiquement sans avoir à authentifier la demande.</p><p>Les conteneurs fournissent les options suivantes pour gérer leur accès :</p><ul><li>Accès en lecture public complet : les données de conteneur et d’objets blob sont lisibles au moyen d’une requête anonyme. Les clients peuvent énumérer les objets blob à l’intérieur du conteneur via une demande anonyme, mais ne peuvent pas énumérer les conteneurs dans le compte de stockage.</li><li>Accès en lecture public pour les objets blob uniquement : Les données d'objets blob à l'intérieur de ce conteneur peuvent être lues via une demande anonyme, mais les données du conteneur ne sont pas disponibles. Les clients ne peuvent pas énumérer les blobs à l’intérieur du conteneur via une demande anonyme.</li><li>Aucun accès en lecture public : seul le propriétaire du compte peut lire les données de conteneur et d’objet blob</li></ul><p>L’accès anonyme est idéal dans les situations où certains blobs doivent toujours être disponibles pour l’accès en lecture anonyme. Pour un contrôle plus fin, il est possible de créer une signature d’accès partagé de manière à déléguer un accès restreint en utilisant des autorisations différentes sur un intervalle de temps spécifié. Garantissez que les conteneurs et les blobs, qui peuvent éventuellement contenir des données sensibles, ne bénéficient pas d’un accès anonyme accidentellement.</p>|

## <a name="grant-limited-access-to-objects-in-azure-storage-using-sas-or-sap"></a><a id="limited-access-sas"></a>Accorder un accès limité aux objets dans le stockage Azure à l’aide de SAS ou SAP

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Stockage Azure |
| **Phase SDL**               | Build |
| **Technologies applicables** | Générique |
| **Attributs**              | N/A |
| **Informations de référence**              | [Signatures d’accès partagé, partie 1 : présentation du modèle SAP](../../storage/common/storage-sas-overview.md), [Signatures d’accès partagé, partie 2 : Créer et utiliser une signature d’accès partagé avec Stockage Blob](../../storage/common/storage-sas-overview.md), [Comment déléguer l’accès aux objets dans votre compte à l’aide de signatures d’accès partagé et stratégies d’accès stockées](../../storage/blobs/security-recommendations.md#identity-and-access-management) |
| **Étapes** | <p>Une signature d’accès partagé (SAP) constitue un moyen efficace pour octroyer aux autres clients un accès limité aux blobs dans un compte de stockage, sans exposer de clé d’accès de compte. La signature d'accès partagé est un URI qui englobe dans ses paramètres de requête toutes les informations nécessaires pour un accès authentifié à une ressource de stockage. Pour accéder aux ressources de stockage avec la signature d'accès partagé, il suffit au client de transmettre cette dernière à la méthode ou au constructeur approprié.</p><p>Vous pouvez utiliser une signature d'accès partagé lorsque vous voulez fournir un accès aux ressources dans votre compte de stockage à un client qui ne peut pas être approuvé avec la clé du compte. Vos clés de compte de stockage incluent une clé primaire et une clé secondaire, qui octroient toutes deux un accès administratif à votre compte et à toutes ses ressources. En exposant l'une ou l'autre des clés de votre compte, vous courez le risque d'une utilisation malveillante ou négligente de ce dernier. Les signatures d'accès partagé offrent une alternative sûre qui permet aux autres clients de lire, d'écrire et de supprimer des données dans votre compte de stockage en fonction des autorisations que vous avez octroyées, sans avoir besoin de la clé du compte.</p><p>Si vous avez un ensemble logique de paramètres qui sont chaque fois similaires, il est plus judicieux d’utiliser une stratégie d’accès stockée. Comme l’utilisation d’une signature d’accès partagé dérivée d’une stratégie d’accès stockée vous permet de révoquer immédiatement cette signature d’accès partagé, il est recommandé de toujours utiliser des stratégies d’accès stockées quand cela est possible.</p>|