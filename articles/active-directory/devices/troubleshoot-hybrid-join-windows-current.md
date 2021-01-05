---
title: Résolution des problèmes liés aux appareils hybrides joints à Azure Active Directory
description: Résolution des problèmes des appareils hybrides Windows 10 et Windows Server 2016 joints à Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jairoc
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: e235bf90568a1382a5ecee3ff4d2283aaa32f10b
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93083214"
---
# <a name="troubleshooting-hybrid-azure-active-directory-joined-devices"></a>Résolution des problèmes liés aux appareils hybrides joints à Azure Active Directory

Le contenu de cet article s’applique aux appareils exécutant Windows 10 ou Windows Server 2016.

Pour les autres clients Windows, consultez l’article [Dépanner des appareils hybrides de bas niveau joints à Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md).

Cet article suppose que vous avez [configuré des appareils hybrides joints à Azure Active Directory](hybrid-azuread-join-plan.md) de façon à prendre en charge les scénarios suivants :

- Accès conditionnel basé sur les appareils
- [Itinérance d’entreprise des paramètres](./enterprise-state-roaming-overview.md)
- [Windows Hello Entreprise](/windows/security/identity-protection/hello-for-business/hello-identity-verification)

Ce document fournit des conseils de dépannage pour résoudre les problèmes potentiels.

Pour Windows 10 et Windows Server 2016, la jonction Azure Active Directory hybride prend en charge la mise à jour Windows du 10 novembre 2015 et au-delà.

## <a name="troubleshoot-join-failures"></a>Résoudre les problèmes d’échecs de jointure

### <a name="step-1-retrieve-the-join-status"></a>Étape 1 : Récupérer l’état de jonction

**Pour récupérer l’état de jonction :**

1. Ouvrez une invite de commandes en tant qu’administrateur.
2. Saisissez `dsregcmd /status`

```
+----------------------------------------------------------------------+
| Device State                                                         |
+----------------------------------------------------------------------+

    AzureAdJoined: YES
 EnterpriseJoined: NO
         DeviceId: 5820fbe9-60c8-43b0-bb11-44aee233e4e7
       Thumbprint: B753A6679CE720451921302CA873794D94C6204A
   KeyContainerId: bae6a60b-1d2f-4d2a-a298-33385f6d05e9
      KeyProvider: Microsoft Platform Crypto Provider
     TpmProtected: YES
     KeySignTest: : MUST Run elevated to test.
              Idp: login.windows.net
         TenantId: 72b988bf-xxxx-xxxx-xxxx-2d7cd011xxxx
       TenantName: Contoso
      AuthCodeUrl: https://login.microsoftonline.com/msitsupp.microsoft.com/oauth2/authorize
   AccessTokenUrl: https://login.microsoftonline.com/msitsupp.microsoft.com/oauth2/token
           MdmUrl: https://enrollment.manage-beta.microsoft.com/EnrollmentServer/Discovery.svc
        MdmTouUrl: https://portal.manage-beta.microsoft.com/TermsOfUse.aspx
  dmComplianceUrl: https://portal.manage-beta.microsoft.com/?portalAction=Compliance
      SettingsUrl: eyJVcmlzIjpbImh0dHBzOi8va2FpbGFuaS5vbmUubWljcm9zb2Z0LmNvbS8iLCJodHRwczovL2thaWxhbmkxLm9uZS5taWNyb3NvZnQuY29tLyJdfQ==
   JoinSrvVersion: 1.0
       JoinSrvUrl: https://enterpriseregistration.windows.net/EnrollmentServer/device/
        JoinSrvId: urn:ms-drs:enterpriseregistration.windows.net
    KeySrvVersion: 1.0
        KeySrvUrl: https://enterpriseregistration.windows.net/EnrollmentServer/key/
         KeySrvId: urn:ms-drs:enterpriseregistration.windows.net
     DomainJoined: YES
       DomainName: CONTOSO

+----------------------------------------------------------------------+
| User State                                                           |
+----------------------------------------------------------------------+

             NgcSet: YES
           NgcKeyId: {C7A9AEDC-780E-4FDA-B200-1AE15561A46B}
    WorkplaceJoined: NO
      WamDefaultSet: YES
WamDefaultAuthority: organizations
       WamDefaultId: https://login.microsoft.com
     WamDefaultGUID: {B16898C6-A148-4967-9171-64D755DA8520} (AzureAd)
         AzureAdPrt: YES
```

### <a name="step-2-evaluate-the-join-status"></a>Étape 2 : Évaluer l’état de jonction

Examinez les champs suivants et assurez-vous qu’ils disposent des valeurs attendues :

#### <a name="domainjoined--yes"></a>DomainJoined : YES

Ce champ indique si l’appareil est joint à un répertoire Active Directory local ou non. Si la valeur est **NON** , l’appareil ne peut pas effectuer de jonction hybride Azure AD.

#### <a name="workplacejoined--no"></a>WorkplaceJoined : Non

Ce champ indique si l’appareil est inscrit auprès d’Azure AD mais en tant qu’appareil personnel (avec la mention *Joint à l’espace de travail* ). Cette valeur doit être **NON** pour un ordinateur appartenant à un domaine qui est également une jonction hybride Azure AD. Si la valeur est **OUI** , un compte professionnel ou scolaire a été ajouté avant l’achèvement d’une jonction hybride Azure AD. Dans ce cas, le compte est ignoré lors de l’utilisation de Windows 10, version 1607 ou ultérieure.

#### <a name="azureadjoined--yes"></a>AzureAdJoined : YES

Ce champ indique si l’appareil est joint. La valeur sera **Oui** si l’appareil est un appareil joint à Azure AD ou un appareil hybride joint à Azure AD.
Si la valeur est **Non** , la jonction à Azure AD n’est pas encore terminée.

Passez aux étapes suivantes pour poursuivre le dépannage.

### <a name="step-3-find-the-phase-in-which-join-failed-and-the-errorcode"></a>Étape 3 : Rechercher la phase à laquelle la jointure a échoué ainsi que le code d’erreur

#### <a name="windows-10-1803-and-above"></a>Windows 10 version 1803 et ultérieures

Recherchez la sous-section « Previous Registration » dans la section « Diagnostic Data » de la sortie d’état de la jointure. Cette section s’affiche uniquement si l’appareil est joint à un domaine et ne peut pas joindre Azure AD hybride.
Le champ « Error Phase » indique la phase de l’échec de la jointure, tandis que « Client ErrorCode » désigne le code d’erreur de l’opération de jointure.

```
+----------------------------------------------------------------------+
     Previous Registration : 2019-01-31 09:16:43.000 UTC
         Registration Type : sync
               Error Phase : join
          Client ErrorCode : 0x801c03f2
          Server ErrorCode : DirectoryError
            Server Message : The device object by the given id (e92325d0-xxxx-xxxx-xxxx-94ae875d5245) is not found.
              Https Status : 400
                Request Id : 6bff0bd9-820b-484b-ab20-2a4f7b76c58e
+----------------------------------------------------------------------+
```

#### <a name="older-windows-10-versions"></a>Versions antérieures de Windows 10

Servez-vous des journaux de l’Observateur d’événements pour identifier la phase et le code d’erreur des échecs de jointure.

1. Ouvrez les journaux d’événements **User Device Registration** dans l’Observateur d’événements. Il se trouvent sous **Journaux des applications et des services** > **Microsoft** > **Windows** > **User Device Registration**.
2. Recherchez les événements associés aux ID d’événement 304, 305 et 307.

:::image type="content" source="./media/troubleshoot-hybrid-join-windows-current/1.png" alt-text="Capture d’écran de l’observateur d’événements. Un événement ayant l’ID 304 est sélectionné et ses informations sont affichées avec le code d’erreur et la phase en évidence." border="false":::

:::image type="content" source="./media/troubleshoot-hybrid-join-windows-current/2.png" alt-text="Capture d’écran de l’observateur d’événements. Un événement ayant l’ID 305 est visible et ses informations sont affichées avec le code d’erreur en évidence." border="false":::

### <a name="step-4-check-for-possible-causes-and-resolutions-from-the-lists-below"></a>Étape 4 : Rechercher les causes et les résolutions possibles dans les listes suivantes

#### <a name="pre-check-phase"></a>Phase de vérification préalable

Causes possibles de l’échec :

- L’appareil n’a pas le contrôleur de domaine en ligne de mire.
   - L’appareil doit se trouver sur le réseau interne ou le VPN de l’organisation avec le contrôleur de domaine Active Directory (AD) local en ligne de mire.

#### <a name="discover-phase"></a>Phase de découverte

Causes possibles de l’échec :

- Objet SCP (point de connexion de service) mal configuré/incapable de lire l’objet SCP à partir du contrôleur de domaine.
   - Un objet SCP valide est nécessaire dans la forêt AD, à laquelle appartient l’appareil, qui pointe vers un nom de domaine vérifié dans Azure AD.
   - Vous trouverez des détails dans la section [Configurer un point de connexion de service](hybrid-azuread-join-federated-domains.md#configure-hybrid-azure-ad-join).
- Échec de connexion et de récupération des métadonnées de découverte à partir du point de terminaison de découverte.
   - L’appareil doit pouvoir accéder à `https://enterpriseregistration.windows.net`, dans le contexte SYSTEM, pour découvrir les points de terminaison d’inscription et d’autorisation.
   - Si l’environnement local nécessite un proxy sortant, l’administrateur informatique doit vérifier que le compte d’ordinateur de l’appareil peut découvrir le proxy sortant et s’y authentifier en mode silencieux.
- Échec de connexion au point de terminaison du domaine utilisateur et exécution de la découverte de domaine. (Windows 10 version 1809 et ultérieures uniquement)
   - L’appareil doit pouvoir accéder à `https://login.microsoftonline.com`, dans le contexte SYSTEM, pour effectuer une découverte de domaine pour le domaine vérifié et déterminer le type de domaine (managé/fédéré).
   - si l’environnement local nécessite un proxy sortant, l’administrateur informatique doit vérifier que le contexte SYSTEM de l’appareil peut découvrir le proxy sortant et s’y authentifier en mode silencieux.

**Codes d’erreur courants :**

- **DSREG_AUTOJOIN_ADCONFIG_READ_FAILED** (0x801c001d/-2145648611)
   - Motif : impossible de lire l’objet SCP et d’obtenir les informations concernant le locataire Azure AD.
   - Résolution : consultez la section [Configurer un point de connexion de service](hybrid-azuread-join-federated-domains.md#configure-hybrid-azure-ad-join).
- **DSREG_AUTOJOIN_DISC_FAILED** (0x801c0021/-2145648607)
   - Motif : échec de découverte générique. Échec de l’obtention des métadonnées de découverte auprès de DRS.
   - Résolution : recherchez la sous-erreur ci-dessous pour approfondir vos recherches.
- **DSREG_AUTOJOIN_DISC_WAIT_TIMEOUT**  (0x801c001f/-2145648609)
   - Motif : l’opération est arrivée à expiration pendant la découverte.
   - Résolution : vérifiez que `https://enterpriseregistration.windows.net` est accessible dans le contexte SYSTEM. Pour plus d’informations, consultez la section [Exigences de connectivité réseau](hybrid-azuread-join-managed-domains.md#prerequisites).
- **DSREG_AUTOJOIN_USERREALM_DISCOVERY_FAILED** (0x801c0021/-2145648611)
   - Motif : échec de découverte de domaine générique. Impossible de déterminer le type de domaine (managé/fédéré) à partir de STS.
   - Résolution : recherchez la sous-erreur ci-dessous pour approfondir vos recherches.

**Codes de sous-erreur courants :**

Pour rechercher le code de sous-erreur du code d’erreur de découverte, utilisez l’une des méthodes suivantes.

##### <a name="windows-10-1803-and-above"></a>Windows 10 version 1803 et ultérieures

Recherchez « DRS Discovery Test » dans la section « Diagnostic Data » de la sortie d’état de la jointure. Cette section s’affiche uniquement si l’appareil est joint à un domaine et ne peut pas joindre Azure AD hybride.

```
+----------------------------------------------------------------------+
| Diagnostic Data                                                      |
+----------------------------------------------------------------------+

     Diagnostics Reference : www.microsoft.com/aadjerrors
              User Context : UN-ELEVATED User
               Client Time : 2019-06-05 08:25:29.000 UTC
      AD Connectivity Test : PASS
     AD Configuration Test : PASS
        DRS Discovery Test : FAIL [0x801c0021/0x80072ee2]
     DRS Connectivity Test : SKIPPED
    Token acquisition Test : SKIPPED
     Fallback to Sync-Join : ENABLED

+----------------------------------------------------------------------+
```

##### <a name="older-windows-10-versions"></a>Versions antérieures de Windows 10

Servez-vous des journaux de l’Observateur d’événements pour identifier la phase et le code d’erreur des échecs de jointure.

1. Ouvrez les journaux d’événements **User Device Registration** dans l’Observateur d’événements. Il se trouvent sous **Journaux des applications et des services** > **Microsoft** > **Windows** > **User Device Registration**.
2. Recherchez les événements associés à l’ID d’événement 201.

:::image type="content" source="./media/troubleshoot-hybrid-join-windows-current/5.png" alt-text="Capture d’écran de l’observateur d’événements. Un événement ayant l’ID 201 est sélectionné et ses informations sont affichées avec le code d’erreur en évidence." border="false":::

###### <a name="network-errors"></a>Erreurs réseau

- **WININET_E_CANNOT_CONNECT** (0x80072efd/-2147012867)
   - Motif : la connexion avec le serveur n’a pas pu être établie.
   - Résolution : vérifiez la connectivité réseau aux ressources Microsoft nécessaires. Pour plus d’informations, consultez [Exigences de connectivité réseau](hybrid-azuread-join-managed-domains.md#prerequisites).
- **WININET_E_TIMEOUT** (0x80072ee2/-2147012894)
   - Motif : dépassement du délai d’attente général du réseau.
   - Résolution : vérifiez la connectivité réseau aux ressources Microsoft nécessaires. Pour plus d’informations, consultez [Exigences de connectivité réseau](hybrid-azuread-join-managed-domains.md#prerequisites).
- **WININET_E_DECODING_FAILED** (0x80072f8f/-2147012721)
   - Motif : la pile réseau n’a pas pu décoder la réponse du serveur.
   - Résolution : vérifiez que le proxy réseau n’interfère pas et ne modifie pas la réponse du serveur.

###### <a name="http-errors"></a>Erreurs HTTP

- **DSREG_DISCOVERY_TENANT_NOT_FOUND** (0x801c003a/-2145648582)
   - Motif : objet SCP configuré avec un ID de locataire incorrect. Ou aucun abonnement actif n’a été trouvé dans le locataire.
   - Résolution : vérifiez que l’objet SCP est configuré avec l’ID de locataire Azure AD approprié et des abonnements actifs ou qu’il est présent dans le locataire.
- **DSREG_SERVER_BUSY** (0x801c0025/-2145648603)
   - Motif : erreur HTTP 503 du serveur DRS.
   - Résolution : le serveur est actuellement indisponible. Les futures tentatives de jointure aboutiront probablement dès que le serveur sera de nouveau en ligne.

###### <a name="other-errors"></a>Autres erreurs

- **E_INVALIDDATA** (0x8007000d/-2147024883)
   - Motif : la réponse JSON du serveur n’a pas pu être analysée. La raison probable est une erreur HTTP 200 retournée par le proxy avec une page d’authentification HTML.
   - Résolution : si l’environnement local nécessite un proxy sortant, l’administrateur informatique doit vérifier que le contexte SYSTEM de l’appareil peut découvrir le proxy sortant et s’y authentifier en mode silencieux.

#### <a name="authentication-phase"></a>Phase d’authentification

Applicable uniquement pour les comptes de domaine fédérés.

Causes de l’échec :

- Impossible d’obtenir un jeton d’accès en mode silencieux pour la ressource DRS.
   - Les appareils Windows 10 acquièrent un jeton d’authentification auprès du service de fédération en utilisant l’Authentification Windows intégrée sur un point de terminaison WS-Trust actif. Détails : [Configuration du serveur de fédération](hybrid-azuread-join-manual.md#set-up-issuance-of-claims)

**Codes d’erreur courants :**

Utilisez les journaux de l’observateur d’événements pour rechercher le code d’erreur, le code de sous-erreur, le code d’erreur du serveur et le message d’erreur du serveur.

1. Ouvrez les journaux d’événements **User Device Registration** dans l’Observateur d’événements. Il se trouvent sous **Journaux des applications et des services** > **Microsoft** > **Windows** > **User Device Registration**.
2. Recherchez les événements associés à l’ID d’événement 305.

:::image type="content" source="./media/troubleshoot-hybrid-join-windows-current/3.png" alt-text="Capture d’écran de l’observateur d’événements. Un événement ayant l’ID 305 est visible. Ses informations sont affichées avec les codes d’erreur ADAL et l’état en évidence." border="false":::

##### <a name="configuration-errors"></a>Erreurs de configuration

- **ERROR_ADAL_PROTOCOL_NOT_SUPPORTED** (0xcaa90017/-894894057)
   - Motif : le protocole d’authentification n’est pas WS-Trust.
   - Résolution : le fournisseur d’identité local doit prendre en charge WS-Trust.
- **ERROR_ADAL_FAILED_TO_PARSE_XML** (0xcaa9002c/-894894036)
   - Motif : le service de fédération local n’a pas retourné de réponse XML.
   - Résolution : vérifiez que le point de terminaison MEX retourne un fichier XML valide. Vérifiez que le proxy n’interfère pas et ne renvoie pas de réponses non XML.
- **ERROR_ADAL_COULDNOT_DISCOVER_USERNAME_PASSWORD_ENDPOINT** (0xcaa90023/-894894045)
   - Motif : impossible de découvrir le point de terminaison pour l’authentification par nom d’utilisateur/mot de passe.
   - Résolution : vérifiez les paramètres du fournisseur d’identité local. Vérifiez que les points de terminaison WS-Trust sont activés et que la réponse MEX contient ces points de terminaison appropriés.

##### <a name="network-errors"></a>Erreurs réseau

- **ERROR_ADAL_INTERNET_TIMEOUT** (0xcaa82ee2/-894947614)
   - Motif : dépassement du délai d’attente général du réseau.
   - Résolution : vérifiez que `https://login.microsoftonline.com` est accessible dans le contexte SYSTEM. Vérifiez que le fournisseur d’identité local est accessible dans le contexte SYSTEM. Pour plus d’informations, consultez [Exigences de connectivité réseau](hybrid-azuread-join-managed-domains.md#prerequisites).
- **ERROR_ADAL_INTERNET_CONNECTION_ABORTED** (0xcaa82efe/-894947586)
   - Motif : la connexion avec le point de terminaison d’authentification a été abandonnée.
   - Résolution : réessayez ultérieurement ou essayez de vous joindre à partir d’un autre emplacement réseau stable.
- **ERROR_ADAL_INTERNET_SECURE_FAILURE** (0xcaa82f8f/-894947441)
   - Motif : La sécurité TLS (Transport Layer Security), précédemment connue sous le nom de protocole SSL, n’a pas pu valider le certificat envoyé par le serveur.
   - Résolution : vérifiez l’asymétrie temporelle du client. réessayez ultérieurement ou essayez de vous joindre à partir d’un autre emplacement réseau stable.
- **ERROR_ADAL_INTERNET_CANNOT_CONNECT** (0xcaa82efd/-894947587)
   - Motif : la tentative de connexion à `https://login.microsoftonline.com` a échoué.
   - Résolution : vérifiez la connexion réseau à `https://login.microsoftonline.com`.

##### <a name="other-errors"></a>Autres erreurs

- **ERROR_ADAL_SERVER_ERROR_INVALID_GRANT** (0xcaa20003/-895352829)
   - Motif : le jeton SAML du fournisseur d’identité local n’a pas été accepté par Azure AD.
   - Résolution : vérifiez les paramètres du serveur de fédération. Recherchez le code d’erreur du serveur dans les journaux d’authentification.
- **ERROR_ADAL_WSTRUST_REQUEST_SECURITYTOKEN_FAILED** (0xcaa90014/-894894060)
   - Motif : la réponse WS-Trust du serveur a signalé une exception d’erreur et n’a pas pu obtenir d’assertion.
   - Résolution : vérifiez les paramètres du serveur de fédération. Recherchez le code d’erreur du serveur dans les journaux d’authentification.
- **ERROR_ADAL_WSTRUST_TOKEN_REQUEST_FAIL** (0xcaa90006/-894894074)
   - Motif : une erreur s’est produite lors de la tentative d’obtention du jeton d’accès auprès du point de terminaison de jeton.
   - Résolution : recherchez l’erreur sous-jacente dans le journal ADAL.
- **ERROR_ADAL_OPERATION_PENDING** (0xcaa1002d/-895418323)
   - Motif : échec général d’ADAL
   - Résolution : recherchez le code de sous-erreur ou le code d’erreur du serveur dans les journaux d’authentification.

#### <a name="join-phase"></a>Phase de jointure

Causes de l’échec :

Recherchez le type d’inscription et le code d’erreur dans la liste ci-dessous.

#### <a name="windows-10-1803-and-above"></a>Windows 10 version 1803 et ultérieures

Recherchez la sous-section « Previous Registration » dans la section « Diagnostic Data » de la sortie d’état de la jointure. Cette section s’affiche uniquement si l’appareil est joint à un domaine et ne peut pas joindre Azure AD hybride.
Le champ « Registration Type » indique le type de jointure effectué.

```
+----------------------------------------------------------------------+
     Previous Registration : 2019-01-31 09:16:43.000 UTC
         Registration Type : sync
               Error Phase : join
          Client ErrorCode : 0x801c03f2
          Server ErrorCode : DirectoryError
            Server Message : The device object by the given id (e92325d0-7ac4-4714-88a1-94ae875d5245) is not found.
              Https Status : 400
                Request Id : 6bff0bd9-820b-484b-ab20-2a4f7b76c58e
+----------------------------------------------------------------------+
```

#### <a name="older-windows-10-versions"></a>Versions antérieures de Windows 10

Servez-vous des journaux de l’Observateur d’événements pour identifier la phase et le code d’erreur des échecs de jointure.

1. Ouvrez les journaux d’événements **User Device Registration** dans l’Observateur d’événements. Il se trouvent sous **Journaux des applications et des services** > **Microsoft** > **Windows** > **User Device Registration**.
2. Recherchez les événements associés à l’ID d’événement 204.

:::image type="content" source="./media/troubleshoot-hybrid-join-windows-current/4.png" alt-text="Capture d’écran de l’observateur d’événements. Des informations sur un événement ayant l’ID 204 sont affichées avec le code d’erreur, l’état HTTP et le message en évidence." border="false":::

##### <a name="http-errors-returned-from-drs-server"></a>Erreurs HTTP retournées à partir du serveur DRS

- **DSREG_E_DIRECTORY_FAILURE** (0x801c03f2/-2145647630)
   - Motif : réponse d’erreur reçue de DRS avec le code d’erreur : « DirectoryError »
   - Résolution : recherchez les causes et les résolutions possibles dans le code d’erreur du serveur.
- **DSREG_E_DEVICE_AUTHENTICATION_ERROR** (0x801c0002/-2145648638)
   - Motif : réponse d’erreur reçue de DRS avec le code d’erreur : « AuthenticationError » et ErrorSubCode N’EST PAS « DeviceNotFound ».
   - Résolution : recherchez les causes et les résolutions possibles dans le code d’erreur du serveur.
- **DSREG_E_DEVICE_INTERNALSERVICE_ERROR** (0x801c0006/-2145648634)
   - Motif : réponse d’erreur reçue de DRS avec le code d’erreur : « DirectoryError »
   - Résolution : recherchez les causes et les résolutions possibles dans le code d’erreur du serveur.

##### <a name="tpm-errors"></a>Erreurs du module de plateforme sécurisée (TPM)

- **NTE_BAD_KEYSET** (0x80090016/-2146893802)
   - Motif : l’opération du module TPM a échoué ou n’était pas valide
   - Résolution : erreur probablement due à une image sysprep incorrecte. Vérifiez que la machine à partir de laquelle l’image sysprep a été créée n’est pas jointe à Azure AD, jointe à Azure AD en mode hybride ou inscrite à Azure AD.
- **TPM_E_PCP_INTERNAL_ERROR** (0x80290407/-2144795641)
   - Motif : erreur TPM générique.
   - Résolution : désactivez le module de plateforme sécurisée (TPM) sur les appareils présentant cette erreur. Windows 10 version 1809 et ultérieures détecte automatiquement les échecs du module TPM et effectue la jointure Azure AD hybride sans utiliser le module TPM.
- **TPM_E_NOTFIPS** (0x80280036/-2144862154)
   - Motif : le module TPM en mode FIPS n’est actuellement pas pris en charge.
   - Résolution : désactivez le module de plateforme sécurisée (TPM) sur les appareils présentant cette erreur. Windows 1809 détecte automatiquement les échecs du module TPM et effectue la jointure Azure AD hybride sans utiliser le module TPM.
- **NTE_AUTHENTICATION_IGNORED** (0x80090031/-2146893775)
   - Motif : module TPM verrouillé.
   - Résolution : Erreur temporaire. Attendez la période de temps de recharge. La tentative de jointure devrait aboutir après un certain temps. Vous trouverez des informations supplémentaires dans l’article [Principes de base du module de plateforme sécurisée (TPM)](/windows/security/information-protection/tpm/tpm-fundamentals#anti-hammering).

##### <a name="network-errors"></a>Erreurs réseau

- **WININET_E_TIMEOUT** (0x80072ee2/-2147012894)
   - Motif : dépassement du délai d’attente général du réseau pendant la tentative d’inscription de l’appareil sur DRS.
   - Résolution : vérifiez la connectivité réseau à `https://enterpriseregistration.windows.net`.
- **WININET_E_NAME_NOT_RESOLVED** (0x80072ee7/-2147012889)
   - Motif : le nom ou l’adresse du serveur n’a pas pu être résolue.
   - Résolution : vérifiez la connectivité réseau à `https://enterpriseregistration.windows.net`. Vérifiez que la résolution DNS du nom d’hôte est précise dans le réseau et sur l’appareil.
- **WININET_E_CONNECTION_ABORTED** (0x80072efe/-2147012866)
   - Motif : la connexion avec le serveur s’est terminée anormalement.
   - Résolution : réessayez ultérieurement ou essayez de vous joindre à partir d’un autre emplacement réseau stable.

##### <a name="federated-join-server-errors"></a>Erreurs du serveur – Jointure fédérée

| Code d’erreur du serveur | Message d’erreur du serveur | Causes possibles | Résolution |
| --- | --- | --- | --- |
| DirectoryError | Votre requête est limitée temporairement. Réessayez après 300 secondes. | Erreur attendue. Cette erreur peut être due à plusieurs demandes d’inscription successives dans un court laps de temps. | Tentez à nouveau la jointure après la période de temps de recharge. |

##### <a name="sync-join-server-errors"></a>Erreurs du serveur – Jointure de synchronisation

| Code d’erreur du serveur | Message d’erreur du serveur | Causes possibles | Résolution |
| --- | --- | --- | --- |
| DirectoryError | AADSTS90002 : Locataire <UUID> introuvable. Cette erreur peut se produire s’il n’existe pas d’abonnement actif pour le locataire. Vérifiez avec l’administrateur de votre abonnement. | L’ID de locataire de l’objet SCP est incorrect. | Vérifiez que l’objet SCP est configuré avec l’ID de locataire Azure AD approprié et des abonnements actifs et qu’il est présent dans le locataire. |
| DirectoryError | L’objet d’appareil correspondant à l’ID donné est introuvable. | Erreur attendue pour la jointure de synchronisation. L’objet d’appareil n’a pas été synchronisé d’AD vers Azure AD. | Attendez la fin de la synchronisation Azure AD Connect et la prochaine tentative de jointure postérieure à la synchronisation résoudra le problème. |
| AuthenticationError | Vérification du SID de l’ordinateur cible | Le certificat sur l’appareil Azure AD ne correspond pas au certificat utilisé pour signer l’objet blob pendant la jointure de synchronisation. Cette erreur signifie généralement que la synchronisation n’est pas encore terminée. |  Attendez la fin de la synchronisation Azure AD Connect et la prochaine tentative de jointure postérieure à la synchronisation résoudra le problème. |

### <a name="step-5-collect-logs-and-contact-microsoft-support"></a>Étape 5 : Récupérer les journaux et contacter le Support Microsoft

Téléchargez le fichier Auth.zip à partir de [https://github.com/CSS-Windows/WindowsDiag/tree/master/ADS/AUTH](https://github.com/CSS-Windows/WindowsDiag/tree/master/ADS/AUTH)

1. Décompressez les fichiers et renommez les fichiers inclus **start-auth.txt** et **stop-auth.txt** en **start-auth.cmd** et **stop-auth.cmd**.
1. À partir d’une invite de commandes avec élévation de privilèges, exécutez la commande **start-auth.cmd**.
1. Utilisez la commande Changer de compte pour basculer vers une autre session avec l’utilisateur confronté au problème.
1. Reproduisez le problème.
1. Utilisez la commande pour rebasculer vers la session d’administration exécutant le suivi.
1. À partir d’une invite de commandes avec élévation de privilèges, exécutez la commande **stop-auth.cmd**.
1. Compressez et envoyez le dossier **Authlogs** à partir du dossier dans lequel les scripts ont été exécutés.

## <a name="troubleshoot-post-join-issues"></a>Résoudre les problèmes postérieurs à la jointure

### <a name="retrieve-the-join-status"></a>Récupérer l’état de jonction

#### <a name="wamdefaultset-yes-and-azureadprt-yes"></a>WamDefaultSet : OUI et AzureADPrt : YES

Ces champs indiquent que l’utilisateur s’est correctement authentifié auprès d’Azure AD lors de la connexion à l’appareil.
Si les valeurs sont **NON** , il peut être dû :

- Clé de stockage défectueuse dans le module de plateforme sécurisée (TPM) associé à l’appareil lors de l’inscription (vérifiez KeySignTest pendant l’exécution avec élévation de privilèges).
- ID de connexion de substitution
- Proxy HTTP introuvable

## <a name="known-issues"></a>Problèmes connus
- Sous Paramètres -> Comptes -> Accès professionnel ou scolaire, les appareils joints à Azure AD Hybride peuvent afficher deux comptes différents, l’un pour Azure AD et l’autre pour AD local, lorsqu’ils sont connectés à des points d’accès mobiles ou à des réseaux Wi-Fi externes. Il s’agit uniquement d’un problème d’interface utilisateur qui n’a aucun impact sur les fonctionnalités.

## <a name="next-steps"></a>Étapes suivantes

Poursuivez le [dépannage des appareils à l’aide de la commande dsregcmd](troubleshoot-device-dsregcmd.md).

Pour toute question, consultez [FAQ sur la gestion des appareils](faq.md)