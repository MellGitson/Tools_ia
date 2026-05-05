# Phase 6 : Options d'Embeddings

## Status ✅ TESTÉ & FONCTIONNEL

### Résultats des tests

**Test avec multi-provider :**
```
✅ Mistral réussi - 1024 dimensions
✅ Embedding généré: [-0.0236, 0.0469, 0.0311, 0.0319, 0.0420...]
✅ Upsert Pinecone: 1 vecteur stocké
```

## Providers disponibles

### 1️⃣ **Mistral (⭐ ACTUELLEMENT UTILISÉ)**
- **Status**: ✅ Fonctionnel et testé
- **Modèle**: `mistral-embed`
- **Dimensions**: 1024
- **Coût**: Gratuit (tier Mistral)
- **Configuration**: `MISTRAL_API_KEY`
- **Endpoint**: `https://api.mistral.ai/v1/embeddings`

```bash
# Tester
npm run embedding:multi
```

### 2️⃣ **HuggingFace (Alternative gratuite)**
- **Status**: ✅ Implémenté et prêt
- **Modèle**: `sentence-transformers/all-MiniLM-L6-v2`
- **Dimensions**: 384 (plus léger)
- **Coût**: Gratuit
- **Configuration**: `HUGGINGFACE_TOKEN` ✓ (déjà dans .env)
- **Endpoint**: `https://api-inference.huggingface.co/models/...`
- **Avantage**: Peut gérer beaucoup plus de requêtes

### 3️⃣ **Jina AI (Alternative ultra-rapide)**
- **Status**: ✅ Implémenté et prêt
- **Modèle**: `jina-embeddings-v2-base-en`
- **Dimensions**: 768
- **Coût**: ✅ Gratuit (jusqu'à 1000 req/mois)
- **Configuration**: Aucune (API publique)
- **Endpoint**: `https://api.jina.ai/v1/embeddings`
- **Avantage**: Pas de limite de taux pour usage modéré

### 4️⃣ **OpenAI (Alternative premium)**
- **Status**: ✅ Implémenté et prêt
- **Modèle**: `text-embedding-3-small`
- **Dimensions**: 1536
- **Coût**: $0.02 per 1M tokens
- **Configuration**: `OPENAI_API_KEY` (à ajouter)
- **Endpoint**: `https://api.openai.com/v1/embeddings`
- **Avantage**: Meilleure qualité sémantique

### 5️⃣ **Ollama (Alternative locale)**
- **Status**: ✅ Possible (nécessite setup)
- **Modèle**: `nomic-embed-text` (ou autre)
- **Dimensions**: Varie (nomic=768)
- **Coût**: Gratuit (local)
- **Configuration**: `OLLAMA_BASE_URL`
- **Endpoint**: `http://localhost:11434/api/embeddings`
- **Avantage**: Aucune limite, exécution locale
- **Installation**: 
  ```bash
  brew install ollama
  ollama pull nomic-embed-text
  ollama serve  # Lancer en background
  ```

## Stratégie de Fallback (Mode `auto`)

L'implémentation utilise un fallback automatique :

```
Tentative 1: Mistral (1024d) → Si OK, retourne
             ↓ Si erreur/timeout
Tentative 2: HuggingFace (384d) → Si OK, retourne
             ↓ Si erreur
Tentative 3: Jina (768d) → Si OK, retourne
             ↓ Si erreur
Erreur finale → Lance exception
```

## Utilisation

### Mode Auto (Fallback automatique)
```bash
npm run embedding:multi
```

Le système essaie Mistral d'abord, puis bascule sur HuggingFace, puis Jina si nécessaire.

### Mode Spécifique (Force un provider)
```javascript
// Utiliser Mistral uniquement
await getEmbedding(text, 'mistral');

// Utiliser HuggingFace uniquement
await getEmbedding(text, 'huggingface');

// Utiliser Jina uniquement
await getEmbedding(text, 'jina');

// Utiliser OpenAI (si configuré)
await getEmbedding(text, 'openai');
```

## Comparaison des providers

| Provider | Dimensions | Coût | Limite | Qualité | Vitesse |
|----------|-----------|------|--------|---------|---------|
| Mistral | 1024 | Gratuit | Moyen | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| HuggingFace | 384 | Gratuit | Illimitée | ⭐⭐⭐⭐ | ⭐⭐ |
| Jina | 768 | Gratuit | 1000/mois | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| OpenAI | 1536 | $0.02/1M | Illimitée | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Ollama (local) | 768 | Gratuit | Illimitée | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

## Configuration requise

### Pour tous les providers
```env
MISTRAL_API_KEY=...
HUGGINGFACE_TOKEN=...
PINECONE_API_KEY=...
PINECONE_INDEX_HOST=...
```

### Pour OpenAI (optionnel)
```env
OPENAI_API_KEY=...
```

### Pour Ollama (optionnel)
```env
OLLAMA_BASE_URL=http://localhost:11434
```

## Recommandations

1. **Production rapide** : Utiliser Jina (gratuit, illimité, rapide)
2. **Meilleure qualité** : Utiliser OpenAI (le plus cher mais best quality)
3. **Équilibre** : Utiliser Mistral (gratuit, 1024d, bon)
4. **Sans internet** : Utiliser Ollama (local)
5. **Économique** : Utiliser HuggingFace (384d est léger)

## Exemple d'usage avec fallback

```javascript
import { getEmbedding, upsertChunks } from './embedding-multi-provider.js';

const chunks = ['Chunk 1...', 'Chunk 2...'];

try {
  // Mode auto : essaie Mistral → HF → Jina
  await upsertChunks(chunks, 'auto');
  console.log('✅ Upsert réussi');
} catch (error) {
  console.error('❌ Tous les providers ont échoué');
}
```

## Test Démonstration (sans limites)
```bash
npm run embedding:test
```
Utilise un mock embedding pour validation sans appels API.
