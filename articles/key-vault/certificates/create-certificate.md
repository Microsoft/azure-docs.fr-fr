---
title: Méthodes de création de certificats
description: Découvrez les différentes options permettant de créer ou d’importer un certificat Key Vault dans Azure Key Vault. Il existe plusieurs façons de créer un certificat de Key Vault.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: mbaldwin
ms.openlocfilehash: a9545c040809331a5556b11f6cc7536931e2d421
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289582"
---
# <a name="certificate-creation-methods"></a>Méthodes de création de certificats

 Un certificat Key Vault (KV) peut être créé ou importé dans un coffre de clés. Lors de la création d’un certificat KV, la clé privée est créée dans le coffre de clés et n’est jamais exposée au propriétaire du certificat. Vous trouverez ci-dessous des méthodes de création d’un certificat dans Key Vault :  

-   **Créer un certificat auto-signé :** crée une paire de clés publique-privée et l’associe à un certificat. Le certificat est signé par sa propre clé.  

-    **Créer un certificat manuellement :** crée une paire de clés publique-privée et génère une demande de signature de certificat X.509. La demande de signature peut être signée par votre autorité de certification ou autorité d’inscription. Le certificat X509 signé peut être fusionné avec la paire de clés en attente afin de terminer la création du certificat KV dans Key Vault. Bien que cette méthode nécessite davantage d’étapes, elle offre une plus grande sécurité, car la clé privée est créée dans Key Vault et restreinte à ce dernier. Ceci est expliqué dans le diagramme ci-dessous.  

![Créer un certificat avec votre propre autorité de certification](../media/certificate-authority-1.png)  

Les descriptions suivantes correspondent aux étapes écrites en vert dans le diagramme précédent.

1. Dans le diagramme ci-dessus, votre application crée un certificat qui commence en interne par la création d’une clé dans votre coffre de clés.
2. Key Vault retourne une demande de signature de certificat (CSR) à votre application.
3. Votre application transmet la demande CSR à l’autorité de certification que vous avez choisie.
4. L’autorité de certification que vous avez choisie répond avec un certificat X509.
5. Votre application termine la création du certificat par le biais d’une fusion avec le certificat X509 provenant de votre autorité de certification.

-   **Créer un certificat avec un fournisseur émetteur connu :** cette méthode requiert une tâche unique consistant à créer un objet émetteur. Une fois qu’un objet émetteur est créé dans votre coffre de clés, son nom peut être référencé dans la stratégie du certificat KV. La demande de création d’un tel certificat KV crée une paire de clés dans le coffre et communique avec le service de fournisseur émetteur à l’aide des informations dans l’objet émetteur référencé pour obtenir un certificat X509. Le certificat X509 est récupéré à partir du service de l’émetteur et fusionné avec la paire de clés pour terminer la création du certificat KV.  

![Créer un certificat auprès d’une autorité de certification associée à Key Vault](../media/certificate-authority-2.png)  

Les descriptions suivantes correspondent aux étapes écrites en vert dans le diagramme précédent.

1. Dans le diagramme ci-dessus, votre application crée un certificat qui commence en interne par la création d’une clé dans votre coffre de clés.
2. Key Vault envoie une demande de certificat TSL/SSL à l’autorité de certification.
3. Votre application interroge, dans un processus boucle et attente, votre instance Key Vault pour savoir si la création du certificat est terminée. La création de certificats est terminée lorsque Key Vault reçoit la réponse de l’autorité de certification avec un certificat X509.
4. L’autorité de certification répond à la demande de certificat TSL/SSL de Key Vault avec un certificat X509 TSL/SSL.
5. La création de votre nouveau certificat se termine avec la fusion du certificat X.509 TSL/SSL pour l’autorité de certification.

## <a name="asynchronous-process"></a>Processus asynchrone
La création du certificat KV est un processus asynchrone. Cette opération crée une demande de certificat KV et retourne un code d’état HTTP 202 (Accepté). Vous pouvez suivre l’état de la demande en interrogeant l’objet en attente créé par cette opération. L’URI complet de l’objet en attente est retourné dans l’en-tête EMPLACEMENT.  

Quand une demande de création d’un certificat KV est terminée, l’objet en attente passe de l’état « inprogress » (En cours) à « completed » (Terminé), et une nouvelle version du certificat KV est créée. Elle devient la version actuelle.  

## <a name="first-creation"></a>Première création
 Lorsqu’un certificat KV est créé pour la première fois, un secret et une clé adressables sont également créés avec le même nom que celui du certificat. Si le nom est déjà utilisé, l’opération échoue avec un code d’état HTTP 409 (Conflit).
Le secret et la clé adressables obtiennent leurs attributs à partir des attributs du certificat KV. Le secret et la clé adressables ainsi créés sont marqués en tant que secrets et clés managés, dont la durée de vie est gérée par Key Vault. Les secrets et les clés managés sont en lecture seule. Remarque : Si un certificat KV arrive à expiration ou est désactivé, le secret et la clé correspondants deviennent inutilisables.  

 S’il s’agit de la première création d’un certificat KV, alors une stratégie est requise.  Une stratégie peut également être fournie avec les opérations de création suivantes pour remplacer la ressource de la stratégie. Si aucune stratégie n’est fournie, la ressource de la stratégie sur le service est utilisée pour créer une prochaine version du certificat KV. Notez que lorsqu’une demande de création d’une prochaine version est en cours, le certificat KV actuel, ainsi que le secret et la clé adressables correspondants, restent inchangés.  

## <a name="self-issued-certificate"></a>Certificat auto-émis
 Pour créer un certificat auto-émis, définissez le nom de l’émetteur sur « Self » (Auto) dans la stratégie de certificat comme indiqué dans l’extrait de code suivant issu de la stratégie de certificat.  

```  
"issuer": {  
       "name": "Self"  
    }  

```  

 Si le nom de l’émetteur n’est pas spécifié, il est défini sur « Unknown » (Inconnu). Lorsque l’émetteur est « Unknown », le propriétaire du certificat doit obtenir manuellement un certificat X509 auprès de l’émetteur de son choix, puis fusionner le certificat X509 public avec l’objet en attente de certificat du coffre de clés afin de terminer la création du certificat.

```  
"issuer": {  
       "name": "Unknown"  
    }  

```  

## <a name="partnered-ca-providers"></a>Fournisseurs d’autorité de certification associés
La création du certificat peut être effectuée manuellement ou à l’aide d’un émetteur « Self ». Key Vault s’associe également à certains fournisseurs émetteurs pour simplifier la création de certificats. Les types de certificats suivants peuvent être commandés pour Key Vault avec ces fournisseurs émetteurs partenaires.  

|Fournisseur|Type de certificat|Configuration  
|--------------|----------------------|------------------|  
|DigiCert|Key Vault propose des certificats SSL OV ou EV avec DigiCert| [Guide d’intégration](./how-to-integrate-certificate-authority.md)
|GlobalSign|Key Vault propose des certificats SSL OV ou EV avec GlobalSign| [Guide d’intégration](https://support.globalsign.com/digital-certificates/digital-certificate-installation/generating-and-importing-certificate-microsoft-azure-key-vault)

 L’émetteur de certificat est une entité représentée dans Azure Key Vault (KV) en tant que ressource CertificateIssuer. Il fournit des informations sur la source d’un certificat KV, ainsi que le nom de l’émetteur, le fournisseur, les informations d’identification et d’autres détails administratifs.

Notez que lorsqu’une commande est passée auprès du fournisseur émetteur, elle peut respecter ou remplacer les extensions du certificat X509 et la période de validité du certificat en fonction du type de certificat.  

 Autorisation : nécessite l’autorisation de création/certificats.

## <a name="see-also"></a>Voir aussi

 - [Surveiller et gérer la création de certificats](create-certificate-scenarios.md)