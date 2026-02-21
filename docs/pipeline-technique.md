# Pipeline technique — Projet NOPLP

## 1. Objectif du pipeline
Le pipeline technique décrit l’ensemble des étapes permettant à l’utilisateur de réviser une chanson en chantant, puis d’obtenir un retour objectif basé sur une comparaison entre :
- ce qu’il a chanté (audio → texte)
- les paroles de référence (récupérées temporairement)
- un algorithme de similarité textuelle

Ce pipeline doit être fiable, performant, légalement sûr et facilement intégrable dans le système de révision de l’application.

## 2. Vue d’ensemble
Le pipeline se compose de cinq grandes étapes :
1. Enregistrement audio
2. Transcription audio → texte
3. Récupération temporaire des paroles de référence
4. Comparaison texte chanté / texte original
5. Calcul du score + feedback utilisateur

## 3. Étapes détaillées

### 3.1 Enregistrement audio
- L’utilisateur enregistre un extrait via le micro.
- Le fichier audio est envoyé au backend (WAV ou MP3).
- L’audio n’est pas stocké durablement : il est traité puis supprimé.

Entrée : audio brut  
Sortie : fichier audio prêt pour transcription

### 3.2 Transcription audio → texte
- Le backend envoie l’audio à un moteur de reconnaissance vocale (ex. Whisper).
- Le moteur retourne une transcription approximative.
- Le texte est normalisé (ponctuation, casse, espaces).

Entrée : audio  
Sortie : texte chanté

### 3.3 Récupération temporaire des paroles de référence
- Les paroles ne sont jamais stockées dans la base.
- Elles sont récupérées à la volée depuis une source légale.
- Elles sont utilisées uniquement pour la comparaison, puis supprimées.

Entrée : identifiant de la chanson  
Sortie : texte de référence (temporaire)

### 3.4 Comparaison texte chanté / texte référence
- Alignement des deux textes.
- Application d’un algorithme de similarité (Levenshtein, Jaro-Winkler…).
- Identification des différences :
  - mots manquants
  - mots incorrects
  - inversions
  - omissions

Entrée : texte chanté + texte de référence  
Sortie : score brut + liste des erreurs

### 3.5 Calcul du score et feedback
- Calcul du score final (ex. pourcentage de correspondance).
- Génération du feedback :
  - score global
  - erreurs principales
  - lignes correctes
  - suggestions de révision
- Intégration dans le système de révision espacée.

Entrée : score brut + erreurs  
Sortie : feedback utilisateur + données pour la révision

## 4. Contraintes techniques

### Performance
- Transcription rapide (< 2 secondes idéalement).
- Comparaison optimisée.

### Légalité
- Aucune conservation des paroles.
- Aucune redistribution.
- Usage strictement temporaire.

### Sécurité
- Suppression immédiate des fichiers audio.
- Pas de stockage des paroles.

### Compatibilité
- Pipeline indépendant du front.
- API REST ou GraphQL.

## 5. Risques et solutions

| Risque | Solution |
|-------|----------|
| Transcription imprécise | Normalisation + algorithme tolérant |
| Paroles introuvables | Fallback sur une autre source |
| Temps de traitement trop long | Whisper local + optimisation |
| Problèmes légaux | Zéro stockage, usage temporaire |

## 6. Intégration avec le système de révision
- Chaque tentative génère un score.
- Le score influence la date de prochaine révision (SRS).
- Les chansons difficiles remontent plus souvent.
- Les chansons maîtrisées s’espacent.