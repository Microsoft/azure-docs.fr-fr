---
title: Résoudre les problèmes de connexion de point à site Azure
titleSuffix: Azure VPN Gateway
description: Découvrez comment dépanner et résoudre les problèmes courants de connexion point à site et d’autres erreurs et problèmes liés au réseau privé virtuel.
services: vpn-gateway
author: chadmath
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 03/26/2020
ms.author: genli
ms.openlocfilehash: c316aaf02979008b9d2ebc691d54c0fb95a5a52d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994857"
---
# <a name="troubleshooting-azure-point-to-site-connection-problems"></a>Résolution des problèmes : problèmes de connexion point à site Azure

Cet article répertorie les problèmes de connexion de point à site courants que vous pouvez rencontrer. Il décrit également les causes probables de ces problèmes, ainsi que les solutions possibles.

## <a name="vpn-client-error-a-certificate-could-not-be-found"></a>Erreur du client VPN : impossible de trouver un certificat

### <a name="symptom"></a>Symptôme

Lorsque vous essayez de vous connecter à un réseau virtuel Azure à l’aide du client VPN, vous recevez le message d’erreur suivant :

**Impossible de trouver un certificat qui peut être utilisé avec le protocole EAP (Extensible Authentication Protocol). (Erreur 798)**

### <a name="cause"></a>Cause

Ce problème se produit si le certificat client est absent de **Certificats - Utilisateur actuel\Personnel\Certificats**.

### <a name="solution"></a>Solution

Pour résoudre ce problème, effectuez les opérations suivantes :

1. Ouvrez le Gestionnaire de certificats : cliquez sur **Démarrer**, tapez **gérer les certificats d’ordinateur**, puis cliquez sur **gérer les certificats d’ordinateur** dans les résultats de la recherche.

2. Assurez-vous que les certificats suivants se trouvent au bon emplacement :

    | Certificat | Emplacement |
    | ------------- | ------------- |
    | AzureClient.pfx  | Utilisateur actuel\Personnel\Certificats |
    | AzureRoot.cer    | Ordinateur local\Autorités de certification racines de confiance|

3. Accédez à C:\Utilisateurs\<UserName>\AppData\Roaming\Microsoft\Network\Connections\Cm\<GUID>, puis installez manuellement le certificat (fichier *.cer) dans le magasin de l’utilisateur et de l’ordinateur.

Pour en savoir plus sur la façon d’installer le certificat client, consultez la page [Générer et exporter des certificats pour les connexions de point à site](vpn-gateway-certificates-point-to-site.md).

> [!NOTE]
> Lorsque vous importez le certificat client, ne sélectionnez pas l’option **Activer la protection renforcée par clé privée**.

## <a name="the-network-connection-between-your-computer-and-the-vpn-server-could-not-be-established-because-the-remote-server-is-not-responding"></a>La connexion réseau entre votre ordinateur et le serveur VPN n’a pas pu être établie parce que le serveur distant ne répond pas

### <a name="symptom"></a>Symptôme

Lorsque vous essayez de vous connecter à une passerelle de réseau virtuel Azure à l’aide d’IKEv2 sur Windows, vous obtenez le message d’erreur suivant :

**La connexion réseau entre votre ordinateur et le serveur VPN n’a pas pu être établie parce que le serveur distant ne répond pas**

### <a name="cause"></a>Cause
 
 Ce problème se produit lorsqu’une version de Windows ne prend pas en charge la fragmentation IKE.
 
### <a name="solution"></a>Solution

Le protocole IKEv2 est pris en charge sur Windows 10 et Server 2016. Toutefois, pour pouvoir utiliser le protocole IKEv2, vous devez installer les mises à jour et définir une valeur de clé de Registre localement. Les versions du système d’exploitation antérieures à Windows 10 ne sont pas prises en charge et ne peuvent utiliser que SSTP.

Pour préparer Windows 10 ou Server 2016 pour IKEv2 :

1. Installez la mise à jour.

   | Version du SE | Date | Nombre/lien |
   |---|---|---|---|
   | Windows Server 2016<br>Windows 10 version 1607 | 17 janvier 2018 | [KB4057142](https://support.microsoft.com/help/4057142/windows-10-update-kb4057142) |
   | Windows 10 version 1703 | 17 janvier 2018 | [KB4057144](https://support.microsoft.com/help/4057144/windows-10-update-kb4057144) |
   | Windows 10 version 1709 | 22 mars 2018 | [KB4089848](https://www.catalog.update.microsoft.com/search.aspx?q=kb4089848) |
   |  |  |  |  |

2. Définissez la valeur de clé de Registre. Créez ou attribuez la valeur 1 à la clé REG_DWORD `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RasMan\ IKEv2\DisableCertReqPayload` dans le registre.

## <a name="vpn-client-error-the-message-received-was-unexpected-or-badly-formatted"></a>Erreur du client VPN : le message reçu était inattendu ou mis en forme de façon incorrecte

### <a name="symptom"></a>Symptôme

Lorsque vous essayez de vous connecter à un réseau virtuel Azure à l’aide du client VPN, vous recevez le message d’erreur suivant :

**Le message reçu était inattendu ou formaté de façon incorrecte. (Erreur 0x80090326)**

### <a name="cause"></a>Cause

Ce problème se produit si l’une des conditions suivantes est vraie :

- L’itinéraire défini par l’utilisateur par défaut sur le sous-réseau de passerelle n’est pas défini correctement.
- La clé publique du certificat racine n’est pas téléchargée dans la passerelle VPN Azure. 
- La clé est endommagée ou a expiré.

### <a name="solution"></a>Solution

Pour résoudre ce problème, effectuez les opérations suivantes :

1. Supprimez l’itinéraire défini par le l’utilisateur sur le sous-réseau de passerelle. Assurez-vous que l’itinéraire défini par l’utilisateur transmet tout le trafic correctement.
2. Vérifiez l’état du certificat racine dans le portail Azure. Vous pourrez voir si le certificat a été révoqué ou non. S’il n’est pas révoqué, essayez de supprimer le certificat racine et de le télécharger une nouvelle fois. Pour en savoir plus, consultez la section [Créer des certificats](vpn-gateway-howto-point-to-site-classic-azure-portal.md#generatecerts).

## <a name="vpn-client-error-a-certificate-chain-processed-but-terminated"></a>Erreur du client VPN : une chaîne de certificats a été traitée mais s’est terminée 

### <a name="symptom"></a>Symptôme 

Lorsque vous essayez de vous connecter à un réseau virtuel Azure à l’aide du client VPN, vous recevez le message d’erreur suivant :

**Une chaîne de certificats a été traitée mais s’est terminée par un certificat racine qui n’est pas approuvé par le fournisseur d’approbation.**

### <a name="solution"></a>Solution

1. Assurez-vous que les certificats suivants se trouvent au bon emplacement :

    | Certificat | Emplacement |
    | ------------- | ------------- |
    | AzureClient.pfx  | Utilisateur actuel\Personnel\Certificats |
    | Azuregateway-*GUID*.cloudapp.net  | Utilisateur actuel\Autorités de certification racines de confiance|
    | AzureGateway-*GUID*.cloudapp.net, AzureRoot.cer    | Ordinateur local\Autorités de certification racines de confiance|

2. Si les certificats se trouvent déjà dans cet emplacement, essayez de les supprimer et de les réinstaller. Le certificat **azuregateway-*GUID*.cloudapp.net** se trouve dans le package de configuration du client VPN, téléchargé à partir du portail Azure. Vous pouvez vous servir des programmes d’archivage de fichiers afin d’extraire les fichiers du package.

## <a name="file-download-error-target-uri-is-not-specified"></a>Erreur de téléchargement du fichier : l’URI cible n’est pas spécifié

### <a name="symptom"></a>Symptôme

Vous recevez le message d’erreur suivant :

**Erreur de téléchargement du fichier. L’URI cible n’est pas spécifié.**

### <a name="cause"></a>Cause 

Ce problème se produit lorsque le type de passerelle est incorrecte. 

### <a name="solution"></a>Solution

Le type de passerelle VPN doit être défini sur la valeur **VPN**, tandis que le type de VPN doit être défini sur la valeur **RouteBased**.

## <a name="vpn-client-error-azure-vpn-custom-script-failed"></a>Erreur du client VPN : échec du script VPN personnalisé Azure 

### <a name="symptom"></a>Symptôme

Lorsque vous essayez de vous connecter à un réseau virtuel Azure à l’aide du client VPN, vous recevez le message d’erreur suivant :

**Échec du script personnalisé (pour mettre à jour votre table de routage) (Erreur 8007026f)**

### <a name="cause"></a>Cause

Ce problème peut se produire si vous essayez d’ouvrir la connexion VPN de point à site à l’aide d’un raccourci.

### <a name="solution"></a>Solution 

Ouvrez directement le package VPN au lieu de l’ouvrir à partir d’un raccourci.

## <a name="cannot-install-the-vpn-client"></a>Impossible d’installer le client VPN

### <a name="cause"></a>Cause 

Afin de faire confiance à la passerelle VPN pour votre réseau virtuel, un certificat supplémentaire est nécessaire. Le certificat est inclus dans le package de configuration du client VPN, généré à partir du portail Azure.

### <a name="solution"></a>Solution

Extrayez le package de configuration du client VPN et localisez le fichier .cer. Pour installer le certificat, procédez comme suit :

1. Ouvrez mmc.exe.
2. Ajoutez le composant logiciel enfichable **Certificats**.
3. Sélectionnez le compte **Ordinateur** de l’ordinateur local.
4. Cliquez sur le nœud **Autorités de certification racines de confiance** avec le bouton droit de la souris. Cliquez sur **All-Task (Toutes les tâches)**  > **Import**, puis naviguez vers le fichier .cer extrait du package de configuration du client VPN.
5. Redémarrez l'ordinateur. 
6. Essayez d’installer le client VPN.

## <a name="azure-portal-error-failed-to-save-the-vpn-gateway-and-the-data-is-invalid"></a>Erreur du portail Azure : échec de l’enregistrement. Les données de la passerelle VPN ne sont pas valides

### <a name="symptom"></a>Symptôme

Lorsque vous essayez d’enregistrer les modifications apportées à la passerelle VPN dans le portail Azure, vous recevez le message d’erreur suivant :

**Échec de l’enregistrement de la passerelle de réseau virtuel &lt;*nom de la passerelle*&gt;. Les données du certificat &lt;*ID de certificat*&gt; ne sont pas valides.**

### <a name="cause"></a>Cause 

Ce problème peut se produire si la clé publique de certificat racine que vous avez téléchargée contient un caractère non valide, comme un espace.

### <a name="solution"></a>Solution

Vérifiez que les données du certificat ne contiennent pas de caractères non valides, comme des sauts de ligne (retours chariot). La valeur entière doit être une longue ligne. Le texte ci-après est un extrait du certificat :

```text
-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIQFSwsLuUrCIdHwI3hzJbdBjANBgkqhkiG9w0BAQsFADAW
MRQwEgYDVQQDDAtQMlNSb290Q2VydDAeFw0xNzA2MTUwMjU4NDZaFw0xODA2MTUw
MzE4NDZaMBYxFDASBgNVBAMMC1AyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEF
AAOCAQ8AMIIBCgKCAQEAz8QUCWCxxxTrxF5yc5uUpL/bzwC5zZ804ltB1NpPa/PI
sa5uwLw/YFb8XG/JCWxUJpUzS/kHUKFluqkY80U+fAmRmTEMq5wcaMhp3wRfeq+1
G9OPBNTyqpnHe+i54QAnj1DjsHXXNL4AL1N8/TSzYTm7dkiq+EAIyRRMrZlYwije
407ChxIp0stB84MtMShhyoSm2hgl+3zfwuaGXoJQwWiXh715kMHVTSj9zFechYd7
5OLltoRRDyyxsf0qweTFKIgFj13Hn/bq/UJG3AcyQNvlCv1HwQnXO+hckVBB29wE
sF8QSYk2MMGimPDYYt4ZM5tmYLxxxvGmrGhc+HWXzMeQIDAQABozEwLzAOBgNVHQ8B
Af8EBAMCAgQwHQYDVR0OBBYEFBE9zZWhQftVLBQNATC/LHLvMb0OMA0GCSqGSIb3
DQEBCwUAA4IBAQB7k0ySFUQu72sfj3BdNxrXSyOT4L2rADLhxxxiK0U6gHUF6eWz
/0h6y4mNkg3NgLT3j/WclqzHXZruhWAXSF+VbAGkwcKA99xGWOcUJ+vKVYL/kDja
gaZrxHlhTYVVmwn4F7DWhteFqhzZ89/W9Mv6p180AimF96qDU8Ez8t860HQaFkU6
2Nw9ZMsGkvLePZZi78yVBDCWMogBMhrRVXG/xQkBajgvL5syLwFBo2kWGdC+wyWY
U/Z+EK9UuHnn3Hkq/vXEzRVsYuaxchta0X2UNRzRq+o706l+iyLTpe6fnvW6ilOi
e8Jcej7mzunzyjz4chN0/WVF94MtxbUkLkqP
-----END CERTIFICATE-----
```

## <a name="azure-portal-error-failed-to-save-the-vpn-gateway-and-the-resource-name-is-invalid"></a>Erreur du portail Azure : échec de l’enregistrement de la passerelle VPN. Le nom de la ressource n’est pas valide

### <a name="symptom"></a>Symptôme

Lorsque vous essayez d’enregistrer les modifications apportées à la passerelle VPN dans le portail Azure, vous recevez le message d’erreur suivant : 

**Échec de l’enregistrement de la passerelle de réseau virtuel &lt;*nom de la passerelle*&gt;. Le nom de la ressource &lt;*nom du certificat à télécharger*&gt; n’est pas valide**.

### <a name="cause"></a>Cause

Ce problème se produit car le nom du certificat contient un caractère non valide, comme un espace. 

## <a name="azure-portal-error-vpn-package-file-download-error-503"></a>Erreur du portail Azure : téléchargement du fichier de package VPN (Erreur 503)

### <a name="symptom"></a>Symptôme

Lorsque vous essayez de télécharger le package de configuration du client VPN, vous recevez le message d’erreur suivant :

**Impossible de télécharger le fichier. Détails de l’erreur : erreur 503. Le serveur est occupé.**
 
### <a name="solution"></a>Solution

Cette erreur peut être due à un problème réseau temporaire. Patientez quelques minutes, puis essayez à nouveau de télécharger le package VPN.

## <a name="azure-vpn-gateway-upgrade-all-point-to-site-clients-are-unable-to-connect"></a>Mise à niveau de la passerelle VPN Azure : aucun client point à site ne peut se connecter

### <a name="cause"></a>Cause

Si le certificat a atteint plus de 50 % de sa durée de vie, il est restauré.

### <a name="solution"></a>Solution

Pour résoudre ce problème, retéléchargez et redéployez le package point à site sur tous les clients.

## <a name="too-many-vpn-clients-connected-at-once"></a>Trop de clients VPN sont connectés

Le nombre de connexions maximal est atteint. Vous pouvez voir le nombre total de clients connectés dans le portail Azure.

## <a name="vpn-client-cannot-access-network-file-shares"></a>Les clients VPN ne peuvent pas accéder aux partages de fichiers réseau

### <a name="symptom"></a>Symptôme

Le client VPN s’est connecté au réseau virtuel Azure. Toutefois, le client ne peut pas accéder aux partages réseau.

### <a name="cause"></a>Cause

Le protocole SMB est utilisé pour l’accès au partage de fichiers. Au moment où la connexion est établie, le client VPN ajoute les informations d’identification de la session et l’échec se produit. Une fois la connexion établie, le client est forcé d’utiliser les informations d’identification en cache pour l’authentification Kerberos. Ce processus lance les requêtes sur le centre de distribution de clés (un contrôleur de domaine) afin d’obtenir un jeton. Étant donné que le client se connecte à partir d’Internet, il n’est peut-être pas en mesure d’atteindre le contrôleur de domaine. Par conséquent, le client ne peut pas basculer de Kerberos à NTLM. 

Le client est uniquement invité à fournir des informations d’identification quand il possède un certificat valide (avec SAN = UPN) émis par le domaine auquel il est joint. Le client doit également être physiquement connecté au réseau avec domaine. Dans ce cas, le client tente d’utiliser le certificat et d’atteindre le contrôleur de domaine. Le centre de distribution de clés renvoie alors une erreur « KDC_ERR_C_PRINCIPAL_UNKNOWN ». Cette erreur force le client à basculer vers NTLM. 

### <a name="solution"></a>Solution

Pour contourner le problème, désactivez la mise en cache des informations d’identification de domaine à partir de la sous-clé de Registre suivante : 

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\DisableDomainCreds - Set the value to 1`


## <a name="cannot-find-the-point-to-site-vpn-connection-in-windows-after-reinstalling-the-vpn-client"></a>Impossible de trouver la connexion VPN de point à site dans Windows après la réinstallation du client VPN

### <a name="symptom"></a>Symptôme

Supprimez la connexion VPN de point à site, puis réinstallez le client VPN. Dans ce cas, la connexion VPN n’est pas configurée correctement. Vous ne voyez pas la connexion VPN dans les paramètres **Connexions réseau** dans Windows.

### <a name="solution"></a>Solution

Pour résoudre le problème, supprimez les anciens fichiers de configuration du client VPN à partir de **C:\Utilisateurs\NomUtilisateur\AppData\Roaming\Microsoft\Network\Connections\<VirtualNetworkId>** , puis exécutez à nouveau le programme d’installation du client VPN.

## <a name="point-to-site-vpn-client-cannot-resolve-the-fqdn-of-the-resources-in-the-local-domain"></a>Le client VPN de point à site ne peut pas résoudre le nom de domaine complet des ressources dans le domaine local

### <a name="symptom"></a>Symptôme

Lorsque le client se connecte à Azure à l’aide d’une connexion VPN de point à site, il ne peut pas résoudre le nom de domaine complet des ressources de votre domaine local.

### <a name="cause"></a>Cause

Le client VPN de point à site utilise normalement des serveurs Azure DNS configurés dans le réseau virtuel Azure. Les serveurs Azure DNS étant prioritaires sur les serveurs DNS locaux qui sont configurés dans le client (sauf si les métriques de l’interface Ethernet sont inférieures), toutes les requêtes DNS sont envoyées aux serveurs Azure DNS. Si les serveurs Azure DNS ne disposent pas des enregistrements pour les ressources locales, la requête échoue.

### <a name="solution"></a>Solution

Pour résoudre le problème, assurez-vous que les serveurs Azure DNS utilisés sur le réseau virtuel Azure peuvent résoudre les enregistrements DNS pour les ressources locales. Pour ce faire, vous pouvez utiliser des redirecteurs DNS ou des redirecteurs conditionnels. Consultez [Résolution de noms à l’aide de votre propre serveur DNS](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) pour en savoir plus.

## <a name="the-point-to-site-vpn-connection-is-established-but-you-still-cannot-connect-to-azure-resources"></a>La connexion VPN de point à site est établie, mais vous ne pouvez toujours pas vous connecter aux ressources Azure 

### <a name="cause"></a>Cause

Ce problème peut se produire si le client VPN n’obtient pas les itinéraires à partir de la passerelle VPN Azure.

### <a name="solution"></a>Solution

Pour résoudre ce problème, [réinitialisez la passerelle VPN Azure](./reset-gateway.md). Pour s’assurer que les nouvelles routes sont utilisées, les clients VPN point à site doivent être retéléchargés une fois que le peering de réseaux virtuels a été configuré avec succès.

## <a name="error-the-revocation-function-was-unable-to-check-revocation-because-the-revocation-server-was-offlineerror-0x80092013"></a>Erreur : « La fonction de révocation n’a pas pu vérifier la révocation, car le serveur de révocation était hors connexion (Erreur 0x80092013) ».

### <a name="causes"></a>Causes
Ce message d’erreur se produit si le client ne peut pas accéder à http://crl3.digicert.com/ssca-sha2-g1.crl et à http://crl4.digicert.com/ssca-sha2-g1.crl.  La vérification de révocation requiert l’accès à ces deux sites.  Ce problème se produit en général sur le client sur lequel un serveur proxy est configuré. Dans certains environnements, si les requêtes ne passent pas par le serveur proxy, celles-ci seront refusées au niveau du pare-feu Edge.

### <a name="solution"></a>Solution

Vérifiez les paramètres du serveur proxy, assurez-vous que le client peut accéder à http://crl3.digicert.com/ssca-sha2-g1.crl et à http://crl4.digicert.com/ssca-sha2-g1.crl.

## <a name="vpn-client-error-the-connection-was-prevented-because-of-a-policy-configured-on-your-rasvpn-server-error-812"></a>Erreur du client VPN : la connexion a été empêchée en raison d’une stratégie configurée sur votre serveur RAS/VPN. (Erreur 812)

### <a name="cause"></a>Cause

Cette erreur se produit si le serveur RADIUS utilisé pour l’authentification du client VPN comporte des paramètres incorrects ou si Azure ne parvient pas à contacter le serveur Radius.

### <a name="solution"></a>Solution

Assurez-vous que le serveur RADIUS est configuré correctement. Pour plus d’informations, consultez [Intégration de l'authentification RADIUS avec le serveur Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfaserver-dir-radius.md).

## <a name="error-405-when-you-download-root-certificate-from-vpn-gateway"></a>« Erreur 405 » lorsque vous téléchargez le certificat racine à partir de la passerelle VPN

### <a name="cause"></a>Cause

Le certificat racine n’a pas été installé. Le certificat racine est installé dans le magasin **Certificats de confiance** du client.

## <a name="vpn-client-error-the-remote-connection-was-not-made-because-the-attempted-vpn-tunnels-failed-error-800"></a>Erreur du client VPN : la connexion à distance n’a pas été établie car les tunnels VPN essayés ont échoué. (Erreur 800) 

### <a name="cause"></a>Cause

Le pilote de carte d’interface réseau est obsolète.

### <a name="solution"></a>Solution

Mettre à jour le pilote de carte d’interface réseau :

1. Cliquez sur **Démarrer**, tapez **Gestionnaire de périphériques** et sélectionnez-le dans la liste des résultats. Si vous êtes invité à entrer un mot de passe administrateur ou une confirmation, tapez le mot de passe ou confirmez.
2. Dans les catégories **Cartes réseau**, recherchez la carte d’interface réseau que vous souhaitez mettre à jour.  
3. Double-cliquez sur le nom de l’appareil, sélectionnez **Mettre à jour le pilote**, sélectionnez **Rechercher automatiquement un pilote logiciel mis à jour**.
4. Si Windows ne trouve pas de nouveau pilote, recherchez-en un sur le site Web du fabricant de l’appareil et suivez ses instructions.
5. Redémarrez l’ordinateur et réessayez de vous connecter.

## <a name="vpn-client-error-dialing-vpn-connection-vpn-connection-name-status--vpn-platform-did-not-trigger-connection"></a>Erreur du client VPN : Numérotation de la connexion VPN <VPN Connection Name>, Status = La plateforme VPN n’a pas déclenché la connexion

Vous pouvez également voir l’erreur suivante dans l’observateur d’événements à partir de RasClient : « L’utilisateur <User> a composé le numéro de la connexion <VPN Connection Name>. La connexion a échoué. Code d’erreur retourné : 1460. »

### <a name="cause"></a>Cause

Azure VPN Client ne dispose pas de l’autorisation d’application « Applications en arrière-plan » activée dans les paramètres d’application pour Windows.

### <a name="solution"></a>Solution

1. Dans Windows, accédez à Paramètres -> Confidentialité -> Applications en arrière-plan
2. Basculer l’option « Autoriser les applications à s’exécuter en arrière-plan » sur Activé

## <a name="error-file-download-error-target-uri-is-not-specified"></a>Erreur : « Erreur de téléchargement du fichier. L’URI cible n’est pas spécifiée »

### <a name="cause"></a>Cause

Le type de passerelle configuré est incorrect.

### <a name="solution"></a>Solution

Le type de passerelle VPN Azure doit être défini sur la valeur VPN, tandis que le type de VPN doit être défini sur la valeur **RouteBased**.

## <a name="vpn-package-installer-doesnt-complete"></a>L’installation du package VPN ne se termine pas

### <a name="cause"></a>Cause

Ce problème peut être provoqué par des installations précédentes du client VPN. 

### <a name="solution"></a>Solution

Supprimez les anciens fichiers de configuration du client VPN à de **C:\Utilisateurs\NomUtilisateur\AppData\Roaming\Microsoft\Network\Connections\<VirtualNetworkId>** , puis réexécutez le programme d’installation du client VPN. 

## <a name="the-vpn-client-hibernates-or-sleep-after-some-time"></a>Le client VPN se met en veille prolongée ou en veille après un certain temps

### <a name="solution"></a>Solution

Vérifiez les paramètres de mise en veille et veille prolongée sur l’ordinateur qui exécute le client VPN.