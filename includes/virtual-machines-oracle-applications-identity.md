---
author: dlepow
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 07/10/2019
ms.author: danlep
ms.openlocfilehash: 2b28cc742bb981e0db535d39cb0c7fbf027470ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83673762"
---
### <a name="identity-tier"></a>Niveau d'identité 

Le partenariat entre Microsoft et Oracle vous permet de configurer une identité unifiée sur Azure, OCI et votre application Oracle. Pour JD Edwards EnterpriseOne ou la suite d’applications PeopleSoft, une instance du serveur HTTP Oracle (OHS) est nécessaire pour configurer l’authentification unique entre Azure AD et Oracle IDCS.

OHS agit comme un proxy inverse vers la couche Application, ce qui signifie que toutes les demandes envoyées aux applications finales le traversent. Oracle Access Manager WebGate est un plug-in de serveur Web OHS qui intercepte toutes les requêtes adressées à l’application finale. Si une ressource faisant l’objet d’un accès est protégée (nécessite une session authentifiée), WebGate lance le flux d’authentification OIDC avec Identity Cloud Service via le navigateur de l’utilisateur. Pour plus d’informations sur les flux pris en charge par le WebGate OpenID Connect, consultez la [documentation d’Oracle Access Manager](https://docs.oracle.com/cd/E52734_01/oam/AIAAG/GUID-1E927D1B-FB83-425B-8768-85DB441821A4.htm#AIAAG7327).

Avec cette configuration, un utilisateur déjà connecté à Azure AD peut accéder à l’application JD Edwards EnterpriseOne ou PeopleSoft sans se connecter à nouveau, via Oracle Identity Cloud Service. Les clients qui déploient cette solution bénéficient des avantages de l’authentification unique, notamment compris un ensemble unique d’informations d’identification, une expérience d’authentification améliorée, une sécurité accrue et des coûts d’assistance réduits.

Pour en savoir plus sur la configuration de l’authentification unique pour JD Edwards EnterpriseOne ou PeopleSoft avec Azure AD, consultez le [livre blanc Oracle](https://cloud.oracle.com/iaas/whitepapers/deploy_peoplesoft_jdedwards_across_oci_azure.pdf) associé.