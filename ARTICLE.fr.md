# De la demande de prix au G-code en une soirée — une chaîne d'agents IA pour l'usinage de précision

*Retour d'expérience. Une soirée, un modèle orchestrateur, 42 agents IA spécialisés, et une vraie consultation de 7 pièces d'un atelier d'usinage de précision français — menée des plans PDF jusqu'au devis chiffré, aux modèles 3D validés, au programme de perçage CN et au dossier de fabrication complet. Tout ce qui est confidentiel a été retiré ; une pièce de démonstration entièrement synthétique est incluse pour juger la qualité de sortie.*

![Pièce de démonstration synthétique rendue par la chaîne](demo/demo_flange_render.jpg)
*DEMO-FLANGE-001 — pièce créée pour cet article. CAO paramétrique exacte écrite par un agent, export STEP, render Blender Cycles par la même chaîne. Toutes les cotes sont inventées ; aucune donnée client dans ce dépôt.*

## Le problème

Un atelier d'usinage de précision (10-15 personnes, exigences aéro/énergie) vit et meurt par ses devis. Les chiffres du dirigeant lui-même :

- **4 heures** pour chiffrer une pièce à la main,
- **~7 jours** pour répondre à une consultation multi-pièces,
- **2 devis sur 3 perdus**, en partie face à des concurrents plus rapides,
- des consultations déclinées faute de temps : du chiffre d'affaires qui n'a jamais sa chance.

Le goulot n'est pas dans les machines. Il est entre *le PDF du client* et *une réponse signée et défendable*.

## Ce que fait la chaîne

En un seul passage autonome, sur une consultation réelle de 7 pièces (sous NDA — tous les chiffres et plans de ce dépôt sont synthétiques) :

1. **Lecture des plans** — un agent par PDF : cartouche, matière, traitements, tolérances générales, chaque perçage/taraudage/ajustement/état de surface, et proposition de brut avec surépaisseurs.
2. **Devis** — un moteur transparent : masse du brut × prix matière du jour (LME + distributeurs, sourcés et datés), temps par opération, les règles de l'atelier encodées en *Python lisible*. Chaque euro montre sa formule.
3. **CAO** — pas de mesh "IA 3D" : les agents écrivent du code [build123d](https://github.com/gumyr/build123d) (noyau OpenCascade) qui produit une **géométrie paramétrique exacte, exportable en STEP**, qui s'ouvre dans SolidWorks. Chaque modèle est ensuite *validé par mesure* : boîte englobante contre le plan, chaque diamètre nominal présent dans le solide, masse calculée contre le cartouche.
4. **FAO** — G-code de cycles de perçage/taraudage aux positions vérifiées trigonométriquement, et une stratégie d'entrée matière (hélicoïdale / rampe / plongée) documentée que **l'opérateur** valide — pas l'IA.
5. **Dossier de fabrication** — ordre de fabrication, bon de débit avec exigence de certificat matière EN 10204 3.1, fiche de contrôle premier article type AS9102 avec les bornes ISO 286 calculées et contre-vérifiées (Ø140 H7 → +0,040/0), traçabilité.
6. **Vérification adversariale** — un agent séparé dont le seul rôle est de *casser* le travail des autres : il recalcule chaque total, chaque tolérance, et vérifie le G-code numériquement.

**Les chiffres de la session** (réels, anonymisés) : 42 agents sur deux passes · ~1 h 55 de calcul autonome · 3,06 M de tokens (≈ 30 $ d'équivalent API) · 7 pièces chiffrées, 7 modélisées, 70+ fichiers livrés. Temps humain *pendant* l'exécution : zéro. Temps humain *après* : une relecture — et c'est exactement le but.

## Les trois choses qui font que ça marche vraiment

**1. De la CAO exacte, pas de l'« IA 3D ».** Le LLM écrit du code CAO contre un noyau B-rep exact, et la boucle est fermée par de la **mesure déterministe** : sur le vrai dossier, la pièce principale est revenue avec sa boîte englobante exacte au plan et ses dix diamètres nominaux présents dans le solide.

**2. L'honnêteté comme architecture.** Un plan scanné est ambigu. Règle de la chaîne : **une cote illisible ne devient jamais une supposition — elle devient une question.** Chaque modèle est livré avec son fichier d'hypothèses « à valider par le BE ». Sur le dossier réel, la chaîne a même détecté une **incohérence de chaîne de cotes dans le plan du client lui-même** — signalée au lieu d'être moyennée en silence. C'est ce qui positionne l'expert humain en autorité qui tranche, pas en spectateur.

**3. La vérification adversariale.** L'agent vérificateur a trouvé un vrai bug dangereux avant tout humain : l'en-tête du G-code déclarait l'origine Z en haut du brut alors que les coordonnées partaient de la face inférieure — 35 mm de décalage. Détecté par recalcul, corrigé, re-vérifié. La confiance se construit par du contrôle *visible*, pas par de l'assurance.

## Ce qui reste humain — volontairement

L'opérateur valide le bridage et l'attaque matière. Le BE tranche chaque hypothèse. Le chiffreur relit un calcul transparent au lieu de le construire (4 h → une relecture). Rien ne part chez un client sans signature humaine. Le système comprime le temps ; il ne remplace pas le jugement.

## Essayer la démo

Tout le dossier [`demo/`](demo/) est synthétique et autonome : générateur build123d, STEP/STL (s'ouvrent dans SolidWorks/FreeCAD), G-code illustratif avec vitesses et avances calculées, render, et un devis exemple à taux fictifs.

## Qui a fait ça

Je suis **Ismaël Joffroy Chandoutis** — je conçois et déploie des chaînes d'agents IA avec boucles de vérification, pour des PME industrielles (devis, CAO/FAO, documents de production) et des industries créatives. L'ensemble de ce projet — recherche, chaîne, vérification, documentation — a été orchestré en une soirée, par une personne et un environnement IA.

Si votre entreprise chiffre des pièces usinées, lit des plans techniques, ou se noie dans une chaîne PDF-vers-documents, cette approche se transpose directement.

**Contact : contact@ismaeljoffroychandoutis.com**

---

*La mission réelle est sous NDA : rien ici n'identifie l'atelier, ses clients, ses pièces, ses prix ou ses règles métier. Ne faites pas tourner `demo_drilling.nc` sur une vraie machine sans revue professionnelle.*
