---
title: Problèmes lors de l’installation du connecteur d’agent de proxy d’application
description: Comment résoudre les problèmes que vous pouvez rencontrer lors de l’installation du connecteur de l’agent Proxy d’application pour Azure Active Directory.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 05/21/2018
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 0b7fee330f93097b561714ecc938eaf3fee8f2b5
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657327"
---
# <a name="problem-installing-the-application-proxy-agent-connector"></a>Problèmes lors de l’installation du connecteur d’agent de proxy d’application

Le connecteur Proxy d’application de Microsoft Azure Active Directory est un composant de domaine interne qui utilise des connexions sortantes pour établir la connectivité entre le point de terminaison disponible dans le cloud et le domaine interne.

## <a name="general-problem-areas-with-connector-installation"></a>Problèmes généraux avec l’installation du connecteur

En cas d’échec de l’installation d’un connecteur, la cause est généralement liée à l’un des aspects suivants :

1.  **Connectivité** : pour une installation réussie, le nouveau connecteur doit s’inscrire et établir les propriétés d’approbation ultérieures. Pour cela, connectez-vous au service cloud Proxy d’application d’Azure Active Directory.

2.  **Établissement de l’approbation** : le nouveau connecteur crée un certificat auto-signé et s’inscrit auprès du service cloud.

3.  **Authentification de l’administrateur** : pendant l’installation, l’utilisateur doit fournir des informations d’identification d’administrateur pour terminer l’installation du connecteur.

> [!NOTE]
> Les journaux d’installation du connecteur se trouvent dans le dossier %TEMP% et peuvent fournir des informations supplémentaires sur l’origine de l’échec de l’installation.

## <a name="verify-connectivity-to-the-cloud-application-proxy-service-and-microsoft-login-page"></a>Vérification de la connectivité vers le service de proxy d’application cloud et la page de connexion Microsoft

**Objectif** : vérifier que l’ordinateur connecteur peut se connecter au point de terminaison d’inscription de Proxy d’application ainsi qu’à la page de connexion Microsoft.

1.  Sur le serveur du connecteur, exécutez un test de port à l’aide de [telnet](/windows-server/administration/windows-commands/telnet) ou autres outils de tests de port, pour vérifier que les ports 443 et 80 sont ouverts.

2.  Si un de ces ports ne réussit pas, vérifiez que le pare-feu ou proxy principal a accès aux domaines et ports requis. Voir [Préparer votre environnement local](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment).

3.  Ouvrez un navigateur (ou un nouvel onglet) et accédez à la page web suivante : `https://login.microsoftonline.com`. Ensuite, vérifiez que vous pouvez vous connecter à cette page.

## <a name="verify-machine-and-backend-components-support-for-application-proxy-trust-certificate"></a>Vérifier la prise en charge du certificat de confiance du proxy d’application par la machine et les composants back-end

**Objectif** : Vérifier que la machine connecteur, le pare-feu et le proxy back-end peuvent prendre en charge le certificat créé par le connecteur en vue d’une approbation ultérieure, et que le certificat est valide.

>[!NOTE]
>Le connecteur tente de créer un certificat SHA512 pris en charge par TLS1.2. Si l’ordinateur ou le pare-feu principal et le proxy ne prennent pas en charge TLS1.2, l’installation échoue.
>
>

**Passez en revue les prérequis :**

1.  Vérifiez que l’ordinateur prend en charge TLS1.2 (toutes les versions de Windows supérieures à 2012 R2 doivent prendre en charge TLS 1.2). Si votre ordinateur connecteur dispose de la version 2012 R2 ou d’une version antérieure, vérifiez que les bases de connaissances suivantes sont installées sur l’ordinateur : <https://support.microsoft.com/help/2973337/sha512-is-disabled-in-windows-when-you-use-tls-1.2>

2.  Contactez votre administrateur réseau et demandez-lui de vérifier que le proxy principal et le pare-feu ne bloquent par SHA512 pour le trafic sortant.

**Pour vérifier le certificat client :**

Vérifiez l’empreinte numérique du certificat client actuel. Le magasin de certificats est accessible sous `%ProgramData%\microsoft\Microsoft AAD Application Proxy Connector\Config\TrustSettings.xml`.

```
<?xml version="1.0" encoding="utf-8"?>
<ConnectorTrustSettingsFile xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <CloudProxyTrust>
    <Thumbprint>4905CC64B2D81BBED60962ECC5DCF63F643CCD55</Thumbprint>
    <IsInUserStore>false</IsInUserStore>
  </CloudProxyTrust>
</ConnectorTrustSettingsFile>
```

Les valeurs **IsInUserStore** possibles sont **true** et **false**. Une valeur **true** signifie que le certificat renouvelé automatiquement est stocké dans le conteneur personnel du magasin de certificats de l’utilisateur du service réseau. La valeur **false** signifie que le certificat client a été créé pendant l’installation ou l’inscription initiée par la commande Register-AppProxyConnector et qu’il est stocké dans le conteneur personnel du magasin de certificats de l’ordinateur local.

Si la valeur est **true**, procédez comme suit pour vérifier le certificat :
1. Télécharger [PsTools.zip](/sysinternals/downloads/pstools)
2. Extrayez [PsExec](/sysinternals/downloads/psexec) du package et exécutez **psexec -i -u "nt authority\network service" cmd.exe** à partir d’une invite de commandes avec élévation de privilèges.
3. Exécuter **certmgr.msc** dans l’invite de commandes nouvellement apparue
4. Dans la console de gestion, développez le conteneur personnel, puis cliquez sur Certificats.
5. Localiser le certificat émis par **connectorregistrationca.msappproxy.net**

Si la valeur est **false**, procédez comme suit pour vérifier le certificat :
1. Exécuter **certlm.msc**
2. Dans la console de gestion, développez le conteneur personnel, puis cliquez sur Certificats.
3. Localiser le certificat émis par **connectorregistrationca.msappproxy.net**

**Pour renouveler le certificat client :**

Si un connecteur n’est pas connecté au service pendant plusieurs mois, ses certificats ont peut-être expiré. L’échec du renouvellement du certificat aboutit à un certificat expiré. Ceci provoque l’arrêt du fonctionnement du service du connecteur. L’événement 1000 est enregistré dans le journal d’administration du connecteur :

« Échec de la réinscription du connecteur : Le certificat de confiance du connecteur a expiré. Exécutez l’applet de commande PowerShell Register-AppProxyConnector sur l’ordinateur où le connecteur est en cours d’exécution pour réinscrire votre connecteur. »

Dans ce cas, désinstallez et réinstallez le connecteur pour déclencher l’inscription, ou exécutez les commandes PowerShell suivantes :

```
Import-module AppProxyPSModule
Register-AppProxyConnector
```

Pour plus d’informations sur la commande Register-AppProxyConnector, consultez [Créer un script d’installation sans assistance pour le connecteur de proxy d’application Azure AD](./application-proxy-register-connector-powershell.md)

## <a name="verify-admin-is-used-to-install-the-connector"></a>Vérification de l’utilisation d’une connexion administrateur pour l’installation du connecteur

**Objectif** : vérifier que l’utilisateur qui tente d’installer le connecteur est un administrateur disposant des informations d’identification correctes. Actuellement, l’installation requiert que l’utilisateur soit au moins un administrateur d’application.

**Pour vérifier que les informations d’identification sont correctes :**

Connectez-vous à `https://login.microsoftonline.com` en utilisant les mêmes informations d’identification. Vérifiez que la connexion a réussi. Vous pouvez vérifier le rôle utilisateur en sélectionnant **Azure Active Directory** -&gt; **Utilisateurs et groupes** -&gt; **Tous les utilisateurs**. 

Sélectionnez votre compte d’utilisateur, puis « Rôle d’annuaire » dans le menu qui s’affiche. Vérifiez que le rôle sélectionné est « Administrateur d’application ». Si vous ne pouvez accéder à aucune des pages de ces étapes, vous n’avez pas de rôle requis.

## <a name="next-steps"></a>Étapes suivantes
[Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md)