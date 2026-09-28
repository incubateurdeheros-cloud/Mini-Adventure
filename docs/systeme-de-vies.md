# Système de cœurs (vies) — Mini Adventure

> **Version 1.0.0** · 28/09/2026 · Statut : prêt à implémenter
> Spécification pour Claude Code. Le moteur de référence (§7) a été exécuté et testé avant d'être écrit ici : 27 tests, TypeScript strict.

---

## 0. Pour Claude Code : à lire en premier

**Contexte.** Mini Adventure est un jeu mobile pour les 3–10 ans. Ce document décrit le système de vies de Candy Crush (§1), puis la version adaptée à Mini Adventure (§2 à §12). **Implémente la version adaptée, pas celle de Candy Crush.**

**Avant d'écrire du code**

1. Repère dans le projet :
   - le lancement d'un niveau (tap sur la carte, bouton « Réessayer ») ;
   - la fin d'un niveau (victoire, échec, abandon) ;
   - le stockage persistant de la sauvegarde ;
   - le composant `ParentGate` et l'espace parent ;
   - les notifications locales, s'il y en a ;
   - le système d'événements ou de cadeau du jour, s'il y en a.
2. Adapte le moteur de référence (§7) au langage et aux conventions du projet. Il est écrit en TypeScript sans aucune dépendance. S'il faut le porter (Dart, C#, Swift, Kotlin, GDScript), garde les mêmes noms de fonctions et porte aussi les tests (§11).
3. Mets toutes les valeurs réglables dans un seul objet de configuration (§4).

**Règles non négociables**

- Les cœurs ne sont **jamais vendus** : ni en euros, ni en diamants. Aucun bouton d'achat, aucun prix sur les écrans liés aux cœurs.
- Côté enfant, aucun texte du type « Achète… », « Demande à tes parents… ».
- Le temps se calcule à partir d'horodatages, **jamais** avec un minuteur qui décrémente en tâche de fond (§6).
- Une sauvegarde illisible donne des cœurs pleins. Un bug ne doit jamais bloquer un enfant.

**Livraison en 3 lots** (§10), chacun testable séparément. À chaque lot livré, **incrémente le numéro de version de l'app** selon la convention du projet (par exemple `1.4.0` → `1.5.0` pour une nouvelle fonctionnalité, `1.5.0` → `1.5.1` pour un correctif).

---

## 1. Le système de Candy Crush Saga, tel qu'il existe

| Règle | Candy Crush Saga |
|---|---|
| Nombre de vies | 5 au maximum. Des événements et des bonus peuvent temporairement en donner davantage. |
| Recharge | 1 vie toutes les 30 minutes, tant qu'on est sous 5. De 0 à 5 : 2 h 30. |
| Perte d'une vie | Échouer un niveau (plus de coups) **ou** le quitter en cours de partie. Les boosters utilisés sont perdus aussi. |
| Victoire | Aucune vie consommée. |
| À 0 vie | Écran « Plus de vies » : compte à rebours jusqu'à la prochaine vie, recharge payante, demande aux amis. |
| Recharge payante | En lingots d'or (monnaie achetée en argent réel). Ordres de grandeur relevés : 1 vie ≈ 4 lingots, 5 vies ≈ 12 lingots. Les prix varient selon les versions et les tests A/B. |
| Vies illimitées | Une période (2 h, 6 h, 24 h, vendues environ 39 / 69 / 89 lingots) pendant laquelle échouer ne coûte rien. Aussi données en récompense d'événements ou dans des offres groupées. Le compteur affiche ∞ et un minuteur. |
| Amis | On peut demander des vies à ses amis et leur en envoyer. |
| Notification | « Tes vies sont rechargées » quand le compteur est plein. |
| Horloge | Le minuteur s'appuie sur l'heure de l'appareil. Après un changement d'heure (voyage, réglage manuel), King documente des attentes aberrantes de « centaines de minutes » et une procédure manuelle pour les corriger. |
| Réglage du délai | King a déjà fait passer la recharge à 1 h par vie pour une partie des joueurs, ce qui a déclenché une vague de protestations sur son forum. |

**Détail technique fréquent dans ce genre de jeu** : la vie est débitée dès le lancement du niveau, puis rendue en cas de victoire. Fermer l'appli en pleine partie ne permet donc pas d'échapper à la perte. Mini Adventure n'en a pas besoin (§2).

### Pourquoi ça marche, et ce qu'on ne reproduit pas

- **Rythme.** Les sessions sont courtes et on revient plusieurs fois par jour : c'est une « mécanique de rendez-vous ».
- **Enjeu.** Chaque essai compte, donc on joue plus attentivement.
- **Vente au moment de la frustration.** L'échec vide le compteur, et l'écran « Plus de vies » propose de payer à cet instant précis. C'est le cœur du modèle économique de Candy Crush, et c'est exactement ce qu'il ne faut pas faire dans un jeu pour 3–10 ans : règles Apple et Google pour les apps enfants, droit européen de la consommation. Le détail est dans `monetisation.md`.

---

## 2. La version Mini Adventure : ce qu'on garde, ce qu'on change

**Idée directrice.** Chez Candy Crush, les vies servent à vendre. Chez Mini Adventure, les cœurs servent à **donner du rythme et des pauses naturelles**. C'est un argument pour les parents, pas une source de frustration pour l'enfant.

| Sujet | Candy Crush | Mini Adventure | Pourquoi |
|---|---|---|---|
| Nom | Vies | **Cœurs** : le héros se fatigue, puis fait une sieste | Compréhensible sans savoir lire |
| Maximum | 5 | **5** | |
| Recharge | 30 min par vie | **20 min par vie** (de 0 à 5 : 1 h 40) | Une vraie pause, sans attente décourageante. Réglable. |
| Échec | −1 | **−1** | L'essai garde un enjeu |
| Abandon volontaire | −1 | **Gratuit** | L'enfant est appelé à table : on ne le punit pas |
| Appli fermée en pleine partie | −1 | **Gratuit** | Même raison |
| Victoire | 0 | **0** | |
| Premiers niveaux | Payants | **Niveaux 1 à 10 gratuits** | Découvrir le jeu sans pression |
| À 0 cœur | Achat, amis, attente | **Écran « sieste »**, attente, accès libre à la maison | Le temps d'attente devient du temps de jeu calme |
| Achat de cœurs | Oui | **Jamais**, ni en €, ni en 💎 | Pas de boucle frustration → achat |
| Illimité | Acheté ou gagné | **Offert** : par le parent, ou lors d'événements | |
| Cœurs bonus au-delà de 5 | Oui | **Oui, jusqu'à 10** (cadeaux, événements) | |
| Amis | Oui | **Non** | Pas de social ni d'échange de données entre enfants |
| Notification « plein » | Oui, par défaut | **Désactivée par défaut**, activable par le parent | On ne relance pas l'enfant |
| Recharge par le parent | Non | **Oui, gratuite**, derrière `ParentGate` | Le parent garde la main |
| Triche à l'horloge | Combattue | **Tolérée** (§6) | Rien n'est vendu, la triche ne coûte rien |

**Échecs répétés sur un même niveau.** La bonne réponse est un coup de pouce dans le gameplay (indice, coups en plus), pas une règle de cœurs. C'est hors du périmètre de ce document.

---

## 3. Règles détaillées

| # | Règle |
|---|---|
| R1 | `lives` est un entier, `0 ≤ lives ≤ hardCap` (10). |
| R2 | La recharge naturelle n'a lieu que si `lives < maxLives` (5). +1 cœur par `regenInterval`, jamais au-delà de `maxLives`. |
| R3 | Le minuteur démarre quand le compteur passe sous `maxLives`. Perdre un autre cœur ne le remet **pas** à zéro : le cœur en cours continue de se remplir. |
| R4 | Lancer un niveau est autorisé si : niveau ≤ `freeLevelsUpTo`, **ou** illimité actif, **ou** `lives ≥ 1`. Sinon : écran « sieste ». |
| R5 | Rien n'est débité au lancement. En fin de partie : victoire → rien ; abandon → rien (sauf si `loseLifeOnQuit`) ; échec → −1, **sauf** si l'essai était gratuit au lancement ou si l'illimité est actif à la fin. |
| R6 | Les cœurs bonus s'ajoutent jusqu'à `hardCap`. Au-dessus de `maxLives`, pas de recharge. Elle reprend dès qu'on repasse sous `maxLives`. |
| R7 | L'illimité temporaire se cumule : un nouveau don prolonge à partir de la fin en cours. La recharge naturelle continue pendant l'illimité. |
| R8 | L'illimité parent est permanent tant que le réglage est actif. |
| R9 | La recharge parent donne `lives = max(lives, maxLives)`. Elle ne retire jamais de cœurs bonus. |
| R10 | Une sauvegarde absente, illisible ou d'un format inconnu donne des cœurs pleins. |

---

## 4. Configuration

Un seul objet, que l'équipe peut régler sans toucher au code :

```ts
export const DEFAULT_LIVES_CONFIG: LivesConfig = {
  maxLives: 5,
  regenIntervalMs: 20 * 60_000,
  hardCap: 10,
  freeLevelsUpTo: 10,
  loseLifeOnQuit: false,
};
```

| Paramètre | Mini Adventure | Candy Crush | Effet |
|---|---|---|---|
| `maxLives` | 5 | 5 | Nombre d'essais ratés avant la pause |
| `regenIntervalMs` | 20 min | 30 min | Durée de la pause (×5 pour une recharge complète) |
| `hardCap` | 10 | variable | Plafond des cœurs bonus |
| `freeLevelsUpTo` | 10 | 0 | Niveaux de découverte sans enjeu |
| `loseLifeOnQuit` | `false` | `true` | Abandonner coûte-t-il un cœur ? |

Si le jeu numérote ses niveaux par monde (1-1, 1-2…), remplace `levelNumber` par un index global, ou par un booléen `isDiscoveryLevel` calculé à partir des données du niveau.

---

## 5. Modèle de données

```ts
interface LivesState {
  schemaVersion: 1;
  lives: number;                   // 0..hardCap
  regenAnchorMs: number | null;    // début du cœur en cours de recharge ; null si lives >= maxLives
  unlimitedUntilMs: number | null; // fin de l'illimité temporaire (événement)
  parentUnlimited: boolean;        // réglage de l'espace parent
}
```

- C'est tout ce qui est sauvegardé : quelques octets, dans la sauvegarde existante du jeu.
- On n'enregistre ni compteur de secondes ni « heure de la dernière recharge » : tout se déduit de `regenAnchorMs`.
- L'essai en cours (`Attempt`) n'est **pas** sauvegardé. Si l'appli est tuée en pleine partie, l'essai disparaît et aucun cœur n'est débité, ce qui est le comportement voulu.

---

## 6. Le temps : calcul paresseux et horloge

**Principe.** Aucun `setInterval` ne modifie l'état. L'état ne change que lorsqu'on appelle `settle(state, now)`, qui calcule combien de cœurs sont revenus depuis `regenAnchorMs`. Toutes les fonctions du moteur appellent `settle` elles-mêmes.

**Quand appeler le moteur et sauvegarder**

- au démarrage de l'appli ;
- au retour au premier plan (`AppState` → `active`, `onResume`, `applicationDidBecomeActive`…) ;
- au lancement et à la fin de chaque niveau ;
- depuis l'espace parent (recharge, illimité).

**Affichage.** Un tick d'une seconde tourne **uniquement** quand le compteur ou l'écran « sieste » est visible. Il appelle `getLivesView` (fonction pure) et ne sauvegarde rien.

**Horloge**

| Cas | Comportement |
|---|---|
| Horloge reculée (fuseau, réglage manuel) | L'ancre est ramenée à maintenant : l'attente affichée ne dépasse jamais un intervalle. Le bug des « centaines de minutes » de Candy Crush ne peut pas se produire. |
| Horloge avancée | L'enfant récupère des cœurs. **Accepté** : aucun cœur n'est vendu, et un anti-triche côté serveur coûterait plus qu'il ne rapporterait. Si un jour les cœurs avaient une valeur marchande, il faudrait une heure serveur. |
| Changement de fuseau | Sans effet : tout est en epoch ms (UTC). |
| Illimité et horloge reculée | L'illimité peut durer plus longtemps. Accepté pour la même raison. |

`now` vaut `Date.now()` en production, et il est injecté dans les tests.

---

## 7. Moteur de référence (TypeScript, testé)

Fonctions pures, sans dépendance. Chaque fonction prend l'état et `now`, et renvoie un **nouvel** état : c'est à l'appelant de le sauvegarder.

| Fonction | Rôle |
|---|---|
| `createLivesState()` | État initial : cœurs pleins |
| `settle(state, now)` | Applique la recharge écoulée |
| `startAttempt(state, levelNumber, now)` | Autorise ou refuse le lancement, et renvoie un `Attempt` |
| `endAttempt(state, attempt, outcome, now)` | Applique le résultat : `'won'`, `'failed'` ou `'quit'` |
| `grantLives(state, n, now)` | Cœurs bonus, jusqu'à `hardCap` |
| `refillToMax(state, now)` | Recharge offerte par le parent |
| `grantUnlimited(state, durationMs, now)` | Illimité temporaire, cumulable |
| `setParentUnlimited(state, on, now)` | Réglage parent |
| `getLivesView(state, now)` | Données d'affichage, sans effet de bord |
| `loadLivesState(raw)` | Chargement tolérant de la sauvegarde |

```ts
// lives.ts — moteur de cœurs, fonctions pures, aucune dépendance.
// Toutes les heures sont des epoch ms. `now` est toujours passé en paramètre (testable).

export interface LivesConfig {
  maxLives: number;          // plafond de la recharge naturelle
  regenIntervalMs: number;   // durée pour regagner 1 cœur
  hardCap: number;           // plafond absolu (cœurs bonus compris)
  freeLevelsUpTo: number;    // niveaux 1..N : jamais de cœur consommé
  loseLifeOnQuit: boolean;   // abandon volontaire = perte ?
}

export const DEFAULT_LIVES_CONFIG: LivesConfig = {
  maxLives: 5,
  regenIntervalMs: 20 * 60_000,
  hardCap: 10,
  freeLevelsUpTo: 10,
  loseLifeOnQuit: false,
};

export interface LivesState {
  schemaVersion: 1;
  lives: number;
  /** Début du cœur en cours de recharge. null quand lives >= maxLives. */
  regenAnchorMs: number | null;
  /** Fin des cœurs illimités temporaires (événement). */
  unlimitedUntilMs: number | null;
  /** Réglage de l'espace parent : cœurs illimités permanents. */
  parentUnlimited: boolean;
}

export interface Attempt {
  levelNumber: number;
  free: boolean;
  startedAtMs: number;
}

export type StartResult =
  | { ok: true; state: LivesState; attempt: Attempt }
  | { ok: false; state: LivesState; reason: 'NO_LIVES'; msToNextLife: number };

export type Outcome = 'won' | 'failed' | 'quit';

export interface LivesView {
  lives: number;
  maxLives: number;
  isFull: boolean;
  isUnlimited: boolean;
  /** null si pas d'illimité temporaire en cours (ou illimité parent). */
  unlimitedMsLeft: number | null;
  /** null si plein. */
  msToNextLife: number | null;
  /** null si plein. */
  msToFull: number | null;
  /** 0..1, remplissage du cœur en cours ; 1 si plein. */
  nextLifeProgress: number;
}

export function createLivesState(cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesState {
  return { schemaVersion: 1, lives: cfg.maxLives, regenAnchorMs: null, unlimitedUntilMs: null, parentUnlimited: false };
}

/** Applique la recharge écoulée. À appeler avant toute lecture ou écriture. */
export function settle(s: LivesState, now: number, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesState {
  let { lives, regenAnchorMs } = s;
  if (lives >= cfg.maxLives) {
    regenAnchorMs = null;
  } else {
    // Ancre absente (défensif) ou dans le futur (horloge reculée : fuseau, réglage manuel) :
    // on ré-ancre à maintenant, l'attente ne peut donc jamais dépasser un intervalle.
    if (regenAnchorMs === null || regenAnchorMs > now) regenAnchorMs = now;
    const gained = Math.floor((now - regenAnchorMs) / cfg.regenIntervalMs);
    if (gained > 0) {
      lives = Math.min(cfg.maxLives, lives + gained);
      regenAnchorMs = lives >= cfg.maxLives ? null : regenAnchorMs + gained * cfg.regenIntervalMs;
    }
  }
  const unlimitedUntilMs = s.unlimitedUntilMs !== null && s.unlimitedUntilMs <= now ? null : s.unlimitedUntilMs;
  return { ...s, lives, regenAnchorMs, unlimitedUntilMs };
}

export function isUnlimited(s: LivesState, now: number): boolean {
  return s.parentUnlimited || (s.unlimitedUntilMs !== null && now < s.unlimitedUntilMs);
}

export function startAttempt(s0: LivesState, levelNumber: number, now: number, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): StartResult {
  const state = settle(s0, now, cfg);
  const free = levelNumber <= cfg.freeLevelsUpTo || isUnlimited(state, now);
  if (!free && state.lives < 1) {
    return { ok: false, state, reason: 'NO_LIVES', msToNextLife: msToNextLife(state, now, cfg) ?? 0 };
  }
  return { ok: true, state, attempt: { levelNumber, free, startedAtMs: now } };
}

export function endAttempt(s0: LivesState, attempt: Attempt, outcome: Outcome, now: number, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesState {
  const s = settle(s0, now, cfg);
  if (outcome === 'won') return s;
  if (outcome === 'quit' && !cfg.loseLifeOnQuit) return s;
  // Gratuit si l'essai l'était au départ OU si l'illimité est actif maintenant.
  if (attempt.free || isUnlimited(s, now)) return s;
  if (s.lives <= 0) return s;
  const lives = s.lives - 1;
  const regenAnchorMs = lives < cfg.maxLives ? (s.regenAnchorMs ?? now) : null;
  return { ...s, lives, regenAnchorMs };
}

/** Cœurs bonus (cadeau du jour, événement). Peuvent dépasser maxLives jusqu'à hardCap. */
export function grantLives(s0: LivesState, n: number, now: number, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesState {
  const s = settle(s0, now, cfg);
  const lives = Math.min(cfg.hardCap, s.lives + Math.max(0, Math.floor(n)));
  return { ...s, lives, regenAnchorMs: lives >= cfg.maxLives ? null : s.regenAnchorMs };
}

/** Recharge gratuite déclenchée par le parent. Ne retire jamais de cœurs bonus. */
export function refillToMax(s0: LivesState, now: number, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesState {
  const s = settle(s0, now, cfg);
  return { ...s, lives: Math.max(s.lives, cfg.maxLives), regenAnchorMs: null };
}

/** Illimité temporaire. Se cumule : on prolonge à partir de la fin actuelle. */
export function grantUnlimited(s0: LivesState, durationMs: number, now: number, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesState {
  const s = settle(s0, now, cfg);
  const from = Math.max(now, s.unlimitedUntilMs ?? now);
  return { ...s, unlimitedUntilMs: from + durationMs };
}

export function setParentUnlimited(s0: LivesState, on: boolean, now: number, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesState {
  return { ...settle(s0, now, cfg), parentUnlimited: on };
}

function msToNextLife(s: LivesState, now: number, cfg: LivesConfig): number | null {
  if (s.regenAnchorMs === null) return null;
  return s.regenAnchorMs + cfg.regenIntervalMs - now;
}

/** Vue pour l'UI. Pure : ne persiste rien. */
export function getLivesView(s0: LivesState, now: number, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesView {
  const s = settle(s0, now, cfg);
  const next = msToNextLife(s, now, cfg);
  return {
    lives: s.lives,
    maxLives: cfg.maxLives,
    isFull: s.lives >= cfg.maxLives,
    isUnlimited: isUnlimited(s, now),
    unlimitedMsLeft: !s.parentUnlimited && s.unlimitedUntilMs !== null ? s.unlimitedUntilMs - now : null,
    msToNextLife: next,
    msToFull: next === null ? null : next + (cfg.maxLives - s.lives - 1) * cfg.regenIntervalMs,
    nextLifeProgress: next === null ? 1 : (now - (s.regenAnchorMs as number)) / cfg.regenIntervalMs,
  };
}

/** Chargement tolérant : une sauvegarde illisible donne des cœurs pleins, jamais un blocage. */
export function loadLivesState(raw: unknown, cfg: LivesConfig = DEFAULT_LIVES_CONFIG): LivesState {
  const r = raw as Partial<LivesState> | null;
  if (!r || typeof r !== 'object' || r.schemaVersion !== 1 || typeof r.lives !== 'number' || !Number.isFinite(r.lives)) {
    return createLivesState(cfg);
  }
  const num = (v: unknown) => (typeof v === 'number' && Number.isFinite(v) ? v : null);
  return {
    schemaVersion: 1,
    lives: Math.max(0, Math.min(cfg.hardCap, Math.floor(r.lives))),
    regenAnchorMs: num(r.regenAnchorMs),
    unlimitedUntilMs: num(r.unlimitedUntilMs),
    parentUnlimited: r.parentUnlimited === true,
  };
}
```

---

## 8. Intégration dans le jeu

```ts
// Tap sur un niveau de la carte, ou bouton « Réessayer »
function onPlayLevel(level: { number: number }) {
  const r = startAttempt(save.lives, level.number, Date.now());
  save.lives = r.state;
  persist();
  if (!r.ok) {
    showNapScreen(); // §9.3
    return;
  }
  currentAttempt = r.attempt; // en mémoire seulement, jamais sauvegardé
  launchLevel(level);
}

// Fin de niveau
function onLevelEnded(outcome: 'won' | 'failed' | 'quit') {
  if (!currentAttempt) return;
  const now = Date.now();
  const settled = settle(save.lives, now); // pour comparer à un état à jour
  save.lives = endAttempt(settled, currentAttempt, outcome, now);
  currentAttempt = null;
  persist();
  if (save.lives.lives < settled.lives) playHeartLostAnimation(); // §9.2
  rescheduleLivesNotification(); // lot 3
}

// Retour au premier plan
function onAppForeground() {
  save.lives = settle(save.lives, Date.now());
  persist();
  rescheduleLivesNotification();
}
```

- Sauvegarde après chaque changement : l'état est minuscule, inutile de temporiser.
- « Réessayer » depuis l'écran d'échec correspond à un **nouveau** `startAttempt`.
- Si le jeu a déjà une sauvegarde globale versionnée, ajoute `lives` dedans, avec migration : une absence de champ donne `createLivesState()`.

---

## 9. Interface

### 9.1 Compteur (carte du monde, écran d'accueil)

- **5 emplacements de cœur**, pleins ou vides. Le cœur en recharge se remplit progressivement, de bas en haut, selon `nextLifeProgress`. C'est lisible sans savoir lire.
- En dessous, en petit : « Plein », ou le temps avant le prochain cœur au format `mm:ss`.
- Cœurs bonus : un badge doré `+2` à droite des 5 emplacements.
- Illimité : les 5 cœurs deviennent dorés avec un symbole ∞. Minuteur `h:mm` s'il est temporaire, rien s'il vient du parent.
- Tap sur le compteur : bulle courte (avec voix off si le jeu en a) : « Quand tu rates un niveau, [Héros] se fatigue un peu. Ses cœurs reviennent tout seuls. »

### 9.2 Fin de niveau échoué

- Un cœur quitte doucement le compteur (animation de 600 à 800 ms, qui flotte vers le haut et s'efface), sans son dramatique.
- Boutons : « Réessayer » (s'il reste des cœurs, ou si le niveau est gratuit) et « Carte ».
- Si l'essai était gratuit (niveau de découverte, illimité) : pas d'animation de perte.
- S'il ne reste plus de cœur, « Réessayer » ouvre l'écran « sieste ».

### 9.3 Écran « sieste » (0 cœur)

- **Illustration.** Le héros dort, avec une animation lente et des « Zzz ». Une grande jauge se remplit en temps réel.
- **Texte.** « [Héros] fait une petite sieste. Un cœur revient dans 12 min. »
- **Bouton principal.** « Aller à ma maison » : la maison reste accessible pour ranger, décorer et jouer avec les objets animés (voir `monetisation.md`, §7).
- **Bouton secondaire.** « Retour à la carte ». Les niveaux de découverte (1 à 10) restent jouables.
- **Interdit** : bouton d'achat, prix, mention des parents, compte à rebours stressant (pas de rouge, pas de clignotement, pas de son d'alerte).

### 9.4 Espace parent (derrière `ParentGate`)

Section « Cœurs » :

- **Explication**, deux phrases : « Les cœurs créent des pauses naturelles : après 5 niveaux ratés, votre enfant se repose, et un cœur revient toutes les 20 minutes. Ils ne sont jamais vendus. »
- **État actuel.** « 2 cœurs sur 5, recharge complète dans 58 min » (d'après `msToFull`).
- **Bouton** « Recharger les cœurs maintenant » (gratuit, `refillToMax`).
- **Interrupteur** « Cœurs illimités » (`setParentUnlimited`).
- **Interrupteur** « Me prévenir quand les cœurs sont rechargés » : la demande d'autorisation des notifications se fait **ici**, derrière la barrière parentale. Apple exige une barrière parentale avant toute demande de permission dans la catégorie Enfants.

### 9.5 Textes

| Clé | Français |
|---|---|
| `lives.full` | Plein |
| `lives.next_in` | Prochain cœur dans {mm:ss} |
| `lives.tooltip` | Quand tu rates un niveau, {hero} se fatigue un peu. Ses cœurs reviennent tout seuls. |
| `lives.nap.title` | {hero} fait une petite sieste |
| `lives.nap.body` | Un cœur revient dans {minutes} min. |
| `lives.nap.go_home` | Aller à ma maison |
| `lives.nap.back_map` | Retour à la carte |
| `parent.lives.explain` | Les cœurs créent des pauses naturelles : après 5 niveaux ratés, votre enfant se repose, et un cœur revient toutes les 20 minutes. Ils ne sont jamais vendus. |
| `parent.lives.status` | {n} cœurs sur {max}. Recharge complète dans {duration}. |
| `parent.lives.refill` | Recharger les cœurs maintenant |
| `parent.lives.unlimited` | Cœurs illimités |
| `parent.lives.notify` | Me prévenir quand les cœurs sont rechargés |
| `notif.lives.full` | Les cœurs de {hero} sont rechargés. |

Les valeurs `20` et `5` des textes parent viennent de la configuration, pas de chaînes en dur.

---

## 10. Lots de livraison et critères d'acceptation

### Lot 1 : moteur et branchement

- Porter le moteur (§7) et ses tests (§11). Tous les tests passent.
- Brancher `startAttempt` et `endAttempt` sur le lancement et la fin des niveaux.
- Sauvegarder l'état, appeler `settle` au démarrage et au retour au premier plan.
- Affichage minimal provisoire : nombre de cœurs et minuteur en texte.

**Critères d'acceptation**

- [ ] Rater un niveau ≥ 11 retire un cœur ; le réussir n'en retire pas.
- [ ] Quitter un niveau via le menu ne retire rien. Tuer l'appli en pleine partie ne retire rien.
- [ ] Les niveaux 1 à 10 sont jouables à 0 cœur et ne retirent jamais de cœur.
- [ ] À 0 cœur, un niveau ≥ 11 ne se lance pas.
- [ ] Appli fermée 20 min → +1 cœur à la réouverture. Fermée 3 jours → 5 cœurs, pas plus.
- [ ] Reculer l'heure du téléphone de 5 h n'affiche jamais plus de 20 min d'attente.
- [ ] Corrompre la sauvegarde des cœurs → 5 cœurs au démarrage, pas de plantage.
- [ ] Version de l'app incrémentée.

### Lot 2 : interface enfant

- Compteur (§9.1), animation de perte (§9.2), écran « sieste » (§9.3), textes (§9.5).

**Critères d'acceptation**

- [ ] Le cœur en recharge se remplit visiblement en temps réel.
- [ ] L'écran « sieste » mène à la maison, et aucun élément d'achat n'apparaît sur les écrans liés aux cœurs.
- [ ] Le tick d'affichage s'arrête quand le compteur n'est pas visible (vérifier qu'aucun timer ne tourne en arrière-plan).
- [ ] Version de l'app incrémentée.

### Lot 3 : espace parent, notifications, événements

- Section « Cœurs » de l'espace parent (§9.4).
- Notification locale « cœurs rechargés » : désactivée par défaut. Quand le parent l'active et que le compteur passe sous 5, programmer une notification à `now + msToFull`, avec un identifiant fixe (`lives-full`). L'annuler et la reprogrammer à chaque changement d'état. L'annuler quand le compteur est plein ou l'illimité actif.
- Exposer `grantLives` et `grantUnlimited` au système d'événements ou de cadeau du jour (par exemple « Week-end illimité » à Noël, +1 cœur bonus dans le cadeau du dimanche).

**Critères d'acceptation**

- [ ] « Recharger maintenant » remplit à 5 et ne retire pas les bonus.
- [ ] « Cœurs illimités » : on peut rater autant de fois qu'on veut sans rien perdre, le compteur affiche ∞.
- [ ] La permission de notification n'est jamais demandée hors de l'espace parent.
- [ ] Une seule notification programmée à la fois, et aucune quand les cœurs sont pleins.
- [ ] Version de l'app incrémentée.

---

## 11. Tests

Les tests ci-dessous ont été exécutés sur le moteur du §7 (`bun test` : 27 tests réussis). Ils utilisent l'API `describe` / `test` / `expect`, compatible avec Jest et Vitest : seule la ligne d'import change.

| Domaine | Cas couverts |
|---|---|
| Recharge | état initial ; minuteur qui démarre au premier échec ; retour à plein ; minuteur qui ne repart pas à chaque échec ; `msToFull` ; longue absence ; `settle` idempotent |
| Horloge | horloge reculée (attente ≤ 1 intervalle) ; horloge avancée (plafonnée) |
| Règles de partie | victoire ; abandon gratuit ou payant selon config ; niveaux de découverte ; refus à 0 cœur ; dernier cœur |
| Illimité | temporaire puis expiration ; jouable à 0 cœur ; expiration pendant la partie ; cumul ; illimité parent |
| Bonus et parent | au-delà du max ; redescente sous le max ; plafond absolu ; bonus qui ramène au max ; recharge parent |
| Sauvegarde | aller-retour JSON ; sauvegarde illisible ; valeurs hors bornes |

<details>
<summary>Fichier de tests complet (<code>lives.test.ts</code>)</summary>

```ts
import { describe, expect, test } from 'bun:test';
import {
  DEFAULT_LIVES_CONFIG as C, createLivesState, settle, startAttempt, endAttempt, grantLives,
  refillToMax, grantUnlimited, setParentUnlimited, getLivesView, loadLivesState, type LivesState,
} from './lives';

const MIN = 60_000;
const I = C.regenIntervalMs; // 20 min
const T0 = Date.UTC(2026, 8, 28, 18, 0, 0);
const LVL = 42; // niveau payant (> freeLevelsUpTo)

function fail(s: LivesState, now: number, level = LVL) {
  const r = startAttempt(s, level, now);
  if (!r.ok) throw new Error('start refused');
  return endAttempt(r.state, r.attempt, 'failed', now);
}

describe('recharge', () => {
  test('état initial : plein, pas de minuteur', () => {
    const v = getLivesView(createLivesState(), T0);
    expect(v).toMatchObject({ lives: 5, isFull: true, msToNextLife: null, msToFull: null, nextLifeProgress: 1 });
  });
  test('échec à plein : 4 cœurs, minuteur démarre', () => {
    const s = fail(createLivesState(), T0);
    expect(s.lives).toBe(4);
    expect(s.regenAnchorMs).toBe(T0);
    expect(getLivesView(s, T0).msToNextLife).toBe(I);
  });
  test('un intervalle plus tard : plein', () => {
    const s = fail(createLivesState(), T0);
    expect(getLivesView(s, T0 + I)).toMatchObject({ lives: 5, isFull: true, msToNextLife: null });
  });
  test('3 échecs rapprochés : le minuteur ne repart pas à chaque échec', () => {
    let s = createLivesState();
    s = fail(s, T0); s = fail(s, T0 + MIN); s = fail(s, T0 + 2 * MIN);
    expect(s.lives).toBe(2);
    expect(s.regenAnchorMs).toBe(T0);
    expect(getLivesView(s, T0 + I).lives).toBe(3);
    expect(getLivesView(s, T0 + 2 * I).lives).toBe(4);
    expect(getLivesView(s, T0 + 3 * I - 1).lives).toBe(4);
    expect(getLivesView(s, T0 + 3 * I).lives).toBe(5);
  });
  test('msToFull', () => {
    let s = createLivesState();
    for (let i = 0; i < 5; i++) s = fail(s, T0);
    expect(s.lives).toBe(0);
    expect(getLivesView(s, T0).msToFull).toBe(5 * I);
    expect(getLivesView(s, T0 + 7 * MIN).msToFull).toBe(5 * I - 7 * MIN);
    expect(getLivesView(s, T0 + 7 * MIN).msToNextLife).toBe(I - 7 * MIN);
  });
  test('longue absence : plein, jamais au-delà du max', () => {
    let s = createLivesState();
    for (let i = 0; i < 5; i++) s = fail(s, T0);
    const later = settle(s, T0 + 3 * 24 * 60 * MIN);
    expect(later.lives).toBe(5);
    expect(later.regenAnchorMs).toBeNull();
  });
  test("settle est idempotent et conserve l'avancement du cœur en cours", () => {
    let s = createLivesState();
    s = fail(s, T0); s = fail(s, T0);
    const a = settle(s, T0 + I + 5 * MIN);
    const b = settle(a, T0 + I + 5 * MIN);
    expect(b).toEqual(a);
    expect(a.lives).toBe(4);
    expect(getLivesView(a, T0 + I + 5 * MIN).msToNextLife).toBe(I - 5 * MIN);
  });
});

describe('horloge', () => {
  test("recul d'horloge : jamais d'attente > 1 intervalle", () => {
    let s = createLivesState();
    s = fail(s, T0); s = fail(s, T0);
    const v = getLivesView(s, T0 - 5 * 60 * MIN);
    expect(v.msToNextLife).toBe(I);
    expect(v.lives).toBe(3);
  });
  test("avance d'horloge : recharge acceptée, plafonnée au max", () => {
    let s = createLivesState();
    for (let i = 0; i < 5; i++) s = fail(s, T0);
    expect(getLivesView(s, T0 + 10 * 24 * 60 * MIN).lives).toBe(5);
  });
});

describe('règles de partie', () => {
  test('victoire : aucun cœur perdu', () => {
    const r = startAttempt(createLivesState(), LVL, T0);
    if (!r.ok) throw 0;
    expect(endAttempt(r.state, r.attempt, 'won', T0).lives).toBe(5);
  });
  test('abandon : gratuit par défaut, payant si loseLifeOnQuit', () => {
    const r = startAttempt(createLivesState(), LVL, T0);
    if (!r.ok) throw 0;
    expect(endAttempt(r.state, r.attempt, 'quit', T0).lives).toBe(5);
    expect(endAttempt(r.state, r.attempt, 'quit', T0, { ...C, loseLifeOnQuit: true }).lives).toBe(4);
  });
  test('niveaux découverte : jouables à 0 cœur, jamais de perte', () => {
    let s = createLivesState();
    for (let i = 0; i < 5; i++) s = fail(s, T0);
    const r = startAttempt(s, C.freeLevelsUpTo, T0);
    expect(r.ok).toBe(true);
    if (!r.ok) throw 0;
    expect(r.attempt.free).toBe(true);
    expect(endAttempt(r.state, r.attempt, 'failed', T0).lives).toBe(0);
  });
  test('0 cœur sur un niveau payant : refus + délai', () => {
    let s = createLivesState();
    for (let i = 0; i < 5; i++) s = fail(s, T0);
    const r = startAttempt(s, LVL, T0 + 3 * MIN);
    expect(r.ok).toBe(false);
    if (r.ok) throw 0;
    expect(r.reason).toBe('NO_LIVES');
    expect(r.msToNextLife).toBe(I - 3 * MIN);
  });
  test('dernier cœur : on peut jouer, puis 0', () => {
    let s = createLivesState();
    for (let i = 0; i < 4; i++) s = fail(s, T0);
    expect(s.lives).toBe(1);
    s = fail(s, T0);
    expect(s.lives).toBe(0);
  });
});

describe('illimité', () => {
  test('illimité temporaire : pas de perte, puis perte après expiration', () => {
    let s = grantUnlimited(createLivesState(), 60 * MIN, T0);
    s = fail(s, T0 + 10 * MIN);
    expect(s.lives).toBe(5);
    expect(getLivesView(s, T0 + 10 * MIN).unlimitedMsLeft).toBe(50 * MIN);
    s = fail(s, T0 + 61 * MIN);
    expect(s.lives).toBe(4);
    expect(s.unlimitedUntilMs).toBeNull();
  });
  test('illimité : jouable à 0 cœur', () => {
    let s = createLivesState();
    for (let i = 0; i < 5; i++) s = fail(s, T0);
    s = grantUnlimited(s, 30 * MIN, T0);
    expect(startAttempt(s, LVL, T0).ok).toBe(true);
  });
  test("illimité qui expire pendant la partie : l'essai commencé gratuit reste gratuit", () => {
    const s = grantUnlimited(createLivesState(), 5 * MIN, T0);
    const r = startAttempt(s, LVL, T0 + 4 * MIN);
    if (!r.ok) throw 0;
    expect(endAttempt(r.state, r.attempt, 'failed', T0 + 9 * MIN).lives).toBe(5);
  });
  test('cumul : prolonge depuis la fin actuelle', () => {
    let s = grantUnlimited(createLivesState(), 60 * MIN, T0);
    s = grantUnlimited(s, 30 * MIN, T0 + 10 * MIN);
    expect(s.unlimitedUntilMs).toBe(T0 + 90 * MIN);
  });
  test('illimité parent : permanent, sans minuteur affiché', () => {
    let s = setParentUnlimited(createLivesState(), true, T0);
    s = fail(s, T0);
    const v = getLivesView(s, T0);
    expect(v).toMatchObject({ lives: 5, isUnlimited: true, unlimitedMsLeft: null });
  });
});

describe('bonus et parent', () => {
  test('bonus au-delà du max, pas de recharge au-dessus', () => {
    let s = grantLives(createLivesState(), 3, T0);
    expect(s.lives).toBe(8);
    s = fail(s, T0);
    expect(s.lives).toBe(7);
    expect(s.regenAnchorMs).toBeNull();
    expect(getLivesView(s, T0 + 10 * I).lives).toBe(7);
  });
  test('descente sous le max depuis un bonus : minuteur démarre à ce moment', () => {
    let s = grantLives(createLivesState(), 1, T0); // 6
    s = fail(s, T0); // 5
    s = fail(s, T0 + 30 * MIN); // 4
    expect(s.regenAnchorMs).toBe(T0 + 30 * MIN);
  });
  test('plafond absolu', () => {
    expect(grantLives(createLivesState(), 99, T0).lives).toBe(C.hardCap);
  });
  test('bonus qui ramène au max : minuteur arrêté', () => {
    let s = fail(fail(createLivesState(), T0), T0); // 3
    s = grantLives(s, 2, T0 + MIN);
    expect(s).toMatchObject({ lives: 5, regenAnchorMs: null });
  });
  test('recharge parent : remplit, ne retire pas les bonus', () => {
    let s = fail(fail(createLivesState(), T0), T0);
    expect(refillToMax(s, T0).lives).toBe(5);
    expect(refillToMax(grantLives(createLivesState(), 2, T0), T0).lives).toBe(7);
  });
});

describe('sauvegarde', () => {
  test('aller-retour JSON', () => {
    const s = fail(grantUnlimited(createLivesState(), MIN, T0), T0 + 2 * MIN);
    expect(loadLivesState(JSON.parse(JSON.stringify(s)))).toEqual(s);
  });
  test('sauvegarde illisible : cœurs pleins', () => {
    for (const raw of [null, undefined, 'x', {}, { schemaVersion: 2, lives: 3 }, { schemaVersion: 1, lives: NaN }]) {
      expect(loadLivesState(raw)).toEqual(createLivesState());
    }
  });
  test('valeurs hors bornes : ramenées dans les bornes', () => {
    expect(loadLivesState({ schemaVersion: 1, lives: 999 }).lives).toBe(C.hardCap);
    expect(loadLivesState({ schemaVersion: 1, lives: -3 }).lives).toBe(0);
  });
});
```

</details>

---

## 12. Questions ouvertes

1. **Recharge : 20 ou 30 minutes ?** 20 min est proposé pour des enfants. À ajuster après observation de vraies sessions.
2. **Combien de niveaux de découverte ?** 10 est proposé. S'il y a un tutoriel séparé, 5 peuvent suffire.
3. **Cadeau du jour :** faut-il y ajouter un cœur bonus certains jours ? Le branchement existe (`grantLives`).
4. **Mini-jeux ou niveaux spéciaux :** consomment-ils des cœurs ? Par défaut, non : seuls les niveaux de la carte principale sont concernés.

---

## Sources

- Fonctionnement des vies dans Candy Crush Saga : [Candy Crush Saga Wiki, « Lives »](https://candycrush.fandom.com/wiki/Lives), [« Refill Lives »](https://candycrush.fandom.com/wiki/Refill_Lives)
- Attentes aberrantes après un changement d'heure : [aide officielle Candy Crush](https://candycrush.zendesk.com/hc/en-us/articles/7353277804189-The-game-says-I-have-to-wait-hundreds-of-minutes-for-my-next-life-What-happened)
- Passage de la recharge à 1 h : [forum King, « Help us bring back 30-minute lives! »](https://community.king.com/en/candy-crush-saga/discussion/263899/help-us-bring-back-30-minute-lives)
- Barrière parentale et permissions (catégorie Enfants) : [App Review Guidelines, §1.3](https://developer.apple.com/app-store/review/guidelines/)

---

## Historique

| Version | Date | Changements |
|---|---|---|
| 1.0.0 | 28/09/2026 | Première version : description de Candy Crush, adaptation 3–10 ans, moteur de référence testé, UI, espace parent, lots. |
