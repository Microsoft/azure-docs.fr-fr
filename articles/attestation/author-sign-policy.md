---
title: Guide pratique pour créer une stratégie Azure Attestation
description: Explication de la création d’une stratégie d’attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: 3e36de62b79788e2efdc3e9abf711924c4fba0c4
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341805"
---
# <a name="how-to-author-an-attestation-policy"></a>Guide pratique pour créer une stratégie d’attestation

Une stratégie d’attestation est un fichier chargé sur Microsoft Azure Attestation. Azure Attestation offre la possibilité de charger une stratégie dans un format de stratégie spécifique à l’attestation. Une version encodée de la stratégie, au format JSON Web Signature, peut également être chargée. L’administrateur de stratégies est responsable de l’écriture de la stratégie d’attestation. Dans la plupart des scénarios d’attestation, la partie de confiance fait office d’administrateur de stratégies. Le client qui effectue l’appel d’attestation envoie une preuve d’attestation, que le service analyse et convertit en revendications entrantes (ensemble de propriétés, valeur). Le service traite ensuite les revendications, en fonction de ce qui est défini dans la stratégie, et retourne le résultat calculé.

La stratégie contient des règles qui déterminent les critères d’autorisation, les propriétés et le contenu du jeton d’attestation. Voici un exemple de fichier de stratégie :

```
version=1.0;
authorizationrules
{
   c:[type="secureBootEnables", issuer=="AttestationService"]=> permit()
};

issuancerules
{
  c:[type="secureBootEnables", issuer=="AttestationService"]=> issue(claim=c)
  c:[type="notSafeMode", issuer=="AttestationService"]=> issue(claim=c)
};
```
 
Un fichier de stratégie comporte trois sections, comme indiqué ci-dessus :

- **version**  :  la version est le numéro de version de la grammaire qui est suivie. 

    ```
    version=MajorVersion.MinorVersion   
    ```

    La seule version prise en charge est la version « 1.0 ».

- **authorizationrules**  : collection de règles de revendication qui sont vérifiées en premier, pour déterminer si Azure Attestation doit passer à la section **issuancerules**. Les règles de revendication s’appliquent dans l’ordre dans lequel elles sont définies.

- **issuancerules**  : collection de règles de revendication qui sont évaluées pour ajouter des informations supplémentaires au résultat de l’attestation, tel que défini dans la stratégie. Les règles de revendication s’appliquent dans l’ordre dans lequel elles sont définies et sont également facultatives.

Pour plus d’informations, consultez [Revendication et règles de revendication](claim-rule-grammar.md).
   
## <a name="drafting-the-policy-file"></a>Rédaction du fichier de stratégie

1. Créez un fichier.
1. Ajoutez une version au fichier.
1. Ajoutez des sections pour **authorizationrules** et **issuancerules**.

  ```
  version=1.0;
  authorizationrules
  {
  =>deny();
  };
  
  issuancerules
  {
  };
  ```

  Les règles d’autorisation contiennent l’action deny() sans condition, afin qu’aucune règle d’émission ne soit traitée. La règle d’autorisation peut également contenir l’action permit(), afin de permettre le traitement des règles d’émission.
  
4. Ajouter des règles de revendication aux règles d’autorisation

  ```
  version=1.0;
  authorizationrules
  {
  [type=="secureBootEnabled", value==true, issuer=="AttestationService"]=>permit();
  };
  
  issuancerules
  {
  };
  ```

  Si le jeu de revendications entrantes contient une revendication correspondant au type, à la valeur et à l’émetteur, l’action permit() indique au moteur de stratégie de traiter la section **issuancerules**.
  
5. Ajoutez des règles de revendication à la section **issuancerules**.

  ```
  version=1.0;
  authorizationrules
  {
  [type=="secureBootEnabled", value==true, issuer=="AttestationService"]=>permit();
  };
  
  issuancerules
  {
  => issue(type="SecurityLevelValue", value=100);
  };
  ```
  
  Le jeu de revendications sortantes contiendra une revendication avec :

  ```
  [type="SecurityLevelValue", value=100, valueType="Integer", issuer="AttestationPolicy"]
  ```

  Des stratégies complexes peuvent être créées de la même manière. Pour plus d’informations, consultez [Exemples de stratégie d’attestation](policy-examples.md).
  
6. Enregistrez le fichier .

## <a name="creating-the-policy-file-in-json-web-signature-format"></a>Création du fichier de stratégie au format JSON Web Signature

Après avoir créé un fichier de stratégie, pour charger une stratégie au format JWS, suivez les étapes ci-dessous.

1. Générez la signature JWS (RFC 7515) avec la stratégie (encodée en utf-8) comme charge utile.
     - L’identificateur de charge utile pour la stratégie encodée Base64Url doit être « AttestationPolicy ».
     
     Exemple de jeton JWT :
     ```
     Header: {"alg":"none"}
     Payload: {"AttestationPolicy":" Base64Url (policy)"}
     Signature: {}

     JWS format: eyJhbGciOiJub25lIn0.XXXXXXXXX.
     ```

2. (Facultatif) Signez la stratégie. Azure Attestation prend en charge les algorithmes suivants :
     - **Aucun**  : ne pas signer la charge utile de la stratégie.
     - **RS256**  : algorithme pris en charge pour signer la charge utile de la stratégie

3. Chargez la signature JWS et validez la stratégie.
     - Si le fichier de stratégie est exempt d’erreurs de syntaxe, il est accepté par le service.
     - Sinon, ce dernier le rejette.

## <a name="next-steps"></a>Étapes suivantes
- [Configurer Azure Attestation à l’aide de PowerShell](quickstart-powershell.md)
- [Attester une enclave SGX à l’aide d’exemples de code](/samples/browse/?expanded=azure&terms=attestation)