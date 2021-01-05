---
title: Problèmes d’authentification dans Azure HDInsight
description: Problèmes d’authentification dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/24/2020
ms.openlocfilehash: 9f10597023314aed8640ee5a7499a77f952c3694
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93284440"
---
# <a name="authentication-issues-in-azure-hdinsight"></a>Problèmes d’authentification dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

Sur les clusters sécurisés sauvegardés par Azure Data Lake (Gen1 ou Gen2), lorsque les utilisateurs de domaine se connectent aux services de cluster via HDI Gateway (comme la connexion au portail Apache Ambari), HDI Gateway tente d’abord d’obtenir un jeton OAuth d’Azure Active Directory (Azure AD), avant d’obtenir un ticket Kerberos d’Azure AD DS. L’authentification peut échouer dans l’une de ces étapes. Cet article vise à déboguer certains de ces problèmes.

Lorsque l’authentification échoue, vous êtes invité à entrer vos informations d’identification. Si vous annulez cette boîte de dialogue, le message d’erreur est imprimé. Voici quelques messages d’erreur courants :

## <a name="invalid_grant-or-unauthorized_client-50126"></a>invalid_grant ou unauthorized_client, 50126

### <a name="issue"></a>Problème

La connexion échoue pour les utilisateurs fédérés avec le code d’erreur 50126 (la connexion réussie pour les utilisateurs Cloud). Le message d’erreur ressemble à :

```
Reason: Bad Request, Detailed Response: {"error":"invalid_grant","error_description":"AADSTS70002: Error validating credentials. AADSTS50126: Invalid username or password\r\nTrace ID: 09cc9b95-4354-46b7-91f1-efd92665ae00\r\n Correlation ID: 4209bedf-f195-4486-b486-95a15b70fbe4\r\nTimestamp: 2019-01-28 17:49:58Z","error_codes":[70002,50126], "timestamp":"2019-01-28 17:49:58Z","trace_id":"09cc9b95-4354-46b7-91f1-efd92665ae00","correlation_id":"4209bedf-f195-4486-b486-95a15b70fbe4"}
```

### <a name="cause"></a>Cause :

Le code d’erreur 50126 d’Azure AD signifie que la stratégie `AllowCloudPasswordValidation` n’a pas été définie par le locataire.

### <a name="resolution"></a>Résolution

L’administrateur de la société du locataire Azure AD doit autoriser Azure AD à utiliser des hachages de mot de passe pour les utilisateurs d’ADFS.  Appliquez la stratégie `AllowCloudPasswordValidationPolicy` comme montré dans l’article [Utiliser le pack Sécurité Entreprise dans HDInsight](../domain-joined/apache-domain-joined-architecture.md).

---

## <a name="invalid_grant-or-unauthorized_client-50034"></a>invalid_grant ou unauthorized_client, 50034

### <a name="issue"></a>Problème

La connexion échoue avec le code d’erreur 50034. Le message d’erreur ressemble à :

```
{"error":"invalid_grant","error_description":"AADSTS50034: The user account Microsoft.AzureAD.Telemetry.Diagnostics.PII does not exist in the 0c349e3f-1ac3-4610-8599-9db831cbaf62 directory. To sign into this application, the account must be added to the directory.\r\nTrace ID: bbb819b2-4c6f-4745-854d-0b72006d6800\r\nCorrelation ID: b009c737-ee52-43b2-83fd-706061a72b41\r\nTimestamp: 2019-04-29 15:52:16Z", "error_codes":[50034],"timestamp":"2019-04-29 15:52:16Z","trace_id":"bbb819b2-4c6f-4745-854d-0b72006d6800", "correlation_id":"b009c737-ee52-43b2-83fd-706061a72b41"}
```

### <a name="cause"></a>Cause :

Le nom d’utilisateur est incorrect (il n’existe pas). L’utilisateur n’utilise pas le même nom d’utilisateur que celui utilisé dans le Portail Azure.

### <a name="resolution"></a>Résolution

Utilisez le nom d’utilisateur qui fonctionne dans ce portail.

---

## <a name="invalid_grant-or-unauthorized_client-50053"></a>invalid_grant ou unauthorized_client, 50053

### <a name="issue"></a>Problème

Le compte d’utilisateur est verrouillé, code d’erreur 50053. Le message d’erreur ressemble à :

```
{"error":"unauthorized_client","error_description":"AADSTS50053: You've tried to sign in too many times with an incorrect user ID or password.\r\nTrace ID: 844ac5d8-8160-4dee-90ce-6d8c9443d400\r\nCorrelation ID: 23fe8867-0e8f-4e56-8764-0cdc7c61c325\r\nTimestamp: 2019-06-06 09:47:23Z","error_codes":[50053],"timestamp":"2019-06-06 09:47:23Z","trace_id":"844ac5d8-8160-4dee-90ce-6d8c9443d400","correlation_id":"23fe8867-0e8f-4e56-8764-0cdc7c61c325"}
```

### <a name="cause"></a>Cause :

Un trop grand nombre de tentatives de connexion avec un mot de passe incorrect.

### <a name="resolution"></a>Résolution

Patientez 30 minutes, arrêtez toutes les applications susceptibles de tenter de s’authentifier.

---

## <a name="invalid_grant-or-unauthorized_client-50053-2"></a>invalid_grant ou unauthorized_client, 50053 (#2)

### <a name="issue"></a>Problème

Le mot de passe a expiré, code d’erreur 50053. Le message d’erreur ressemble à :

```
{"error":"user_password_expired","error_description":"AADSTS50055: Password is expired.\r\nTrace ID: 241a7a47-e59f-42d8-9263-fbb7c1d51e00\r\nCorrelation ID: c7fe4a42-67e4-4acd-9fb6-f4fb6db76d6a\r\nTimestamp: 2019-06-06 17:29:37Z","error_codes":[50055],"timestamp":"2019-06-06 17:29:37Z","trace_id":"241a7a47-e59f-42d8-9263-fbb7c1d51e00","correlation_id":"c7fe4a42-67e4-4acd-9fb6-f4fb6db76d6a","suberror":"user_password_expired","password_change_url":"https://portal.microsoftonline.com/ChangePassword.aspx"}
```

### <a name="cause"></a>Cause :

Le mot de passe a expiré.

### <a name="resolution"></a>Résolution

Modifiez le mot de passe dans le Portail Azure (sur votre système local), puis patientez 30 minutes que la synchronisation reprenne.

---

## <a name="interaction_required"></a>interaction_required

### <a name="issue"></a>Problème

Réception du message d’erreur `interaction_required`.

### <a name="cause"></a>Cause :

La stratégie d’accès conditionnel ou MFA est appliquée à l’utilisateur. Étant donné que l’authentification interactive n’est pas encore prise en charge, l’utilisateur ou le cluster doit être exempté de l’accès conditionnel/MFA. Si vous choisissez d’exempter le cluster (stratégie d’exemption basée sur l’adresse IP), assurez-vous que les `ServiceEndpoints` AD sont activés pour ce réseau virtuel.

### <a name="resolution"></a>Résolution

Utilisez la stratégie d’accès conditionnel et exemptez les clusters HDInisght de l'authentification MFA comme indiqué dans [Configurer un cluster HDInsight avec le pack Sécurité Entreprise à l’aide des services Azure Active Directory Domain Services](./apache-domain-joined-configure-using-azure-adds.md).

---

## <a name="sign-in-denied"></a>Connexion refusée

### <a name="issue"></a>Problème

La connexion est refusée.

### <a name="cause"></a>Cause :

Pour accéder à cette phase, c’est l’authentification Kerberos qui pose problème, pas l’authentification OAuth. Si ce cluster est sauvegardé par ADLS, la connexion OAuth a réussi avant la tentative d’authentification Kerberos. Sur les clusters WASB, la connexion OAuth n’est pas tentée. Il y a de nombreuses raisons d’échec pour l’authentification Kerberos, comme les hachages de mot de passe non synchonisés, les comptes d’utilisateur bloqués dans Azure AD DS, etc. La synchronisation des hachages de mot de passe se fait uniquement lorsque l’utilisateur modifie le mot de passe. Lorsque vous créez l’instance Azure AD DS, elle commence à synchroniser les mots de passe qui sont modifiés après la création. Elle ne synchronise pas rétroactivement les mots de passe qui ont été définis avant son commencement.

### <a name="resolution"></a>Résolution

Si vous pensez que les mots de passe ne sont pas synchronisés, essayez de modifier le mot de passe et attendez quelques minutes pour la synchronisation.

Essayez de vous connecter par SSH. Vous devez essayer de vous authentifier (kinit) via les mêmes informations d'identification, depuis une machine jointe au domaine. Connectez-vous par SSH dans le nœud de périphérie / principal avec un utilisateur local et exécutez kinit.

---

## <a name="kinit-fails"></a>échec de kinit

### <a name="issue"></a>Problème

Échec de kinit.

### <a name="cause"></a>Cause :

Varie.

### <a name="resolution"></a>Résolution

Pour que kinit réussisse, vous devez connaître votre `sAMAccountName` (il s’agit du nom de compte court sans le domaine). `sAMAccountName` constitue généralement le préfixe du compte (comme bob dans `bob@contoso.com`). Pour certains utilisateurs, il peut être différent. Vous devez avoir la possibilité de parcourir/rechercher le répertoire pour connaître votre `sAMAccountName`.

Moyens de rechercher `sAMAccountName` :

* Si vous pouvez vous connecter à Ambari à l’aide de l’administrateur Ambari local, examinez la liste des utilisateurs.

* Si vous disposez d'un [ordinateur Windows joint à un domaine](../../active-directory-domain-services/tutorial-create-management-vm.md), vous pouvez utiliser les outils Windows AD standard pour parcourir. Cela nécessite un compte opérationnel dans le domaine.

* À partir du nœud principal, vous pouvez utiliser des commandes SAMBA pour effectuer une recherche. Cela nécessite une session Kerberos valide (kinit réussi). net ads search "(userPrincipalName=bob*)"

    Les résultats de la recherche/navigation doivent vous indiquer l’attribut `sAMAccountName`. En outre, vous pouvez examiner d’autres attributs tels que `pwdLastSet`, `badPasswordTime`, `userPrincipalName`, etc., pour voir si ces propriétés correspondent à ce que vous attendez.

---

## <a name="kinit-fails-with-preauthentication-failure"></a>kinit échoue avec échec de préauthentification

### <a name="issue"></a>Problème

kinit échoue avec échec `Preauthentication`.

### <a name="cause"></a>Cause :

Nom d’utilisateur ou mot de passe incorrect.

### <a name="resolution"></a>Résolution

Vérifiez votre nom d’utilisateur et votre mot de passe. Vérifiez également les autres propriétés décrites ci-dessus. Pour activer le débogage détaillé, exécutez `export KRB5_TRACE=/tmp/krb.log` à partir de la session avant d’essayer kinit.

---

## <a name="job--hdfs-command-fails-due-to-tokennotfoundexception"></a>La commande Job/HDFS échoue à cause de TokenNotFoundException

### <a name="issue"></a>Problème

La commande Job/HDFS échoue à cause de `TokenNotFoundException`.

### <a name="cause"></a>Cause :

Le jeton d’accès OAuth requis est introuvable pour que le travail/la commande aboutisse. Le pilote ADLS/ABFS essaiera de récupérer le jeton d’accès OAuth auprès du service d’informations d’identification avant d’effectuer des demandes de stockage. Ce jeton est inscrit lorsque vous vous connectez au portail Ambari à l’aide du même utilisateur.

### <a name="resolution"></a>Résolution

Vérifiez que vous êtes connecté au portail Ambari une seule fois via le nom d’utilisateur dont l’identité est utilisée pour exécuter le travail.

---

## <a name="error-fetching-access-token"></a>Erreur lors de la récupération du jeton d’accès

### <a name="issue"></a>Problème

L’utilisateur reçoit un message d’erreur `Error fetching access token`.

### <a name="cause"></a>Cause :

Cette erreur se produit par intermittence lorsque les utilisateurs essaient d’accéder au ADLS Gen2 à l’aide de listes de contrôle d’accès et que le jeton Kerberos a expiré.

### <a name="resolution"></a>Résolution

* Pour Azure Data Lake Storage Gen1, nettoyez le cache du navigateur et reconnectez-vous à Ambari.

* Par Azure Data Lake Storage Gen2, exécutez `/usr/lib/hdinsight-common/scripts/RegisterKerbTicketAndOAuth.sh <upn>` pour l’utilisateur auquel l’utilisateur tente de se connecter.

---

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]