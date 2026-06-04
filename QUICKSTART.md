### QUICKSTART.md (Copy and Paste This)

```markdown
# Quick Start — Governor Layer

Get Governor Layer running locally in 5 minutes.

## Prerequisites

- Docker & Docker Compose
- Python 3.11+ (for local development)
- Node.js 18+ (for frontend)

## Local Setup (Docker)

```bash
# Clone the repo
git clone https://github.com/ceyptoslim/governor-layer.git
cd governor-layer

# Start the stack
docker-compose up

# Services:
# API Dashboard: http://localhost:3000
# API Server: http://localhost:8000
# PostgreSQL: localhost:5432
# Redis: localhost:6379
