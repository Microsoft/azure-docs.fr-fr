---
title: Configurer l’extension NPS pour Azure AD MFA - Azure Active Directory
description: Après avoir installé l’extension NPS, procédez comme suit pour la configuration avancée (par exemple, liste des adresses IP approuvées et remplacement de l’UPN).
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/11/2018
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6d436414393d77c83acc835110f17e55e491dce1
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97503487"
---
# <a name="advanced-configuration-options-for-the-nps-extension-for-multi-factor-authentication"></a>Options de configuration avancée de l’extension de serveur NPS pour l’authentification multifacteur

L’extension de serveur NPS (Network Policy Server) étend vos fonctionnalités Azure AD Multi-Factor Authentication basées sur le cloud à votre infrastructure locale. Cet article suppose que l’extension est déjà installée et que vous souhaitez maintenant savoir comment personnaliser l’extension en fonction de vos propres besoins.

> [!NOTE]
> Cet article contient des références au terme *liste verte*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

## <a name="alternate-login-id"></a>ID de connexion de substitution

Étant donné que l’extension de serveur NPS se connecte à vos répertoires locaux et cloud, vous pouvez rencontrer un problème de correspondance entre les noms d’utilisateur principal (UPN) locaux et ceux du cloud. Pour résoudre ce problème, utilisez des ID de connexion de substitution. 

Dans l’extension de serveur NPS, vous pouvez désigner un attribut Active Directory à utiliser à la place de l’UPN pour Azure AD Multi-Factor Authentication. Cela vous permet de protéger vos ressources locales avec la vérification en deux étapes sans modifier vos UPN locaux. 

Pour configurer un ID de connexion de substitution, accédez à `HKLM\SOFTWARE\Microsoft\AzureMfa` et modifiez les valeurs de registre suivantes :

| Nom | Type | Valeur par défaut | Description |
| ---- | ---- | ------------- | ----------- |
| LDAP_ALTERNATE_LOGINID_ATTRIBUTE | string | Vide | Spécifiez le nom de l’attribut Active Directory que vous souhaitez utiliser au lieu de l’UPN. Cet attribut est utilisé en tant qu’attribut AlternateLoginId. Si cette valeur de registre est définie sur un [attribut Active Directory valide](/windows/win32/adschema/attributes-all) (par exemple, mail ou displayName), la valeur de l’attribut est utilisée à la place de l’UPN de l’utilisateur pour l’authentification. Si cette valeur de registre est vide ou n’est pas configurée, AlternateLoginId est désactivé et l’UPN de l’utilisateur est utilisé pour l’authentification. |
| LDAP_FORCE_GLOBAL_CATALOG | boolean | False | Utilisez cet indicateur pour forcer l’utilisation du catalogue global dans le cadre des recherches LDAP d’AlternateLoginId. Configurez un contrôleur de domaine en tant que catalogue global, ajoutez l’attribut AlternateLoginId dans le catalogue global, puis activez cet indicateur. <br><br> Si LDAP_LOOKUP_FORESTS est configuré (n’est pas vide), **cet indicateur est appliqué comme true**, quelle que soit la valeur du paramètre du registre. Dans ce cas, l’extension de serveur NPS nécessite la configuration du catalogue global avec l’attribut AlternateLoginId pour chaque forêt. |
| LDAP_LOOKUP_FORESTS | string | Vide | Fournissez une liste séparée par des points-virgules pour les forêts à rechercher. Par exemple, *contoso.com;foobar.com*. Si cette valeur de registre est configurée, l’extension NPS effectue une recherche itérative sur toutes les forêts dans l’ordre dans lequel elles ont été répertoriées, et elle retourne la première valeur AlternateLoginId correcte. Si cette valeur de registre n’est pas configurée, la recherche AlternateLoginId est limitée au domaine actuel.|

Pour résoudre les problèmes liés à des ID de connexion de substitution, utilisez les étapes recommandées dans [Erreurs d’ID de connexion de substitution](howto-mfa-nps-extension-errors.md#alternate-login-id-errors).

## <a name="ip-exceptions"></a>Exceptions liées aux adresses IP

Si vous devez surveiller la disponibilité du serveur, par exemple si les équilibreurs de charge vérifient quels serveurs sont en cours d’exécution avant l’envoi de charges de travail, vous ne souhaitez pas que ces contrôles soient bloqués par des requêtes de vérification. Au lieu de cela, créez une liste d’adresses IP utilisées par les comptes de service, puis désactivez les exigences de l’authentification multifacteur pour cette liste.

Pour configurer une liste autorisée d’adresses IP, accédez à `HKLM\SOFTWARE\Microsoft\AzureMfa` et configurez la valeur de registre suivante :

| Nom | Type | Valeur par défaut | Description |
| ---- | ---- | ------------- | ----------- |
| IP_WHITELIST | string | Vide | Fournissez une liste séparée par des points-virgules pour les adresses IP. Incluez les adresses IP des machines d’origine des requêtes de service, telles que le serveur NAS/VPN. Les plages d’adresses IP et les sous-réseaux ne sont pas pris en charge. <br><br> Par exemple, *10.0.0.1;10.0.0.2;10.0.0.3*.

> [!NOTE]
> Cette clé de Registre n’est pas créée par défaut par le programme d’installation et une erreur s’affiche dans le journal AuthZOptCh lorsque le service est redémarré. Cette erreur dans le journal peut être ignorée, mais si cette clé de Registre est créée et laissée vide lorsqu’elle n’est pas nécessaire, le message d’erreur ne retourne pas.

Lorsqu’une requête provient d’une adresse IP de la `IP_WHITELIST`, la vérification en deux étapes est ignorée. La liste d’adresses IP est comparée à l’adresse IP fournie dans l’attribut *ratNASIPAddress* de la requête RADIUS. Si une requête RADIUS est fournie sans l’attribut ratNASIPAddress, l’avertissement suivant est enregistré : « P_WHITE_LIST_WARNING::IP Whitelist is being ignored as source IP is missing in RADIUS request in NasIpAddress attribute. » (P_WHITE_LIST_WARNING::IP Whitelist est ignoré, car l’adresse IP source est manquante dans la requête RADIUS de l’attribut NasIpAddress.)

## <a name="next-steps"></a>Étapes suivantes

[Résoudre les messages d'erreur liés à l'extension NPS d'Azure AD Multi-Factor Authentication](howto-mfa-nps-extension-errors.md)