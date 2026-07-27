# V.I.S.O.R — Note de passation projet

## Qu'est-ce que c'est
Une interface vocale conversationnelle façon HUD futuriste (esprit JARVIS/Iron Man), utilisable dans un navigateur mobile (Chrome Android testé). L'utilisateur parle ou tape, l'IA (Claude, via l'API Anthropic) répond à l'oral et à l'écrit. Nom : **V.I.S.O.R** = **V**ision · **I**ntelligence · **S**anté · **O**rganisation · **R**aisonnement — reprend les 4 zones thématiques de vie de l'utilisateur (cognition/travail, santé/relations, projets futurs, + raisonnement global transversal).

**Fichier principal actuel : `/mnt/user-data/outputs/visor.html`** (fichier HTML unique autonome, ~670 lignes, aucune dépendance externe hors polices Google Fonts et l'API Anthropic).

Les autres fichiers `.html` dans le même dossier (`jarvis-voice*.html`, `sentry-proposal-*.html`, `sentry-final.html`, `test-tts.html`) sont des **versions antérieures / brouillons de travail** conservés par accident d'itération — `visor.html` est la version de référence à faire évoluer. Les autres peuvent être ignorés ou supprimés.

## Stack technique
- HTML/CSS/JS vanilla, un seul fichier, aucun build.
- **Reconnaissance vocale** : Web Speech API navigateur (`SpeechRecognition`/`webkitSpeechRecognition`), `lang='fr-FR'`.
- **Synthèse vocale** : Web Speech API (`speechSynthesis`), voix française sélectionnée dynamiquement.
- **IA** : appel direct à l'API Anthropic (`https://api.anthropic.com/v1/messages`, modèle `claude-sonnet-4-6`) depuis le navigateur (pattern "Claude in Claude" — l'environnement Claude.ai gère la clé API automatiquement, pas de clé en dur dans le code).
- **Mémoire persistante** : `window.storage` (API de stockage clé-valeur propre aux artefacts Claude.ai), non partagée (`shared:false`), clé `sentry:conversation-history`. Chaque message a un champ `zone` (cerveau/coeur/oeil/general) pour permettre le filtrage thématique.
- **Déploiement** : le fichier est "publié" depuis l'interface Claude.ai (bouton Publish sur l'artefact), ce qui génère une URL publique ouvrable dans n'importe quel navigateur.

## Fonctionnalités déjà implémentées
1. Chat vocal + texte avec Claude, réponses lues à voix haute.
2. Mémoire de conversation persistante entre sessions (survit à la fermeture du navigateur), avec bouton de réinitialisation.
3. 4 zones thématiques cliquables (Cognition/Vitals/Horizon/Global) avec tag automatique par mots-clés sur chaque message, et filtrage du transcript par zone (base posée pour un vrai système de catégorisation plus tard).
4. Interface HUD dense : anneaux multiples animés autour d'un noyau central, brackets de ciblage, balayage radar, grille hexagonale + scanline en fond, règles graduées latérales, panneaux de tableau de bord décoratifs animés en arrière-plan.
5. **Bascule de thème dynamique** : cyan par défaut → passe automatiquement en rouge ("mode Mark") pendant que l'IA réfléchit, revient au cyan après. Implémenté via une classe CSS `mark-mode` sur `<body>` qui redéfinit les variables CSS `--cyan`/`--cyan-soft`/`--cyan-dim`/`--cyan-rgb`.
6. 4 cadrans satellites autour du noyau : PWR (vraie batterie via Battery API), CPU (vrais cœurs logiques + charge simulée — un navigateur ne peut pas lire l'usage CPU réel du téléphone), MEM (heap JS réel si Chrome l'expose, sinon simulé), NET (type de connexion réel via `navigator.connection`).
7. Couleurs d'état distinctes : écoute = cyan, réflexion = violet (ou rouge en mode Mark), réponse = doré, erreur = rouge.

## État des 4 chantiers (mis à jour par Claude Code, session du 27/07)
Les quatre points ci-dessous ont été traités dans `visor.html`. Détail de ce qui a été fait et pourquoi :

1. **Bug d'empilement** : le vrai problème n'était pas les 4 satellites (PWR/CPU/MEM/NET étaient déjà correctement positionnés) mais les **panneaux décoratifs d'arrière-plan** (`.bg-panels`, `.bp-ring`) qui chevauchaient les zone-chips, le noyau et la console sur mobile — il n'y a tout simplement pas de marge libre à côté d'un noyau de 262px sur un écran de ~375px de large. **Fix** : ces panneaux sont maintenant masqués sous 700px de large (`@media (max-width: 700px)`) et ne s'affichent que sur des fenêtres plus larges (desktop/tablette) où ils ont vraiment la place de respirer dans les marges, sans toucher au contenu réel. Vérifié visuellement à 375px (mobile) et 1100px (desktop).
2. **Icône centrale "œil vivant"** : remplacée. C'est maintenant une iris mécanique (12 "lames" radiales tournant lentement, anneau d'iris, pupille avec un cœur qui respire en continu) avec un clignement périodique (paupières SVG qui se referment brièvement toutes les ~7.5s). La pupille se dilate/contracte et change de couleur selon l'état (cyan dilatée = écoute, violette contractée = réflexion, dorée pulsée en rythme = réponse), en réutilisant les anneaux `.core-inner-ring`/`.core-pulse` déjà en place comme texture d'iris. Respecte `prefers-reduced-motion`.
3. **Police** : "Michroma" remplacée par **"Iceland"** pour le wordmark `V.I.S.O.R` (taille et letter-spacing réajustés pour ce nouveau tracé plus fin). Les autres polices (Electrolize pour l'état, JetBrains Mono pour les données, Rajdhani pour le texte) sont inchangées.
4. **Wake word "Hey Visor" / "Ok Visor"** : implémenté avec un bouton **HEY VISOR** dans la barre de télémétrie (désactivable à tout moment). Une seconde instance de `SpeechRecognition` tourne en continu quand le mode est armé, uniquement pendant que Visor est au repos (elle se coupe automatiquement pendant l'écoute manuelle, la réflexion et la synthèse vocale, pour éviter que Visor s'entende elle-même). Détection tolérante ("hey/ok/eh/salut" + "visor"), redémarrage automatique après les coupures silencieuses de Chrome mobile. À la première activation, un message dans la console prévient explicitement des limites (batterie, fiabilité) — testé : si la permission micro est refusée, le mode se désarme proprement avec un message clair plutôt que de planter.

**Non testé sur device réel** : Claude Code a vérifié le rendu visuel et les transitions d'état (texte → API → erreur) dans un navigateur sandboxé sans accès micro. La reconnaissance vocale, la synthèse et le wake word en conditions réelles (Xiaomi/Chrome Android) restent à valider sur le téléphone.

## Historique de debug important (contexte utile pour Claude Code)
L'utilisateur est sur un téléphone **Xiaomi/MIUI**, navigateur **Chrome for Android**. Deux séries de bugs déjà rencontrées et résolues, à garder en tête si des problèmes similaires reviennent :

- **Micro (reconnaissance vocale) qui ne fonctionnait pas** : cause finale = permissions Android/MIUI en cascade (Chrome → Réglages Android → Autorisations → Microphone → "Pendant l'utilisation de l'app", PAS "Demander à chaque fois"). Résolu.
- **Synthèse vocale (TTS) muette** : cause finale = le téléphone n'avait **aucun moteur TTS Google installé**, seulement "Mi AI Speech Engine" (MIUI), incompatible avec l'API Web Speech de Chrome. Résolu en installant l'app "Speech Recognition & Synthesis from Google" (Play Store) et en la sélectionnant comme moteur préféré dans Réglages → Langues et saisie → Synthèse vocale, puis en téléchargeant la voix française.
- Un fichier `test-tts.html` existe dans le dossier de sortie — c'est l'outil de diagnostic utilisé pour isoler ce bug TTS (bouton "lister les voix disponibles" + bouton "parler"). Peut être réutilisé si un nouveau problème audio survient.
- **Chrome garde du cache tenace sur les artefacts publiés** : à chaque changement, il a fallu renommer le fichier (nouvelle URL de publication) pour forcer le rechargement — Claude Code devra probablement faire pareil, ou utiliser un versioning de cache-busting plus robuste (ex. query string `?v=timestamp` sur l'URL publiée, si l'outil de publication le permet).

## Prompt système actuel de l'IA
```
Tu es Visor (Vision, Intelligence, Santé, Organisation, Raisonnement), un assistant vocal.
Réponds de façon concise et naturelle à l'oral, en français, sans markdown ni listes à puces.
Tu peux aussi faire des liens entre plusieurs domaines de la vie de l'utilisateur
(travail/cognition, santé et relations, projets futurs) quand c'est pertinent pour donner
une réponse plus complète et un raisonnement global, pas cloisonné par thème.
```

## Pistes d'évolution mentionnées par l'utilisateur mais pas encore traitées
- Rendre les 4 zones thématiques **vraiment fonctionnelles** (filtrage/recherche réel dans l'historique par thème), pas juste décoratives — la structure de données (`zone` par message) est déjà prête pour ça.
- Éventuellement connecter à la smart home (Google Home/Home Assistant) — sujet discuté en amont dans la conversation mais mis de côté au profit du travail sur l'interface.

## Prochaine étape
Les 4 chantiers ci-dessus sont faits côté code — la priorité pour la prochaine session est de **tester sur le téléphone réel** (Xiaomi/Chrome Android) : rendu de l'œil animé, lisibilité de la police Iceland à la taille réelle, et surtout fiabilité du wake word "Hey Visor" en conditions réelles (bruit ambiant, redémarrage après silence, non-interférence avec la synthèse vocale).
