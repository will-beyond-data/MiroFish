# MiroFish Local Setup (Zero API Cost)

## Prerequisites
- Gaming laptop with NVIDIA GPU (8GB+ VRAM)
- Docker Desktop installed
- Git

## Step 1: Install Ollama
```bash
# Windows: Download from https://ollama.com/download
# Or via winget:
winget install Ollama.Ollama
```

## Step 2: Pull Models
```bash
# Fast model for simulation agent actions (bulk of the calls)
ollama pull qwen2.5:7b

# Smarter model for ontology/profiles/reports (fewer calls, quality matters)
# Pick based on your VRAM:
#   8GB VRAM:  ollama pull qwen2.5:7b   (use same model for both)
#   12GB VRAM: ollama pull qwen2.5:14b
#   16GB+ VRAM: ollama pull qwen2.5:32b
ollama pull qwen2.5:14b
```

## Step 3: Start Neo4j (replaces Zep Cloud)
```bash
docker run -d \
  --name mirofish-neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/mirofish123 \
  -v neo4j_data:/data \
  neo4j:latest
```

## Step 4: Configure Environment
```bash
cp .env.example .env
```

Edit `.env`:
```env
# ===== LOCAL LLM (Ollama) =====
# Fast model for simulation agent actions
LLM_API_KEY=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL_NAME=qwen2.5:7b

# Smarter model for ontology/profiles/reports
LLM_BOOST_API_KEY=ollama
LLM_BOOST_BASE_URL=http://localhost:11434/v1
LLM_BOOST_MODEL_NAME=qwen2.5:14b

# ===== Zep Cloud (still needed for graph memory) =====
# Free tier: https://app.getzep.com/
# TODO: Replace with Neo4j once MiroFish-Offline integration is merged
ZEP_API_KEY=your_free_tier_key
```

## Step 5: Install & Run
```bash
npm run setup:all
npm run dev
```
Frontend: http://localhost:3000
Backend: http://localhost:5001

## Performance Expectations (Local)

| Scenario | API (~5 min) | Local 8B (~30-60 min) | Local 14B (~45-90 min) |
|----------|-------------|----------------------|----------------------|
| 10 agents, 20 rounds | $3-4 Haiku | Free | Free |
| 30 agents, 40 rounds | $14-18 Haiku | 2-4 hours | 3-6 hours |

## Hybrid Option (Best of Both)
Run simulation locally (free), use cheap API for report only:
```env
# Simulation: local
LLM_API_KEY=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL_NAME=qwen2.5:7b

# Reports: API (Haiku = ~$0.50 per report)
LLM_BOOST_API_KEY=your_anthropic_key
LLM_BOOST_BASE_URL=https://api.anthropic.com/v1
LLM_BOOST_MODEL_NAME=claude-3-5-haiku-20241022
```

## Known Limitations
- Zep Cloud is still required for graph memory (free tier works for testing)
- Full Neo4j replacement requires porting from MiroFish-Offline fork
- Local models produce lower quality agent personas than API models
- Simulation is sequential per GPU — patience required

## Reference Forks
- [MiroFish-Offline](https://github.com/nikmcfly/MiroFish-Offline) — Neo4j + Ollama, most complete local version
- [MiroShark](https://github.com/aaronjmars/MiroShark) — Another local adaptation

## What This Is
MiroFish is a multi-agent swarm intelligence engine. Feed it a document (news, policy, novel),
it builds a knowledge graph, generates thousands of AI agents with unique personalities, and
simulates their social media interactions. Inject variables ("Fed cuts rates") and watch the
digital world reorganise. Built by a 20-year-old, backed by $4.1M from Shanda Group.

Relevant for Beyond Data: demo-worthy AI capability for client conversations about what
multi-agent systems can do. The "digital twin of a market" concept.
