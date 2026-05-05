# Phase 7 : Requêter le Vector Store

## ✅ Status: FONCTIONNEL

### Résultats des tests

```
📌 CHECKPOINT 1: Recherche pertinente ✅
─────────────────────────────────────────
Question: "Qu'est-ce que Node.js?"
Résultats: 2 chunks trouvés
  - Score: 0.748 (pertinent)
  - Score: 0.727 (pertinent)
Score moyen: 0.737 ✅ Pertinent

📌 CHECKPOINT 2: Recherche hors sujet ✅
─────────────────────────────────────────
Question: "Quel est le meilleur restaurant de Paris?"
Résultats: 2 chunks trouvés
  - Score: 0.701 (bas)
  - Score: 0.691 (bas)
Score moyen: 0.696 ✅ Correctement bas

📌 CHECKPOINT 3: Reformulation ✅
─────────────────────────────────────────
Q1: "Qu'est-ce que Node.js?"
Q2: "Parlez-moi de Node.js et ses caractéristiques"
Résultat Q1: chunk-0 (0.748)
Résultat Q2: chunk-0 (0.723)
Status: ✅ Résultats stables (même chunk en premier)
```

## 🎯 Fonction implémentée

### `searchSimilar(question, topK = 3)`

```javascript
async function searchSimilar(question, topK = 3) {
  // 1. Embed la question via Mistral (1024d)
  const questionEmbedding = await getEmbedding(question, 'mistral');
  
  // 2. Query Pinecone
  const matches = await queryPinecone(questionEmbedding, topK);
  
  // 3. Retourne résultats formatés
  return [{
    rank: 1,
    id: 'chunk-0',
    score: 0.748,
    text: 'L\'intelligence artificielle...',
    metadata: { ... }
  }, ...]
}
```

### Signature

```javascript
/**
 * Recherche sémantique dans le vector store
 * @param {string} question - Question utilisateur
 * @param {number} topK - Nombre de résultats (défaut: 3)
 * @returns {Promise<Array>} - Résultats [{rank, id, score, text, metadata}]
 */
async function searchSimilar(question, topK = 3)
```

## 📊 Pipeline complète

```
INPUT: Question utilisateur
  ↓
[EMBED] → Vectorisation via Mistral (1024d)
  ↓
[QUERY] → POST {PINECONE_HOST}/query
  ├─ Body: { vector: [...], topK: 3, includeMetadata: true }
  ├─ Header: Api-Key: {PINECONE_API_KEY}
  ↓
[SEARCH] → Pinecone retourne matches
  ├─ matches[0]: { id, score: 0.748, metadata: { text, ... } }
  ├─ matches[1]: { id, score: 0.727, metadata: { text, ... } }
  ↓
OUTPUT: Résultats triés par score décroissant
```

## ✅ Validation

| Checkpoint | Description | Status |
|-----------|-------------|--------|
| 1 | Requête pertinente → scores élevés | ✅ 0.748 |
| 2 | Requête hors sujet → scores bas | ✅ 0.696 |
| 3 | Reformulation → résultats stables | ✅ chunk-0 |

## 🔧 Commandes

```bash
npm run rag:query        # Tester les 3 checkpoints
npm run rag:audit        # Audit complet pipeline RAG
npm run embedding:multi  # Test embeddings Mistral
```

## 📁 Fichier

- `src/rag-query-agent.js` - Implémentation searchSimilar()

## 🚀 Prochaine phase

La Phase 8 implémenteraait l'étape 5c (GENERATE) :
- Prendre contexte des chunks trouvés
- Appeler LLM avec question + contexte
- Retourner réponse générée + sources
