# Knowledge Base MCP Server - API Documentation

## Overview

The Knowledge Base MCP Server provides two main tools for interacting with knowledge bases:

1. `list_knowledge_bases` - Lists available knowledge bases
2. `retrieve_knowledge` - Retrieves content from a specific knowledge base

## Tool Specifications

### 1. list_knowledge_bases

Lists all available knowledge bases on the system.

#### Parameters
- None required

#### Returns
```json
{
  "knowledge_bases": [
    {
      "name": "string",
      "path": "string", 
      "description": "string (optional)",
      "last_updated": "string (ISO timestamp)"
    }
  ]
}
```

#### Example Usage
```javascript
// List all knowledge bases
const result = await mcp.callTool("list_knowledge_bases");
console.log(result.knowledge_bases);
```

### 2. retrieve_knowledge

Retrieves relevant content from a specified knowledge base based on a query.

#### Parameters
- `kb_name` (string, required): Name of the knowledge base to search in
- `query` (string, required): Search query or question
- `limit` (integer, optional): Maximum number of results to return (default: 5)

#### Returns
```json
{
  "results": [
    {
      "file_path": "string",
      "content": "string",
      "relevance_score": "number",
      "metadata": {
        "word_count": "integer",
        "last_modified": "string (ISO timestamp)"
      }
    }
  ],
  "total_found": "integer",
  "search_time_ms": "number"
}
```

#### Example Usage
```javascript
// Retrieve knowledge from a specific knowledge base
const result = await mcp.callTool("retrieve_knowledge", {
  kb_name: "my-knowledge-base",
  query: "How to configure embeddings?",
  limit: 3
});

console.log(`Found ${result.total_found} relevant documents`);
result.results.forEach(doc => {
  console.log(`File: ${doc.file_path}`);
  console.log(`Content: ${doc.content}`);
});
```

## Error Handling

### Common Error Codes

| Code | Description | Resolution |
|------|-------------|------------|
| `KB_NOT_FOUND` | Knowledge base does not exist | Check `list_knowledge_bases` for valid names |
| `EMBEDDING_ERROR` | Failed to generate embeddings | Verify embedding provider configuration |
| `INDEX_CORRUPTED` | FAISS index is corrupted | Delete `.index` directory and retry |
| `PERMISSION_DENIED` | Cannot write to index directory | Check file permissions on FAISS_INDEX_PATH |

### Error Response Format
```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": "object (optional)"
  }
}
```

## Integration Examples

### Basic Integration
```javascript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

const transport = new StdioClientTransport({
  command: "node",
  args: ["build/index.js"],
  env: {
    KNOWLEDGE_BASES_ROOT_DIR: "/path/to/kbs",
    EMBEDDING_PROVIDER: "huggingface"
  }
});

const client = new Client({
  name: "example-client",
  version: "1.0.0"
}, {
  capabilities: {}
});

await client.connect(transport);

// List knowledge bases
const kbs = await client.listTools();
console.log(kbs.tools);

// Use a tool
const result = await client.callTool("retrieve_knowledge", {
  kb_name: "docs",
  query: "What is the setup process?"
});
```

## Configuration Reference

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `KNOWLEDGE_BASES_ROOT_DIR` | Yes | - | Root directory containing knowledge base folders |
| `FAISS_INDEX_PATH` | No | `.faiss_index` | Path to store FAISS index files |
| `EMBEDDING_PROVIDER` | No | `huggingface` | Embedding provider: `huggingface`, `ollama`, or `openai` |
| `HUGGINGFACE_MODEL_NAME` | Conditional | `BAAI/bge-small-en-v1.5` | HuggingFace model name |
| `OLLAMA_MODEL` | Conditional | `nomic-embed-text` | Ollama embedding model |
| `OPENAI_MODEL_NAME` | Conditional | `text-embedding-ada-002` | OpenAI embedding model |
| `LOG_FILE` | No | stderr only | Optional log file path |

## Best Practices

1. **Always call `list_knowledge_bases` first** to get valid knowledge base names
2. **Handle errors gracefully** - network issues with embedding providers are common
3. **Use appropriate limits** - very large queries may timeout
4. **Cache results when possible** - repeated queries on the same content can be expensive
5. **Monitor search times** - if searches take too long, consider preprocessing content

## Troubleshooting

### Slow Searches
- Check if you're using an efficient embedding model
- Consider preprocessing and re-indexing your knowledge base
- Monitor memory usage during searches

### Connection Issues
- Verify Node.js version (16+ required)
- Ensure all dependencies are installed (`npm install`)
- Check environment variables are set correctly

### Index Problems
- Delete the `.index` directory in your knowledge base to force re-indexing
- Ensure write permissions to the FAISS index location