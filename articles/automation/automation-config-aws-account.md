---
title: Authentifier les runbooks Azure Automation avec Amazon Web Services
description: Cet article explique comment authentifier des runbooks avec Amazon Web Services.
keywords: authentification AWS, configurer aws
services: automation
ms.subservice: process-automation
ms.date: 04/23/2020
ms.topic: conceptual
ms.openlocfilehash: acb056fb959fda320a14059e2b36a0f5e3fb0b37
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83837181"
---
# <a name="authenticate-runbooks-with-amazon-web-services"></a>Authentifier des runbooks avec Amazon Web Services

Il est possible d’automatiser les tâches courantes avec les ressources Amazon Web Services (AWS) à l’aide des runbooks Automation dans Azure. Vous pouvez automatiser de nombreuses tâches dans AWS à l’aide des runbooks Automation, tout comme avec les ressources dans Azure. Pour l’authentification, vous devez disposer d’un abonnement Azure.

## <a name="obtain-aws-subscription-and-credentials"></a>Obtenir un abonnement et des informations d’identification AWS

Pour vous authentifier auprès d’AWS, vous devez obtenir un abonnement AWS et spécifier un ensemble d’informations d'identification AWS pour authentifier vos runbooks s’exécutant à partir d’Azure Automation. Les informations d’identification spécifiques requises sont la clé d’accès et la clé secrète AWS. Consultez [Utilisation des informations d’identification AWS](https://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html).

## <a name="configure-automation-account"></a>Configuration d’un compte Automation

Vous pouvez utiliser un compte Automation existant pour vous authentifier auprès d’AWS. Vous pouvez également dédier un compte aux runbooks ciblant des ressources AWS. Dans ce cas, créez un [compte Automation](automation-create-standalone-account.md).  

## <a name="store-aws-credentials"></a>Stocker des informations d’identification AWS

Vous devez stocker les informations d’identification AWS en tant que ressources dans Azure Automation. Pour obtenir des instructions sur la création de la clé d’accès et de la clé secrète, consultez [Gestion des clés d’accès de votre compte AWS](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html). Lorsque les clés sont disponibles, copiez l’ID de clé d’accès et l’ID de clé secrète dans un endroit sûr. Vous pouvez télécharger votre fichier de clé pour le stocker en lieu sûr.

## <a name="create-credential-asset"></a>Créer une ressource d’informations d’identification

Une fois que vous avez créé et copié vos clés de sécurité AWS, vous devez créer une ressource d’informations d’identification avec le compte Automation. La ressource vous permet de stocker en toute sécurité les clés AWS et de les référencer dans votre runbooks. Consultez [Créer une ressource d’informations d’identification avec le portail Azure](shared-resources/credentials.md#create-a-new-credential-asset-with-the-azure-portal). Entrez les informations d’AWS suivantes dans les champs prévus à cet effet :
    
* **Nom** - **AWScred** ou une valeur appropriée suivant vos normes d’affectation de noms
* **Nom d’utilisateur** : votre ID d’accès.
* **Mot de passe** : nom de votre clé secrète. 

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir comment créer des runbooks pour automatiser des tâches dans AWS, consultez [Déployer une machine virtuelle Amazon Web Services avec un runbook](automation-scenario-aws-deployment.md).