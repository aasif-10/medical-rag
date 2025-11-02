# Frontend-Backend Integration Guide

## Current Setup
- **Frontend Repo**: https://github.com/Sathishraj16/rag_model
- **Backend API**: https://medical-rag-m61y.onrender.com
- **Backend Endpoint**: `/query` (POST)

## Integration Steps

### 1. Create API Service File

Create `frontend/src/services/api.js`:

```javascript
// API configuration
const API_BASE_URL = process.env.REACT_APP_API_URL || 'https://medical-rag-m61y.onrender.com';

// Query API
export const queryAPI = {
  // Query documents
  search: async (query, top_k = 5) => {
    try {
      const response = await fetch(`${API_BASE_URL}/query`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          query: query,
          top_k: top_k
        })
      });

      if (!response.ok) {
        throw new Error(`API Error: ${response.status}`);
      }

      const data = await response.json();
      
      // Transform to frontend format
      return {
        answer: data.answer,
        sourceChunks: data.contexts.map((context, index) => ({
          id: index.toString(),
          documentName: extractDocumentName(context),
          content: context
        }))
      };
    } catch (error) {
      console.error('Query API Error:', error);
      throw error;
    }
  }
};

// Helper function to extract document name from context
function extractDocumentName(context) {
  // Context format: "[DocumentName, Page X] content..."
  const match = context.match(/^\[([^\]]+)\]/);
  return match ? match[1] : 'Unknown Document';
}

// Upload API (placeholder - implement when backend has upload endpoint)
export const uploadAPI = {
  uploadDocument: async (file) => {
    // TODO: Implement when backend has upload endpoint
    throw new Error('Upload endpoint not yet implemented in backend');
  },
  
  getDocuments: async () => {
    // TODO: Implement when backend has documents list endpoint
    return [];
  },
  
  reEmbedAll: async () => {
    // TODO: Implement when backend has re-embed endpoint
    throw new Error('Re-embed endpoint not yet implemented in backend');
  }
};
```

### 2. Update Query Component

Modify `frontend/src/pages/Query.jsx`:

```javascript
import React, { useState } from 'react';
import { Button } from '../components/ui/button';
import { Input } from '../components/ui/input';
import { Checkbox } from '../components/ui/checkbox';
import { Search, X, Loader2 } from 'lucide-react';
import { queryAPI } from '../services/api';  // Changed from mockAPI
import { toast } from '../hooks/use-toast';

const Query = () => {
  const [query, setQuery] = useState('');
  const [answer, setAnswer] = useState('');
  const [sourceChunks, setSourceChunks] = useState([]);
  const [showSources, setShowSources] = useState(false);
  const [isSearching, setIsSearching] = useState(false);
  
  const handleSearch = async () => {
    if (!query.trim()) {
      toast({
        title: 'Empty query',
        description: 'Please enter a question to search.',
        variant: 'destructive'
      });
      return;
    }
    
    setIsSearching(true);
    
    try {
      const response = await queryAPI.search(query, 5);  // Changed to real API
      setAnswer(response.answer);
      setSourceChunks(response.sourceChunks);
      setShowSources(true);
      
      toast({
        title: 'Search successful',
        description: `Found ${response.sourceChunks.length} relevant contexts.`
      });
    } catch (error) {
      toast({
        title: 'Search failed',
        description: error.message || 'Unable to process your query. Please try again.',
        variant: 'destructive'
      });
    } finally {
      setIsSearching(false);
    }
  };
  
  // ... rest of the component stays the same
};

export default Query;
```

### 3. Create Environment File

Create `frontend/.env`:

```env
# Production API
REACT_APP_API_URL=https://medical-rag-m61y.onrender.com

# For local development (when running backend locally)
# REACT_APP_API_URL=http://localhost:8000
```

### 4. Test the Integration

```bash
# In frontend directory
cd frontend
npm install
npm start

# Test queries:
# 1. "What causes hypertension?"
# 2. "Creatine is synthesized from:\n\nA. amino acids in the muscles.\nB. amino acids in the liver.\nC. amino acids in the kidneys.\nD. creatinine in the kidneys."
```

## API Endpoint Details

### POST /query

**Request:**
```json
{
  "query": "What causes hypertension?",
  "top_k": 5
}
```

**Response:**
```json
{
  "answer": "Hypertension is a risk factor for various health conditions...",
  "contexts": [
    "[EmergencyMedicine, Page 1099] Treatment of High Blood Pressure...",
    "[Cardiology, Page 928] Worldwide, hypertension causes..."
  ]
}
```

## Features

✅ **Working:**
- Medical Q&A queries
- MCQ questions with A/B/C/D options
- Context retrieval (up to 5 contexts)
- Cohere API with anti-hallucination safeguards
- 85 hardcoded contexts for common questions

⚠️ **Not Yet Implemented in Backend:**
- Document upload
- Document listing
- Re-embedding documents

## Next Steps

1. **Deploy frontend changes** to see live integration
2. **Add upload endpoint** to backend when needed
3. **Add document management** endpoints if required

## CORS Configuration

Your backend already has CORS enabled for all origins (`*`), so frontend requests will work from any domain.

## Production URLs

- **Frontend**: (Deploy to Vercel/Netlify/etc)
- **Backend**: https://medical-rag-m61y.onrender.com
