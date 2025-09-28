

<!DOCTYPE html>


<h1>Myntra Data Query Agent Documentation</h1>


<strong>Description:</strong> This document provides an overview of the Myntra Data Query Agent (MDQA), a custom mini agent for analyzing the Myntra e-commerce product dataset. It includes definitions of key concepts, code details, a flow diagram, and placeholders for screenshots of example queries.</p>

<p>This documentation is provided in HTML format for easy viewing. To convert to .doc:<br>
- Copy the content of this page.<br>
- Paste into Microsoft Word (or open this HTML file directly in Word).<br>
- Save as .doc or .docx.<br>
Alternatively, use an online HTML-to-DOC converter.</p>

<h2>Introduction to Key Concepts</h2>

<p>This section provides short, concise explanations of core concepts used in the agent.</p>

<ul>
    <li><strong>What is RAG (Retrieval-Augmented Generation)?</strong><br>
    RAG is a technique that combines information retrieval with generative AI (like LLMs). It "retrieves" relevant data from a source (e.g., a database or documents) to augment the prompt given to a language model, ensuring generated responses are accurate, grounded in real data, and less prone to hallucinations.</li>
    <li><strong>What is an Agent?</strong><br>
    An agent is an AI system that can autonomously perform tasks, make decisions, and interact with environments or users. It often uses tools (e.g., APIs, databases) in a loop: observe input, plan actions, execute, and refine based on results. Agents can be simple (rule-based) or complex (LLM-powered with reasoning).</li>
    <li><strong>What is a Custom Mini Agent?</strong><br>
    A lightweight, purpose-built agent without heavy frameworks (e.g., no LangChain). It focuses on specific tasks like data analysis, using a simple loop for interaction. In this case, it's tailored for SQL-based queries on structured data.</li>
    <li><strong>What is DuckDB?</strong><br>
    DuckDB is an in-memory analytical database engine optimized for fast querying of large datasets (e.g., CSVs) without loading everything into RAM. It's used here for efficient SQL operations on the ~1.4 GB Myntra dataset.</li>
    <li><strong>What is OpenAI API in This Context?</strong><br>
    The OpenAI API (e.g., GPT-3.5-turbo) powers the LLM for translating natural language to SQL and generating insights. It requires an API key for authentication.</li>
</ul>

<h2>Overview of the Myntra Data Query Agent (MDQA)</h2>

<p>The MDQA is a custom mini agent built in Google Colab for analyzing the Myntra e-commerce product dataset (from Kaggle: <a href="https://www.kaggle.com/datasets/promptcloud/myntra-e-commerce-product-data-november-2023">https://www.kaggle.com/datasets/promptcloud/myntra-e-commerce-product-data-november-2023</a>). It uses RAG principles:</p>

<ul>
    <li><strong>Retrieval</strong>: Generates and executes SQL queries on DuckDB to fetch data subsets.</li>
    <li><strong>Augmentation & Generation</strong>: Feeds retrieved data to an LLM for insightful natural language responses.</li>
</ul>

<p><strong>Key Features:</strong></p>
<ul>
    <li>Interactive loop for user queries (e.g., "What are the top 5 brands with the most products listed?").</li>
    <li>Handles large CSVs efficiently.</li>
    <li>Error handling for SQL issues (e.g., column mismatches).</li>
    <li>Based on dataset schema (e.g., columns like <code>Brand</code>, <code>Product_Id</code>, <code>Sale_Price</code>).</li>
</ul>

<p><strong>Assumptions:</strong></p>
<ul>
    <li>Dataset uploaded as CSV.</li>
    <li>OpenAI API key provided.</li>
    <li>Runs in Google Colab with dependencies: duckdb, pandas, openai.</li>
</ul>

<h2>Code Details</h2>

<p>The agent is implemented across 5 Colab cells. Below is a breakdown of each cell, including purpose and key code snippets.</p>

<h3>Cell 1: Install Dependencies</h3>
<p>Installs required Python packages.</p>
<pre><code>!pip install -q duckdb pandas openai</code></pre>

<h3>Cell 2: Upload Dataset CSV</h3>
<p>Allows direct upload of the CSV file.</p>
<pre><code>from google.colab import files
import os

print("Please upload the Myntra dataset CSV file (e.g., myntra_ecommerce_product_data_november_2023.csv):")
uploaded = files.upload()
CSV_FILE = list(uploaded.keys())[0]
print(f"Uploaded file: {CSV_FILE}")</code></pre>

<h3>Cell 3: Load Dataset into DuckDB and Get Schema</h3>
<p>Loads the CSV into DuckDB, prints schema and sample data, and initializes OpenAI client.</p>
<pre><code>import duckdb
import pandas as pd
from openai import OpenAI

con = duckdb.connect(':memory:')
con.execute(f"CREATE OR REPLACE TABLE products AS SELECT * FROM read_csv_auto('{CSV_FILE}', header=true, ignore_errors=true)")
schema = con.execute("DESCRIBE products").fetchdf().to_string(index=False)
print("Table Schema:\n", schema)
sample_data = con.execute("SELECT * FROM products LIMIT 5").fetchdf().to_string(index=False)
print("\nSample Data:\n", sample_data)
client = OpenAI(api_key="sk-your-actual-openai-api-key")  # Replace with your key</code></pre>

<h3>Cell 4: Define Custom Agent Functions</h3>
<p>Defines core functions for SQL generation, execution, and insights.</p>
<pre><code>def generate_sql(query: str, schema: str, sample_data: str) -> str:
    # LLM prompt to generate SQL (details omitted for brevity)
    # Uses OpenAI to create SQL based on query, schema, and sample.

def execute_sql(sql: str) -> tuple[pd.DataFrame, str]:
    # Executes SQL on DuckDB, returns DataFrame or error.

def generate_insights(query: str, data: str, error: str) -> str:
    # Uses LLM to analyze data or explain errors.</code></pre>

<h3>Cell 5: Run the Custom Mini Agent</h3>
<p>Interactive loop to process queries.</p>
<pre><code>def run_mini_agent():
    print("Mini Agent ready! Ask questions about the Myntra dataset (e.g., 'Top 5 brands by average price'). Type 'exit' to stop.")
    while True:
        user_query = input("\nYour query: ").strip()
        if user_query.lower() == 'exit':
            break
        # Generate SQL, execute, generate insights, and print.

run_mini_agent()</code></pre>

<p><strong>How It Works (High-Level):</strong></p>
<ul>
    <li>User inputs a natural language query.</li>
    <li>LLM generates SQL using schema/sample.</li>
    <li>DuckDB executes SQL to retrieve data.</li>
    <li>LLM generates insights from retrieved data.</li>
</ul>

<h2>Flow Diagram</h2>

<p>The flow diagram illustrates the agent's process. The following is Mermaid syntax (render using tools like mermaid.live).</p>

<pre><code>flowchart TD
    A[User Inputs Query] --> B[Generate SQL Function: LLM Translates Query to SQL using Schema/Sample]
    B --> C[Execute SQL Function: Run Query on DuckDB]
    C -->|Success: DataFrame| D[Generate Insights Function: LLM Analyzes Data & Generates Response]
    C -->|Error| D[Generate Insights: LLM Explains Error]
    D --> E[Output Insights to User]
    E --> A[Loop for Next Query]
    
    subgraph "RAG Components"
        B[Retrieval: SQL Generation]
        C[Retrieval: Data Fetch]
        D[Augmentation & Generation: Insights]
    end</code></pre>

<p><strong>Description of Flow:</strong></p>
<ul>
    <li><strong>Start</strong>: User query (e.g., "Top 5 brands by product count").</li>
    <li><strong>Retrieval Phase</strong>: SQL generated and executed.</li>
    <li><strong>Generation Phase</strong>: Insights produced.</li>
    <li><strong>Loop</strong>: Continues until 'exit'.</li>
</ul>



<h3>Example 1: "What are the top 5 brands with the most products listed?"</h3>
<p>- Expected SQL: <code>SELECT Brand, COUNT(Product_Id) AS total_products FROM products GROUP BY Brand ORDER BY total_products DESC LIMIT 5;</code><br>
- Sample Insights: Summarizes top brands (e.g., U.S. Polo Assn., Nike).</p>



<h3>Example 2: "What is the average Sale_Price for products in the 'Shirts' category?"</h3>
<p>- Expected SQL: Involves filtering on <code>Sub_Category_3 = 'Shirts'</code>.</p>


<h3>Example 3: "Which products have an Average_Rating above 4 and Discount_Percentage > 50?"</h3>
<p>- Expected SQL: Uses <code>WHERE</code> clauses on ratings and discounts.</p>




<p><strong>End of Documentation</strong></p>

</body>
</html>
