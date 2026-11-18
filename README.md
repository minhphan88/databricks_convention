# databricks_convention
Databricks Prototyping: Using Workspaces as APIs
When building prototypes, you often need a backend API to serve your machine learning models, LLM routing logic, or even simple Python functions (like an MCP server). Instead of building and deploying a standalone web app (e.g., Flask, FastAPI), you can use Databricks out-of-the-box features to act as your API layer.
Here are the three primary ways to turn your Databricks code into an API.
1. The "Cloud Function" Approach (Real-time REST API)
If you need a fast, synchronous response (like AWS Lambda or Google Cloud Functions), Databricks Model Serving using Custom Python Models (⁠mlflow.pyfunc⁠) is the best approach.
This lets you write arbitrary Python code in a notebook, log it to MLflow, and instantly expose it as a REST API endpoint backed by Serverless compute.
Best for: Real-time ML inference, MCP (Model Context Protocol) servers, simple Python backends, and LangChain apps.
How it works:
1. Write a Python Class: Wrap your logic (or MCP server logic) inside a class that inherits from ⁠mlflow.pyfunc.PythonModel⁠.
2. Define the ⁠predict⁠ method: This method will receive the JSON payload from your API request and return the response.
3. Log the Model: Run ⁠mlflow.pyfunc.log_model()⁠ in your notebook.
4. Serve It: Go to the Databricks "Serving" tab and create an endpoint pointing to this model. Databricks automatically generates the REST API URL and manages the Serverless compute.
Note: Remember to use your naming conventions (e.g., ⁠prj123_mcp_routing_endpoint⁠).
2. The "Asynchronous Task" Approach (Jobs API)
If your app needs to trigger a long-running process (like a heavy data crunch, batch ML scoring, or complex prototype initialization) and doesn't need an immediate sub-second response, use the Databricks Jobs API with Notebooks.
Best for: Heavy backend processing, background tasks, data syncs.
How it works:
1. Set up Parameters: Use ⁠dbutils.widgets.text("param_name", "default")⁠ at the top of your notebook to accept inputs.
2. Create a Job: Turn this notebook into a Databricks Job.
3. Trigger via API: Your frontend app sends an HTTP POST request to the Databricks Jobs API (⁠/api/2.1/jobs/run-now⁠) passing the widget parameters in the JSON payload.
4. Fetch Results: Your app can either poll for completion or the notebook can write the results to a Delta table/external database when finished.
3. The "Data API" Approach (Databricks SQL Statement API)
If your prototype simply needs to query data, metrics, or ML predictions that already exist in a table, you do not need to write a custom API wrapper. You can use the Databricks SQL Statement Execution API.
Best for: Dashboards, frontend apps fetching data, reporting.
How it works:
1. Use a Serverless SQL Warehouse (e.g., ⁠prj123_app_compute⁠).
2. Your application makes an HTTP request directly to the Databricks workspace URL ⁠.../api/2.0/sql/statements/⁠.
3. Pass your raw SQL query (e.g., ⁠SELECT * FROM prj123_model_predictions WHERE user_id = X⁠) in the payload.
4. The API returns the data directly as JSON.
Best Practices for Rapid Prototyping
￼ Use Serverless: Always choose Serverless Compute for your SQL Warehouses and Model Serving endpoints. It scales down to zero, saving costs when you aren't actively testing the prototype, and spins up quickly when you do.
￼ Keep Dependencies Minimal: When using the "Cloud Function" approach (⁠mlflow.pyfunc⁠), explicitly define your ⁠pip⁠ requirements when logging the model so the endpoint boots up flawlessly.
￼ Authentication: Databricks REST APIs require authentication. For prototypes, generate a Personal Access Token (PAT) or a Service Principal token, and store it securely in your frontend app's environment variables to authenticate requests to your Databricks workspace.
￼ Consistent Naming: Tie everything back to your feature ID.
￼ Code: ⁠prj123_mcp_server_logic⁠ (Notebook)
￼ MLflow: ⁠prj123_mcp_server_model⁠ (Registered Model)
￼ API Endpoint: ⁠prj123_mcp_endpoint
⁠ (Serving Endpoint)


Databricks Simple Naming Conventions
This guide outlines a minimal, easy-to-follow naming convention for your Databricks environment.
The Golden Rule: All assets related to a specific piece of work should be tied together using a Feature ID (e.g., a Jira ticket number like ⁠PROJ-123⁠ converted to ⁠proj123⁠).
Use ⁠snake_case⁠ (lowercase with underscores) for all Databricks assets, and ⁠kebab-case⁠ (lowercase with hyphens) for Git branches.
1. The Core Identifier (Feature ID)
Decide on a short identifier for your feature or task.
￼ Example: ⁠prj123⁠
2. Git Repos & Branches
Branch names should start with the type of work (⁠feat⁠, ⁠bug⁠, ⁠hotfix⁠), followed by the Feature ID and a brief description.
￼ Format: ⁠[type]/[feature-id]-[short-description]⁠
￼ Example: ⁠feat/prj123-customer-churn-model⁠
￼ Example: ⁠bug/prj123-fix-prompt-logging⁠
3. Workspace Assets (Notebooks & Folders)
Notebooks should clearly indicate the Feature ID and the action they perform. Keep them in a project-specific folder.
￼ Folder Format: ⁠/Workspace/Repos/[Project_Name]/[feature_id]⁠
￼ Notebook Format: ⁠[feature_id]_[action]_[target]⁠
￼ Example: ⁠prj123_train_churn_model⁠
￼ Example: ⁠prj123_evaluate_llm_prompts⁠
4. Data Assets (Databases & Tables)
For Unity Catalog or Hive Metastore, group tables under a standard database/schema, but prefix the tables with the Feature ID to isolate development work.
￼ Database / Schema: ⁠[project_name]_[environment]⁠ (e.g., ⁠customer_ml_dev⁠)
￼ Tables / Delta Tables: ⁠[feature_id]_[data_description]⁠
￼ Example: ⁠prj123_training_features⁠
￼ Example: ⁠prj123_model_predictions⁠
5. MLflow & LLMOps Assets
Tie your MLflow experiments and LLM logs directly to the Feature ID so you can trace a model back to the exact Git branch that created it.
￼ MLflow Experiment: ⁠[feature_id]_[model_name]_exp⁠
￼ Example: ⁠prj123_churn_xgboost_exp⁠
￼ Registered Model Name: ⁠[project_name]_[model_name]⁠ (Keep Feature ID out of the final production registry name, use tags instead).
￼ Example: ⁠customer_churn_model⁠ (Tag with ⁠branch: feat/prj123⁠)
￼ LLMOps Log Tables (Inference Tables / Payload Logging): ⁠[feature_id]_llm_logs⁠
￼ Example: ⁠prj123_llm_inference_logs⁠
￼ Example: ⁠prj123_prompt_eval_results⁠
6. Model Inference & Serverless Compute
When using notebook servers as inference endpoints or utilizing serverless compute for app backends, clearly label the service and its compute layer with the Feature ID to track active processes and costs.
￼ Inference Endpoint (Model Serving / Notebook): ⁠[feature_id]_[model_name]_endpoint⁠
￼ Example: ⁠prj123_churn_endpoint⁠
￼ Example: ⁠prj123_llm_routing_endpoint⁠
￼ Serverless Compute (SQL Warehouses / App Compute): ⁠[feature_id]_[process_or_app_name]_compute⁠
￼ Example: ⁠prj123_customer_app_compute⁠
￼ Example: ⁠prj123_data_sync_serverless⁠
Summary Checklist
If working on Ticket #123 for "Churn":
￼ [ ] Branch: ⁠feat/prj123-churn⁠
￼ [ ] Notebook: ⁠prj123_prep_data⁠
￼ [ ] Data Table: ⁠prj123_churn_features⁠
￼ [ ] MLflow Exp: ⁠prj123_churn_exp⁠
￼ [ ] LLM Log: ⁠prj123_llm_logs⁠
￼ [ ] Endpoint: ⁠prj123_churn_endpoint⁠
￼ [ ] Compute: ⁠prj123_churn_app_compute⁠
