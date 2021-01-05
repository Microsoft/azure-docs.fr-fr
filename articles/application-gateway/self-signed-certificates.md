---
title: Générer un certificat auto-signé avec une autorité de certification racine personnalisée
titleSuffix: Azure Application Gateway
description: Découvrez comment générer un certificat auto-signé Azure Application Gateway avec une autorité de certification racine personnalisée
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 07/23/2019
ms.author: victorh
ms.openlocfilehash: e60aa9f072a447af97aa7cc66534e6e893fdbcf6
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93396938"
---
# <a name="generate-an-azure-application-gateway-self-signed-certificate-with-a-custom-root-ca"></a>Générer un certificat auto-signé Azure Application Gateway avec une autorité de certification racine personnalisée

La référence SKU Application Gateway v2 introduit l’utilisation de certificats racines approuvés pour autoriser les serveurs backend. Cela supprime les certificats d’authentification requis dans la référence SKU v1. Le *certificat racine* est un certificat racine au format X.509 codé en base 64 (.CER) provenant des serveurs de certificats back-ends. Il identifie l’autorité de certification racine qui a émis le certificat de serveur et le certificat de serveur est ensuite utilisé pour la communication TLS/SSL.

Application Gateway approuve le certificat de votre site web par défaut s’il est signé par une autorité de certification connue (par exemple GoDaddy ou DigiCert). Vous n’avez pas besoin de charger explicitement le certificat racine dans ce cas. Pour plus d’informations, consultez [Présentation de la terminaison TLS et du chiffrement TLS de bout en bout sur la passerelle Application Gateway](ssl-overview.md). Toutefois, si vous disposez d’un environnement de développement/test et que vous ne souhaitez pas acheter un certificat signé par une autorité de certification, vous pouvez créer votre propre autorité de certification personnalisée et créer un certificat auto-signé avec celle-ci. 

> [!NOTE]
> Les certificats auto-signés ne sont pas approuvés par défaut et peuvent être difficiles à gérer. En outre, ils peuvent utiliser des suites de hachage et de chiffrement obsolètes qui peuvent ne pas être fortes. Pour une meilleure sécurité, achetez un certificat signé par une autorité de certification connue.

Dans cet article, vous allez apprendre à :

- Créer votre propre autorité de certification personnalisée
- Créer un certificat auto-signé signé par votre autorité de certification personnalisée
- Charger un certificat racine auto-signé dans Application Gateway pour authentifier le serveur backend

## <a name="prerequisites"></a>Prérequis

- **[OpenSSL](https://www.openssl.org/) sur un ordinateur exécutant Windows ou Linux** 

   Bien qu’il existe d’autres outils disponibles pour la gestion des certificats, ce tutoriel utilise OpenSSL. OpenSSL est fourni avec de nombreuses distributions Linux, telles que Ubuntu.
- **Un serveur web**

   Par exemple, Apache, IIS ou NGINX pour tester les certificats.

- **Une référence SKU Application Gateway v2**
   
  Si vous n’avez pas de passerelle d’application, consultez [Démarrage rapide : diriger le trafic web avec Azure Application Gateway - Portail Azure](quick-create-portal.md).

## <a name="create-a-root-ca-certificate"></a>Créer un certificat d’autorité de certification racine

Créez votre certificat d’autorité de certification racine à l’aide d’OpenSSL.

### <a name="create-the-root-key"></a>Créer la clé racine

1. Connectez-vous à l’ordinateur sur lequel OpenSSL est installé et exécutez la commande suivante. Cela crée une clé protégée par mot de passe.

   ```
   openssl ecparam -out contoso.key -name prime256v1 -genkey
   ```
1. À l’invite, tapez un mot de passe fort. Par exemple, contenant au moins neuf caractères, dont des majuscules, minuscules, chiffres et symboles.

### <a name="create-a-root-certificate-and-self-sign-it"></a>Créer un certificat racine et le signer automatiquement

1. Utilisez les commandes suivantes pour générer la requête de signature de certificat et le certificat.

   ```
   openssl req -new -sha256 -key contoso.key -out contoso.csr

   openssl x509 -req -sha256 -days 365 -in contoso.csr -signkey contoso.key -out contoso.crt
   ```
   Les commandes précédentes créent le certificat racine. Vous utiliserez ceci pour signer votre certificat de serveur.

1. Lorsque vous y êtes invité, tapez le mot de passe de la clé racine et les informations d’organisation de l’autorité de certification personnalisée, telles que Pays/Région, État, Org, OU et le nom de domaine complet (domaine de l’émetteur).

   ![créer un certificat racine](media/self-signed-certificates/root-cert.png)

## <a name="create-a-server-certificate"></a>Créer un certificat de serveur

Ensuite, vous allez créer un certificat de serveur à l’aide d’OpenSSL.

### <a name="create-the-certificates-key"></a>Créer la clé du certificat

Utilisez la commande suivante pour générer la clé pour le certificat de serveur.

   ```
   openssl ecparam -out fabrikam.key -name prime256v1 -genkey
   ```

### <a name="create-the-csr-certificate-signing-request"></a>Créer une requête de signature de certificat

La requête de signature de certificat est une clé publique qui est donnée à une autorité de certification lors de la demande d’un certificat. L’autorité de certification émet le certificat pour cette demande spécifique.

> [!NOTE]
> Le nom commun (CN) pour le certificat de serveur doit être différent du domaine de l’émetteur. Par exemple, dans ce cas, le nom commun de l’émetteur est `www.contoso.com` et le nom commun du certificat du serveur est `www.fabrikam.com`.


1. Utilisez la commande suivante pour générer la requête de signature de certificat :

   ```
   openssl req -new -sha256 -key fabrikam.key -out fabrikam.csr
   ```

1. Lorsque vous y êtes invité, tapez le mot de passe de la clé racine et les informations d’organisation de l’autorité de certification personnalisée : Pays/Région, État, Org, OU et le nom de domaine complet. Il s’agit du domaine du site web qui doit être différent de celui de l’émetteur.

   ![Certificat serveur](media/self-signed-certificates/server-cert.png)

### <a name="generate-the-certificate-with-the-csr-and-the-key-and-sign-it-with-the-cas-root-key"></a>Générer le certificat avec la requête de signature de certificat et la clé et le signer avec la clé racine de l’autorité de certification

1. Exécutez la commande suivante pour créer le certificat :

   ```
   openssl x509 -req -in fabrikam.csr -CA  contoso.crt -CAkey contoso.key -CAcreateserial -out fabrikam.crt -days 365 -sha256
   ```
### <a name="verify-the-newly-created-certificate"></a>Vérifier le certificat que vous venez de créer

1. Utilisez la commande suivante pour imprimer la sortie du fichier CRT et vérifier son contenu :

   ```
   openssl x509 -in fabrikam.crt -text -noout
   ```

   ![Vérification du certificat](media/self-signed-certificates/verify-cert.png)

1. Vérifiez les fichiers de votre répertoire et vérifiez que vous disposez des fichiers suivants :

   - contoso.crt
   - contoso.key
   - fabrikam.crt
   - fabrikam.key

## <a name="configure-the-certificate-in-your-web-servers-tls-settings"></a>Configurer le certificat dans les paramètres TLS de votre serveur web

Dans votre serveur web, configurez TLS à l’aide des fichiers fabrikam.crt et fabrikam.key. Si votre serveur web ne peut pas prendre deux fichiers, vous pouvez les combiner en un seul fichier .pem ou. pfx à l’aide des commandes OpenSSL.

### <a name="iis"></a>IIS

Pour obtenir des instructions sur l’importation d’un certificat et son téléchargement sous la forme d’un certificat de serveur sur IIS, consultez [Procédure : Installer des certificats importés sur un serveur web dans Windows Server 2003](https://support.microsoft.com/help/816794/how-to-install-imported-certificates-on-a-web-server-in-windows-server).

Pour obtenir des instructions de liaison TLS, consultez [Configuration de SSL sur IIS 7](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis#create-an-ssl-binding-1).

### <a name="apache"></a>Apache

La configuration suivante est un exemple d’[hôte virtuel configuré pour SSL](https://cwiki.apache.org/confluence/display/HTTPD/NameBasedSSLVHosts) dans Apache :

```
<VirtualHost www.fabrikam:443>
      DocumentRoot /var/www/fabrikam
      ServerName www.fabrikam.com
      SSLEngine on
      SSLCertificateFile /home/user/fabrikam.crt
      SSLCertificateKeyFile /home/user/fabrikam.key
</VirtualHost>
```

### <a name="nginx"></a>NGINX

La configuration suivante est un exemple de [bloc NGINX Server](https://nginx.org/docs/http/configuring_https_servers.html) avec une configuration TLS :

![NGINX avec TLS](media/self-signed-certificates/nginx-ssl.png)

## <a name="access-the-server-to-verify-the-configuration"></a>Accéder au serveur pour vérifier la configuration

1. Ajoutez le certificat racine au magasin racine approuvé de votre ordinateur. Lorsque vous accédez au site web, assurez-vous que l’intégralité de la chaîne de certificats est visible dans le navigateur.

   ![Certificats racines approuvés](media/self-signed-certificates/trusted-root-cert.png)

   > [!NOTE]
   > Il est supposé que DNS a été configuré pour pointer le nom du serveur web (dans cet exemple, www.fabrikam.com) vers l’adresse IP de votre serveur web. Si ce n’est pas le cas, vous pouvez modifier le [fichier hosts](https://answers.microsoft.com/en-us/windows/forum/all/how-to-edit-host-file-in-windows-10/7696f204-2aaf-4111-913b-09d6917f7f3d) pour résoudre le nom.
1. Accédez à votre site web, puis cliquez sur l’icône de verrou dans la zone d’adresse de votre navigateur pour vérifier le site et les informations de certificat.

## <a name="verify-the-configuration-with-openssl"></a>Vérifier la configuration avec OpenSSL

Vous pouvez également utiliser OpenSSL pour vérifier le certificat.

```
openssl s_client -connect localhost:443 -servername www.fabrikam.com -showcerts
```

![Vérification du certificat OpenSSL](media/self-signed-certificates/openssl-verify.png)

## <a name="upload-the-root-certificate-to-application-gateways-http-settings"></a>Charger le certificat racine dans les paramètres HTTP d’Application Gateway

Pour charger le certificat dans Application Gateway, vous devez exporter le certificat .crt dans un format .cer encodé en base 64. Étant donné que.crt contient déjà la clé publique dans le format encodé en base 64, il vous suffit de renommer l’extension de fichier .crt en .cer. 

### <a name="azure-portal"></a>Portail Azure

Pour charger le certificat racine approuvé à partir du portail, sélectionnez **Paramètres HTTP** et choisissez le protocole **HTTPS**.

![Ajouter un certificat avec le portail](media/self-signed-certificates/portal-cert.png)

### <a name="azure-powershell"></a>Azure PowerShell

Vous pouvez utiliser Azure CLI ou Azure PowerShell pour charger le certificat racine. Le code suivant est un exemple Azure PowerShell.

> [!NOTE]
> L’exemple suivant ajoute un certificat racine approuvé à la passerelle Application Gateway, crée un paramètre HTTP et ajoute une nouvelle règle, en supposant que le pool backend et l’écouteur existent déjà.

```azurepowershell
## Add the trusted root certificate to the Application Gateway

$gw=Get-AzApplicationGateway -Name appgwv2 -ResourceGroupName rgOne

Add-AzApplicationGatewayTrustedRootCertificate `
   -ApplicationGateway $gw `
   -Name CustomCARoot `
   -CertificateFile "C:\Users\surmb\Downloads\contoso.cer"

$trustedroot = Get-AzApplicationGatewayTrustedRootCertificate `
   -Name CustomCARoot `
   -ApplicationGateway $gw

## Get the listener, backend pool and probe

$listener = Get-AzApplicationGatewayHttpListener `
   -Name basichttps `
   -ApplicationGateway $gw

$bepool = Get-AzApplicationGatewayBackendAddressPool `
  -Name testbackendpool `
  -ApplicationGateway $gw

Add-AzApplicationGatewayProbeConfig `
  -ApplicationGateway $gw `
  -Name testprobe `
  -Protocol Https `
  -HostName "www.fabrikam.com" `
  -Path "/" `
  -Interval 15 `
  -Timeout 20 `
  -UnhealthyThreshold 3

$probe = Get-AzApplicationGatewayProbeConfig `
  -Name testprobe `
  -ApplicationGateway $gw

## Add the configuration to the HTTP Setting and don't forget to set the "hostname" field
## to the domain name of the server certificate as this will be set as the SNI header and
## will be used to verify the backend server's certificate. Note that TLS handshake will
## fail otherwise and might lead to backend servers being deemed as Unhealthy by the probes

Add-AzApplicationGatewayBackendHttpSettings `
  -ApplicationGateway $gw `
  -Name testbackend `
  -Port 443 `
  -Protocol Https `
  -Probe $probe `
  -TrustedRootCertificate $trustedroot `
  -CookieBasedAffinity Disabled `
  -RequestTimeout 20 `
  -HostName www.fabrikam.com

## Get the configuration and update the Application Gateway

$backendhttp = Get-AzApplicationGatewayBackendHttpSettings `
  -Name testbackend `
  -ApplicationGateway $gw

Add-AzApplicationGatewayRequestRoutingRule `
  -ApplicationGateway $gw `
  -Name testrule `
  -RuleType Basic `
  -BackendHttpSettings $backendhttp `
  -HttpListener $listener `
  -BackendAddressPool $bepool

Set-AzApplicationGateway -ApplicationGateway $gw 
```

### <a name="verify-the-application-gateway-backend-health"></a>Vérifier l’intégrité du backend Application Gateway

1. Cliquez sur l’affichage **Intégrité principale** de votre passerelle d’application pour vérifier si la sonde est saine.
1. Vous devez voir que l’état est **sain** pour la sonde HTTPS.

![Sonde HTTPS](media/self-signed-certificates/https-probe.png)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur SSL/TLS dans Application Gateway, consultez [Présentation de la terminaison TLS et du chiffrement TLS de bout en bout sur la passerelle Application Gateway](ssl-overview.md).