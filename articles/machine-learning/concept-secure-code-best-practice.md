---
title: Bonnes pratiques pour la sécurisation du code
titleSuffix: Azure Machine Learning
description: Découvrez les menaces de sécurité potentielles qui peuvent exister lors du développement pour Azure Machine Learning, les atténuation de risques et les meilleures pratiques.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: cgronlun
author: cjgronlund
ms.date: 11/12/2019
ms.openlocfilehash: 37cb70bdbd1e3c87eeb994e0959c6214822d22ad
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322977"
---
# <a name="secure-code-best-practices-with-azure-machine-learning"></a>Bonnes pratiques pour la sécurisation du code avec Azure Machine Learning

Dans Azure Machine Learning, vous pouvez charger des fichiers et du contenu à partir de n’importe quelle source dans Azure. Le contenu dans les scripts ou les notebooks Jupyter que vous chargez peut potentiellement lire les données de vos sessions, accéder aux données de votre organisation dans Azure ou exécuter des processus malveillants en votre nom.

> [!IMPORTANT]
> Exécutez uniquement des notebooks ou des scripts provenant de sources approuvées, par exemple l’emplacement où votre équipe de sécurité ou vous-même avez revu le notebook ou le script.

## <a name="potential-threats"></a>Menaces potentielles

Le développement avec Azure Machine Learning implique souvent des environnements de développement basés sur le web (Notebooks & Azure ML Studio). Quand vous utilisez des environnements de développement web, les menaces potentielles sont les suivantes :

* [Scripts intersites (XSS)](https://owasp.org/www-community/attacks/xss/)

    * __Injection basée sur le modèle DOM__  : ce type d’attaque peut modifier l’interface utilisateur affichée dans le navigateur, par exemple en changeant le comportement du bouton Exécuter dans un notebook Jupyter.
    * __Jeton d’accès/cookies__  : les attaques XSS peuvent également accéder au stockage local et aux cookies du navigateur. Votre jeton d’authentification Azure Active Directory (AAD) est stocké dans le stockage local. Une attaque XSS peut utiliser ce jeton pour effectuer des appels d’API en votre nom, puis envoyer les données à une API ou un système externe.

* [Falsification de requête intersite (CSRF)](https://owasp.org/www-community/attacks/csrf) : Cette attaque peut remplacer l’URL d’une image ou d’un lien par celle d’un script ou d’une API malveillant(e). Quand l’image est chargée, ou quand l’utilisateur clique sur le lien, un appel est effectué à l’URL.

## <a name="azure-ml-studio-notebooks"></a>Notebooks Azure ML Studio

Azure Machine Learning Studio fournit une expérience de notebook hébergé dans votre navigateur. Les cellules d’un notebook peuvent générer des fragments ou des documents HTML qui contiennent du code malveillant.  Lorsque la sortie est restituée, le code peut être exécuté.

__Menaces possibles__  :
* Scripts intersites (XSS)
* Falsification de requête intersite (CSRF)

__Atténuations des risques fournies par Azure Machine Learning__  :
* La __sortie de la cellule de code__ est mise en bac à sable (sandbox) dans un iframe. L’iframe empêche le script d’accéder au modèle DOM parent, aux cookies ou au stockage de session.
* Le contenu de la __cellule Markdown__ est nettoyé à l’aide de la bibliothèque dompurify. Cela empêche les scripts malveillants de s’exécuter pendant le rendu des cellules Markdown.
* L’ __URL d’image__ et les __liens Markdown__ sont envoyés à un point de terminaison Microsoft, qui vérifie s’il n’y a pas de valeurs malveillantes. Si une valeur malveillante est détectée, le point de terminaison rejette la requête.

__Actions recommandées__  :
* Vérifiez que le contenu des fichiers est digne de confiance avant de le charger dans Studio. Lors du chargement, vous devez confirmer que vous chargez des fichiers approuvés.
* Lors de la sélection d’un lien pour ouvrir une application externe, vous serez invité à approuver l’application.

## <a name="azure-ml-compute-instance"></a>Instance de calcul Azure ML

L’instance de calcul Azure Machine Learning héberge __Jupyter__ et __Jupyter Lab__. Quand vous utilisez l’un ou l’autre, les cellules d’un notebook ou du code peuvent générer des fragments ou des documents HTML qui contiennent du code malveillant. Lorsque la sortie est restituée, le code peut être exécuté. Les mêmes menaces s’appliquent également quand vous utilisez __RStudio__ hébergé sur une instance de calcul.

__Menaces possibles__  :
* Scripts intersites (XSS)
* Falsification de requête intersite (CSRF)

__Atténuations des risques fournies par Azure Machine Learning__  :
* Aucun. Jupyter et Jupyter Lab sont des applications open source hébergées sur l’instance de calcul Azure Machine Learning.

__Actions recommandées__  :
* Vérifiez que le contenu des fichiers est digne de confiance avant de le charger dans Studio. Lors du chargement, vous devez confirmer que vous chargez des fichiers approuvés.

## <a name="report-security-issues-or-concerns"></a>Signaler des problèmes de sécurité ou des préoccupations 

Azure Machine Learning est éligible dans le cadre du programme de primes Microsoft Azure. Pour plus d’informations, consultez [https://www.microsoft.com/msrc/bounty-microsoft-azure](https://www.microsoft.com/msrc/bounty-microsoft-azure).

## <a name="next-steps"></a>Étapes suivantes

* [Sécurité de l’entreprise pour Azure Machine Learning](concept-enterprise-security.md)