# Documentation — Mini Adventure

| Document | Version | Date | Contenu |
|---|---|---|---|
| [systeme-de-vies.md](systeme-de-vies.md) | 1.0.0 | 28/09/2026 | Le système de vies de Candy Crush, puis sa version adaptée aux 3–10 ans (« cœurs »). Moteur de référence testé, interface, espace parent, 3 lots. |
| [monetisation.md](monetisation.md) | 1.0.0 | 28/09/2026 | Diamants à acheter, objets extraordinaires, agrandir son chez-moi. Cadre légal, économie chiffrée, parcours d'achat parent, code de référence testé, 5 lots. |

Le code de référence des deux documents a été exécuté : 47 tests réussis, TypeScript strict.

## Consignes à coller dans Claude Code

**Système de cœurs, lot 1**

```
Lis docs/systeme-de-vies.md en entier. Implémente le lot 1 (§10) : porte le moteur du §7 et ses tests du §11 dans la stack du projet, branche-le sur le lancement et la fin des niveaux, et sauvegarde l'état. Respecte les règles non négociables du §0. Incrémente la version de l'app. Montre-moi les tests qui passent.
```

**Monétisation, lot A**

```
Lis docs/monetisation.md en entier. Implémente le lot A (§11) : porte wallet.ts et ses tests (§9.1), crée le fichier de configuration du §8, branche les gains du §3.2 avec leurs identifiants de journal. Aucun achat réel à ce stade. Respecte la liste noire du §2.1. Incrémente la version de l'app.
```

Enchaîner ensuite lot par lot (cœurs : 2, 3 ; monétisation : B, C, D, E), en vérifiant les critères d'acceptation de chaque lot avant de passer au suivant.

## Règle de version

Chaque modification d'un document incrémente sa version (en-tête, historique en bas du document et tableau ci-dessus) :
- correctif ou précision : `1.0.0` → `1.0.1` ;
- ajout d'une section ou changement de règle : `1.0.0` → `1.1.0` ;
- refonte : `1.0.0` → `2.0.0`.
