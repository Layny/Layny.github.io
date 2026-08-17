---
title: "Retour sur l'eJPT : Mon avis honnête, l'examen et la réalité du terrain"
date: 2026-08-16 18:16:32 +0200
categories: [Certifications, eJPT]
tags: [retex, pentest, osint, certification]
---

Obtenir la certification eJPT est souvent considéré comme le PREMIER véritable passage pour quiconque souhaite mettre un pied dans la cybersécurité offensive. Après avoir validé l'examen et pris un peu de recul, il est temps de faire un bilan complet et sans filtre. 

Certains jugent qu'il n'est pas utile de faire un retour sur une certification "débutante", mais personnellement je pense que ça peut répondre à des questions.

### Une excellente base méthodologique

Le point fort de cette certification réside dans l'apprentissage de la méthode. L'eJPT fait un excellent travail pour structurer la phase de reconnaissance (passive et active), tout en insistant sur le cadre légal et les limites à ne pas franchir lors d'un audit.

La cartographie de la cible est une étape importante (domaines, IPs, hébergement, services). On y apprend à manipuler certains outils dans différents sujets :

*   **Reconnaissance Passive & OSINT :** Utilisation des Google Dorks, requêtes LinkedIn, fouille des `robots.txt` et `sitemap.xml`.
*   **Profilage Web :** Utilisation d'extensions comme BuiltWith ou Wappalyzer pour décortiquer l'infrastructure et les plugins d'un site.
*   **Analyse DNS :** Sublist3r, dnsdumpster, `dnsrecon`.
*   **Reconnaissance Active :** La maîtrise indispensable de Nmap, l'utilisation de `httrack`, ou encore `wafw00f` pour la détection de pare-feux applicatifs.

### L'arsenal d'exploitation classique

La phase d'exploitation du cours est bien approfondi pour un profil junior. Elle permet d'éclaircir les environnements Linux et Windows, et surtout d'apprendre à utiliser le framework Metasploit (`msfconsole`). 

Les vidéos et exercices abordent certains fondamentaux comme :
*   L'exploitation du système Linux ou Windows pour l'adaptation de la cible
*   L'identification et l'exploitation des failles web les plus courantes (XSS, SQLi).
*   La création de reverse shells et l'utilisation de payloads.
*   Les mécaniques d'élévation de privilèges (PrivEsc), notamment avec l'utilisation indispensable des GTFOBins sous Linux.

### Le format de l'examen : 100% pratique

S'il y a bien un point sur lequel toute la communauté est unanime, c'est le format de l'examen : l'eJPT vous plonge dans un environnement de type *Black Box* avec un QCM pour certaines réponses.

On est face à un réseau d'entreprise simulé, et l'objectif est clair : énumérer, exploiter, pivoter et répondre à une série de questions directement liées à vos compromissions. L'examen est à livre ouvert, ce qui reflète parfaitement la réalité du métier : un pentester n'est pas une encyclopédie, il doit savoir chercher l'information, utiliser ses notes et s'adapter.

Attention ! Sans pratique ou exploitation du système, il est impossible de deviner ou répondre aux questions lors de l'examen (même s'il y a un QCM).
Une réponse à une question est aussi un indice à la réponse suivante.

### ⚠️ Les limites : Un décalage avec les défenses modernes

C'est ici que l'honnêteté s'impose. Si la certification est excellente pour poser les bases, elle montre des limites évidentes face aux environnements d'entreprise actuels.

1.  **L'absence cruelle de l'Active Directory :** Dans le monde réel du pentest d'infrastructure, l'Active Directory est le nerf de la guerre. L'eJPT n'aborde pas (ou de façon beaucoup trop superficielle) cet aspect pourtant fondamental.
2.  **Un côté "Old School" :** On apprend à générer des payloads via MSFvenom et à utiliser des encodeurs classiques comme *shikata_ga_nai*. C'est très intéressant pour comprendre la théorie de l'évasion d'antivirus. Cependant, dans la pratique d'aujourd'hui, n'importe quel EDR (Endpoint Detection and Response) ou AV moderne va flagger et bloquer ces attaques de manière instantanée. Les techniques enseignées sont bruyantes et obsolètes face aux défenses contemporaines.

### Conclusion

L'eJPT reste une excellente certification d'introduction. C'est un tremplin parfait pour débuter, assimiler la méthodologie d'audit et se familiariser avec le côté offensive. Elle donne confiance et prouve que l'on sait mener une attaque basique de A à Z.

Parcontre, il ne faut pas s'y tromper. Beaucoup pense qu'elle suffit pour être pleinement opérationnel sur le marché du travail d'aujourd'hui. Pour affronter de vraies infrastructures, il est nécessaire d'aller beaucoup plus loin en se formant en continu. 

On peut se tourner vers une future certification comme la **CPTS** de Hack The Box pour approfondir les points manquants et agrandir notre arbre de compétences.