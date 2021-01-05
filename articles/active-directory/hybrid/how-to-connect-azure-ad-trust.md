---
title: Azure AD Connect - Gérer l’approbation AD FS avec Azure AD à l’aide d’Azure AD Connect | Microsoft Docs
description: Détails opérationnels sur la gestion des approbations Azure AD par Azure AD Connect.
keywords: AD FS, ADFS, gestion AD FS, AAD Connect, Connect, Azure AD, approbation, AAD, revendication, règles de revendication, émission, transformation, règles, sauvegarde, restauration
services: active-directory
documentationcenter: ''
ms.reviewer: anandyadavmsft
manager: daveba
ms.subservice: hybrid
ms.assetid: 2593b6c6-dc3f-46ef-8e02-a8e2dc4e9fb9
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/28/2018
ms.author: billmath
author: billmath
ms.custom: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 13d56ec321cd257412c2b0abbe0be655c6cb4dbf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85360093"
---
# <a name="manage-ad-fs-trust-with-azure-ad-using-azure-ad-connect"></a>Gérer l’approbation AD FS avec Azure AD à l’aide d’Azure AD Connect

## <a name="overview"></a>Vue d’ensemble

Azure AD Connect peut gérer la fédération entre le service de fédération Active Directory (AD FS) local et Azure AD. Cet article fournit une vue d’ensemble des éléments suivants :

* divers paramètres configurés sur l’approbation par Azure AD Connect ;
* règles de transformation d’émission (règles de revendication) définies par Azure AD Connect ;
* procédures de sauvegarde et de restauration des règles de revendication entre les mises à niveau et les mises à jour de configuration. 

## <a name="settings-controlled-by-azure-ad-connect"></a>Paramètres contrôlés par Azure AD Connect

Azure AD Connect gère **uniquement** les paramètres associés à l’approbation Azure AD. Azure AD Connect ne modifie aucun paramètre d’autres approbations de partie de confiance dans AD FS. Le tableau ci-après indique les paramètres qui sont contrôlés par Azure AD Connect.

| Paramètre | Description |
| :--- | :--- |
| Certificat de signature de jetons | Vous pouvez utiliser Azure AD Connect pour réinitialiser et recréer l’approbation avec Azure AD. Azure AD Connect effectue une substitution immédiate à usage unique des certificats de signature de jetons pour AD FS et met à jour les paramètres de fédération de domaine Azure AD.|
| Token signing algorithm (Algorithme de signature de jetons) | Microsoft recommande d’utiliser l’algorithme de signature de jetons SHA-256. Azure AD Connect peut détecter si l’algorithme de signature de jetons est défini sur une valeur moins sécurisée que SHA-256. Si tel est le cas, il redéfinit le paramètre sur SHA-256 au cours de la prochaine opération de configuration possible. L’autre approbation de partie de confiance doit être mise à jour pour utiliser le nouveau certificat de signature de jetons. |
| Azure AD trust identifier (Identificateur d’approbation Azure AD) | Azure AD Connect définit la valeur d’identificateur correcte pour l’approbation Azure AD. AD FS identifie de manière unique l’approbation Azure AD à l’aide de la valeur d’identificateur. |
| Points de terminaison Azure AD | Azure AD Connect vérifie que les points de terminaison configurés pour l’approbation Azure AD sont toujours conformes aux dernières valeurs recommandées en matière de résilience et de performances. |
| Règles de transformation d’émission | Plusieurs règles de revendication sont nécessaires pour optimiser les performances des fonctionnalités d’Azure AD dans un environnement fédéré. Azure AD Connect vérifie que l’approbation Azure AD est toujours configurée avec l’ensemble adéquat de règles de revendication recommandées. |
| Alternate-id (Alternate-id) | Si la synchronisation est configurée pour utiliser le paramètre Alternate-id, Azure AD Connect configure AD FS pour qu’il effectue l’authentification à l’aide de ce paramètre. |
| Automatic metadata update (Mise à jour automatique des métadonnées) | L’approbation avec Azure AD est configurée pour la mise à jour automatique des métadonnées. AD FS vérifie régulièrement les métadonnées de l’approbation Azure AD et les actualise si elles sont modifiées côté Azure AD. |
| Authentification Windows intégrée (IWA) | Au cours de l’opération de jointure Azure AD Hybride, l’authentification Windows intégrée est activée pour l’inscription d’appareils afin de faciliter la jointure Azure AD Hybride pour les appareils de bas niveau. |

## <a name="execution-flows-and-federation-settings-configured-by-azure-ad-connect"></a>Flux d’exécution et paramètres de fédération configurés par Azure AD Connect

Azure AD Connect ne met pas à jour la totalité des paramètres de l’approbation Azure AD lors des flux de configuration. Les paramètres modifiés varient selon le flux de tâches ou d’exécution en cours d’exécution. Le tableau ci-après répertorie les paramètres affectés dans les différents flux d’exécution.

| Flux d’exécution | Paramètres affectés |
| :--- | :--- |
| Installation au premier passage (rapide) | None |
| Installation au premier passage (nouvelle batterie de serveurs AD FS) | Une nouvelle batterie de serveurs AD FS est créée, et une approbation avec Azure AD est créée à partir de zéro. |
| Installation au premier passage (batterie de serveurs AD FS existante, approbation Azure AD existante) | Azure AD trust identifier (Identificateur d’approbation Azure AD), Règles de transformation d’émission, Points de terminaison Azure AD, Alternate-id (Alternate-id) (si nécessaire), Automatic metadata update (Mise à jour automatique des métadonnées) |
| Réinitialiser l’approbation Azure AD | Certificat de signature de jetons, Token signing algorithm (Algorithme de signature de jetons), Azure AD trust identifier (Identificateur d’approbation Azure AD), Règles de transformation d’émission, Points de terminaison Azure AD, Alternate-id (Alternate-id) (si nécessaire), Automatic metadata update (Mise à jour automatique des métadonnées) |
| Ajouter un serveur de fédération | None |
| Ajouter un serveur WAP | None |
| Options de l’appareil | Règles de transformation d’émission, Authentification Windows intégrée pour l’inscription d’appareils |
| Ajouter un domaine fédéré | Si le domaine est ajouté pour la première fois, autrement dit, si la configuration évolue d’une fédération de domaine unique vers une fédération multidomaine, Azure AD Connect recrée l’approbation à partir de zéro. Si l’approbation avec Azure AD est déjà configurée pour plusieurs domaines, seules les règles de transformation d’émission sont modifiées. |
| Mettre à jour TLS | None |

Dans le cadre de toutes les opérations au cours desquelles un paramètre est modifié, Azure AD Connect sauvegarde les paramètres d’approbation actuels à l’emplacement **%ProgramData%\AADConnect\ADFS**.

![Page Azure AD Connect affichant un message concernant une sauvegarde d’approbation Azure AD existante](./media/how-to-connect-azure-ad-trust/backup2.png)

> [!NOTE]
> Avant la version 1.1.873.0, la sauvegarde comprenait uniquement les règles de transformation d’émission et s’effectuait dans le fichier journal de trace d’Assistant.

## <a name="issuance-transform-rules-set-by-azure-ad-connect"></a>Règles de transformation d’émission définies par Azure AD Connect

Azure AD Connect vérifie que l’approbation Azure AD est toujours configurée avec l’ensemble adéquat de règles de revendication recommandées. Microsoft vous recommande d’utiliser Azure AD Connect pour la gestion de votre approbation Azure AD. Cette section répertorie les règles de transformation d’émission définies et en fournit une description.

| Nom de la règle | Description |
| --- | --- |
| Issue UPN (Émettre UPN) | Cette règle interroge la valeur de l’élément userprincipalname à partir de l’attribut configuré dans les paramètres de synchronisation pour userprincipalname.|
| Query objectguid and msdsconsistencyguid for custom ImmutableId claim (Interroger objectguid et msdsconsistencyguid pour la revendication ImmutableId personnalisée) | Cette règle ajoute une valeur temporaire dans le pipeline pour la valeur objectguid et la valeur msdsconsistencyguid si elle existe. |
| Check for the existence of msdsconsistencyguid (Vérifier l’existence de msdsconsistencyguid) | Selon que la valeur de msdsconsistencyguid existe ou non, nous définissons un indicateur temporaire pour spécifier l’élément à utiliser comme ImmutableId. |
| Issue msdsconsistencyguid as Immutable ID if it exists (Émettre msdsconsistencyguid comme ID non modifiable s’il existe) | Cette règle émet msdsconsistencyguid en tant qu’ID non modifiable s’il existe. |
| Issue objectGuidRule if msdsConsistencyGuid rule does not exist (Émettre objectGuidRule si la règle msdsConsistencyGuid n’existe pas) | Si la valeur de msdsconsistencyguid n’existe pas, la valeur d’objectguid sera émise en tant qu’ImmutableId. |
| Issue nameidentifier (Émettre nameidentifier) | Cette règle émet la valeur de la revendication nameidentifier.|
| Issue accounttype for domain-joined computers (Émettre accounttype pour les ordinateurs joints à un domaine) | Si l’entité en cours d’authentification est un appareil joint à un domaine, cette règle émet le type de compte sous la forme DJ, qui signifie appareil joint à un domaine. |
| Issue AccountType with the value USER when it is not a computer account (Émettre AccountType avec la valeur UTILISATEUR lorsqu’il ne s’agit pas d’un compte d’ordinateur) | Si l’entité en cours d’authentification est un utilisateur, cette règle émet le type de compte sous la forme Utilisateur. |
| Issue issuerid when it is not a computer account (Émettre issuerid lorsqu’il ne s’agit pas d’un compte d’ordinateur) | Cette règle émet la valeur issuerId lorsque l’entité d’authentification n’est pas un appareil. La valeur est créée par le biais d’une expression régulière, qui est configurée par Azure AD Connect. L’expression régulière est créée après la prise en compte de tous les domaines fédérés à l’aide d’Azure AD Connect. |
| Issue issuerid for DJ computer auth (Émettre issuerid pour l’authentification d’ordinateur DJ) | Cette règle émet la valeur issuerId lorsque l’entité d’authentification est un appareil. |
| Issue onpremobjectguid for domain-joined computers (Émettre onpremobjectguid pour les ordinateurs joints à un domaine) | Si l’entité en cours d’authentification est un appareil joint à un domaine, cette règle émet l’objectguid local pour l’appareil. |
| Pass through primary SID (Transmettre directement le SID principal) | Cette règle émet l’identificateur de sécurité principal de l’entité d’authentification. |
| Pass through claim - insideCorporateNetwork (Transmettre directement la revendication - insideCorporateNetwork) | Cette règle émet une revendication qui aide Azure AD à savoir si l’authentification provient du réseau d’entreprise interne ou de l’extérieur. |
| Pass Through Claim - Psso (Transmettre directement la revendication - Psso) |   |
| Issue Password Expiry Claims (Émettre des revendications d’expiration de mot de passe) | Cette règle émet trois revendications pour la date d’expiration du mot de passe, le nombre de jours restants avant l’expiration du mot de passe de l’entité en cours d’authentification et l’URL permettant de modifier le mot de passe.|
| Pass through claim – authnmethodsreferences (Transmettre directement la revendication - authnmethodsreferences) | La valeur de la revendication émise sous cette règle indique le type d’authentification exécuté pour l’entité. |
| Pass through claim - multifactorauthenticationinstant (Transmettre directement la revendication - multifactorauthenticationinstant) | La valeur de cette revendication spécifie l’heure, au format UTC, à laquelle l’utilisateur a effectué la dernière authentification multifacteur. |
| Pass through claim - AlternateLoginID (Transmettre directement la revendication - AlternateLoginID) | Cette règle émet la revendication AlternateLoginID si l’authentification a été effectuée à l’aide de l’ID de connexion alternatif. |

> [!NOTE]
> Les règles de revendication pour Issue UPN et ImmutableId diffèrent si vous n’utilisez pas le choix par défaut lors de la configuration d’Azure AD Connect.

## <a name="restore-issuance-transform-rules"></a>Restaurer les règles de transformation d’émission

La version Azure AD Connect 1.1.873.0 ou les versions ultérieures sauvegardent les paramètres d’approbation Azure AD chaque fois que ces derniers font l’objet d’une mise à jour. Les paramètres d’approbation Azure AD sont sauvegardés à l’emplacement **%ProgramData%\AADConnect\ADFS**. Le nom de fichier présente le format AadTrust-&lt;date&gt;-&lt;heure&gt;.txt ; par exemple : AadTrust-20180710-150216.txt

![Capture d’écran d’exemple de sauvegarde d’approbation Azure AD](./media/how-to-connect-azure-ad-trust/backup.png)

Vous pouvez restaurer les règles de transformation d’émission à l’aide de la procédure recommandée ci-après.

1. Ouvrez l’interface utilisateur de gestion AD FS dans le Gestionnaire de serveur.
2. Ouvrez les propriétés d’approbation Azure AD en accédant à **AD FS &gt; Approbations de partie de confiance &gt; Plateforme d’identité Microsoft Office 365 &gt; Editer la stratégie d’émission de revendication**.
3. Cliquez sur **Ajouter une règle**.
4. Dans le modèle de règles de revendication, sélectionnez « Envoyer les revendications en utilisant une règle personnalisée », puis cliquez sur **Suivant**.
5. Copiez le nom de la règle de revendication à partir du fichier de sauvegarde et collez-le dans le champ **Nom de la règle de revendication**.
6. Copiez la règle de revendication à partir du fichier de sauvegarde dans le champ de texte pour **Règle personnalisée**, puis cliquez sur **Terminer**.

> [!NOTE]
> Assurez-vous que vos autres règles n’entrent pas en conflit avec les règles configurées par Azure AD Connect.

## <a name="next-steps"></a>Étapes suivantes
* [Gérer et personnaliser Active Directory Federation Services à l’aide d’Azure AD Connect](how-to-connect-fed-management.md)
