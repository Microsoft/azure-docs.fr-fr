---
title: Méthode d’authentification de jetons OATH – Azure Active Directory
description: Apprenez-en davantage sur l’utilisation des jetons OATH dans Azure Active Directory pour contribuer à l’amélioration et à la sécurisation des événements de connexion
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: d04075b415bace4104a58e8221d764355d3318d0
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744276"
---
# <a name="authentication-methods-in-azure-active-directory---oath-tokens"></a>Méthodes d’authentification dans Azure Active Directory – Jetons OATH

Les mots de passe à usage unique et durée définie (TOTP) OATH forment une norme ouverte qui spécifie le mode de génération des codes de mot de passe (OTP) à usage unique. Les mots de passe à usage unique et durée définie OATH peuvent être implémentés à l’aide de logiciels ou de matériels permettant de générer des codes. Azure AD ne prend pas en charge les HOTP OATH, une norme de génération de code différente.

## <a name="oath-software-tokens"></a>Jetons logiciels OATH

Les jetons logiciels OATH sont généralement des applications telles que l’application Microsoft Authenticator et d’autres applications d’authentification. Azure AD génère la clé secrète, ou la valeur de départ, qui est entrée dans l’application et utilisée pour générer chaque code OTP.

L’application Authenticator génère automatiquement des codes lorsqu’elle est configurée pour effectuer des notifications push, afin que l’utilisateur dispose d’une sauvegarde, même si son appareil n’a pas de connectivité. Les applications tierces qui utilisent les TOTP OATH pour générer des codes peuvent également être utilisées.

Certains jetons matériels TOTP OATH sont programmables, ce qui signifie qu’ils ne sont pas fournis avec une clé secrète ou une valeur de départ préprogrammée. Ces jetons matériels programmables peuvent être configurés à l’aide de la clé secrète ou de la valeur initiale obtenue à partir du flux d’installation du jeton logiciel. Les clients peuvent acheter ces jetons auprès du fournisseur de leur choix et utiliser la clé secrète ou la valeur initiale dans le processus d’installation de leur fournisseur.

## <a name="oath-hardware-tokens-preview"></a>Jetons matériels OATH (version préliminaire)

Azure AD prend en charge l’utilisation de jetons OATH-TOTP SHA-1 qui actualisent les codes toutes les 30 ou 60 secondes. Les clients peuvent acheter ces jetons auprès du fournisseur de leur choix.

Les jetons matériels OATH TOTP sont généralement fournis avec une clé secrète, ou valeur initiale, préprogrammée dans le jeton. Ces clés doivent être entrées dans Azure AD comme décrit dans les étapes suivantes. Les clés secrètes sont limitées à 128 caractères et cette limite peut ne pas être compatible avec tous les jetons. La clé secrète peut contenir uniquement les caractères *a à z* ou *A à Z* et les chiffres *1 à 7*, et doit être encodée en *Base32*.

Vous pouvez également configurer des jetons matériels OATH TOTP programmables qui peuvent être réamorcés avec Azure AD dans le processus d’installation des jetons logiciels.

Les jetons matériels OATH sont pris en charge dans le cadre d’une préversion publique. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

![Chargement des jetons OATH dans le panneau de jetons OATH MFA](media/concept-authentication-methods/mfa-server-oath-tokens-azure-ad.png)

Après avoir obtenu les jetons, vous devez les charger dans un fichier de valeurs séparées par des virgules (CSV), contenant l'UPN, le numéro de série, la clé secrète, l’intervalle de temps, le fabricant et le modèle, comme indiqué dans l’exemple suivant :

```csv
upn,serial number,secret key,time interval,manufacturer,model
Helga@contoso.com,1234567,1234567abcdef1234567abcdef,60,Contoso,HardwareKey
```

> [!NOTE]
> Veillez à inclure la ligne d’en-tête dans votre fichier CSV.

Une fois que le fichier a été correctement mis en forme au format CSV, l’administrateur peut ensuite se connecter au Portail Azure et accéder à **Azure Active Directory > Sécurité > MFA > Jetons OATH** afin de charger le fichier CSV créé.

L’opération peut prendre plusieurs minutes selon la taille du fichier CSV. Sélectionnez le bouton **Actualiser** pour obtenir l’état actuel. Si le fichier contient des erreurs, vous pouvez télécharger un fichier CSV qui répertorie les erreurs à résoudre. Les noms de champs dans le fichier CSV téléchargé sont différents de ceux de la version chargée.

Une fois que toutes les erreurs ont été résolues, l’administrateur peut activer chaque clé en sélectionnant **Activer** pour le jeton et en entrant l’OTP affiché sur le jeton.

Les utilisateurs peuvent combiner jusqu’à cinq jetons matériels OATH ou des applications d’authentification, comme l’application Microsoft Authenticator, configurées pour une utilisation à tout moment.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en plus sur la configuration des méthodes d’authentification à l’aide de l’[API REST Microsoft Graph bêta](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta).
