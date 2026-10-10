# ATTOL — Dossier de conception

> Agenda architectural. Chaque journée est un atoll : un anneau de terre construit autour d'un lagon.
> On ne lit plus une liste d'heures, on lit un paysage.

*Edel & Nomen — v0.1*

---

## 0. Verdict

**Le concept est fort et différenciant.** Aucun agenda ne traite la *charge* comme matière première visuelle ; tous traitent le *temps*. Attol inverse ça : le temps est la forme, la charge est le volume. C'est le bon axe.

**Trois risques majeurs, à traiter dès le prototype :**

| Risque | Pourquoi c'est grave | Parade |
|---|---|---|
| **Joli mais illisible** | Si l'utilisateur doit « déchiffrer » la ville, l'app perd contre une liste. La promesse est « d'un regard ». | Grammaire visuelle stricte : **1 variable visuelle = 1 information**. Test des 3 secondes à chaque itération. |
| **Lourd sur mobile** | 3D + trafic + jour/nuit + météo = batterie et chaleur. | Rendu instancié low-poly, caméra contrainte, LOD sémantique, animation à la demande (pas en continu). |
| **Saisie trop coûteuse** | Type + sous-type + compétences + charge à chaque tâche = abandon en 1 semaine. | Inférence par défaut (le type propose tout), saisie en 1 geste, import calendrier existant. |

**Position de Nomen :** la 3D doit être **orthographique/isométrique à caméra contrainte** (vue du dessus inclinée, rotation par crans), pas une caméra libre. La 3D libre sert la démo, pas l'usage quotidien. Garder la liberté pour un « mode contemplation ».

---

## 1. Le modèle spatial — grammaire de l'atoll

### 1.1 Forme

```
                    MIDI
              ┌───────────────┐
         ╱  après-midi          ╲
       ╱    ▓▓▓  ▒▒   ░░  ▓▓      ╲
  matin  ▓▓                  ▒▒   soir
     │ ░░      ~~~~~~~~~~~      ░░ │
     │ ▓▓     ~~  LAGON  ~~     ▓▓ │
     │ ▒▒      ~~~~═══~~~~      ▒▒ │   ═══ pont (dépendance)
       ╲   ░░   (parc)     ▓▓    ╱
         ╲     sur-heure / nuit ╱
              └──── récif ────┘
```

- **Anneau = journée.** L'angle encode l'heure (lecture « horloge » immédiate, culturellement acquise).
- **Lagon = centre.** Espace de respiration, de transit, et support des **ponts** (liens entre tâches).
- **Récif extérieur = sur-heures / nuit.** Ce qui déborde de la journée normale déborde physiquement de l'anneau : on *voit* la journée qui s'étale.
- **Semaine = archipel.** Sept atolls ; un report de tâche = une barge qui traverse vers l'atoll suivant.

### 1.2 Découpage géométrique

- 1 heure = **1 secteur annulaire** (parcelle).
- 1 parcelle = **4 sous-unités de 15 min** (réglable : 2 × 30 / 4 × 15 / 6 × 10 / 12 × 5). 15 min est le bon défaut : granularité réelle d'un agenda, pas de micro-gestion.
- Plage visible par défaut : **heure de lever → heure de coucher** de l'utilisateur, pas 24 h (sinon la nuit vide mange la moitié de l'anneau). La nuit est compressée en récif.
- Géométrie : secteurs annulaires subdivisés, contours adoucis (congés arrondis, *chamfer*) → harmonie sans perdre la lisibilité des bords de créneau.

### 1.3 Grammaire visuelle — une variable, une information

C'est le cœur du produit. À figer et ne jamais violer.

| Variable visuelle | Information encodée | Remarque |
|---|---|---|
| **Position angulaire** | Heure de début / durée | Lecture horloge |
| **Emprise au sol** (nb de sous-unités) | Durée | Une tâche de 1h30 = 6 sous-unités |
| **Hauteur** | Intensité / charge instantanée | Le signal principal de surcharge |
| **Densité** (nb de bâtiments par parcelle, ruelles étroites) | Fragmentation (nombre de tâches, changements de contexte) | 4 réunions de 15 min = quartier saturé |
| **Style architectural** | Type de tâche | Bureau, école, commerce, stade… |
| **Matériau / palette** | Compétence dominante mobilisée | Voir §3 |
| **Enseigne / panneau** | Sous-type précis (pharmacie, coiffeur…) | Visible seulement en zoom (LOD) |
| **Trafic sur les ponts** | Flux générés entre tâches (suivis, dépendances, bascules) | Embouteillage = surcharge relationnelle |
| **Végétation / parc** | Récupération (prévue ou suggérée) | Vert = respiration |
| **Lumière (fenêtres allumées)** | Tâche en cours / restante | Le soir, les fenêtres encore allumées = ce qui reste à faire |
| **Fumée / grue / échafaudage** | Tâche en retard, en cours de construction (non planifiée finement) | Grue = « à préciser » |

**Règle :** si un élément visuel ne porte aucune information, il est décoratif et doit être discret (eau, sable, ciel). Le décor ne concurrence jamais le signal.

---

## 2. Taxonomie exhaustive des tâches

4 sphères + 2 transversales. Chaque sous-type porte : **style architectural**, **icône d'enseigne**, **compétences proposées**, **coefficient de charge par défaut** (0.2 → 1.5), **trafic généré par défaut**.

### 2.1 Sphère PROFESSIONNELLE — *Quartier d'affaires / zone d'activités*

| Sous-type | Architecture | Enseigne | Compétences proposées | Charge | Trafic |
|---|---|---|---|---|---|
| Administratif / paperasse | Immeuble de bureaux rectiligne, fenêtres en grille | Classeur | Rigueur, Organisation, Concentration | 0.8 | moyen |
| Réunion interne | Tour de verre, salle vitrée en toiture | Table ronde | Communication, Écoute, Synthèse | 0.9 | **fort** |
| Contact client / commercial | Tour à atrium, hall ouvert | Poignée de main | Relationnel, Négociation, Gestion émotionnelle | 1.1 | **fort** |
| Appel / visio | Antenne en toiture, petit bureau | Combiné / caméra | Communication, Écoute | 0.7 | moyen |
| Mails / messagerie | Bureau de poste | Enveloppe | Synthèse, Rédaction | 0.6 | fort (génère) |
| Recherche / veille | Bibliothèque, coupole | Loupe | Analyse, Concentration, Curiosité | 1.0 | faible |
| Science / laboratoire | Bâtiment à sheds, cheminées blanches | Fiole | Analyse, Rigueur, Méthode | 1.1 | faible |
| Rédaction / conception | Atelier à verrière | Plume | Créativité, Concentration, Rédaction | 1.0 | faible |
| Création / design | Atelier d'artiste, façade colorée | Palette | Créativité, Sens esthétique | 0.9 | faible |
| Développement / technique | Data-center, façade à LED | Chevrons `</>` | Logique, Concentration, Résolution | 1.1 | faible |
| Management / RH | Bâtiment à colonnades | Organigramme | Leadership, Écoute, Décision | 1.2 | fort |
| Formation donnée | Amphithéâtre | Tableau | Pédagogie, Communication | 1.1 | moyen |
| Travaux / chantier | Chantier, grues, palissades | Casque | Effort physique, Coordination, Sécurité | 1.2 | moyen |
| Ouvrier / manutention | Hangar, quais de chargement | Clé | Effort physique, Endurance, Précision | 1.2 | faible |
| Industrie / production | Usine, cheminées, silos | Engrenage | Vigilance, Endurance, Procédure | 1.1 | moyen |
| Soin / service à la personne | Bâtiment à croix, auvent | Cœur | Empathie, Gestion émotionnelle, Endurance | 1.3 | moyen |
| Commerce / vente (côté vendeur) | Galerie marchande | Caisse | Relationnel, Endurance | 1.0 | fort |
| Déplacement pro | Gare, aérogare | Valise | Organisation | 0.6 | — (route) |
| Deadline / livraison | Bâtiment avec horloge en façade | Sablier | (hérite) ×1.3 | **×1.3** | — |

### 2.2 Sphère PERSONNELLE — *Ville basse, rues commerçantes, quartiers résidentiels*

| Sous-type | Architecture | Enseigne | Compétences | Charge |
|---|---|---|---|---|
| **Courses** — supermarché | Hangar commercial, parking | Caddie | Organisation | 0.5 |
| Boulangerie / marché | Échoppe, étals | Pain / panier | — | 0.2 |
| Pharmacie | Échoppe | Croix verte | — | 0.2 |
| Coiffeur / esthétique | Boutique, store rayé | Ciseaux | — | 0.2 |
| Banque / poste / guichet | Petit bâtiment à fronton | € / enveloppe | Patience, Rigueur | 0.6 |
| Administration (préfecture, impôts, CAF) | Bâtiment officiel, drapeau | Tampon | Rigueur, Patience | 0.9 |
| Garage / réparation | Atelier, pont élévateur | Clé plate | — | 0.4 |
| **Santé** — médecin généraliste | Cabinet, plaque | Stéthoscope | Gestion émotionnelle | 0.5 |
| Spécialiste / examen | Clinique | Croix | Gestion émotionnelle | 0.7 |
| Hôpital / intervention | Hôpital | H | Gestion émotionnelle, Endurance | 1.3 |
| Dentiste | Cabinet | Dent | — | 0.6 |
| Kiné / thérapie | Cabinet, jardinet | Main | — | 0.3 |
| Psy / accompagnement | Maison, jardin clos | Bulle | Introspection | 0.8 |
| **Formation reçue** — école / cours | École, préau | Livre | Concentration, Mémoire | 0.9 |
| Université / examen | Faculté, amphithéâtre | Toque | Concentration, Mémoire, Gestion du stress | 1.3 |
| Apprentissage autonome | Bibliothèque de quartier | Livre ouvert | Concentration, Curiosité | 0.8 |
| **Sport** — cardio | Stade, piste | Chaussure | Endurance | 0.7 (physique) |
| Musculation / salle | Gymnase | Haltère | Effort physique | 0.7 (physique) |
| Collectif | Terrain, tribunes | Ballon | Coopération, Endurance | 0.7 |
| Aquatique | Piscine (en bord de lagon) | Vague | Endurance | 0.6 |
| Yoga / méditation | Pavillon, jardin zen | Lotus | Attention | **récupérant** |
| **Détente** — lecture | Kiosque | Livre | — | récupérant |
| Jeu / écran | Salle d'arcade | Manette | — | neutre |
| Création perso (musique, écriture, art) | Atelier, maison d'artiste | Note | Créativité | 0.3 / nourrissant |
| Sieste | Hamac, plage | Lune | — | récupérant |
| **Maison** — ménage / entretien | Maison avec seau | Balai | Effort physique | 0.5 |
| Cuisine | Maison, cheminée fumante | Casserole | Organisation | 0.4 |
| Bricolage / jardinage | Cabane, potager | Marteau / pousse | Effort physique, Précision | 0.5 |
| Administratif perso (factures, papiers) | Petit bureau domestique | Facture | Rigueur | 0.8 |

### 2.3 Sphère FAMILIALE — *Quartier résidentiel, maisons, équipements de proximité*

| Sous-type | Architecture | Enseigne | Compétences | Charge |
|---|---|---|---|---|
| Trajet école / crèche | Abribus, passage piéton | Cartable | Organisation, Ponctualité | 0.5 |
| Devoirs / accompagnement scolaire | Maison + préau | Crayon | Pédagogie, Patience | 0.8 |
| Activité enfant (club, sport) | Équipement sportif junior | Étoile | Organisation | 0.4 |
| Repas familial | Maison, table en terrasse | Couverts | — | nourrissant |
| Soin d'un proche / aidant | Maison à croix | Main tendue | Empathie, Endurance, Gestion émotionnelle | 1.2 |
| Rendez-vous médical enfant/proche | Cabinet | Stéthoscope + cœur | Gestion émotionnelle | 0.6 |
| Visite famille | Maison avec jardin | Maison | Relationnel | 0.4 – 0.9 (variable) |
| Événement familial (anniversaire, fête) | Maison illuminée, guirlandes | Cadeau | Organisation | 0.6 |
| Gestion du foyer (budget, planning) | Bureau domestique | Calendrier | Organisation, Rigueur | 0.7 |
| Garde / temps parent | Parc de jeux | Pelle et seau | Vigilance, Patience | 0.6 |

### 2.4 Sphère SOCIALE — *Place centrale, terrasses, salles*

| Sous-type | Architecture | Enseigne | Compétences | Charge |
|---|---|---|---|---|
| Café / verre entre amis | Terrasse de café | Tasse | — | nourrissant |
| Repas / restaurant | Restaurant, auvent | Fourchette | — | nourrissant |
| Sortie culturelle | Théâtre, cinéma, musée | Masque / pellicule | — | nourrissant |
| Concert / festival | Scène, chapiteau | Note | — | 0.4 |
| Association / bénévolat | Maison des associations | Mains | Coopération, Organisation | 0.7 |
| Engagement civique | Mairie, place | Urne | Communication | 0.7 |
| Culte / spiritualité | Édifice dédié (neutre, générique) | Flamme | — | récupérant |
| Événement réseau / pro-social | Salle de réception | Badge | Relationnel | 0.9 |
| Rendez-vous amoureux | Belvédère, ponton sur le lagon | Cœur | — | nourrissant |

### 2.5 Transversal — DÉPLACEMENT — *Routes, ponts, gares*

| Sous-type | Rendu |
|---|---|
| Trajet quotidien (domicile-travail) | Route périphérique de l'anneau, trafic proportionnel à la durée |
| Transport en commun | Ligne de tram / train sur l'anneau |
| Voiture | Route + véhicules |
| Vélo / marche | Piste cyclable, sentier |
| Voyage (train, avion) | Gare / aérogare ; l'avion traverse le ciel de l'atoll |
| Voyage multi-jours | L'atoll entier change de biome (nouveau décor), port d'arrivée |

### 2.6 Transversal — REPOS & SUSPENSIONS — *Le vide est une information*

| Sous-type | Rendu |
|---|---|
| Pause courte (5-15 min) | Banc, fontaine |
| Pause café | **Café en terrasse** |
| Marche / respiration | **Parc** avec allée |
| Déjeuner | Place avec arbres et tables |
| Temps non planifié | Sable / dunes (terrain libre, constructible) |
| Vacances / congés | Plage, ponton, voiliers sur le lagon ; l'atoll devient « île de vacances » |
| Maladie / arrêt | Brume douce, bâtiments fermés, volets clos |
| Jour férié | Pavoisement, drapeaux |

---

## 3. Compétences — le modulateur du quartier

### 3.1 Principe
L'utilisateur choisit **au maximum 3 compétences** par tâche. Le type en **propose** (pré-cochées), l'utilisateur ajuste ou crée les siennes.

Le maximum de 3 est juste : il force à hiérarchiser et garde le rendu lisible (3 matériaux max par bâtiment).

### 3.2 Les 5 réservoirs d'énergie
Chaque compétence puise dans un ou plusieurs **réservoirs**. C'est ce qui rend le moteur de charge crédible (une journée de chantier ne fatigue pas comme une journée de négociation).

| Réservoir | Couleur / matériau | Compétences typiques |
|---|---|---|
| **Cognitif** (attention, mémoire de travail) | Verre / bleu acier | Concentration, Analyse, Logique, Rigueur, Mémoire, Synthèse, Rédaction |
| **Relationnel** (exposition sociale) | Brique / terracotta | Communication, Écoute, Négociation, Pédagogie, Leadership, Relationnel |
| **Émotionnel** (régulation) | Pierre rose / ocre | Empathie, Gestion émotionnelle, Gestion du stress, Patience, Décision |
| **Physique** | Béton / métal brut | Effort physique, Endurance, Précision manuelle, Vigilance |
| **Créatif** (divergence) | Bois / couleurs vives | Créativité, Curiosité, Sens esthétique, Imagination |

Une tâche peut aussi être **récupérante** pour un réservoir (sport → recharge cognitif mais vide physique ; café entre amis → recharge émotionnel, coûte un peu de relationnel pour les introvertis — **paramétrable par profil**).

### 3.3 Effet sur le rendu
- Matériau des façades = réservoir dominant.
- Bâtiment « mixte » si 2-3 réservoirs différents → architecture composite (socle brique + tour de verre).
- Un réservoir qui s'épuise sur la journée → les bâtiments correspondants se **patinent / se fissurent** progressivement (signal tardif, subtil).

---

## 4. Moteur de charge & de trafic — la logique qui manque à tous les agendas

C'est **le service structurel principal** d'Attol. Sans lui, l'app n'est qu'un skin 3D.

### 4.1 Charge d'une tâche
```
charge(t) = durée × intensité(type) × Σ poids(compétences)
            × enjeu (1.0 / 1.2 / 1.5)
            × incertitude (1.0 si connue, 1.2 si « première fois »)
            × échéance (1.3 si deadline le jour même)
```

### 4.2 Fatigue cumulée (par réservoir)
Courbe de remplissage/vidange par réservoir sur la journée :
```
niveau_r(h+Δ) = niveau_r(h) − coût_r(tâche) × Δ + récupération_r(pause) × Δ
```
+ modulation **chronotype** (matinal / vespéral, déclaré ou appris) : une tâche cognitive à 14h coûte plus à un matinal.

### 4.3 Coût de bascule (context switching)
Entre deux tâches consécutives A → B :
```
bascule(A,B) = distance(compétences_A, compétences_B) × (1 − pause_entre/15min)
```
→ rendu : **carrefour encombré** entre les deux bâtiments. 6 tâches différentes en 2h = quartier saturé de carrefours.

### 4.4 Trafic généré (les ponts)
Certaines tâches **produisent** du travail ailleurs dans la journée ou la semaine :

| Tâche source | Génère | Rendu |
|---|---|---|
| Réunion | Compte-rendu, actions, mails | Pont vers le créneau de suivi, flux de véhicules |
| Contact client | Devis, relance, suivi | Pont + camions de livraison |
| Mails | Réponses, micro-tâches | Pluie de petits véhicules |
| Rendez-vous médical | Pharmacie, examen, nouveau RDV | Pont vers commerce |
| Courses | Cuisine | Petit pont |
| Voyage | Préparation (veille), récupération (lendemain) | Ponts vers les atolls voisins |

- **Pont** = dépendance (explicite saisie, ou implicite inférée par type).
- **Densité du trafic sur le pont** = volume de travail induit non encore planifié.
- **Embouteillage** = travail induit > capacité libre restante.

C'est ici que la promesse « comprendre ce que les tâches génèrent les unes sur les autres » devient tangible : **une réunion n'occupe pas 1h, elle occupe 1h + son sillage**.

### 4.5 Seuils & signaux
| Indicateur | Vert | Orange | Rouge |
|---|---|---|---|
| Charge totale / capacité du jour | < 70 % | 70-95 % | > 95 % |
| Réservoir au plus bas | > 40 % | 20-40 % | < 20 % |
| Bascules / heure | ≤ 1 | 2-3 | ≥ 4 |
| Plus long bloc sans pause | < 90 min | 90-150 | > 150 |
| Travail induit non planifié | 0 | < 1h | > 1h |

### 4.6 Conseiller (aide à l'organisation)
Règles d'abord (explicables), apprentissage ensuite :
1. **Insertion de respiration** : bloc > 90 min → propose un *café en terrasse* (10 min) ; réservoir cognitif < 30 % → propose un *parc* (marche 20 min).
2. **Regroupement** (*batching*) : 3 appels dispersés → les regrouper en un « quartier d'appels ».
3. **Déplacement** : tâche à haute charge cognitive placée au creux du chronotype → propose un créneau meilleur.
4. **Report** : journée > 100 % → propose les tâches *déplaçables* à plus faible enjeu ; montre l'atoll de demain qui grandit.
5. **Planifier le sillage** : réunion sans créneau de suivi → propose un bloc de 30 min ensuite.
6. **Protection** : sommeil, repas, temps familial marqués « sanctuaires » → jamais proposés au déplacement.

Chaque suggestion est **un fantôme** posé sur l'atoll (bâtiment translucide) : on accepte d'un tap, on refuse d'un glissement.

---

## 5. Génération procédurale

Tout est **déterministe** (graine = date + id utilisateur) : le même jour ressemble toujours à lui-même → mémoire spatiale, l'utilisateur reconnaît « son mardi ».

| Élément | Méthode |
|---|---|
| **Parcelles** | Secteurs annulaires subdivisés, congés arrondis (géométrie paramétrique) |
| **Bâtiments** | Grammaire de formes (*shape grammar*) par style : socle + corps + couronnement ; hauteur = charge, emprise = durée ; kit de modules instanciés (≈ 30-50 pièces par style) |
| **Routes** | Anneau périphérique (trajets) + rues radiales entre parcelles ; largeur = flux ; carrefours aux bascules |
| **Ponts** | Courbes de Bézier au-dessus du lagon entre deux parcelles liées ; type de pont selon nature (passerelle = léger, pont routier = fort trafic, pont suspendu = dépendance critique) |
| **Parcs** | Remplissage des parcelles libres / pauses par distribution Poisson-disk d'arbres, allées par chemins lissés |
| **Panneaux de façade** | Système modulaire : *slot* d'enseigne (icône du sous-type) + *slot* d'info (heure, lieu, personne) + *slot* d'état (retard, terminé ✓, déplacé ↺) |
| **Trafic** | Particules instanciées le long des splines de route ; densité = flux ; couleur = sphère ; animation seulement à l'écran actif |
| **Lagon** | Shader d'eau simple (normal map animée), reflets selon heure ; bateaux = tâches flottantes (non datées / inbox) |

### 5.1 Jour / nuit
- La lumière suit **l'heure réelle** ; le soleil tourne autour de l'atoll → l'ombre portée pointe vers « maintenant ».
- **Fenêtres allumées** = tâches à venir ; **fenêtres éteintes** = tâches faites. Le soir, l'atoll montre littéralement ce qui reste.
- Véhicules : phares la nuit.
- **Indicateur « maintenant »** : un phare (*lighthouse*) au bord du lagon dont le faisceau pointe sur l'heure courante. Élément signature.

### 5.2 Météo réelle (arrière-plan)
Ciel, nuages, pluie, vent sur le lagon via API météo localisée.
**Règle :** la météo réelle reste dans le *ciel et l'eau*, jamais sur les bâtiments — sinon conflit avec la grammaire de charge (la « tempête » ne doit pas être confondue avec une surcharge).
Usage utile : une course à pied à 17h + pluie prévue → micro-alerte sur le stade.

---

## 6. Vues, zoom, LOD sémantique

| Niveau | Vue | Ce qu'on voit | Ce qu'on fait |
|---|---|---|---|
| **L0 — Archipel** | Semaine / mois | 7 atolls, taille et hauteur moyenne, couleur de charge | Repérer la journée critique, déplacer entre jours |
| **L1 — Atoll** | Journée entière | Silhouettes, hauteurs, ponts, parcs, phare ; aucun texte sauf heures cardinales | Lecture « d'un regard » |
| **L2 — Quartier** | Matin / après-midi / soir / sur-heure | Styles architecturaux, enseignes, trafic, carrefours | Réorganiser, accepter les suggestions |
| **L3 — Parcelle** | 1-2 heures | Panneaux de façade lisibles, personnes, lieu, compétences | Éditer finement |
| **L4 — Fiche** | Feuille modale | Formulaire, notes, pièces jointes, sillage | Saisie, détails |

**LOD = sémantique, pas seulement géométrique** : à chaque niveau on change *ce qui est dit*, pas juste le nombre de polygones. Au L1, un bâtiment est un volume coloré ; au L3, c'est une façade avec une enseigne.

---

## 7. Interface & expérience mobile

### 7.1 Écran principal (portrait)
```
┌────────────────────────────┐
│ Ven 10 oct.   ☁ 14°   ⚑ 87%│  ← date, météo, jauge de charge du jour
│                            │
│        [ ATOLL 3D ]        │  ← 70 % de l'écran
│                            │
│ ◔ cog ◑ rel ◕ emo ◔ phy ◑ cr│  ← 5 réservoirs (mini-jauges)
├────────────────────────────┤
│ ▸ Maintenant : Réunion RH  │  ← bandeau contextuel (tâche courante / suivante)
│   ensuite : café 10'  (💡) │
├────────────────────────────┤
│  ◀ jour   [ + ]   semaine ▶│  ← barre d'action, bouton central = créer
└────────────────────────────┘
```

### 7.2 Gestes
| Geste | Action |
|---|---|
| Pincer | Changer de niveau L0 ↔ L3 |
| Glisser horizontal (L1) | Jour précédent / suivant |
| Rotation deux doigts | Tourner l'atoll par crans de 3 h |
| Tap bâtiment | Panneau rapide (titre, heure, compétences) |
| Appui long bâtiment | **Saisir** pour déplacer |
| Glisser bâtiment sur l'anneau | Replanifier (aperçu en direct) |
| Glisser bâtiment dans le lagon | **Mettre en flottaison** (inbox, non daté) |
| Glisser bâtiment hors écran → atoll voisin | Reporter |
| Tap sur terrain libre (sable) | Créer une tâche à ce créneau |

### 7.3 Replanification — « voir l'effet immédiat »
Le moment le plus important de l'app.
1. Appui long → le bâtiment se soulève, les autres deviennent semi-transparents.
2. Pendant le glissement : **simulation en direct** — hauteurs voisines recalculées, ponts qui se tendent, carrefours qui se vident ou se remplissent, jauge de charge qui bouge.
3. Un **delta chiffré** flotte : `Charge 94 % → 78 %  ·  bascules −2  ·  sillage OK`.
4. Les créneaux « bons » s'illuminent doucement en vert sous le doigt (aimantation).
5. Lâcher → animation de construction (le bâtiment se pose, échafaudage → fini en 300 ms). **Annuler** toujours disponible (toast + secouer).

Mode **« Et si… »** : bouton qui duplique la journée en bac à sable ; on peut tout casser, comparer avant/après côte à côte, puis valider ou jeter.

### 7.4 Création de tâche (objectif : < 5 s)
1. Tap sur le sable (créneau pré-rempli) ou bouton `+`.
2. **Champ unique en langage naturel** : « RDV dentiste jeudi 14h30 » → type, sous-type, durée, enseigne inférés.
3. Ruban de **3 compétences proposées** (pré-cochées, modifiables).
4. Curseur d'**enjeu** (faible / normal / fort) — optionnel.
5. Valider → le bâtiment se construit sous les yeux.

Les champs avancés (lieu, personnes, récurrence, sillage, sanctuaire, déplaçable oui/non) restent repliés.

### 7.5 Accessibilité & alternatives (non négociable)
- **Mode Plan 2D** : même atoll vu à plat, en anneau coloré. Pour les appareils faibles, la batterie basse, et la lecture rapide.
- **Mode Liste** : agenda classique synchronisé. L'utilisateur ne doit jamais être prisonnier de la métaphore.
- Palettes daltonisme-sûres (la charge est aussi encodée par la hauteur, jamais par la couleur seule).
- `prefers-reduced-motion` : trafic figé, transitions coupées.
- Lecteur d'écran : résumé textuel généré (« Journée chargée à 87 %. Pic entre 10h et 12h : 3 réunions. Aucune pause avant 13h. »).
- Grandes zones de toucher (les petits bâtiments ont une zone de tap élargie).

### 7.6 Ton du conseiller
Jamais culpabilisant. Pas de « vous êtes en retard ». Plutôt : « Le pont de 15h est encombré — un café de 10 min avant soulagerait le quartier. » La métaphore urbaine permet de parler de la charge **sans parler de l'utilisateur**. C'est une force à exploiter.

### 7.7 Widgets & surfaces hors app
- Widget écran d'accueil : mini-atoll 2D + jauge + prochaine tâche.
- Notification de pause : une icône de café/parc, pas un texte injonctif.
- Montre : réservoirs en anneaux concentriques.

---

## 8. Ce qui manque structurellement (services & logique)

Classé par criticité.

### Indispensable dès le MVP
| Manque | Pourquoi |
|---|---|
| **Synchronisation calendrier** (Google, Apple/CalDAV, Outlook/Graph, import .ics) | Personne ne ressaisit son agenda. Sans import, pas d'adoption. Mapping automatique événement → type (mots-clés, lieu, participants). |
| **Local-first + hors-ligne** | Un agenda doit marcher sans réseau. Base locale (SQLite) + synchro. |
| **Tâches non datées (inbox / lagon)** | Beaucoup de « tâches » n'ont pas d'heure. Elles flottent sur le lagon jusqu'à ce qu'on les amarre. |
| **Récurrence** (RRULE) | Sport tous les mardis, réunion hebdo. |
| **Rappels / notifications** | Fonction de base d'un agenda. |
| **Contraintes dures vs souples** | « Déplaçable : oui/non », « sanctuaire ». Sans ça, le conseiller propose des absurdités. |
| **Annuler / historique** | Replanification par glisser = erreurs fréquentes. |

### Nécessaire pour tenir la promesse
| Manque | Pourquoi |
|---|---|
| **Temps de trajet** (API cartographique) | Un RDV à 14h à 20 min de route crée une route occupée de 13h40 à 14h. Le trajet est une charge invisible. |
| **Boucle estimé / réel** | Fin de tâche : « c'était plus / moins lourd que prévu ? » (1 tap). Calibre les coefficients personnels. Sans retour, le moteur de charge reste théorique. |
| **Profil énergétique** | Chronotype, heures de sommeil, sensibilité sociale (introverti/extraverti), capacité journalière de référence. |
| **Bilan de journée / semaine** | Le soir : l'atoll « vécu » vs l'atoll « prévu ». Hebdo : archipel avec tendances. |
| **Moteur de planification** (solveur de contraintes) | Pour les suggestions de déplacement : recherche de créneaux respectant dures/souples, minimisant charge et bascules. Heuristique d'abord, OR-Tools/CP-SAT plus tard si besoin. |
| **Météo API** | Ciel + alertes pour activités extérieures. |

### Différenciants / v2
| Manque | Pourquoi |
|---|---|
| **Atolls partagés (famille / équipe)** | Voir l'atoll du conjoint pour les trajets d'école ; ponts entre atolls = tâches partagées. Énorme pour la sphère familiale. |
| **Projets multi-jours** | Un projet = un quartier qui se construit sur plusieurs atolls (chantier qui avance). |
| **Objectifs / habitudes** | Sport 3×/semaine = phare secondaire, parc qui s'agrandit avec la régularité. |
| **Wearables** (sommeil, fréquence cardiaque) | Ajuster la capacité du jour selon la nuit réelle. Strictement optionnel. |
| **Assistant conversationnel** | « Allège mon jeudi » → propositions sur l'atoll. |

### Cadre légal & confiance
- **RGPD** : les RDV médicaux, psy, culte, sont des **données sensibles** (art. 9). Chiffrement local, consentement explicite, aucune analyse serveur de ces catégories par défaut.
- Export complet (ICS + JSON), suppression de compte réelle.
- Pas de revente, pas de pub. Le modèle économique doit être clair (freemium : styles, archipel partagé, conseiller avancé).

---

## 9. Architecture logicielle

### 9.1 Principe : séparer **le moteur** du **rendu**
```
┌────────────────────────────────────────────────────────────┐
│  APP MOBILE                                                │
│  ┌──────────────┐  ┌─────────────────┐  ┌───────────────┐  │
│  │ UI (écrans,  │  │ RENDU           │  │ Mode 2D /     │  │
│  │ fiches,      │  │ 3D atoll        │  │ Liste         │  │
│  │ gestes)      │  │ (scène, LOD,    │  │               │  │
│  └──────┬───────┘  │ instancing)     │  └──────┬────────┘  │
│         │          └────────┬────────┘         │           │
│         └───────────┬───────┴──────────────────┘           │
│              ┌──────▼────────────────────────┐             │
│              │ CŒUR (pur, testable, partagé) │             │
│              │ · Modèle (tâches, types, comp)│             │
│              │ · Moteur de charge & trafic   │             │
│              │ · Conseiller (règles)         │             │
│              │ · Générateur procédural       │             │
│              │   → produit un « plan de      │             │
│              │     ville » (données, pas 3D) │             │
│              └──────┬────────────────────────┘             │
│              ┌──────▼──────┐                               │
│              │ Stockage    │  SQLite local + file de synchro│
│              └──────┬──────┘                               │
└─────────────────────┼──────────────────────────────────────┘
                      │
┌─────────────────────▼──────────────────────────────────────┐
│  SERVICES                                                  │
│  · Synchro & compte      · Connecteurs calendrier (OAuth)  │
│  · Météo (proxy/cache)   · Trajets (proxy/cache)           │
│  · Notifications push    · Partage d'atolls (v2)           │
└────────────────────────────────────────────────────────────┘
```

Le générateur produit un **plan de ville** (JSON : parcelles, bâtiments {style, hauteur, modules}, routes, ponts, parcs, panneaux). Le rendu 3D, le mode 2D et les tests consomment le même plan. On peut changer de moteur graphique sans toucher à la logique.

### 9.2 Choix techniques recommandés
| Couche | Recommandation | Alternative |
|---|---|---|
| Prototype de validation | **Web (Three.js) en PWA** — itération ultra-rapide sur la lisibilité | — |
| App | **React Native (Expo) + react-three-fiber / expo-gl** ; cœur en TypeScript partagé avec le prototype | Flutter + moteur 3D ; Unity (plus lourd, meilleur rendu, UI moins native) |
| Assets | Kit modulaire low-poly (glTF), une texture atlas par style, instancing | — |
| Stockage | SQLite (expo-sqlite / WatermelonDB) | — |
| Synchro | Backend léger (Supabase / Postgres) ; CRDT seulement si partage d'atolls | — |
| Calendriers | Google Calendar API, CalDAV, Microsoft Graph, ICS | — |
| Budget perf | 60 fps sur milieu de gamme 2022, < 150 draw calls, < 80 k triangles à L1, rendu à la demande (pas de boucle continue au repos) | — |

### 9.3 Modèle de données (esquisse)
```ts
type Sphere = 'pro' | 'perso' | 'famille' | 'social' | 'deplacement' | 'repos';
type Reservoir = 'cognitif' | 'relationnel' | 'emotionnel' | 'physique' | 'creatif';

interface TaskType {
  id: string;               // 'pro.reunion'
  sphere: Sphere;
  label: string;
  archStyle: string;        // 'tour_verre'
  signIcon: string;         // 'table_ronde'
  defaultSkills: string[];  // ≤ 3
  baseIntensity: number;    // 0.2 – 1.5 ; < 0 = récupérant
  generates?: { typeId: string; ratio: number }[]; // sillage : 0.5 = 30 min par heure
}

interface Skill {
  id: string;
  label: string;
  costs: Partial<Record<Reservoir, number>>; // négatif = recharge
  custom?: boolean;
}

interface Task {
  id: string;
  title: string;
  typeId: string;
  start?: string;           // ISO ; absent = flotte sur le lagon
  durationMin: number;
  skills: string[];         // max 3 — validé
  stakes: 'faible' | 'normal' | 'fort';
  movable: boolean;
  sanctuary: boolean;
  deadline?: string;
  location?: { label: string; lat?: number; lng?: number };
  people?: string[];
  dependsOn?: string[];     // ponts explicites
  recurrence?: string;      // RRULE
  source?: { provider: 'google' | 'caldav' | 'graph' | 'ics'; externalId: string };
  feedback?: { felt: -1 | 0 | 1; actualDurationMin?: number };
}

interface DayPlan {          // sortie du générateur
  date: string;
  seed: number;
  load: { total: number; byReservoir: Record<Reservoir, number[]> };
  parcels: Parcel[];
  buildings: Building[];
  roads: Road[];
  bridges: Bridge[];
  parks: Park[];
  suggestions: Suggestion[];
}
```

---

## 10. Feuille de route

| Phase | Contenu | Critère de sortie |
|---|---|---|
| **0 — Preuve de lisibilité** (3-4 sem.) | Prototype web : atoll 2.5D, 6 styles, hauteur = charge, ponts statiques, glisser-replanifier avec delta. Données fictives. | **Test des 3 secondes** : 8 personnes sur 10 identifient la journée la plus chargée et le pic horaire sans explication. |
| **1 — MVP mobile** | Import Google/Apple, 4 sphères × sous-types principaux, compétences ≤ 3, moteur de charge v1, conseiller règles 1-2-5, L1/L2/L4, mode Liste, hors-ligne. | Usage 5 jours / 7 pendant 2 semaines chez 20 testeurs. |
| **2 — V1** | Jour/nuit, météo, trafic animé, enseignes complètes, L3, archipel semaine, boucle estimé/réel, trajets, widgets, mode « Et si ». | Rétention J30. |
| **3 — V2** | Atolls partagés famille/équipe, projets multi-jours, habitudes, wearables, assistant. | — |

---

## 11. Points de désaccord / à trancher

1. **Anneau vs grille.** L'anneau est plus poétique et lisible (horloge), mais les créneaux du soir et du matin sont *proches* sur un anneau de 24h alors qu'ils sont éloignés dans le vécu. Avec la nuit compressée en récif, le problème disparaît en grande partie. **Position Nomen : anneau, nuit compressée.**
2. **Hauteur = charge OU durée ?** Tentation de mélanger. **Non : durée = emprise, charge = hauteur.** Sinon une longue tâche légère (lecture 3h) paraît écrasante.
3. **Combien de styles ?** La liste ci-dessus en contient ~70. Pour le MVP : **~15 styles d'architecture**, la variété fine passe par les **enseignes** (beaucoup moins chères à produire qu'un bâtiment).
4. **Le trafic doit signifier une seule chose.** Proposition : trafic = *travail induit* (sillage). Les bascules sont des *carrefours*, pas du trafic. Sinon l'utilisateur ne sait plus quoi corriger.
5. **Gamification ?** Tentation d'ajouter des points, des récompenses. **Non** : la ville elle-même est la récompense (un atoll équilibré est beau). Pas de badges.

---

*Edel & Nomen — Attol, dossier de conception v0.1*
