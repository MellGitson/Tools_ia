# Phase 6 : Embedder un document

## Status ✅ IMPLÉMENTÉ

### Fonctionnalités implémentées

#### 1. `getEmbedding(text)` 
Génère un embedding pour un texte via l'API Mistral (1024 dimensions)
- Endpoint: `POST https://api.mistral.ai/v1/embeddings`
- Model: `mistral-embed`
- Retourne: `number[]` (1024 dimensions)

#### 2. `simpleChunk(text, maxWords = 50)`
Découpe un texte en chunks de taille fixe
- Découpe par mots (défaut: 50 mots par chunk)
- Retourne: `string[]`
- Exemple: Un texte de 82 mots → 3 chunks (30, 30, 22 mots)

#### 3. `upsertChunks(chunks)`
Insère les chunks et leurs embeddings dans Pinecone
- Génère les embeddings via Mistral
- Crée des vecteurs avec structure: `{ id, values, metadata }`
- Upload dans Pinecone via: `POST {PINECONE_INDEX_HOST}/vectors/upsert`
- Retourne: `{ upsertedCount }`

### Tests disponibles

#### Test avec démonstration (recommandé)
```bash
npm run embedding:test
```
✅ Fonctionne sans limite d'API (utilise un mock embedding)

Résultat:
- ✓ Texte découpé en 3 chunks
- ✓ 3 embeddings générés (1024 dimensions)
- ✓ Structure JSON valide pour Pinecone
- ✓ Chaque vecteur inclut ID, values et métadonnées

#### Test complet (production)
```bash
npm run embedding
```
Utilise l'API Mistral réelle (limité par quota)

### Structure des données

#### Format d'un chunk
```javascript
"L'intelligence artificielle est un domaine en rapide évolution. Les modèles d'IA comme les transformers ont révolutionné..."
```

#### Format d'un vecteur Pinecone
```json
{
  "id": "chunk-0",
  "values": [0.2065, 0.2058, 0.2052, ...],  // 1024 dimensions
  "metadata": {
    "text": "L'intelligence artificielle...",
    "chunk_index": 0,
    "created_at": "2026-05-05T..."
  }
}
```

### Flux d'exécution

1. **Découpage** : Texte → N chunks
2. **Embedding** : Chaque chunk → Vecteur 1024d via Mistral
3. **Métadonnées** : Ajout du texte original et index
4. **Upsert** : Envoi à Pinecone via API REST

### Variables d'environnement requises

```
MISTRAL_API_KEY=...
PINECONE_API_KEY=...
PINECONE_INDEX_HOST=mini-perplexity-yd3wzck.svc.aped-4627-b74a.pinecone.io
```

### Notes importantes

- Chaque embedding coûte un appel API Mistral
- Les limites de quota peuvent être atteintes avec de gros documents
- Le mock test permet de valider la structure sans limites API
- Les métadonnées permettront les recherches sémantiques futures
