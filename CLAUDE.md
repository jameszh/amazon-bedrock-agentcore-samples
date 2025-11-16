# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains samples, tutorials, and use cases for **Amazon Bedrock AgentCore** - a framework-agnostic and model-agnostic service for deploying and operating AI agents securely at scale. The repository demonstrates how to use AgentCore components with various agentic frameworks (Strands Agents, LangGraph, CrewAI, LlamaIndex, etc.) and models.

## Repository Structure

```
01-tutorials/          # Component-specific tutorials (Runtime, Gateway, Memory, Identity, Tools, Observability)
02-use-cases/          # End-to-end production-ready applications
03-integrations/       # Framework integrations (agentic frameworks, observability, UX examples)
04-infrastructure-as-code/  # CDK and CloudFormation templates
```

## AgentCore Core Components

- **Runtime**: Serverless runtime for deploying and scaling agents
- **Gateway**: Converts APIs/Lambda functions to MCP-compatible tools
- **Memory**: Fully-managed memory infrastructure for personalized agent experiences
- **Identity**: Agent identity and access management (AWS + 3rd party apps)
- **Tools**: Built-in Code Interpreter and Browser tools
- **Observability**: OpenTelemetry-compatible tracing, debugging, and monitoring

## Development Commands

### Package Management

This repository uses **`uv`** as the primary package manager (NOT pip):

```bash
# Install dependencies
uv sync

# Install with dev dependencies
uv sync --dev

# Activate virtual environment
source .venv/bin/activate

# Run Python scripts
uv run python script.py

# Run commands with uv
uv run pytest
uv run streamlit run app.py
```

### Running Notebooks

Notebooks are the primary tutorial format (100+ in this repo):

```bash
# Setup
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m ipykernel install --user --name=notebook-venv --display-name="Python (notebook-venv)"

# Launch Jupyter
jupyter notebook path/to/notebook.ipynb
```

**Important**: Select the "Python (notebook-venv)" kernel in Jupyter after opening.

### AgentCore CLI (bedrock-agentcore-starter-toolkit)

The `agentcore` CLI is used for deploying agents:

```bash
# Configure agent deployment
agentcore configure --entrypoint main.py -er <iam-role-arn> --name <agent-name>

# Deploy to AWS
agentcore launch

# Test deployed agent
agentcore invoke '{"prompt": "your message"}'
```

**Important**: Before `agentcore launch`, delete `.agentcore.yaml` if it exists to avoid configuration conflicts.

### Testing (where applicable)

Some use cases include test suites using pytest:

```bash
# Run all tests
uv run pytest

# Run specific test
uv run pytest tests/unit/test_memory.py

# Run with verbose output
uv run pytest -v
```

### Quality Checks (SRE-agent example pattern)

Some projects include Makefiles with quality commands:

```bash
make quality      # Run all checks (format, lint, typecheck, security)
make format       # Format with black
make lint         # Lint with ruff
make lint-fix     # Auto-fix linting issues
make typecheck    # Type check with mypy
make security     # Security scan with bandit
make test         # Run pytest
make install-dev  # Install dev dependencies
make clean        # Clean build artifacts
```

## Architecture Patterns

### Agent Entry Points

Agents use the `BedrockAgentCoreApp` pattern with an `@app.entrypoint` decorator:

```python
from bedrock_agentcore.runtime import BedrockAgentCoreApp

app = BedrockAgentCoreApp()

@app.entrypoint
async def invoke(payload, context):
    user_message = payload["prompt"]
    session_id = context.session_id
    # Agent logic here
    return result

if __name__ == "__main__":
    app.run()
```

### Multi-Component Integration

Production use cases typically integrate multiple AgentCore components:

1. **Gateway**: Created via `scripts/agentcore_gateway.py` - exposes APIs as MCP tools
2. **Memory**: Created via `scripts/agentcore_memory.py` - stores conversation/user preferences
3. **Identity**: Credential providers (Cognito, Google OAuth) via scripts
4. **Runtime**: Deployed via `agentcore configure` + `agentcore launch`

### Configuration Management

Projects use **AWS Systems Manager (SSM) Parameter Store** for configuration:

- Store values: Gateway ARNs, Memory IDs, IAM roles, OAuth URLs
- Naming pattern: `/app/<project-name>/agentcore/<config-key>`
- Utility: `scripts/utils.py` typically has `get_ssm_parameter()` helper

### Testing Deployed Components

Use cases provide test scripts in `test/` or `scripts/tests/`:

```bash
# Test gateway integration
uv run python test/test_gateway.py --prompt "your test query"

# Test memory
uv run python test/test_memory.py load-conversation

# Test agent runtime
uv run python test/test_agent.py <agent-name> -p "Hi"
```

## Common Workflows

### Creating a New Use Case

1. Study similar use cases in `02-use-cases/`
2. Create project structure: `agent_config/`, `scripts/`, `test/`, `main.py`, `pyproject.toml`
3. Set up infrastructure scripts (gateway, memory, identity if needed)
4. Implement agent logic with `BedrockAgentCoreApp`
5. Configure and deploy with `agentcore` CLI
6. Test with provided test scripts

### Working with Tutorials

1. Navigate to component folder in `01-tutorials/`
2. Read component README for overview
3. Follow numbered notebooks sequentially
4. Use `01-tutorials/utils.py` for helper functions (Cognito setup, IAM role creation)

### Adding Framework Integrations

1. Reference `03-integrations/agentic-frameworks/` for patterns
2. Each framework (Strands, LangGraph, CrewAI) has different integration approaches
3. Key pattern: Wrap framework agent in `BedrockAgentCoreApp.entrypoint`

## Prerequisites for Development

- **Python**: 3.10+ (some projects require 3.12+)
- **AWS CLI**: Configured with credentials (`aws configure`)
- **IAM Permissions**: `BedrockAgentCoreFullAccess`, `AmazonBedrockFullAccess`
- **Model Access**: Anthropic Claude 4.0 enabled in Bedrock console
- **Docker/Finch**: For local agent testing (before deployment)
- **uv**: Install via `curl -LsSf https://astral.sh/uv/install.sh | sh`

## Important Notes

- **Naming Convention**: Many use cases require resource names to start with project prefix (e.g., `customersupport-*`)
- **Region**: Default is `us-east-1`; set `AWS_DEFAULT_REGION` to override
- **Cleanup**: Each use case provides cleanup scripts to delete AWS resources
- **SSM Parameters**: Use `scripts/list_ssm_parameters.sh` to view stored configuration
- **Streamlit Apps**: When running Streamlit UIs, use port 8501: `uv run streamlit run app.py --server.port 8501`

## MCP (Model Context Protocol)

Many examples use MCP for tool integration:

- MCP servers can be TypeScript or Python
- Gateway converts APIs to MCP-compatible format
- Toolkit packages: `strands-agents-tools`, `langchain-mcp-adapters`, `mcp>=1.9.0`

## Key Dependencies

Most projects depend on:
- `bedrock-agentcore` - Core AgentCore SDK
- `bedrock-agentcore-starter-toolkit` - Provides `agentcore` CLI
- `strands-agents` - Strands agent framework
- `langgraph`, `langchain`, `langchain-aws` - LangChain ecosystem
- `boto3` - AWS SDK for Python
- `mcp` - Model Context Protocol

## Infrastructure as Code

CDK and CloudFormation templates are in `04-infrastructure-as-code/`:
- CDK examples for TypeScript/Python deployments
- CloudFormation templates for declarative infrastructure
