# TP TypeScript — Donjon

Le sujet : [`SUJET.md`](SUJET.md).

Node 22.12 minimum (`node -v`).

```bash
npm install
npm run dev          # le jeu dans le navigateur
npm run palier 0     # dans un second terminal
```

| Commande | Rôle |
|---|---|
| `npm run dev` | Lancer le jeu |
| `npm run palier 3` | Vérifier le palier 3 : compilateur, puis tests |
| `npm run bilan` | Vérifier tous les paliers |
| `npm run build` | Vérifier les types et construire |

| Chemin | Rôle |
|---|---|
| `SUJET.md` | Le sujet |
| `index.html` | La page |
| `src/main.ts` | Point d'entrée, seul fichier qui utilise jQuery |
| `src/` | Les règles du jeu : squelettes en commentaire, à compléter |
| `tests/` | Un fichier de test par palier, à ne pas modifier |
