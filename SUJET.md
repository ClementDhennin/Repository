# TP TypeScript — Donjon

Un jeu de rôle dans le navigateur : un héros traverse un donjon de 8 salles (coffres, monstres, combat au tour par
tour) et finit face à un dragon.

À écrire : les types, le corps des fonctions, puis l'écran (HTML, Tailwind, jQuery). Le squelette des fonctions est
déjà dans `src/`, en commentaire.

Aperçu du sujet dans VS Code : clic droit sur `SUJET.md` → Ouvrir l'aperçu.

---

## Fonctionnement

### Commandes

| Commande | Rôle |
|---|---|
| `npm install` | Installer le projet (une fois) |
| `npm run dev` | Lancer le jeu dans le navigateur |
| `npm run palier 3` | Vérifier le palier 3 : compilateur, puis tests |
| `npm run bilan` | Vérifier tous les paliers |

Deux terminaux : un pour `npm run dev`, un pour `npm run palier`.

### Règles

1. Les noms demandés sont imposés (fichiers, types, fonctions, champs) : les tests s'appuient dessus. Le reste est
   libre : corps des fonctions, fonctions d'aide, textes, mise en page.
2. `any` interdit. Valeur de type inconnu : `unknown`.
3. Pas de mutation. Une fonction qui « modifie » un héros rend un nouveau héros (`{ ...heros, pv: 12 }`). Les tests
   le vérifient.
4. jQuery uniquement dans `src/main.ts`. Les autres fichiers de `src/` contiennent les règles du jeu et ne
   connaissent pas l'écran.
5. Ne pas modifier `tests/`. Le lire est utile : chaque nom de test est une règle du jeu.

### Squelettes

Chaque fichier de `src/` contient déjà, en commentaire, les imports et la signature des fonctions du palier.

1. Écrire les types demandés (seul leur nom est donné).
2. Décommenter les imports en haut du fichier.
3. Décommenter une fonction, écrire son corps, lancer `npm run palier N`, passer à la suivante.

Ne décommenter que ce qui est écrit : une fonction décommentée sans corps ne compile pas, et une erreur de
compilation dans un fichier bloque tous les paliers.

`src/main.ts` : squelette découpé par palier. Décommenter les imports du palier en cours, pas ceux des suivants.

### Imports et exports

```ts
// Tout ce qui sert dans un autre fichier est exporté
export type Arme = { … }
export function creerHeros(…) { … }

// Une valeur s'importe avec import, un type avec import type
import { creerHeros } from "./heros"
import type { Heros } from "./heros"
```

### Validation d'un palier

`npm run palier 3` lance, dans l'ordre :

1. le compilateur (`tsc`) sur `src/` et sur le test du palier. Une erreur de type : les tests ne sont pas lancés ;
2. les tests.

Dans les tests, une ligne précédée de `// @ts-expect-error` doit être refusée par le compilateur. Si elle passe,
c'est qu'un type est trop large (un `string` à la place d'une union, un `readonly` oublié). Message obtenu :
`Unused '@ts-expect-error' directive`. Le commentaire sur la ligne dit ce qui est attendu.

### Paliers

Les durées sont indicatives.

| Palier | Titre | Notions | Vérification | Durée |
|---|---|---|---|---|
| 0 | Démarrage | — | tests | 10 min |
| 1 | Le héros | types d'objets, littéraux, `readonly`, `?` | tests | 25 min |
| 2 | La carte du héros | type de retour littéral, jQuery, Tailwind | tests + écran | 35 min |
| 3 | Le bestiaire | `as const`, tableaux, type de fonction, `find` | tests | 25 min |
| 4 | Les dégâts | paramètre facultatif, immuabilité, typage structurel | tests | 25 min |
| 5 | Premier combat à l'écran | événements, état | écran | 30 min |
| 6 | Le sac et les objets | union discriminée, `never`, `reduce` | tests | 35 min |
| 7 | Les actions | union discriminée, `switch` exhaustif, narrowing | tests | 40 min |
| 8 | Le journal | classe | tests | 20 min |
| 9 | Le combat complet à l'écran | délégation, `data-*` | écran | 30 min |
| 10 | Le donjon | unions, tableaux | tests | 30 min |
| 11 | La partie | états de la partie | tests | 45 min |
| 12 | Le jeu complet à l'écran | `unknown`, `switch` sur l'état | tests + écran | 45 min |
| 13 | Bonus : générique | `<T>` | tests | 20 min |
| 14 | Bonus : scores | `unknown`, `in`, `localStorage` | tests | 30 min |
| 15 | Bonus libres | — | — | — |

Palier 9 terminé : le combat est jouable. Palier 12 terminé : le jeu est complet.

---

## Règles du jeu

### Classes de héros

| Classe | PV | Mana | Attaque | Défense | Arme de départ |
|---|---|---|---|---|---|
| `guerrier` | 100 | 10 | 13 | 5 | Épée courte (bonus 2) |
| `mage` | 80 | 80 | 8 | 3 | aucune |
| `voleur` | 100 | 30 | 14 | 4 | aucune |

Au départ : PV et mana au maximum, 0 or, et 2 potions de 30 PV (à partir du palier 6).

### Combat

- Dégâts = attaque − défense, minimum 1. Coup critique : dégâts doublés.
- Attaque totale du héros = attaque + bonus de l'arme s'il en a une.
- Un tour : le héros joue une action, puis le monstre riposte s'il est vivant.

| Action | Effet |
|---|---|
| Attaquer | Dé à 6 faces : 6 = coup critique. Le monstre perd les dégâts. |
| Boule de feu | 10 de mana. Le monstre perd 25 PV, défense ignorée. |
| Soin | 8 de mana. Le héros regagne 30 PV, sans dépasser son maximum. |
| Potion | Le héros boit la première potion du sac. |
| Fuir | Dé à 6 faces : 4, 5 ou 6 = fuite réussie. |

### Donjon

- 8 salles. Les 7 premières au dé : 1 = vide, 2 = coffre, 3 à 6 = monstre. La 8ᵉ : le dragon.
- Salles 1 à 3 : monstres de niveau 1. Salles 4 à 6 : niveau 2. Ensuite : niveau 3.
- Salle vide : +20 PV et +10 de mana.
- Monstre vaincu : son butin en or. Score = or + valeur des trésors du sac.

---

## Palier 0 — Démarrage

1. Ouvrir le dossier dans VS Code.
2. `node -v` : 22.12 minimum, ou une version 24.
3. `npm install`
4. `npm run dev`, ouvrir l'adresse affichée. Texte attendu, sur fond sombre :
   « Le poste est prêt : jQuery 4.0.0 répond, Tailwind met en forme. »
5. Dans un second terminal : `npm run palier 0`.

**Validé si** : `Palier 0 validé.`

| Fichier | Rôle |
|---|---|
| `index.html` | La page : HTML et classes Tailwind |
| `src/main.ts` | Point d'entrée, seul fichier qui utilise jQuery |
| `src/style.css` | `@import "tailwindcss"` |
| `src/*.ts` | Un fichier par palier, avec le squelette des fonctions en commentaire |
| `tests/` | Un fichier de test par palier |
| `tsconfig.json` | Réglages du compilateur, dont `"strict": true` |

---

## Palier 1 — Le héros

**Notions** : type d'objet, union de littéraux, `readonly`, champ facultatif `?`, fonctions typées, `switch`, `?.`
et `??`, spread.

**Fichier** : `src/heros.ts` (squelette fourni).

1. Ouvrir `src/heros.ts`.
2. Exporter le type `ClasseHeros` : trois valeurs possibles, `"guerrier"`, `"mage"`, `"voleur"`.
3. Exporter le type `Arme` : `nom` (texte), `bonus` (nombre). Les deux en lecture seule.
4. Exporter le type `Heros` :

   | Champ | Type | Remarque |
   |---|---|---|
   | `nom` | texte | lecture seule |
   | `classe` | `ClasseHeros` | lecture seule |
   | `pvMax` | nombre | lecture seule |
   | `manaMax` | nombre | lecture seule |
   | `pv` | nombre | |
   | `mana` | nombre | |
   | `attaque` | nombre | |
   | `defense` | nombre | |
   | `or` | nombre | |
   | `arme` | `Arme` | facultatif |

5. Exporter la fonction `creerHeros(nom, classe)` → `Heros`.
   a. Valeurs de départ : tableau des classes ci-dessus.
   b. `pv` = `pvMax`, `mana` = `manaMax`, `or` = 0.
   c. Guerrier : `arme` vaut `{ nom: "Épée courte", bonus: 2 }`. Mage et voleur : pas de champ `arme`.
   d. Un `switch` sur la classe, un `case` par classe.
6. Exporter la fonction `attaqueTotale(heros)` → nombre : attaque + bonus de l'arme s'il y en a une.
7. Exporter la fonction `equiper(heros, arme)` → nouveau `Heros` qui porte l'arme. L'ancien n'est pas modifié.
8. Exporter la fonction `decrireHeros(heros)` → texte contenant le nom, la classe, les PV sous la forme `80/80`, et
   le nom de l'arme s'il y en a une. Formulation libre.
9. `npm run palier 1`

**Validé si** : `Palier 1 validé.`

**À essayer** :

- Commenter un `case` de `creerHeros`, lire l'erreur, le remettre.
- Écrire `creerHeros("Zed", "dragon")` dans `src/main.ts`, lire l'erreur, supprimer la ligne.

<details>
<summary>Indice : creerHeros</summary>

```ts
export function creerHeros(nom: string, classe: ClasseHeros): Heros {
  switch (classe) {
    case "guerrier":
      return { nom, classe, pvMax: 100, pv: 100, /* … */ }
    // les deux autres classes
  }
}
```

Pas de `default` : avec les trois classes traitées, le compilateur sait que la fonction rend toujours un héros.

</details>

<details>
<summary>Indice : arme facultative</summary>

`heros.arme?.bonus` vaut le bonus ou `undefined`. `?? 0` remplace `undefined` par 0.

</details>

---

## Palier 2 — La carte du héros

**Notions** : type de retour littéral, jQuery dans un fichier TypeScript, Tailwind.

**Fichiers** : `src/affichage.ts` (squelette fourni), `index.html`, `src/main.ts`.

### A. Calculs (testés)

1. Ouvrir `src/affichage.ts`.
2. Exporter la fonction `pourcentage(valeur, max)` → nombre.
   a. Résultat entier, arrondi avec `Math.round` : 1 sur 3 donne 33.
   b. Résultat toujours entre 0 et 100.
   c. `max` inférieur ou égal à 0 : résultat 0.
3. Exporter la fonction `couleurBarre(pourcent)` → une classe Tailwind :

   | Pourcentage | Résultat |
   |---|---|
   | plus de 50 | `"bg-emerald-500"` |
   | de 21 à 50 | `"bg-amber-500"` |
   | 20 et moins | `"bg-red-500"` |

   Type de retour : ces trois valeurs exactement, pas `string`. Créer un type pour ça (`CouleurBarre`).

Rappel Tailwind : pas de nom de classe construit par morceaux (`"bg-" + couleur + "-500"`), Tailwind ne le voit pas.

### B. Écran

4. Dans `index.html`, remplacer le paragraphe `#message` par une carte. Structure minimale, à styler avec Tailwind
   (10 minutes maximum sur le style) :

   ```html
   <article id="heros">
     <h2 id="heros-nom"></h2>
     <p id="heros-classe"></p>

     <p>PV : <span id="heros-pv"></span></p>
     <div><!-- fond de la barre -->
       <div id="heros-pv-barre"></div><!-- partie colorée -->
     </div>

     <p>Mana : <span id="heros-mana"></span></p>
     <p>Attaque : <span id="heros-attaque"></span></p>
     <p>Arme : <span id="heros-arme"></span></p>
   </article>
   ```

5. Dans `src/main.ts` (squelette fourni, palier par palier) :
   a. supprimer la ligne qui écrit dans `#message` ;
   b. créer un héros avec `creerHeros` ;
   c. décommenter et écrire `afficherHeros(heros: Heros): void`, qui remplit la carte avec `.text(…)` ;
   d. largeur de la barre : `.css("width", …)` ; couleur : `.addClass(couleurBarre(…))` ;
   e. appeler `afficherHeros`.
6. Tester la barre avec un héros blessé : `{ ...creerHeros("Aria", "mage"), pv: 12 }`.
7. `npm run palier 2`

**Validé si** : `Palier 2 validé.` et les points « à vérifier à l'écran » sont bons.

<details>
<summary>Indice : barre de vie</summary>

Deux `div` imbriquées : le fond (`h-3 rounded-full bg-slate-800`) et la partie colorée (`h-3 rounded-full`), dont
la largeur est réglée par `.css("width", "60%")`.

Changement de couleur : retirer l'ancienne avant d'ajouter la nouvelle.
`.removeClass("bg-emerald-500 bg-amber-500 bg-red-500").addClass(couleurBarre(pourcent))`

</details>

---

## Palier 3 — Le bestiaire

**Notions** : `as const`, type dérivé d'une constante, tableau `readonly`, `&`, type de fonction, `filter`, `find`
et `undefined`.

**Fichiers** : `src/de.ts` et `src/monstres.ts` (squelette fourni).

### A. Le dé

1. Ouvrir `src/de.ts`.
2. Exporter le type `De` : une fonction qui reçoit un nombre de faces et rend un nombre.
3. Exporter la constante `lancerDe`, de type `De` : un entier au hasard entre 1 et le nombre de faces.
   Formule : `Math.floor(Math.random() * faces) + 1`.

Toute fonction qui a besoin de hasard reçoit le dé en paramètre au lieu d'appeler `Math.random`. Le jeu passe
`lancerDe`, les tests passent un dé truqué.

### B. Les monstres

4. Ouvrir `src/monstres.ts`.
5. Exporter la constante `NIVEAUX` : `[1, 2, 3]`, avec `as const`.
6. Exporter le type `Niveau`, dérivé de `NIVEAUX`.
7. Exporter le type `ModeleMonstre`. Tous les champs en lecture seule.

   | Champ | Type |
   |---|---|
   | `nom` | texte |
   | `niveau` | `Niveau` |
   | `pvMax` | nombre |
   | `attaque` | nombre |
   | `defense` | nombre |
   | `butin` | nombre |

8. Exporter le type `Monstre` : `ModeleMonstre` et un champ `pv` (nombre, modifiable). Utiliser `&`.
9. Exporter la constante `BESTIAIRE` : tableau `readonly` de `ModeleMonstre`. Données à copier :

   ```ts
   [
     { nom: "Rat géant", niveau: 1, pvMax: 18, attaque: 5, defense: 0, butin: 3 },
     { nom: "Gobelin", niveau: 1, pvMax: 25, attaque: 7, defense: 1, butin: 5 },
     { nom: "Chauve-souris", niveau: 1, pvMax: 14, attaque: 6, defense: 0, butin: 2 },
     { nom: "Squelette", niveau: 2, pvMax: 40, attaque: 10, defense: 3, butin: 10 },
     { nom: "Loup-garou", niveau: 2, pvMax: 50, attaque: 12, defense: 2, butin: 12 },
     { nom: "Araignée géante", niveau: 2, pvMax: 35, attaque: 11, defense: 2, butin: 9 },
     { nom: "Troll", niveau: 3, pvMax: 70, attaque: 14, defense: 4, butin: 25 },
     { nom: "Spectre", niveau: 3, pvMax: 55, attaque: 16, defense: 2, butin: 22 },
   ]
   ```

10. Exporter la constante `BOSS`, de type `ModeleMonstre` :
    `{ nom: "Dragon", niveau: 3, pvMax: 100, attaque: 17, defense: 4, butin: 100 }`.
11. Exporter la fonction `creerMonstre(modele)` → `Monstre` avec tous ses PV.
12. Exporter la fonction `trouverModele(nom)` → le modèle de ce nom, ou `undefined`. Le type de retour doit le dire.
13. Exporter la fonction `modelesDeNiveau(niveau)` → tableau des modèles de ce niveau, dans l'ordre du bestiaire.
14. Exporter la fonction `monstreAuHasard(niveau, de)` → `Monstre`.
    a. Un seul lancer de dé, avec autant de faces que de monstres de ce niveau.
    b. 1 = premier monstre du niveau, 3 = troisième.
15. `npm run palier 3`

**Validé si** : `Palier 3 validé.`

**À essayer** : mettre `niveau: 4` sur un monstre du bestiaire, lire l'erreur, corriger.

<details>
<summary>Indice : type dérivé d'une constante</summary>

```ts
const STATUTS = ["libre", "reserve"] as const
type Statut = (typeof STATUTS)[number]   // "libre" | "reserve"
```

</details>

<details>
<summary>Indice : type de fonction</summary>

```ts
type Comparateur = (a: number, b: number) => boolean
const plusGrand: Comparateur = (a, b) => a > b
```

</details>

---

## Palier 4 — Les dégâts

**Notions** : paramètre facultatif, immuabilité (spread), typage structurel.

**Fichier** : `src/combat.ts` (squelette fourni).

1. Ouvrir `src/combat.ts`.
2. Exporter la fonction `calculerDegats(attaque, defense, critique?)` → nombre. `critique` : booléen facultatif.
   a. Dégâts = attaque − défense.
   b. Minimum 1.
   c. `critique` à `true` : résultat doublé, après le minimum (1 devient 2).
3. Exporter la fonction `blesserHeros(heros, degats)` → nouveau `Heros` avec les PV en moins. Minimum 0. L'ancien
   héros n'est pas modifié.
4. Exporter la fonction `blesserMonstre(monstre, degats)` : pareil pour un `Monstre`.
5. Exporter la fonction `soigner(heros, points)` → nouveau `Heros` avec les PV en plus, sans dépasser `pvMax`.
6. Exporter la fonction `reposer(heros)` → nouveau `Heros` avec +20 PV et +10 de mana, sans dépasser `pvMax` ni
   `manaMax`. Réutiliser `soigner`.
7. Exporter la fonction `estVivant(combattant)` → `true` si les PV sont supérieurs à 0. Doit accepter un héros
   comme un monstre : le paramètre est « un objet qui a un champ `pv` de type nombre ».
8. `npm run palier 4`

**Validé si** : `Palier 4 validé.`

`blesserHeros` et `blesserMonstre` font doublon : voir le palier 13.

<details>
<summary>Indice : copie avec un champ changé</summary>

```ts
const plusVieux = { ...personne, age: personne.age + 1 }
```

`Math.max(0, x)` : pas en dessous de 0. `Math.min(plafond, x)` : pas au-dessus du plafond.

</details>

---

## Palier 5 — Premier combat à l'écran

**Notions** : événements jQuery, état dans des variables.

**Fichiers** : `index.html`, `src/main.ts`.

1. Dans `index.html`, ajouter :
   a. une carte `<article id="monstre">` sur le modèle de celle du héros (nom, PV, barre, attaque, défense) ;
   b. un bouton `<button id="attaquer" type="button">Attaquer</button>` ;
   c. un paragraphe `<p id="annonce"></p>`.
2. Dans `src/main.ts`, deux variables `let` : le héros (à la place de celui du palier 2), et un monstre tiré par
   `monstreAuHasard(1, lancerDe)`.
3. Écrire `afficherMonstre(monstre: Monstre): void`.
4. Écrire `afficher(): void` :
   a. appelle `afficherHeros` et `afficherMonstre` ;
   b. écrit dans `#annonce` qui a gagné, si le combat est fini ;
   c. désactive le bouton si le combat est fini : `.prop("disabled", true)`.
5. Au clic sur `#attaquer` :
   a. remplacer le monstre par un monstre blessé (`blesserMonstre`, `calculerDegats`, `attaqueTotale`) ;
   b. si le monstre est vivant, remplacer le héros par un héros blessé (attaque du monstre, défense du héros) ;
   c. appeler `afficher()`.
6. Appeler `afficher()` au démarrage.
7. `npm run palier 5` (pas de test : compilation, puis vérification à l'écran).

**Validé si** : `Palier 5 validé.` et les points « à vérifier à l'écran » sont bons.

L'état du jeu est dans les deux variables, pas dans le DOM. On change les variables, puis on réaffiche tout.

---

## Palier 6 — Le sac et les objets

**Notions** : union discriminée, narrowing, `switch` exhaustif avec `never`, `filter`, `reduce`.

**Fichiers** : `src/objets.ts` (squelette fourni), `src/heros.ts`.

1. Ouvrir `src/objets.ts`.
2. Exporter le type `Objet` : union discriminée par le champ `type`.

   | `type` | Autres champs |
   |---|---|
   | `"potion"` | `soin` : nombre |
   | `"arme"` | `arme` : `Arme` |
   | `"tresor"` | `nom` : texte, `valeur` : nombre |

3. Exporter la fonction `decrireObjet(objet)` → texte : le nombre de PV pour une potion, le nom de l'arme pour une
   arme, le nom du trésor pour un trésor. `switch` exhaustif, `default` en `never`.
4. À essayer : ajouter une quatrième forme à `Objet` (`{ type: "cle" }`), lire l'erreur dans `decrireObjet`, retirer
   la forme.
5. Dans `src/heros.ts`, ajouter au type `Heros` le champ `sac` : tableau `readonly` d'`Objet`.
   a. Corriger ce que le compilateur signale.
   b. Sac de départ : 2 potions de 30 PV.
   c. `heros.ts` et `objets.ts` s'importent l'un l'autre : sans problème avec `import type`.
6. Exporter la fonction `compterPotions(sac)` → nombre de potions.
7. Exporter la fonction `valeurDuSac(sac)` → somme des `valeur` des trésors. Utiliser `reduce`.
8. Exporter la fonction `ramasser(heros, objet)` → nouveau `Heros`.
   a. Objet ajouté à la fin du sac.
   b. Si c'est une arme au bonus strictement supérieur à celui de l'arme actuelle (pas d'arme = 0) : elle est
      équipée.
9. Exporter la fonction `boirePotion(heros)` → nouveau `Heros`.
   a. Pas de potion : héros rendu tel quel.
   b. Sinon : +`soin` PV de la première potion du sac, sans dépasser `pvMax`.
   c. Cette potion, et elle seule, est retirée du sac.
10. `npm run palier 6`, puis `npm run bilan`.

**Validé si** : `Palier 6 validé.` et bilan bon jusqu'au palier 6.

<details>
<summary>Indice : switch exhaustif</summary>

```ts
switch (objet.type) {
  case "potion":
    return …          // objet.soin accessible
  case "arme":
    return …          // objet.arme accessible
  case "tresor":
    return …          // objet.nom et objet.valeur accessibles
  default: {
    const jamais: never = objet
    return jamais
  }
}
```

</details>

<details>
<summary>Indice : reduce sur une union</summary>

```ts
sac.reduce((total, objet) => (objet.type === "tresor" ? total + objet.valeur : total), 0)
```

</details>

<details>
<summary>Indice : boirePotion</summary>

1. Position de la première potion : `findIndex` (`-1` si aucune).
2. `-1` : rendre le héros.
3. Lire l'objet à cette position et tester son `type` pour accéder à `soin`.
4. Nouveau sac sans cette position : `sac.filter((objet, position) => position !== …)`.
5. Rendre le héros soigné avec le nouveau sac.

</details>

---

## Palier 7 — Les actions

**Notions** : union discriminée avec données, `switch` exhaustif, narrowing d'un texte.

**Fichier** : `src/actions.ts` (squelette fourni).

1. Ouvrir `src/actions.ts`.
2. Exporter le type `Sort` : `"boule de feu"` ou `"soin"`.
3. Exporter le type `Action` : union discriminée par le champ `type`.

   | `type` | Autres champs |
   |---|---|
   | `"attaquer"` | aucun |
   | `"sort"` | `sort` : `Sort` |
   | `"potion"` | aucun |
   | `"fuir"` | aucun |

4. Exporter le type `Combat` : `heros` (`Heros`), `monstre` (`Monstre`).
5. Exporter le type `Tour` : `heros`, `monstre`, `fuite` (booléen), `messages` (tableau de textes).
6. Exporter la fonction `agir(combat, action, de)` → `Tour`. Action du héros seulement, sans la riposte.

   | Action | Règle |
   |---|---|
   | attaquer | Un lancer de dé à 6 faces. 6 = critique. Dégâts : `calculerDegats` avec l'attaque totale du héros et la défense du monstre. |
   | sort « boule de feu » | 10 de mana, −25 PV au monstre, défense ignorée. Mana insuffisant : rien ne change. |
   | sort « soin » | 8 de mana, +30 PV au héros, sans dépasser le maximum. Mana insuffisant : rien ne change. |
   | potion | `boirePotion`. Pas de potion : rien ne change. |
   | fuir | Un lancer de dé à 6 faces. 4, 5 ou 6 : `fuite` à `true`. |

   a. `fuite` à `false` dans tous les autres cas.
   b. `messages` : au moins une phrase dans tous les cas, y compris quand rien ne change. Textes libres.
   c. Le combat reçu n'est pas modifié.
   d. `switch` exhaustif sur `action.type`.
7. Exporter la fonction `jouerTour(combat, action, de)` → `Tour`.
   a. Le héros agit (`agir`).
   b. Fuite réussie ou monstre mort : fin du tour.
   c. Sinon le monstre riposte : `calculerDegats` avec l'attaque du monstre et la défense du héros, sans dé ni
      critique. Le héros est blessé, une phrase est ajoutée aux messages.
8. Exporter la fonction `lireAction(texte)` → `Action` ou `null`. `texte` : `string | undefined`.

   | Texte | Résultat |
   |---|---|
   | `"attaquer"` | `{ type: "attaquer" }` |
   | `"boule de feu"` | `{ type: "sort", sort: "boule de feu" }` |
   | `"soin"` | `{ type: "sort", sort: "soin" }` |
   | `"potion"` | `{ type: "potion" }` |
   | `"fuir"` | `{ type: "fuir" }` |
   | autre | `null` |

9. `npm run palier 7`

**Validé si** : `Palier 7 validé.`

`lireAction` sert au palier 9 : chaque bouton porte un attribut `data-action`, que jQuery lit comme un
`string | undefined`.

<details>
<summary>Indice : ordre de travail pour agir</summary>

Écrire d'abord le `switch` avec ses quatre `case` et le `default` en `never`, chaque `case` rendant un tour où rien
ne change :

```ts
const { heros, monstre } = combat
const rien: Tour = { heros, monstre, fuite: false, messages: ["…"] }
```

Remplir ensuite les cas un par un, en relançant `npm run palier 7`. Pour le sort : une fonction à part, avec son
propre `switch`.

</details>

---

## Palier 8 — Le journal

**Notions** : classe, `private`, `readonly`, accesseur.

**Fichier** : `src/journal.ts` (squelette fourni).

1. Ouvrir `src/journal.ts`, décommenter la classe `Journal`.
2. Champ privé `lignes` : tableau de textes, vide au départ.
3. Champ privé et `readonly` `capacite` : nombre de lignes gardées, reçu par le constructeur (`new Journal(6)`).
4. Méthode `ajouter(message)` : ajoute à la fin. Au-delà de la capacité, les lignes les plus anciennes sont
   supprimées.
5. Méthode `ajouterTous(messages)` : ajoute un tableau de messages, dans l'ordre.
6. Accesseur `get taille()` : nombre de lignes (`journal.taille`, pas `journal.taille()`).
7. Méthode `lire()` : les lignes, de la plus ancienne à la plus récente, en tableau `readonly`.
8. Méthode `vider()`.
9. `npm run palier 8`

**Validé si** : `Palier 8 validé.`

Attention : `constructor(private capacite: number)` est refusé par le réglage `erasableSyntaxOnly` du projet.
Déclarer le champ, puis l'affecter dans le constructeur.

<details>
<summary>Indice : garder les N derniers</summary>

`tableau.slice(-3)` : les 3 derniers éléments.

</details>

---

## Palier 9 — Le combat complet à l'écran

**Notions** : délégation d'événements, attribut `data-*`, `null`.

**Fichiers** : `index.html`, `src/main.ts`.

1. Dans `index.html` :
   a. remplacer le bouton `#attaquer` par un bloc `<div id="actions">` avec cinq boutons ;
   b. chaque bouton porte un `data-action` dont la valeur est un des cinq textes de `lireAction` :
      `<button type="button" data-action="boule de feu">Boule de feu</button>` ;
   c. ajouter `<ol id="journal"></ol>` ;
   d. ajouter le nombre de potions dans la carte du héros (`<span id="heros-potions"></span>`).
2. Dans `src/main.ts`, l'état devient :
   a. `let combat`, de type `Combat` (le héros et un monstre de niveau 2) ;
   b. `let termine`, booléen, `false` au départ ;
   c. `const journal`, un `Journal` de capacité 6.
3. Un seul écouteur de clic pour les cinq boutons, par délégation sur `#actions` :
   a. écouteur écrit avec `function`, pas en fléchée (pour `this`) ;
   b. lire l'attribut : `$(this).attr("data-action")`. Type : `string | undefined` ;
   c. le passer à `lireAction`. Résultat `null` : `return` ;
   d. appeler `jouerTour`, remplacer `combat` par le héros et le monstre du tour ;
   e. `termine` passe à `true` si fuite réussie ou si l'un des deux est mort ;
   f. ajouter les messages du tour au journal ;
   g. appeler `afficher()`.
4. Écrire `afficherJournal()` : vider la liste (`.empty()`), ajouter un `<li>` par ligne avec `.text(…)`.
5. Compléter `afficher()` : les deux cartes, le journal, le nombre de potions, les boutons désactivés si `termine`.
6. `npm run palier 9`

**Validé si** : `Palier 9 validé.` et les points « à vérifier à l'écran » sont bons.

**À essayer** : remplacer `.attr("data-action")` par `.data("action")` et survoler : le type est `any`. Remettre
`.attr(…)`.

<details>
<summary>Indice : délégation en jQuery</summary>

```ts
$("#liste").on("click", ".reserver", function () {
  $(this)   // le bouton cliqué
})
```

Sélecteur pour « tout élément qui a un attribut `data-action` » : `"[data-action]"`.

</details>

---

## Palier 10 — Le donjon

**Notions** : union discriminée, fonction qui rend un littéral, construction d'un tableau.

**Fichier** : `src/donjon.ts` (squelette fourni).

1. Ouvrir `src/donjon.ts`.
2. Exporter le type `Salle` : union discriminée par le champ `type`.

   | `type` | Autres champs |
   |---|---|
   | `"vide"` | aucun |
   | `"coffre"` | `objet` : `Objet` |
   | `"monstre"` | `monstre` : `Monstre` |
   | `"boss"` | `monstre` : `Monstre` |

3. Exporter la constante `COFFRES` : tableau `readonly` d'`Objet`. Données à copier :

   ```ts
   [
     { type: "potion", soin: 30 },
     { type: "potion", soin: 50 },
     { type: "arme", arme: { nom: "Dague", bonus: 1 } },
     { type: "arme", arme: { nom: "Épée longue", bonus: 4 } },
     { type: "arme", arme: { nom: "Bâton runique", bonus: 6 } },
     { type: "tresor", nom: "Bourse de pièces", valeur: 15 },
     { type: "tresor", nom: "Rubis", valeur: 40 },
     { type: "tresor", nom: "Couronne", valeur: 80 },
   ]
   ```

4. Exporter la fonction `niveauDeLaSalle(numero)` → `Niveau`. La première salle porte le numéro 1. Salles 1 à 3 :
   niveau 1. Salles 4 à 6 : niveau 2. Ensuite : niveau 3.
5. Exporter la fonction `objetAuHasard(de)` → un objet de `COFFRES`. Un lancer, autant de faces que d'objets,
   1 = le premier.
6. Exporter la fonction `genererSalle(numero, de)` → `Salle`.
   a. D'abord un lancer de dé à 6 faces.
   b. 1 : salle vide.
   c. 2 : coffre, objet tiré par `objetAuHasard`.
   d. 3 et plus : monstre, tiré par `monstreAuHasard` au niveau de la salle.
7. Exporter la fonction `genererDonjon(taille, de)` → tableau de `Salle`.
   a. Exactement `taille` salles.
   b. Les `taille − 1` premières : `genererSalle`, numéros 1, 2, 3…
   c. La dernière : `{ type: "boss", … }`, monstre créé à partir de `BOSS`.
8. Exporter la fonction `decrireSalle(salle)` → texte libre. Pour un monstre ou un boss, le texte contient le nom du
   monstre. `switch` exhaustif.
9. `npm run palier 10`

**Validé si** : `Palier 10 validé.`

**À essayer** : écrire `return 4` dans `niveauDeLaSalle`, lire l'erreur, corriger.

<details>
<summary>Indice : tableau de N éléments</summary>

Une boucle `for` qui remplit un tableau local, ou :

```ts
Array.from({ length: 3 }, (_, index) => index + 1)   // [1, 2, 3]
```

</details>

---

## Palier 11 — La partie

**Notions** : union discriminée pour l'état de la partie, narrowing, fonctions de transition.

**Fichier** : `src/partie.ts` (squelette fourni).

Transitions :

| De | Par | Vers |
|---|---|---|
| accueil | `commencer` | exploration |
| exploration | `avancer`, salle vide ou coffre | exploration |
| exploration | `avancer`, monstre ou boss | combat |
| combat | `combattre`, les deux vivants | combat |
| combat | `combattre`, fuite réussie ou monstre vaincu | exploration |
| combat | `combattre`, boss vaincu | victoire |
| combat | `combattre`, héros à 0 PV | defaite |

1. Ouvrir `src/partie.ts`.
2. Exporter le type `Partie` : union discriminée par le champ `etat`. Chaque état ne porte que les données qui
   existent dans cet état.

   | `etat` | Autres champs |
   |---|---|
   | `"accueil"` | aucun |
   | `"exploration"` | `heros`, `donjon` (tableau `readonly` de `Salle`), `position` (nombre) |
   | `"combat"` | `heros`, `donjon`, `position`, `monstre` (le monstre affronté, avec ses PV actuels) |
   | `"victoire"` | `heros` |
   | `"defaite"` | `heros`, `position` |

   `position` : index, dans `donjon`, de la salle où l'on va entrer (ou de celle où l'on se bat). Départ : 0.
3. Exporter le type `Etape` : `partie` (`Partie`), `messages` (tableau de textes).
4. Exporter la constante `TAILLE_DONJON` : 8.
5. Exporter la fonction `nouvellePartie()` → partie à l'état `"accueil"`.
6. Exporter la fonction `commencer(nom, classe, de)` → `Partie` à l'état `"exploration"` : nouveau héros, donjon de
   `TAILLE_DONJON` salles, position 0.
7. Exporter la fonction `avancer(partie)` → `Etape`.
   a. Partie hors exploration : rien ne change, `messages` vide.
   b. Sinon, entrer dans la salle à `position` :

   | Salle | Partie rendue |
   |---|---|
   | vide | exploration, héros reposé (`reposer`), `position + 1` |
   | coffre | exploration, héros qui ramasse l'objet (`ramasser`), `position + 1` |
   | monstre ou boss | combat contre le monstre de la salle, même `position` |

   c. `messages` : au moins une phrase (`decrireSalle`).
8. Exporter la fonction `combattre(partie, action, de)` → `Etape`.
   a. Partie hors combat : rien ne change, `messages` vide.
   b. Sinon, jouer un tour (`jouerTour`), puis tester dans cet ordre :

   | Après le tour | Partie rendue |
   |---|---|
   | héros mort | `"defaite"`, avec le héros et la `position` |
   | fuite réussie | exploration, même `position` |
   | monstre vivant | combat, avec le héros et le monstre du tour |
   | monstre mort, dernière salle | `"victoire"`, héros + `butin` en or |
   | monstre mort, autre salle | exploration, `position + 1`, héros + `butin` en or |

   c. `messages` : ceux du tour, plus d'autres si besoin.
9. Exporter la fonction `score(heros)` → or + valeur du sac.
10. `npm run palier 11`

**Validé si** : `Palier 11 validé.`

**À essayer** : dans `avancer`, écrire `partie.heros` avant d'avoir testé `partie.etat`, lire l'erreur.

Après une fuite, le monstre retrouve tous ses PV sans rien faire de spécial : le monstre blessé est dans l'état
`"combat"`, le donjon n'a jamais été modifié.

<details>
<summary>Indice : écarter les mauvais états</summary>

```ts
export function avancer(partie: Partie): Etape {
  if (partie.etat !== "exploration") {
    return { partie, messages: [] }
  }
  // ici, partie.heros, partie.donjon et partie.position existent
}
```

</details>

---

## Palier 12 — Le jeu complet à l'écran

**Notions** : `unknown`, narrowing d'une saisie, `switch` exhaustif sur l'état.

**Fichiers** : `src/saisie.ts` (squelette fourni), `index.html`, `src/main.ts`.

### A. Saisie (testée)

`.val()` sur un champ de formulaire rend `string | number | string[] | undefined`.

1. Ouvrir `src/saisie.ts`.
2. Exporter la fonction `lireNom(valeur)` → texte ou `null`. `valeur` : `unknown`.
   a. Pas un texte : `null`.
   b. Retirer les espaces autour (`trim`).
   c. Moins de 2 ou plus de 20 caractères : `null`.
3. Exporter la fonction `lireClasse(valeur)` → `ClasseHeros` ou `null`. `valeur` : `unknown`.
4. `npm run palier 12` : les tests doivent passer.

### B. Écran

5. Dans `index.html`, trois sections :
   a. `#ecran-accueil` : un formulaire `#formulaire` avec un champ `#nom`, un `<select id="classe">` (`value` :
      `guerrier`, `mage`, `voleur`), un paragraphe d'erreur `#erreur`, un bouton d'envoi ;
   b. `#ecran-partie` : les deux cartes, un bouton `#avancer` (« Ouvrir la porte suivante »), les cinq boutons
      d'action, le journal, un texte de progression (« Salle 3 sur 8 ») ;
   c. `#ecran-fin` : un titre, un texte, un bouton `#rejouer`.
6. Dans `src/main.ts`, l'état devient une seule variable : `let partie: Partie = nouvellePartie()`. Le journal
   reste. `combat` et `termine` disparaissent.
7. Réécrire `afficher()` avec un `switch` exhaustif sur `partie.etat` :

   | État | Affichage |
   |---|---|
   | accueil | le formulaire |
   | exploration | carte du héros, progression, bouton pour avancer |
   | combat | les deux cartes, boutons d'action |
   | victoire | écran de fin : nom du héros, score |
   | defaite | écran de fin : nom du héros, salle, score |

   Afficher ou cacher : `.show()`, `.hide()`, `.toggle(condition)`.
8. Écrire `appliquer(etape: Etape): void` : remplace `partie`, ajoute les messages au journal, appelle `afficher()`.
9. Événements :
   a. envoi du formulaire : `preventDefault`, `lireNom`, `lireClasse`. Un `null` : message d'erreur et arrêt.
      Sinon : journal vidé, `partie = commencer(…)`, `afficher()` ;
   b. bouton pour avancer : `appliquer(avancer(partie))` ;
   c. boutons d'action (délégation) : `appliquer(combattre(partie, action, lancerDe))` ;
   d. bouton « Rejouer » : `partie = nouvellePartie()`, journal vidé, `afficher()`.
10. `npm run palier 12`, puis jouer une partie entière.

**Validé si** : `Palier 12 validé.`, points « à vérifier à l'écran » bons, `npm run bilan` bon de 0 à 12.

<details>
<summary>Indice : narrowing d'un unknown</summary>

`typeof valeur === "string"` donne accès aux méthodes des textes. Pour la classe : comparer la valeur à chacun des
trois textes (`===` ou `switch`).

</details>

<details>
<summary>Indice : squelette de afficher</summary>

```ts
function afficher(): void {
  $("#ecran-accueil").toggle(partie.etat === "accueil")
  $("#ecran-partie").toggle(partie.etat === "exploration" || partie.etat === "combat")
  $("#ecran-fin").toggle(partie.etat === "victoire" || partie.etat === "defaite")

  switch (partie.etat) {
    case "accueil":
      return
    case "exploration":
      afficherHeros(partie.heros)
      // …
      return
    // les trois autres états, puis le default en never
  }
}
```

</details>

---

## Palier 13 — Bonus : générique

**Notions** : paramètre de type `<T>`. Pas encore vu en cours : suivre les étapes dans l'ordre.

**Fichier** : `src/hasard.ts` (squelette fourni).

1. Relire `monstreAuHasard` et `objetAuHasard` : même logique (un dé à autant de faces que d'éléments, on rend
   l'élément tiré).
2. Dans `src/hasard.ts`, écrire une première version pour des nombres (décommenter l'import de `De`) :

   ```ts
   export function tirerAuHasard(elements: readonly number[], de: De): number {
     return elements[de(elements.length) - 1]
   }
   ```

3. Limite : pour des textes ou des monstres, il faudrait la recopier. Et `any` supprimerait le typage.
4. Remplacer `number` par un paramètre de type `T`, déclaré entre chevrons après le nom de la fonction. Ce qui
   sort a le type de ce qui entre :

   ```ts
   export function tirerAuHasard<T>(elements: readonly T[], de: De): T {
     return elements[de(elements.length) - 1]
   }
   ```

5. Dans `src/main.ts`, écrire `const tirage = tirerAuHasard(["pile", "face"], lancerDe)` et survoler `tirage` :
   `T` vaut `string`. Essayer avec `BESTIAIRE`. Supprimer ces lignes.
6. Utiliser `tirerAuHasard` dans `monstreAuHasard` et `objetAuHasard`.
7. `npm run palier 13`, puis `npm run bilan`.

**Validé si** : `Palier 13 validé.` et bilan toujours bon.

Pour aller plus loin (sans test) : fusionner `blesserHeros` et `blesserMonstre` en une fonction `blesser`, avec un
`T` contraint à « quelque chose qui a des PV » :

```ts
export function blesser<T extends { pv: number }>(cible: T, degats: number): T
```

---

## Palier 14 — Bonus : scores

**Notions** : `unknown`, narrowing avec `typeof` et `in`, `localStorage`.

**Fichier** : `src/scores.ts` (squelette fourni).

Le contenu de `localStorage` est un texte modifiable par n'importe qui. `JSON.parse` rend `any` : le traiter comme
un `unknown` et vérifier sa forme.

1. Ouvrir `src/scores.ts`.
2. Exporter le type `Score` : `nom` (texte), `classe` (`ClasseHeros`), `points` (nombre).
3. Exporter la fonction `lireScores(texte)` → tableau de `Score`. `texte` : `string | null` (ce que rend
   `localStorage.getItem`).
   a. `null` : tableau vide.
   b. JSON invalide : tableau vide, sans plantage (`try` / `catch`).
   c. JSON qui n'est pas un tableau : tableau vide (`Array.isArray`).
   d. Ne garder que les éléments valides : objet non `null`, `nom` texte, `classe` valide (`lireClasse`), `points`
      nombre.
   e. Champs en trop ignorés : reconstruire chaque score avec ses trois champs.
4. Exporter la fonction `ajouterScore(scores, score)` → nouveau tableau, trié du meilleur au moins bon, limité aux
   5 meilleurs. Le tableau reçu n'est pas modifié.
5. `npm run palier 14`
6. À l'écran : à la victoire et à la défaite, relire les scores, ajouter celui de la partie, réécrire dans
   `localStorage` (`JSON.stringify`), afficher le tableau sur l'écran de fin. Une partie = un seul enregistrement.

**Validé si** : `Palier 14 validé.` et le tableau est toujours là après un rechargement de la page.

<details>
<summary>Indice : vérifier la forme d'un unknown</summary>

```ts
if (typeof valeur === "object" && valeur !== null && "nom" in valeur) {
  valeur.nom   // accessible, de type unknown : reste à tester typeof valeur.nom === "string"
}
```

</details>

---

## Palier 15 — Bonus libres

Pas de test.

1. Une action « se défendre » : la prochaine riposte est divisée par deux. L'ajouter à `Action` et suivre les
   erreurs du compilateur.
2. Un troisième sort, ajouté à `Sort`.
3. Une boutique entre deux salles : un nouvel état dans `Partie`, l'or s'échange contre des potions.
4. De l'expérience : des points par monstre vaincu, +2 en attaque tous les 50 points.
5. Une attaque spéciale du dragon : un tour sur trois, des dégâts qui ignorent la défense.
6. Un mode difficile choisi à l'accueil : un type `Difficulte`, un donjon de 12 salles.
7. L'interface : animation des barres de vie, version téléphone, coups critiques en couleur dans le journal.

---

## Erreurs du compilateur les plus fréquentes

| Message | Cause |
|---|---|
| `Type '"dragon"' is not assignable to type 'ClasseHeros'` | Valeur hors de l'union. |
| `Property 'soin' does not exist on type …` | Champ lu avant d'avoir testé le discriminant (`type`, `etat`). |
| `'x' is possibly 'undefined'` | Cas d'absence non traité (`?.`, `??` ou `if`). |
| `Cannot assign to 'nom' because it is a read-only property` | Champ `readonly` : rendre une copie. |
| `Function lacks ending return statement…` | Un chemin ne rend rien : un `case` manque. |
| `Type … is not assignable to type 'never'` | `switch` exhaustif : le cas nommé est oublié. |
| `'Heros' is a type and must be imported using a type-only import` | `import type { Heros }`. |
| `Parameter 'x' implicitly has an 'any' type` | Paramètre sans type. |
| `Unused '@ts-expect-error' directive` (dans un test) | Type trop large : lire le commentaire de la ligne. |
