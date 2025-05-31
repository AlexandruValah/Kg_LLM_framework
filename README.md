# KG-LLM Fitness Framework - Setup Guide

This guide helps you set up and run the Knowledge Graph-LLM Fitness Framework on your machine.

## Prerequisites

1. **Neo4j Desktop 5.x** - [Download here](https://neo4j.com/download/)
2. **APOC Plugin 5.x** - Install via Neo4j Desktop (Manage → Plugins → Install APOC)
3. **Python 3.8+** with required packages
4. **OpenRouter API Key** - Get one from [OpenRouter](https://openrouter.ai/)

## Repository Structure

```
KG_LLM_FRAMEWORK/
├── Health_KG_LLM/
│   ├── resources/
│   │   ├── Exercises/
│   │   ├── Guidelines/
│   │   ├── HealthCondition_Contraindications/
│   │   ├── apoc.conf
│   │   ├── nodes.csv
│   │   └── rels.csv
│   └── results/
│       ├── Deepseek-Results.pdf
│       └── Gemini-Results.pdf
├── inject_exercises.ipynb
├── inject_guidelines.ipynb
├── inject_prescriptions.ipynb
├── KG_LLM_framework.ipynb
└── Kg_LLM_framework.iml
```

## Setup Instructions

### 1. Configure Neo4j

1. **Copy APOC configuration:**
   - In Neo4j Desktop: Click your database → Manage → Open Folder → Configuration 
   - Copy `Health_KG_LLM/resources/apoc.conf` to this folder
   - Restart the database

2. **Copy data files:**
   - In Neo4j Desktop: Click your database → Manage → Open Folder → Import
   - Copy `Health_KG_LLM/resources/nodes.csv` and `Health_KG_LLM/resources/rels.csv` to this folder

### 2. Load the Knowledge Graph

Open Neo4j Browser and run these queries in order:

```cypher
// 1. Import nodes
LOAD CSV WITH HEADERS FROM 'file:///nodes.csv' AS row
WITH
  apoc.convert.fromJsonList(row.labelList) AS labs,
  apoc.convert.fromJsonMap(row.props) AS props,
  row.id AS srcId
CALL {
  WITH labs, props, srcId
  CALL apoc.create.node(labs, props) YIELD node
  SET node.__srcId__ = srcId
  RETURN node
}
RETURN count(*) AS nodesCreated;

// 2. Import relationships
LOAD CSV WITH HEADERS FROM 'file:///rels.csv' AS row
WITH
  apoc.convert.fromJsonMap(row.props) AS rprops,
  row.type AS rtype,
  row.startId AS sid,
  row.endId AS eid
MATCH (a {__srcId__: sid}), (b {__srcId__: eid})
CALL {
  WITH a, b, rtype, rprops
  CALL apoc.create.relationship(a, rtype, rprops, b) YIELD rel
  RETURN rel
}
RETURN count(*) AS relsCreated;

// 3. Clean up temporary properties
MATCH (n) WHERE exists(n.__srcId__) REMOVE n.__srcId__;
```

### 3. Configure the Python Application

**Set up environment variables:**
Update the following values in the notebook:

```python
# Neo4j connection settings
NEO4J_URL = "bolt://localhost:7687"  # Your Neo4j URL
NEO4J_USER = "neo4j"                  # Your Neo4j username
NEO4J_PWD = "your-password"           # Your Neo4j password

# OpenRouter API key
openrouter_api_key = "your-openrouter-api-key"
```

**Model selection:**
You can switch between models by changing this line:
```python
model="google/gemini-2.0-flash-exp:free"  # or "tngtech/deepseek-r1t-chimera:free"
```

### 4. Run the Application

1. Open `KG_LLM_framework.ipynb` in Jupyter Notebook
2. Run all cells sequentially
3. The notebook will automatically:
   - Connect to Neo4j
   - Load the knowledge base
   - Initialize the LLM
   - Start accepting queries

### 5. Optional: Data Injection Notebooks

If you need to add more data to the knowledge graph:

- `inject_exercises.ipynb` - Add new exercises
- `inject_guidelines.ipynb` - Add new fitness guidelines
- `inject_prescriptions.ipynb` - Add new exercise prescriptions

## Troubleshooting

- **"File not found"**: Ensure CSV files are in the Neo4j import folder
- **APOC errors**: Verify APOC plugin is installed and apoc.conf is in place
- **Connection errors**: Check Neo4j is running and credentials are correct
- **API errors**: Verify your OpenRouter API key is valid

## Usage

Once set up, the system can process queries like:

- "What exercises are safe for someone with osteoarthritis?"
- "Create a workout plan for chronic low back pain"
- "What should I avoid with severe arthritis?"

The framework will use the knowledge graph to provide personalized, safe exercise recommendations.

## Results

All the results from the experiments can be found in:

- `Health_KG_LLM/results/Deepseek-Results.pdf`
- `Health_KG_LLM/results/Gemini-Results.pdf`
