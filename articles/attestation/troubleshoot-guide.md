---
title: Guide de résolution des problèmes Azure Attestation
description: Guide de résolution des problèmes courants
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 07/20/2020
ms.author: mbaldwin
ms.openlocfilehash: 72d7a40c38f3629a70260f223074b456dff9ce38
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182833"
---
# <a name="microsoft-azure-attestation-troubleshooting-guide"></a>Guide de résolution des problèmes Microsoft Azure Attestation

La gestion des erreurs dans Azure Attestation est implémentée conformément aux [instructions de l’API REST de Microsoft](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#7102-error-condition-responses). La réponse d’erreur retournée par les API d’Azure Attestation contient le code d’état HTTP et les paires nom-valeur avec les noms « code » et « message ». La valeur de « code » est lisible par l’utilisateur et constitue un indicateur du type d’erreur. La valeur de « message » vise à aider l’utilisateur et fournit les détails relatifs à l’erreur.

Si votre problème n’est pas traité dans cet article, vous pouvez également envoyer une demande de support Azure sur la [page du support Azure](https://azure.microsoft.com/support/options/).

Voici quelques exemples des erreurs renvoyées par Azure Attestation :

## <a name="1-http401--unauthorized-exception"></a>1. HTTP–401 : Exception non autorisée

### <a name="http-status-code"></a>Code d'état HTTP
401

**Code d’erreur** Unauthorized

**Exemples de scénarios**
  - Échec de l’attestation si l’utilisateur n’est pas affecté à un rôle Lecteur d’attestation
  - Impossible de gérer les stratégies d’attestation, car l’utilisateur n’est pas affecté aux rôles appropriés
  - Impossible de gérer les signataires de la stratégie d’attestation, car l’utilisateur n’est pas affecté aux rôles appropriés

Utilisateur ayant un rôle Lecteur et tentant de modifier une stratégie d’attestation dans PowerShell 

  ```powershell
  Set-AzAttestationPolicy : Operation returned HTTP Status Code 401
At line:1 char:1
+ Set-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzAttestationPolicy], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.SetAzureAttestationPolicy
  ```

**Étapes de dépannage**

Pour afficher les stratégies d’attestation ou les signataires de stratégie, un utilisateur Azure AD a besoin de l’autorisation pour « Actions » :
- Microsoft.Attestation/attestationProviders/attestation/read

  Cette autorisation peut être attribuée à un utilisateur AD par le biais d’un rôle tel que « Propriétaire » (autorisations génériques), « Lecteur » (autorisations génériques) ou « Lecteur d’attestation » (autorisations spécifiques pour Azure Attestation uniquement).

Pour ajouter ou supprimer des signataires de stratégie ou pour configurer des stratégies, un utilisateur Azure AD doit avoir les autorisations suivantes pour « Actions » :
- Microsoft.Attestation/attestationProviders/attestation/write
- Microsoft.Attestation/attestationProviders/attestation/delete

  Ces autorisations peuvent être attribuées à un utilisateur AD par le biais d’un rôle tel que « Propriétaire » (autorisations génériques), « Contributeur » (autorisations génériques) ou « Contributeur d’attestation » (autorisations spécifiques pour Azure Attestation uniquement).

Les clients peuvent choisir d’utiliser le fournisseur par défaut pour l’attestation ou créer leurs propres fournisseurs avec des stratégies personnalisées. Pour envoyer des demandes d’attestation à des fournisseurs d’attestations personnalisées, le rôle « Propriétaire » (autorisations génériques), « Lecteur » (autorisations génériques) ou « Lecteur d’attestation » est requis pour l’utilisateur. Les fournisseurs par défaut sont accessibles par tout utilisateur Azure AD.

Pour vérifier les rôles dans PowerShell, exécutez la commande ci-dessous :

a. Lancez PowerShell et connectez-vous à Azure via la cmdlet « Connect-AzAccount ».

b. Vérifiez les paramètres d’attribution de rôle Azure.


  ```powershell
  $c = Get-AzContext
  Get-AzRoleAssignment -ResourceGroupName $attestationResourceGroup -ResourceName $attestationProvider -ResourceType Microsoft.Attestation/attestationProviders -SignInName $c.Account.Id
  ```

  Un résultat semblable à celui-ci doit s’afficher :

  ```
  RoleAssignmentId   :/subscriptions/subscriptionId/providers/Microsoft.Authorization/roleAssignments/roleAssignmentId
  
  Scope              : /subscriptions/subscriptionId
  
  DisplayName        : displayName
  
  SignInName         : signInName
  
  RoleDefinitionName : Reader
  
  RoleDefinitionId   : roleDefinitionId
  
  ObjectId           : objectid
  
  ObjectType         : User
  
  CanDelegate        : False
 
  ```

c. Si vous ne trouvez pas une attribution de rôle appropriée dans la liste, suivez les instructions indiquées [ici](../role-based-access-control/role-assignments-powershell.md).

## <a name="2-http--400-errors"></a>2. Erreurs HTTP–400

### <a name="http-status-code"></a>Code d'état HTTP
400

Il existe différentes raisons pour lesquelles une requête peut retourner une erreur 400. Voici quelques exemples d’erreurs renvoyées par les API d’Azure Attestation :

### <a name="21-attestation-failure-due-to-policy-evaluation-errors"></a>2.1. Échec de l’attestation en raison d’erreurs d’évaluation de la stratégie

La stratégie d’attestation comprend des règles d’autorisation et des règles d’émission. La preuve d’enclave est évaluée en fonction des règles d’autorisation. Les règles d’émission définissent les revendications à inclure dans le jeton d’attestation. Si les revendications dans les preuves d’enclave ne sont pas conformes aux règles d’autorisation, les appels d’attestation renvoient une erreur d’évaluation de la stratégie. 

**Code d’erreur** PolicyEvaluationError

**Exemples de scénarios** Lorsque les revendications du devis d’enclave ne correspondent pas aux règles d’autorisation de la stratégie d’attestation

```
Native operation failed with 65518: G:\Az\security\Attestation\src\AttestationServices\Instance\NativePolicyWrapper\NativePolicyEngine.cpp(168)\(null)!00007FF801762308: (caller: 00007FF80143DCC8) Exception(0) 83FFFFEE Policy Evaluation Error has occurred Msg:[Policy Engine Exception: A Deny claim was issued, authorization failed.]

G:\Az\security\Attestation\src\AttestationServices\Instance\Enclave\api.cpp(840)\(null)!00007FF801739FF3: (caller: 00007FF801232801) LogHr(0) 83FFFFEE Policy Evaluation Error has occurred Msg:[Unhandled Enclave Exception: "Policy Evaluation Error has occurred"]

```

**Étapes de dépannage** Les utilisateurs peuvent évaluer les preuves d’enclave par rapport à une stratégie d’attestation SGX avant de les configurer.

Envoyez une demande d’attestation d’API en fournissant le texte de la stratégie dans le paramètre « draftPolicyForAttestation ». L’API AttestSgxEnclave utilise ce document de stratégie lors de l’appel de l’attestation et celui-ci peut être utilisé pour tester les stratégies d’attestation avant qu’elles ne soient consommées. Quand ce champ est présent, le jeton d’attestation généré n’est pas sécurisé.

Voir des [exemples de stratégie d’attestation](./policy-examples.md)

### <a name="22-attestation-failure-due-to-invalid-input"></a>2.2. Échec de l’attestation en raison d’une entrée non valide

**Code d’erreur** InvalidParameter

**Exemples de scénarios** Échec de l’attestation SGX en raison d’une entrée non valide. Voici quelques exemples de messages d’erreur :
- Le devis spécifié n’était pas valide en raison d’une erreur dans la documentation connexe du devis. 
- Le devis spécifié n’était pas valide, car l’appareil sur lequel le devis a été généré ne répond pas aux exigences de base d’Azure.
- Le devis spécifié n’est pas valide, car le TCBInfo ou le QEID fourni par le service de cache PCK n’était pas valide.

**Étapes de dépannage**

Microsoft Azure Attestation prend en charge l’attestation de devis SGX générés par les Kits de développement logiciel (SDK) Intel et Open Enclave.

Reportez-vous aux [exemples de code](/samples/browse/?expanded=azure&terms=attestation) pour effectuer une attestation à l’aide des Kits de développement logiciel (SDK) Open Enclave ou Intel.

### <a name="23-invalid-certificate-chain-error-while-uploading-policypolicy-signer"></a>2.3. Erreur de chaîne de certificat non valide lors du chargement de la stratégie/du signataire de stratégie

**Code d’erreur** InvalidParameter

**Exemples de scénarios** Configurer une stratégie signée ou ajouter/supprimer un signataire de stratégie, qui est signé avec une chaîne de certificat non valide (par exemple, lorsque l’extension Basic Constraints du certificat racine n’est pas définie sur Subject Type = CA)

```
Native operation failed with 65529: C:\source\src\AttestationServices\Instance\SgxPal\sgxcert.cpp(1074)\(null)!00007FFA285CDAED: (caller: 00007FFA285C36E8) Exception(0) 83FFFFF9 The requested item is not found    Msg:[Unable to find issuer certificate CN=attestationsigningcert]
C:\source\src\AttestationServices\Instance\Enclave\api.cpp(618)\(null)!00007FFA286DCBF8: (caller: 00007FFA285860D3) LogHr(0) 83FFFFF9 The requested item is not found    Msg:[Unhandled Enclave Exception: "The requested item is not found"]
At line:1 char:1
+ Set-AzAttestationPolicy -Name "testpolicy1" -ResourceGroupName "BugBa ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzAttestationPolicy], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.SetAzureAttestationPolicy

```

**Étapes de dépannage** Le certificat racine doit être signalé comme étant émis par une autorité de certification (les contraintes de base X.509), sinon il ne sera pas considéré comme un certificat valide. 

Assurez-vous que l’extension Basic Constraints du certificat racine est définie pour indiquer Subject Type = CA.

Sinon, la chaîne de certificat est considérée comme non valide.

Voir des exemples de [signataire de stratégie](./policy-signer-examples.md) et de [stratégie](./policy-examples.md) 

### <a name="24-adddelete-policy-signer-failure"></a>2.4. Échec de l’ajout ou de la suppression du signataire de stratégie

**Code d’erreur** InvalidOperation

**Exemples de scénarios**

Lorsque l’utilisateur charge JWS sans revendication « maa-policyCertificate »

```
Add-AzAttestationPolicySigner : Operation returned HTTP Status Code 400
Code: InvalidOperation
Message: Native operation failed with 74: ..\Enclave\enclave.cpp(2213)\(null)!: (caller: ) Exception(0) 83FF004A Bad
message    Msg:[Could not find "maa-policyCertificate" claim in policy token]
..\Enclave\api.cpp(496)\(null)!: (caller: ) LogHr(0) 83FF004A Bad message    Msg:[Unhandled Enclave Exception: "Bad
message"]
At line:1 char:1
+ Add-AzAttestationPolicySigner -Name $attestationProvider -ResourceGro ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Add-AzAttestationPolicySigner], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.AddAzureAttestationPolicySigner

```

Lorsque l’utilisateur ne charge pas de certificat au format JWS

```
Add-AzAttestationPolicySigner : Operation returned HTTP Status Code 400
Code: InvalidOperation
Message: Native operation failed with 74: ..\JsonWebToken\jsonwebtoken.cpp(375)\(null)!: (caller: ) LogHr(0) 83FF004A
Bad message    Msg:[RETURN_IF_TRUE('(firstPeriod == std::string::npos)') failed with 0x4a: Malformed JWT: Could not
find first period in the token.]
..\Enclave\enclave.cpp(2106)\(null)!: (caller: ) LogHr(0) 83FF004A Bad message
Msg:[THROW_IF_ERROR('DecomposeJsonWebSignature(&policyJws, encodedJoseHeader, encodedJwsBody, jwsSignature)') failed
with 0x4a: 'Bad message']
..\Enclave\enclave.cpp(2106)\(null)!: (caller: ) Exception(0) 83FF004A Bad message
..\Enclave\api.cpp(496)\(null)!: (caller: ) LogHr(0) 83FF004A Bad message    Msg:[Unhandled Enclave Exception: "Bad
message"]
At line:1 char:1
+ Add-AzAttestationPolicySigner -Name $attestationProvider -ResourceGro ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Add-AzAttestationPolicySigner], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.AddAzureAttestationPolicySigner
```

**Étapes de dépannage** Pour ajouter/supprimer un nouveau certificat de signataire de stratégie, utilisez RFC7519 JSON Web Token (JWT) avec une revendication nommée « x-ms-policyCertificate ». La valeur de la revendication est une clé web JSON RFC7517, qui contient le certificat à ajouter. JWT doit être signé avec une clé privée de l’un des certificats de signataire de stratégie valides associés au fournisseur. Voir des [exemples de signataire de stratégie](./policy-signer-examples.md).

### <a name="25-attestation-policy-configuration-failure"></a>2.5. Échec de la configuration de la stratégie d’attestation

**Code d’erreur** PolicyParsingError

**Exemples de scénarios** Stratégie fournie avec une syntaxe incorrecte (par exemple, point-virgule manquant ou absence de stratégie JWT valide)

```
Native operation failed with 65526: ..\NativePolicyWrapper\NativePolicyEngine.cpp(31)\(null)!: (caller: ) Exception(0) 83FFFFF6 Invalid policy was specified    Msg:[Policy Parser Exception Thrown: Offending
symbol: '['
Line: '2', Column: '1'
Failure message: 'mismatched input '[' expecting ';''
Failing rule: 'policy > versionInfo']
..\Enclave\api.cpp(618)\(null)!: (caller: ) LogHr(0) 83FFFFF6 Invalid policy was specified    Msg:[Unhandled Enclave Exception: "Invalid policy was specified"]
At line:1 char:1
+ set-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzAttestationPolicy], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.SetAzureAttestationPolicy
```

**Code d’erreur** InvalidOperation

**Exemples de scénarios** Contenu non valide fourni (par exemple, chargement d’une stratégie non signée lorsque la signature de la stratégie est obligatoire)

```
Native operation failed with 74: ..\Shared\base64url.h(226)\(null)!: (caller: ) Exception(0) 83FF004A Bad message    Msg:[Unknown base64 character: 41 (')')]
..\Enclave\api.cpp(618)\(null)!: (caller: ) LogHr(0) 83FF004A Bad message    Msg:[Unhandled Enclave Exception: "Bad message"]
At line:1 char:1
+ set-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzAttestationPolicy], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.SetAzureAttestationPolicy
```

**Étapes de dépannage** S’assurer que la stratégie au format texte est UTF-8.

Si la signature de stratégie est requise, la stratégie d’attestation doit être configurée uniquement au format RFC7519 JSON Web Token (JWT). Si vous n’avez pas besoin de la signature de stratégie, la stratégie peut être configurée au format texte ou JWT.

Pour configurer une stratégie au format JWT, utilisez JWT avec une revendication nommée « AttestationPolicy ». La valeur de la revendication est la version codée en Base64URL du texte de la stratégie. Si le fournisseur d’attestation est configuré avec des certificats de signataire de stratégie, JWT doit être signé avec la clé privée de l’un des certificats de signataire de stratégie valides associés au fournisseur. 

Pour configurer une stratégie au format texte, spécifiez directement le texte de la stratégie.

Dans PowerShell, pour PolicyFormat, spécifiez JWT afin de configurer la stratégie au format JWT. Le format de stratégie par défaut est Texte.

Voir des [exemples de stratégie d’attestation](./policy-examples.md) et [comment créer une stratégie d’attestation](./author-sign-policy.md) 

## <a name="3-azattestation-installation-issues-in-powershell"></a>3. Problèmes d’installation d’Az.Attestation dans PowerShell

Impossible d’installer les modules Az ou Az.Attestation dans PowerShell

### <a name="error"></a>Error

AVERTISSEMENT : Impossible de résoudre la source du package « https://www.powershellgallery.com/api/v2  » PackageManagement\Install-Package : Aucune correspondance n’a été trouvée pour les critères de recherche et le nom de module spécifiés

### <a name="troubleshooting-steps"></a>Étapes de dépannage

PowerShell Gallery déconseille les versions 1.0 et 1.1 du protocole TLS (Transport Layer Security). 

TLS 1.2 ou une version ultérieure est recommandé. 

Pour continuer à interagir avec PowerShell Gallery, exécutez la commande suivante avant les commandes Install-Module

**[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12**

## <a name="4-policy-accessconfiguration-issues-in-powershell"></a>4. Problèmes d’accès ou de configuration pour la stratégie dans PowerShell

Utilisateur affecté aux rôles appropriés, mais faisant face à des problèmes d’autorisation lors de la gestion des stratégies d’attestation via PowerShell.

### <a name="error"></a>Error
Le client ayant l’ID d’objet <object Id> n’a pas l’autorisation d’effectuer l’action Microsoft.Authorization/roleassignments/write sur l’étendue « subcriptions/<subscriptionId>resourcegroups/secure_enclave_poc/providers/Microsoft.Authorization/roleassignments/<role assignmentId> », ou l’étendue n’est pas valide. Si l’accès a été accordé récemment, actualisez vos informations d’identification.

### <a name="troubleshooting-steps"></a>Étapes de dépannage

La version minimale des modules Az nécessaires pour prendre en charge les opérations d’attestation sont les suivantes : 

 **Az 4.5.0** 
 
 **Az.Accounts 1.9.2**
 
 **Az.Attestation 0.1.8** 

Exécutez la commande ci-dessous pour vérifier la version installée de tous les modules Az : 

```powershell
Get-InstalledModule 
```

Si les versions ne correspondent pas à la configuration minimale requise, exécutez des commandes Update-Module.

Par ex. : - Update-Module -Name Az.Attestation