---
title: Sécurité des communications pour Microsoft Threat Modeling Tool
titleSuffix: Azure
description: Découvrez l’atténuation des menaces de sécurité de communication exposées dans Microsoft Threat Modeling Tool. Découvrez les informations sur l’atténuation et les exemples de code.
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
ms.custom: devx-track-csharp
ms.openlocfilehash: d9a4eabf37101622ac69ae05f3bec232fb8d2fe6
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517527"
---
# <a name="security-frame-communication-security--mitigations"></a>Infrastructure de sécurité : sécurité des communications | mesures d’atténuation 
| Produit/Service | Article |
| --------------- | ------- |
| **Azure Event Hub** | <ul><li>[Sécuriser les communications vers Event Hub à l’aide du protocole SSL/TLS](#comm-ssltls)</li></ul> |
| **Dynamics CRM** | <ul><li>[Vérifier les privilèges de compte de service et vérifier que les services personnalisés ou les pages ASP.NET respectent la sécurité CRM](#priv-aspnet)</li></ul> |
| **Azure Data Factory**. | <ul><li>[Utiliser la passerelle de gestion des données lors de la connexion du SQL Server local à Azure Data Factory](#sqlserver-factory)</li></ul> |
| **Serveur d’identité** | <ul><li>[Garantir que l’intégralité du trafic vers IdentityServer est sur la connexion HTTPS](#identity-https)</li></ul> |
| **Application Web** | <ul><li>[Vérifier les certificats X.509 utilisés pour authentifier les connexions SSL, TLS et DTLS](#x509-ssltls)</li><li>[Configurer le certificat TLS/SSL pour le domaine personnalisé dans Azure App Service](#ssl-appservice)</li><li>[Forcer l’intégralité du trafic vers Azure App Service sur la connexion HTTPS](#appservice-https)</li><li>[Activer le protocole HTTP Strict Transport Security (HSTS)](#http-hsts)</li></ul> |
| **Sauvegarde de la base de données** | <ul><li>[Assurer le chiffrement de la connexion SQL Server et la validation des certificats](#sqlserver-validation)</li><li>[Forcer des communications chiffrées vers SQL Server](#encrypted-sqlserver)</li></ul> |
| **Stockage Azure** | <ul><li>[Vérifier que la communication vers le stockage Azure est sur HTTPS](#comm-storage)</li><li>[Valider le hachage MD5 après le téléchargement de blobs si le protocole HTTPS ne peut pas être activé](#md5-https)</li><li>[Utiliser un client compatible SMB 3.0 pour garantir le chiffrement des données en transit vers les partages de fichiers Azure](#smb-shares)</li></ul> |
| **Client mobile** | <ul><li>[Implémenter l’épinglage de certificat](#cert-pinning)</li></ul> |
| **WCF** | <ul><li>[Activer le protocole HTTPS - Sécuriser le canal de transport](#https-transport)</li><li>[WCF : définir le niveau de protection de la sécurité des messages sur EncryptAndSign](#message-protection)</li><li>[WCF : utiliser un compte avec des privilèges minimum pour exécuter votre service WCF](#least-account-wcf)</li></ul> |
| **API Web** | <ul><li>[Forcer l’intégralité du trafic vers les API web sur la connexion HTTPS](#webapi-https)</li></ul> |
| **Cache Azure pour Redis** | <ul><li>[Vérifier que la communication vers le Cache Redis Azure s’effectue via TLS](#redis-ssl)</li></ul> |
| **Passerelle de champ IoT** | <ul><li>[Sécuriser la communication entre l’appareil et la passerelle de champ](#device-field)</li></ul> |
| **Passerelle de cloud IoT** | <ul><li>[Sécuriser la communication entre l’appareil et la passerelle de cloud à l’aide du protocole SSL/TLS](#device-cloud)</li></ul> |

## <a name="secure-communication-to-event-hub-using-ssltls"></a><a id="comm-ssltls"></a>Sécuriser les communications vers Event Hub à l’aide du protocole SSL/TLS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure Event Hub | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Présentation du modèle de sécurité et de l’authentification Event Hubs](../../event-hubs/authenticate-shared-access-signature.md) |
| **Étapes** | Sécuriser les connexions AMQP ou HTTP vers Event Hub à l’aide du protocole SSL/TLS |

## <a name="check-service-account-privileges-and-check-that-the-custom-services-or-aspnet-pages-respect-crms-security"></a><a id="priv-aspnet"></a>Vérifier les privilèges de compte de service et vérifier que les services personnalisés ou les pages ASP.NET respectent la sécurité CRM

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Dynamics CRM | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Vérifiez les privilèges de compte de service et vérifiez que les services personnalisés ou les pages ASP.NET respectent la sécurité CRM. |

## <a name="use-data-management-gateway-while-connecting-on-premises-sql-server-to-azure-data-factory"></a><a id="sqlserver-factory"></a>Utiliser la passerelle de gestion des données lors de la connexion du SQL Server local à Azure Data Factory

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure Data Factory | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | Types de services liés - Azure et local |
| **Informations de référence**              |[Déplacement de données entre des sources locales et Azure Data Factory](../../data-factory/v1/data-factory-move-data-between-onprem-and-cloud.md#create-gateway), [Passerelle de gestion de données](../../data-factory/v1/data-factory-data-management-gateway.md) |
| **Étapes** | <p>L’outil de passerelle de gestion des données est requis pour se connecter aux sources de données qui sont protégées derrière un pare-feu ou un réseau d’entreprise.</p><ol><li>Le verrouillage de la machine isole l’outil de passerelle de gestion des données et empêche les programmes ne fonctionnant pas correctement d’endommager la machine hébergeant les sources de données ou d’espionner celle-ci. (Par exemple, les dernières mises à jour doivent être installées, activer le nombre minimal de ports requis, approvisionnement des comptes contrôlés, audit activé, chiffrement activé, etc.).</li><li>La clé de passerelle de données doit pivoter à intervalles réguliers ou chaque fois que le mot de passe du compte de service de passerelle de gestion des données est renouvelé.</li><li>Les transits de données via le service lié doivent être chiffrés.</li></ol> |

## <a name="ensure-that-all-traffic-to-identity-server-is-over-https-connection"></a><a id="identity-https"></a>Garantir que l’intégralité du trafic vers IdentityServer est sur la connexion HTTPS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | IdentityServer | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [IdentityServer3 - Keys, Signatures and Cryptography](https://identityserver.github.io/Documentation/docsv2/configuration/crypto.html) (IdentityServer3 - Clés, signatures et chiffrement), [IdentityServer3 - Deployment](https://identityserver.github.io/Documentation/docsv2/advanced/deployment.html) (IdentityServer3 - Déploiement) |
| **Étapes** | Par défaut, IdentityServer requiert que toutes les connexions entrantes proviennent du protocole HTTPS. Il est absolument indispensable que les communications avec IdentityServer s’effectuent sur les transports sécurisés uniquement. Il existe certains scénarios de déploiement (p. ex. déchargement TLS) dans lesquels cette exigence peut être assouplie. Consultez la page sur le déploiement d’IdentityServer dans la section Références pour plus d’informations. |

## <a name="verify-x509-certificates-used-to-authenticate-ssl-tls-and-dtls-connections"></a><a id="x509-ssltls"></a>Vérifier les certificats X.509 utilisés pour authentifier les connexions SSL, TLS et DTLS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Les applications qui utilisent les protocoles SSL, TLS ou DTLS doivent intégralement vérifier les certificats X.509 des entités auxquelles elles se connectent. Cela inclut la vérification des certificats pour les éléments suivants :</p><ul><li>Nom de domaine</li><li>Dates de validité (dates de début et dates d’échéance).</li><li>État de révocation.</li><li>Utilisation (par exemple, authentification serveur pour les serveurs, authentification client pour les clients).</li><li>Chaîne d’approbation. Les certificats doivent être rattachés à une autorité de certification racine approuvée par la plateforme ou explicitement configurés par l’administrateur.</li><li>La longueur de clé de la clé publique du certificat doit être supérieure à 2 048 bits.</li><li>L’algorithme de hachage doit être SHA256 et supérieur. |

## <a name="configure-tlsssl-certificate-for-custom-domain-in-azure-app-service"></a><a id="ssl-appservice"></a>Configurer le certificat TLS/SSL pour le domaine personnalisé dans Azure App Service

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | EnvironmentType - Azure |
| **Informations de référence**              | [Activer le protocole HTTPS pour une application dans Azure App Service](../../app-service/configure-ssl-bindings.md) |
| **Étapes** | Par défaut, Azure active déjà le protocole HTTPS pour toutes les applications grâce à un certificat générique pour le domaine *.azurewebsites.net. Comme tous les domaines génériques, il n’est cependant pas aussi sécurisé qu’un domaine personnalisé avec votre propre certificat. [Consultez cet article](https://casecurity.org/2014/02/26/pros-and-cons-of-single-domain-multi-domain-and-wildcard-certificates/). Il est recommandé d’activer le protocole TLS pour le domaine personnalisé qui sera accessible via l’application déployée|

## <a name="force-all-traffic-to-azure-app-service-over-https-connection"></a><a id="appservice-https"></a>Forcer l’intégralité du trafic vers Azure App Service sur la connexion HTTPS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | EnvironmentType - Azure |
| **Informations de référence**              | [Appliquer le protocole HTTPS sur Azure App Service](../../app-service/configure-ssl-bindings.md#enforce-https) |
| **Étapes** | <p>Bien qu’Azure active déjà le protocole HTTPS pour les services d’application Azure grâce à un certificat générique pour le domaine *.azurewebsites.net, le domaine n’applique pas le protocole HTTPS. Les visiteurs peuvent toujours accéder à l’application à l’aide du protocole HTTP, ce qui peut compromettre la sécurité de l’application. Par conséquent, le protocole HTTPS doit être appliqué de manière explicite. Les applications ASP.NET MVC doivent utiliser le [filtre RequireHttps](/dotnet/api/system.web.mvc.requirehttpsattribute) qui force une demande HTTP non sécurisée à être renvoyée sur HTTPS.</p><p>Sinon, vous pouvez utiliser le module de réécriture d’URL, qui est inclus avec Azure App Service, pour appliquer le protocole HTTPS. Le module de réécriture d’URL permet aux développeurs de définir des règles qui sont appliquées aux demandes entrantes avant qu’elles ne soient transmises à votre application. Les règles de réécriture d’URL sont définies dans un fichier web.config stocké à la racine de l’application.</p>|

### <a name="example"></a>Exemple
L’exemple suivant contient une règle de réécriture d’URL basique qui force tout le trafic entrant à utiliser le protocole HTTPS.
```XML
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Force HTTPS" enabled="true">
          <match url="(.*)" ignoreCase="false" />
          <conditions>
            <add input="{HTTPS}" pattern="off" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" appendQueryString="true" redirectType="Permanent" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```
Cette règle fonctionne en renvoyant le code d’état HTTP 301 (redirection permanente) lorsque l’utilisateur demande une page utilisant le protocole HTTP. Le code 301 redirige la demande vers la même URL que celle requise par le visiteur, mais remplace la partie HTTP de la demande par HTTPS. Par exemple, `HTTP://contoso.com` est redirigé vers `HTTPS://contoso.com`. 

## <a name="enable-http-strict-transport-security-hsts"></a><a id="http-hsts"></a>Activer le protocole HTTP Strict Transport Security (HSTS)

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [OWASP HTTP Strict Transport Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html) (Aide-mémoire sur HTTP Strict Transport Security par l’OWASP) |
| **Étapes** | <p>HTTP Strict Transport Security (HSTS) est une amélioration de sécurité à accepter qui est spécifiée par une application web via l’utilisation d’un en-tête de réponse spécial. Une fois qu’un navigateur pris en charge reçoit cet en-tête, ce navigateur empêchera toutes les communications d’être envoyés sur HTTP vers le domaine spécifié et enverra à la place toutes les communications sur HTTPS. Il empêche également les invites HTTPS sur lesquelles cliquer sur les navigateurs.</p><p>Pour implémenter HSTS, l’en-tête de réponse suivant doit être configuré pour un site web de manière globale, dans le code ou dans la configuration. Strict-Transport-Security : max-age = 300 ; includeSubDomains HSTS répond aux menaces suivantes :</p><ul><li>L’utilisateur marque `https://example.com` avec un signet ou saisit cette adresse et fait face à une attaque de l’intercepteur : HSTS redirige automatiquement les requêtes HTTP vers HTTPS pour le domaine cible</li><li>L’application web qui est destinée à être purement HTTPS contient par inadvertance des liens HTTP ou traite du contenu sur HTTP : HSTS redirige automatiquement les requêtes HTTP vers HTTPS pour le domaine cible</li><li>Un intercepteur tente d’intercepter le trafic d’un utilisateur victime à l’aide d’un certificat non valide et espère que l’utilisateur acceptera le certificat incorrect : HSTS n’autorise pas un utilisateur à remplacer le message de certificat par un certificat non valide</li></ul>|

## <a name="ensure-sql-server-connection-encryption-and-certificate-validation"></a><a id="sqlserver-validation"></a>Assurer le chiffrement de la connexion SQL Server et la validation des certificats

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | SQL Azure  |
| **Attributs**              | Version SQL - V12 |
| **Informations de référence**              | [Best Practices on Writing Secure Connection Strings for SQL Database](https://social.technet.microsoft.com/wiki/contents/articles/2951.windows-azure-sql-database-connection-security.aspx#best) (Bonnes pratiques sur l’écriture de chaînes de connexion sécurisées pour SQL Database) |
| **Étapes** | <p>Toutes les communications entre SQL Database et une application cliente sont chiffrées en permanence à l’aide du protocole TLS (Transport Layer Security), anciennement SSL (Secure Sockets Layer). SQL Database ne prend pas en charge les connexions non chiffrées. Pour valider des certificats avec le code ou les outils de l’application, demandez explicitement une connexion chiffrée et ne faites pas confiance aux certificats de serveur. Si le code ou les outils de votre application ne demandent pas une connexion chiffrée, ils recevront pourtant des connexions chiffrées.</p><p>Cependant, comme ils ne peuvent pas valider les certificats de serveur, ils restent vulnérables aux attaques de « l’homme du milieu ». Pour valider des certificats avec le code d’application ADO.NET, définissez `Encrypt=True` et `TrustServerCertificate=False` dans la chaîne de connexion de base de données. Pour valider des certificats via SQL Server Management Studio, ouvrez la boîte de dialogue « Se connecter au serveur ». Cliquez sur « Chiffrer la connexion » dans l’onglet « Propriétés de connexion ».</p>|

## <a name="force-encrypted-communication-to-sql-server"></a><a id="encrypted-sqlserver"></a>Forcer des communications chiffrées vers SQL Server

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Local |
| **Attributs**              | Version SQL - MsSQL2016, Version SQL - MsSQL2012, Version SQL - MsSQL2014 |
| **Informations de référence**              | [Activer les connexions chiffrées dans le moteur de base de données](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine)  |
| **Étapes** | L’activation du chiffrement TLS augmente la sécurité des données transmises sur les réseaux entre les instances de SQL Server et les applications. |

## <a name="ensure-that-communication-to-azure-storage-is-over-https"></a><a id="comm-storage"></a>Vérifier que la communication vers le stockage Azure est sur HTTPS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Stockage Azure | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Chiffrement au niveau du transport – Utilisation de HTTPS](../../storage/blobs/security-recommendations.md#networking) |
| **Étapes** | Pour garantir la sécurité des données du stockage Azure en transit, utilisez toujours le protocole HTTPS lors de l’appel des API REST ou de l’accès aux objets dans le stockage. De plus, les signatures d’accès partagé, qui peuvent être utilisées pour déléguer l’accès aux objets de stockage Azure, incluent une option pour spécifier que seul le protocole HTTPS est autorisé avec les signatures d’accès partagé. Cette option garantit que le protocole approprié est utilisé par tous ceux qui envoient des liens avec des jetons SAP.|

## <a name="validate-md5-hash-after-downloading-blob-if-https-cannot-be-enabled"></a><a id="md5-https"></a>Valider le hachage MD5 après le téléchargement de blobs si le protocole HTTPS ne peut pas être activé

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Stockage Azure | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | StorageType - Blob |
| **Informations de référence**              | [Windows Azure Blob MD5 Overview](https://blogs.msdn.microsoft.com/windowsazurestorage/2011/02/17/windows-azure-blob-md5-overview/) (Vue d’ensemble de la vérification MD5 du service Blob Windows Azure) |
| **Étapes** | <p>Le service BLOB Windows Azure fournit des mécanismes permettant de garantir l’intégrité des données au niveau des couches de transport et d’application. Si, pour une raison quelconque, vous devez utiliser le protocole HTTP au lieu de HTTPS et que vous travaillez avec des objets blobs de blocs, vous pouvez utiliser la vérification MD5 pour vérifier l’intégrité des blobs transférés.</p><p>Ceci contribuera à la protection contre les erreurs au niveau du réseau/transport, mais pas nécessairement contre les attaques intermédiaires. Si vous pouvez utiliser le protocole HTTPS, qui fournit une sécurité au niveau du transport, alors l’utilisation de la vérification MD5 est redondant et inutile.</p>|

## <a name="use-smb-30-compatible-client-to-ensure-in-transit-data-encryption-to-azure-file-shares"></a><a id="smb-shares"></a>Utiliser un client compatible SMB 3.0 pour garantir le chiffrement des données en transit vers les partages de fichiers Azure

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Client mobile | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | StorageType - Fichier |
| **Informations de référence**              | [Azure File Storage, now generally available](https://azure.microsoft.com/blog/azure-file-storage-now-generally-available/#comment-2529238931) (Le stockage de fichiers Azure est désormais disponible de manière générale), [Prise en main du stockage de fichiers Azure sur Windows](../../storage/files/storage-dotnet-how-to-use-files.md#understanding-the-net-apis) |
| **Étapes** | Azure File Storage prend en charge le protocole HTTPS avec l’API REST, mais il est plus couramment utilisé comme partage de fichiers SMB attaché à une machine virtuelle. SMB 2.1 ne prend pas en charge le chiffrement. Les connexions sont donc autorisées uniquement dans la même région Azure. Toutefois, SMB 3.0 prend en charge le chiffrement et peut être utilisé avec Windows Server 2012 R2, Windows 8, Windows 8.1 et Windows 10, ce qui rend possibles les connexions d’accès entre régions et même les accès sur le bureau. |

## <a name="implement-certificate-pinning"></a><a id="cert-pinning"></a>Implémenter l’épinglage de certificat

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Stockage Azure | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, Windows Phone |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Certificate and Public Key Pinning](https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning) (Épinglage de clé publique et de certificat) |
| **Étapes** | <p>L’épinglage de certificat assure une protection contre les interceptions. L’épinglage consiste à associer un hôte à sa clé publique ou à son certificat X509 attendu. Une fois qu’un certificat ou une clé publique est connu ou vu par un hôte, le certificat ou la clé publique est associé ou « épinglé » à l’hôte. </p><p>Par conséquent, lorsqu’un pirate tente une interception de TLS, lors de la liaison TLS, la clé du serveur du pirate sera différente de la clé du certificat épinglé et la demande sera rejetée, empêchant ainsi l’interception. L’épinglage de certificat peut être obtenu en implémentant le délégué `ServerCertificateValidationCallback` de ServicePointManager.</p>|

### <a name="example"></a>Exemple
```csharp
using System;
using System.Net;
using System.Net.Security;
using System.Security.Cryptography;

namespace CertificatePinningExample
{
    class CertificatePinningExample
    {
        /* Note: In this example, we're hardcoding a the certificate's public key and algorithm for 
           demonstration purposes. In a real-world application, this should be stored in a secure
           configuration area that can be updated as needed. */

        private static readonly string PINNED_ALGORITHM = "RSA";

        private static readonly string PINNED_PUBLIC_KEY = "3082010A0282010100B0E75B7CBE56D31658EF79B3A1" +
            "294D506A88DFCDD603F6EF15E7F5BCBDF32291EC50B2B82BA158E905FE6A83EE044A48258B07FAC3D6356AF09B2" +
            "3EDAB15D00507B70DB08DB9A20C7D1201417B3071A346D663A241061C151B6EC5B5B4ECCCDCDBEA24F051962809" +
            "FEC499BF2D093C06E3BDA7D0BB83CDC1C2C6660B8ECB2EA30A685ADE2DC83C88314010FFC7F4F0F895EDDBE5C02" +
            "ABF78E50B708E0A0EB984A9AA536BCE61A0C31DB95425C6FEE5A564B158EE7C4F0693C439AE010EF83CA8155750" +
            "09B17537C29F86071E5DD8CA50EBD8A409494F479B07574D83EDCE6F68A8F7D40447471D05BC3F5EAD7862FA748" +
            "EA3C92A60A128344B1CEF7A0B0D94E50203010001";


        public static void Main(string[] args)
        {
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("https://azure.microsoft.com");
            request.ServerCertificateValidationCallback = (sender, certificate, chain, sslPolicyErrors) =>
            {
                if (certificate == null || sslPolicyErrors != SslPolicyErrors.None)
                {
                    // Error getting certificate or the certificate failed basic validation
                    return false;
                }

                var targetKeyAlgorithm = new Oid(certificate.GetKeyAlgorithm()).FriendlyName;
                var targetPublicKey = certificate.GetPublicKeyString();
                
                if (targetKeyAlgorithm == PINNED_ALGORITHM &&
                    targetPublicKey == PINNED_PUBLIC_KEY)
                {
                    // Success, the certificate matches the pinned value.
                    return true;
                }
                // Reject, either the key or the algorithm does not match the expected value.
                return false;
            };

            try
            {
                var response = (HttpWebResponse)request.GetResponse();
                Console.WriteLine($"Success, HTTP status code: {response.StatusCode}");
            }
            catch(Exception ex)
            {
                Console.WriteLine($"Failure, {ex.Message}");
            }
            Console.WriteLine("Press any key to end.");
            Console.ReadKey();
        }
    }
}
```

## <a name="enable-https---secure-transport-channel"></a><a id="https-transport"></a>Activer le protocole HTTPS - Sécuriser le canal de transport

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | NET Framework 3 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_transport_security_enabled) |
| **Étapes** | La configuration de l’application doit garantir l’utilisation du protocole HTTPS pour tous les accès à des informations sensibles.<ul><li>**EXPLICATION :** Si une application gère des informations sensibles et n’utilise pas le chiffrement au niveau du message, elle doit être uniquement autorisée à communiquer sur un canal de transport chiffré.</li><li>**RECOMMANDATIONS :** Assurez-vous que le transport HTTP est désactivé et activez le transport HTTPS à la place. Par exemple, remplacez `<httpTransport/>` par la balise `<httpsTransport/>`. Ne comptez pas sur une configuration réseau (pare-feu) pour garantir l’accès à l’application sur un canal sécurisé uniquement. D’un point de vue théorique, l’application ne doit pas dépendre du réseau pour sa sécurité.</li></ul><p>D’un point de vue pratique, les responsables de la sécurisation du réseau ne suivent pas en permanence les exigences de sécurité de l’application à mesure de leur évolution.</p>|

## <a name="wcf-set-message-security-protection-level-to-encryptandsign"></a><a id="message-protection"></a>WCF : définir le niveau de protection de la sécurité des messages sur EncryptAndSign

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | .NET Framework 3 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff650862(v=pandp.10)) |
| **Étapes** | <ul><li>**EXPLICATION :** Quand le niveau de protection est défini sur « aucun », la protection des messages est désactivée. La confidentialité et l’intégrité sont obtenues grâce à un niveau approprié de paramétrage.</li><li>**RECOMMANDATIONS :**<ul><li>Quand `Mode=None`, la protection des messages est désactivée.</li><li>Quand `Mode=Sign`, les messages sont signés mais pas chiffrés. Cette valeur doit être utilisée lorsque l’intégrité du message est primordiale.</li><li>Quand `Mode=EncryptAndSign`, les messages sont signés et chiffrés.</li></ul></li></ul><p>Pensez à désactiver le chiffrement et signez uniquement votre message lorsque vous avez simplement besoin de valider l’intégrité des informations, sans vous soucier de la confidentialité. Cela peut être utile pour les opérations ou les contrats de service pour lesquels vous avez besoin de valider l’expéditeur d’origine, mais qu’aucune donnée sensible n’est transmise. Lorsque vous réduisez le niveau de protection, veillez à ce que le message ne contienne pas de données à caractère personnel.</p>|

### <a name="example"></a>Exemple
La configuration du service et de l’opération permettant de signer uniquement le message est indiquée dans les exemples suivants. Exemple de contrat de service de `ProtectionLevel.Sign` : vous trouverez ci-dessous un exemple d’utilisation de ProtectionLevel.Sign au niveau du contrat de service : 
```
[ServiceContract(Protection Level=ProtectionLevel.Sign] 
public interface IService 
  { 
  string GetData(int value); 
  } 
```

### <a name="example"></a>Exemple
Exemple de contrat d’opération de `ProtectionLevel.Sign` (pour un contrôle granulaire) : Voici un exemple d’utilisation de `ProtectionLevel.Sign` au niveau du contrat d’opération :

```
[OperationContract(ProtectionLevel=ProtectionLevel.Sign] 
string GetData(int value);
``` 

## <a name="wcf-use-a-least-privileged-account-to-run-your-wcf-service"></a><a id="least-account-wcf"></a>WCF : utiliser un compte avec des privilèges minimum pour exécuter votre service WCF

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | .NET Framework 3 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648826(v=pandp.10)) |
| **Étapes** | <ul><li>**EXPLICATION :** N’exécutez pas de services WCF sous un compte administrateur ou avec des privilèges élevés. En cas de compromission des services, l’impact sera important.</li><li>**RECOMMANDATIONS :** Utilisez un compte avec des privilèges minimum pour héberger votre service WCF, car cela réduira la surface d’attaque de votre application ainsi que les dommages potentiels si vous êtes attaqué. Si le compte de service requiert des droits d’accès supplémentaires sur les ressources d’infrastructure tels que MSMQ, le journal des événements, les compteurs de performance et le système de fichiers, des autorisations appropriées doivent être accordées à ces ressources pour que le service WCF puisse s’exécuter correctement.</li></ul><p>Si votre service a besoin d’accéder à des ressources spécifiques pour le compte de l’appelant d’origine, utilisez l’emprunt d’identité et la délégation pour faire circuler l’identité de l’appelant pour une vérification d’autorisation en aval. Dans un scénario de développement, utilisez le compte de service réseau local, qui est un compte spécial intégré disposant de privilèges réduits. Dans un scénario de production, créez un compte de service de domaine personnalisé avec des privilèges minimum.</p>|

## <a name="force-all-traffic-to-web-apis-over-https-connection"></a><a id="webapi-https"></a>Forcer l’intégralité du trafic vers les API web sur la connexion HTTPS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Enforcing SSL in a Web API Controller](https://www.asp.net/web-api/overview/security/working-with-ssl-in-web-api) (Application de SSL dans un contrôleur d’API Web) |
| **Étapes** | Si une application a une liaison HTTP et une liaison HTTPS, les clients peuvent toujours utiliser HTTP pour accéder au site. Pour éviter cela, utilisez un filtre d’action afin de vous assurer que les demandes envoyées aux API protégées sont toujours sur HTTPS.|

### <a name="example"></a>Exemple 
Le code suivant montre un filtre d’authentification d’API web qui recherche TLS : 
```csharp
public class RequireHttpsAttribute : AuthorizationFilterAttribute
{
    public override void OnAuthorization(HttpActionContext actionContext)
    {
        if (actionContext.Request.RequestUri.Scheme != Uri.UriSchemeHttps)
        {
            actionContext.Response = new HttpResponseMessage(System.Net.HttpStatusCode.Forbidden)
            {
                ReasonPhrase = "HTTPS Required"
            };
        }
        else
        {
            base.OnAuthorization(actionContext);
        }
    }
}
```
Ajoutez ce filtre à toutes les actions de l’API web nécessitant TLS : 
```csharp
public class ValuesController : ApiController
{
    [RequireHttps]
    public HttpResponseMessage Get() { ... }
}
```
 
## <a name="ensure-that-communication-to-azure-cache-for-redis-is-over-tls"></a><a id="redis-ssl"></a>Vérifier que la communication vers le Cache Redis Azure s’effectue via TLS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Cache Azure pour Redis | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Prise en charge de TLS par Azure Redis](../../azure-cache-for-redis/cache-faq.md) |
| **Étapes** | Le serveur Redis n’offre pas une prise en charge intégrée de TLS, contrairement au Cache Azure pour Redis. Si vous vous connectez à Azure Cache pour Redis et que votre client est compatible avec TLS, par exemple StackExchange.Redis, vous devez utiliser TLS. Par défaut, le port non TLS est désactivé pour les nouvelles instances Cache Azure pour Redis. Assurez-vous que les valeurs par défaut sécurisées ne sont pas modifiées sauf s’il existe une dépendance sur la prise en charge TLS pour les clients Redis. |

Notez que Redis est conçu pour être accessible par les clients approuvés dans des environnements de confiance. Cela signifie que généralement ce n’est pas une bonne idée d’exposer l’instance Redis directement sur Internet ou, en règle générale, dans un environnement où les clients non approuvés peuvent directement accéder au port TCP Redis ou socket UNIX. 

## <a name="secure-device-to-field-gateway-communication"></a><a id="device-field"></a>Sécuriser la communication entre l’appareil et la passerelle de champ

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle de champ IoT | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Pour les appareils basés sur IP, le protocole de communication peut généralement être encapsulé dans un canal SSL/TLS pour protéger les données en transit. Pour d’autres protocoles qui ne prennent pas en charge le protocole SSL/TLS, recherchez s’il existe des versions sécurisées du protocole qui assurent la sécurité au niveau du transport ou du message. |

## <a name="secure-device-to-cloud-gateway-communication-using-ssltls"></a><a id="device-cloud"></a>Sécuriser la communication entre l’appareil et la passerelle de cloud à l’aide du protocole SSL/TLS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle cloud IoT | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Guide du développeur Azure IoT Hub](../../iot-hub/iot-hub-devguide.md) |
| **Étapes** | Sécuriser les protocoles HTTP/AMQP ou MQTT à l’aide du protocole SSL/TLS. |