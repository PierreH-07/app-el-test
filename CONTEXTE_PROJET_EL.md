# Contexte — Application Eau Libre (FFN · Centre de la Performance)

> Document de référence à placer dans les fichiers du projet.  
> À partager avec Claude en début de chat pour reprendre le développement sans perte de contexte.  
> Dernière mise à jour : 01/05/2026

---

## 1. Vue d'ensemble du projet

Application web d'analyse de natation en eau libre, développée pour le Centre de la Performance de la FFN. Hébergée sur **GitHub Pages** (pas de backend, pas de base de données). Tous les fichiers sont statiques.

**URL de production :** `https://pierreh-07.github.io/app-el/`

**Stack technique :**
- HTML / CSS / JS vanilla (pas de framework)
- Thème sombre "marine" (voir palette ci-dessous)
- Données : fichiers JSON externes chargés via `fetch()`
- Hébergement : GitHub Pages (dépôt `pierreh-07/app-el`)

---

## 2. Architecture des fichiers

```
app-el/
├── index.html                      ← page d'accueil, liens vers toutes les applis
├── Analyse_KO_Eau_libre.html       ← analyse résultats courses KO (3km Sprint)
├── analyse_course_el.html          ← analyse résultats courses 10km
├── Ponton10km.html                 ← ponton startlist 10km (avec météo)
├── ponton_ko_ibiza2026.html        ← ponton startlist KO Ibiza 2026 (modèle)
├── video_analyse_el.html           ← outil tagging vidéo (bureau)
├── observer_el.html                ← outil observation terrain (mobile)
├── data/
│   ├── resultats_ko_el.json        ← SOURCE DE VÉRITÉ courses KO
│   └── resultats_10km_el.json      ← SOURCE DE VÉRITÉ courses 10km
└── startlists/
    └── startlist_golfo26_h.json    ← exemple format startlist
```

### Règle fondamentale d'architecture

Les fichiers HTML **ne contiennent plus de données embarquées**. Ils chargent les JSON via `fetch()` au chargement de la page. Cela ne fonctionne qu'avec un serveur HTTP (GitHub Pages ✓, Live Server VS Code ✓). **Ne fonctionne pas en `file://`.**

```js
// Pattern standard de chargement dans tous les fichiers HTML
fetch('data/resultats_ko_el.json')
  .then(r => r.json())
  .then(json => {
    var DATA  = json.KO_DATA;
    var HF    = json.KO_HF;
    var HH    = json.KO_HH;
    var FLAGS = json.KO_FLAGS;
    // ... tout le code JS qui utilise ces variables
  })
  .catch(e => {
    document.body.innerHTML = '<p style="color:red;padding:2rem">Erreur chargement : ' + e + '</p>';
  });
```

---

## 3. Les deux fichiers JSON — Sources de vérité

### 3.1 `data/resultats_ko_el.json`

Structure de premier niveau :
```json
{
  "KO_DATA": { ... },   // résultats de chaque course KO
  "KO_HF":  { ... },   // historique individuel femmes
  "KO_HH":  { ... },   // historique individuel hommes
  "KO_FLAGS": { ... }  // emojis drapeaux par NOC
}
```

**Convention de nommage des clés dans `KO_DATA` :**
```
{ville}{année2chiffres}_{genre}
Exemples : golfo26_f · ibiza26_h · singapour25_f · starigrad25_h
```

**Structure d'une course dans `KO_DATA` :**
```json
"ibiza26_f": {
  "conditions": "17.3°C · Vent NE 18.2 km/h rafales 42.8 km/h",
  "label": "CdM Ibiza 2026",
  "date": "25/04/2026",
  "genre": "F",
  "type": "CDM",           // CDM | CHM | CHE | JO
  "nb_series": 2,
  "nq_par_serie": 10,
  "fastest_serie": 2,
  "stats_series": {
    "1": {
      "1er": "18:20.1",
      "dernier_q": "18:25.6",
      "premier_elim": "18:26.6",
      "ecart_1_q": 5.5,
      "marge_coupure": 1.0,
      "nb_starts": 28,
      "nb_qualifies": 10
    }
  },
  "series": { "1": [ /* nageurs */ ], "2": [ /* nageurs */ ] },
  "placement_demi": [ /* ordre de départ */ ],
  "demi": [ /* résultats */ ],
  "placement_finale": [ /* ordre de départ */ ],
  "finale": [ /* résultats */ ]
}
```

**Structure d'un nageur dans une série/demi/finale KO :**
```json
{
  "rang": 1,
  "nom": "FABIAN Bettina",
  "noc": "HUN",
  "temps": "18:20.1",
  "temps_s": 1100.1,
  "ecart_s": 0.0,
  "qualifie": true,
  "t1500": 987.39,    // temps 1er tour en secondes (optionnel)
  "t800": 517.33,     // temps tour 800m en secondes (optionnel)
  "t400": 248.86,     // temps tour 400m en secondes (optionnel)
  "rv": 13,           // rang virtuel (classement mondial au moment de la course)
  "di": 12            // delta index = rv - rang obtenu (positif = surperformance)
}
```

**Structure d'un nageur dans `KO_HF` / `KO_HH` :**
```json
"FABIAN Bettina": [
  {
    "l": "CdM Ibiza 2026",   // label lisible
    "d": "2026",              // année
    "t": "CDM",               // type : CDM | CHM | CHE | JO
    "r": 4,                   // rang final (finale, ou série si pas de finale)
    "n": 48,                  // nombre total de participants
    "rv": 13,                 // rang virtuel
    "di": 12                  // delta index
  }
]
```

**⚠️ Point critique — Correspondance des noms :**
Les clés dans `KO_HF`/`KO_HH` doivent correspondre **exactement** au champ `nom` de la startlist pour que l'historique s'affiche dans le ponton. Problèmes fréquents :
- PDF tronque les prénoms : `BRANDT DE MACEDO L.` → compléter en `BRANDT DE MACEDO Leonard`
- Doublons de clés : `DE VALDES Maria` et `de VALDES Maria` peuvent coexister → vérifier
- Accents et tirets : vérifier la cohérence entre les sources

---

### 3.2 `data/resultats_10km_el.json`

Structure analogue au JSON KO. Les sections principales sont (à confirmer selon la structure exacte du fichier généré) :

```json
{
  "DATA_F": { ... },   // résultats par course 10km femmes
  "DATA_H": { ... },   // résultats par course 10km hommes
  "HF": { ... },       // historique individuel femmes 10km
  "HH": { ... }        // historique individuel hommes 10km
}
```

**Convention de nommage des clés :** même convention que le KO (`golfo26_h`, etc.)

**Structure d'un nageur dans une course 10km :**
```json
{
  "rang": 1,
  "nom": "VELLY Sacha",
  "noc": "FRA",
  "temps": "1:49:07.5",
  "temps_s": 6547.5,
  "ecart_s": 0.0,
  "t_1666": 1061.5,   // temps cumulé au passage 1666m en secondes
  "t_3333": 2148.4,
  "t_5000": 3262.9,
  "t_6666": 4373.6,
  "t_8333": 5463.3,
  "rv": null,         // rang virtuel (peut être null si non renseigné)
  "di": null,         // delta index (peut être null)
  "statut": "FIN"     // FIN | DNF | DNS | DSQ
}
```

---

## 4. Les fichiers d'analyse

### 4.1 `Analyse_KO_Eau_libre.html`

Outil d'analyse des courses KO (3km Sprint). Charge `data/resultats_ko_el.json`.

**Variables JS utilisées après le fetch :**
```js
var DATA  = json.KO_DATA;   // données de toutes les courses
var HF    = json.KO_HF;     // historique femmes
var HH    = json.KO_HH;     // historique hommes
var FLAGS = json.KO_FLAGS;  // drapeaux
var GENRE = 'F';            // genre actif ('F' ou 'H')
var COMP  = null;           // course sélectionnée
```

**Fonctionnalités :**
- Sélecteur genre F/H
- Sélecteur de course (liste des clés de `KO_DATA`)
- Affichage des résultats séries / demi / finale
- Profil nageur au clic : historique personnel, rang virtuel, delta index
- Barres visuelles de comparaison des temps

---

### 4.2 `analyse_course_el.html`

Outil d'analyse des courses 10km. Charge `data/resultats_10km_el.json`.

**Fonctionnalités :**
- Sélecteur genre F/H
- Sélecteur de course
- Affichage des résultats avec temps de passage par tour
- Stratégie "cigare" : analyse de la régularité des tours (variance des vitesses)
- Profil nageur : historique + graphique des vitesses par tour

---

## 5. Les fichiers pontons

### 5.1 Rôle des pontons

Les pontons sont des pages utilisées **la veille et le jour de la course** pour visualiser la startlist, les profils des nageurs, la météo prévue et la carte du plan d'eau.

### 5.2 `Ponton10km.html`

Ponton pour les courses 10km. Charge `data/resultats_10km_el.json` pour afficher l'historique de chaque nageur.

**Structure de la startlist locale (embarquée dans le HTML) :**
```js
var STARTLIST_F = [
  { bib: 1, nom: "JOHNSON Moesha", noc: "AUS", couloir: 1 },
  { bib: 2, nom: "TADDEUCCI Ginevra", noc: "ITA", couloir: 2 },
  // ...
];
var STARTLIST_H = [ /* idem */ ];
```

**Météo (embarquée dans le HTML, à mettre à jour manuellement avant chaque course) :**
```js
var METEO_F = [
  { h: "09h", t: 18.5, v: 12.3, r: 28.6, dir: "NE" },
  // une entrée par heure, de -3h à +3h autour du départ
];
var METEO_H = [ /* idem */ ];
```

| Champ | Description | Unité |
|-------|-------------|-------|
| `h` | Heure | `"10h"` |
| `t` | Température | °C |
| `v` | Vitesse vent | km/h |
| `r` | Rafales | km/h |
| `dir` | Direction | points cardinaux |

**Source météo recommandée :** Open-Meteo (gratuit, sans clé API)
```
https://api.open-meteo.com/v1/forecast?latitude=41.00&longitude=9.63
  &hourly=temperature_2m,windspeed_10m,windgusts_10m,winddirection_10m
  &timezone=Europe/Rome&forecast_days=1
```

**Config de la compétition (embarquée dans le HTML) :**
```js
var CONFIG = {
  titre: "Ponton 10km · Golfo Aranci 2026",
  date_f: "1er mai 2026",
  heure_f: "12h00",
  date_h: "1er mai 2026",
  heure_h: "09h00",
  lieu: "Golfo Aranci, Sardaigne",
};
```

**Fonctionnalités affichées :**
- Visualisation du ponton (position des nageurs sur le ponton de départ)
- Profil individuel au clic : historique des courses, indices EL, stratégie cigare
- Météo heure par heure (différenciée F/H selon les heures de départ)
- Carte du plan d'eau

**Indices EL et stratégie cigare :**
- **Indice EL** : calculé à partir du champ `di` (delta index) de l'historique. Mesure la capacité à sur/sous-performer par rapport au classement mondial.
- **Stratégie cigare** : analyse des vitesses par tour (`t_1666`, `t_3333`, etc.). Calcule la variance pour déterminer si le nageur est régulier (cigare fermé) ou irrégulier (cigare ouvert). Nécessite que les données de passage soient disponibles dans le JSON.

---

### 5.3 `ponton_ko_ibiza2026.html`

Modèle de ponton pour les courses KO. Même structure que `Ponton10km.html` mais charge `data/resultats_ko_el.json` et utilise les données KO (séries, qualifiés, temps de passage à 400/800/1500m).

**Pour créer un nouveau ponton KO :**
1. Dupliquer ce fichier
2. Renommer selon la convention : `ponton_ko_{ville}{annee}.html`
3. Mettre à jour `CONFIG`, `STARTLIST_F`, `STARTLIST_H`, `METEO_F`, `METEO_H`
4. Vérifier la correspondance des noms avec les clés du JSON

---

## 6. Les applis de terrain

### 6.1 `video_analyse_el.html` — Outil bureau (tagging vidéo)

Utilisé sur laptop en parallèle d'une vidéo de course. Permet de tagger des événements (passage bouée, leader, position, ravito...) avec un chrono synchronisé.

**Architecture :** fichier HTML autonome, tout en mémoire JS, pas de fetch. Export/import JSON.

**Layout 3 colonnes :**
```
| COL GAUCHE (280px)     | COL CENTRE (flex)      | COL DROITE (320px)   |
| Chrono + Config course | Boutons d'action       | Log des événements   |
| Config tours/bouées    | Contexte passage actuel|                      |
| Import nageurs JSON    | Analyse + Export       |                      |
```

**Format startlist attendu à l'import :**
```json
[{ "bib": 5, "nom": "FONTAINE Logan", "noc": "FRA", "couloir": 5 }]
```

**Format export JSON de session :**
```json
{
  "version": "1.0",
  "generated": "2026-05-01T09:00:00Z",
  "course": { "nom": "CdM Golfo Aranci 2026", "genre": "H", "type": "CDM" },
  "tour_config": { "1": { "pts": [{"lbl":"T1-B1","dist":"500"}] } },
  "nageurs": [...],
  "events": [...],
  "duree_totale_ms": 6547000
}
```

**Types d'événements taguables :**

| Type JS | Libellé | Modal |
|---------|---------|-------|
| `passage` | Passage bouée | Non |
| `leader` | Nouveau leader | Oui |
| `leader_confirm` | Confirmer leader | Oui |
| `position` | Changement position | Oui (1/2/3/4/5/>5/?) |
| `pointes` | 2 pointes | Oui |
| `formation` | Quinquonce / En ligne | Non |
| `ravito` | Ravitaillement | Non |
| `interet` | Point d'intérêt | Oui |

---

### 6.2 `observer_el.html` — Outil mobile (observation terrain)

Utilisé sur téléphone par des observateurs en bord de plan d'eau. Chaque observateur suit 1 ou 2 nageurs. **Nécessite un hébergement HTTP** (GitHub Pages) pour le système de partage par URL.

**Flux d'utilisation :**
1. Analyste configure l'appli (course + tours + startlist) → génère un lien
2. Envoie le lien aux observateurs (WhatsApp, SMS)
3. Observateur ouvre le lien, choisit ses nageurs, démarre au coup de pistolet
4. Fin de course : chaque observateur exporte son JSON
5. Analyste fusionne les exports

**Système de partage par URL :**
- Encode en `base64` un objet `{cfg, nageurs, ts}` dans le paramètre `?d=` de l'URL
- L'URL contient toute la config : le destinataire n'a qu'à choisir son nom et ses nageurs

**3 écrans : `[SETUP]` → `[OBSERVATION]` → `[ANALYSE]`**

**Boutons disponibles en observation :**

| Bouton | Type | Comportement |
|--------|------|-------------|
| 👑 Leader | `leader` | Modal confirmation |
| 📍 Position | `position` | Sélecteur 2/3/4/5/top5/peloton |
| 🔵 Bouée | `bouee` | Incrémente auto tour + bouée |
| 👥 Peloton | `peloton` | Sélection formation + position |
| 💧 Début ravito | `ravito_debut` | — |
| ✓ Fin ravito | `ravito_fin` | Calcule durée auto |

**Format export :**
```json
{
  "version": "1.0",
  "course": { "nom": "CdM Golfo Aranci 2026", "genre": "H", "tours": 6 },
  "observateur": "JD",
  "duree_totale_ms": 6547000,
  "nageurs": [
    {
      "nageur": { "nom": "FONTAINE Logan", "noc": "FRA", "bib": 5, "color": "#e8a020" },
      "events": [...]
    }
  ]
}
```

---

## 7. Palette de couleurs commune

Tous les fichiers utilisent le même thème sombre "marine" :

```css
--bg-dark:    #07101c;
--bg-medium:  #0d1520;
--bg-light:   #1a2535;
--blue-dark:  #003087;
--blue-light: #1560d4;
--red:        #C8102E;
--gold-dark:  #b8860b;
--gold-light: #e8a020;
--green-dark: #1a7a3c;
--green-light:#10b060;
--text:       #e8edf2;
--text-muted: #8899aa;
```

---

## 8. Workflow de mise à jour

### Après une course KO

1. Ouvrir `data/resultats_ko_el.json`
2. Ajouter la course dans `KO_DATA` (clé `{ville}{annee}_{genre}`) avec séries / demi / finale
3. Pour chaque nageur classé, ajouter une entrée dans `KO_HF` ou `KO_HH`
4. `git commit` + `git push` → GitHub Pages se met à jour en 1-2 min

### Après une course 10km

Même principe avec `data/resultats_10km_el.json`.

### Avant un nouveau ponton

1. Extraire la startlist du PDF officiel World Aquatics
2. Vérifier la correspondance exacte des noms avec les clés JSON (⚠️ prénoms tronqués !)
3. Mettre à jour `CONFIG`, `STARTLIST_F/H` dans le fichier ponton
4. Récupérer la météo sur Open-Meteo et remplir `METEO_F/H`
5. Tester en local avec Live Server
6. `git push`

### Test en local

```bash
# Option 1 — VS Code Live Server (recommandé)
# Clic droit sur index.html → Open with Live Server

# Option 2 — Python
python3 -m http.server 8000
# puis ouvrir http://localhost:8000
```

---

## 9. Bugs connus / Points de vigilance

- **Noms tronqués dans les PDF** : les PDF World Aquatics tronquent parfois les prénoms longs. Toujours compléter avant d'insérer dans le JSON. Ex : `BRANDT DE MACEDO L.` → `BRANDT DE MACEDO Leonard`.
- **Indices EL et stratégie cigare** : nécessitent que les données de passage par tour soient présentes dans le JSON. Si `rv` ou `di` sont `null`, l'indice ne s'affiche pas. Si les temps de passage sont absents, la stratégie affiche "Pas assez de données".
- **Correspondance des noms** : la moindre différence (espace, tiret, majuscule) entre le nom dans la startlist du ponton et la clé dans `KO_HF/HH` ou `HF/HH` empêche l'affichage de l'historique.
- **fetch() en file://** : impossible. Toujours tester avec un serveur local.
- **Doublons de clés** : certains nageurs ont des variantes de noms dans le JSON historique (`DE VALDES Maria` et `de VALDES Maria`). Consolider avant usage.

---

## 10. Backlog / Évolutions prévues

### Applis analyse
- [ ] Couplage avec résultats officiels pour afficher les positions de **tous** les nageurs aux points de passage (pas seulement celui sélectionné)

### Pontons
- [ ] Automatiser la récupération de la météo via l'API Open-Meteo au chargement (au lieu de l'embarquer manuellement)

### Appli mobile (`observer_el.html`)
- [ ] Fusion automatique de plusieurs exports JSON côté analyste
- [ ] Vue comparée multi-observateurs sur un même nageur
- [ ] Synchronisation temps réel (nécessiterait un backend Firebase)

### Maintenance JSON
- [ ] Script Python pour extraire automatiquement les résultats depuis les PDF World Aquatics et mettre à jour les JSON (partiellement développé)
