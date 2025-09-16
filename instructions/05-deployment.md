## Create app.yaml for Databricks Deployment

### Step 1: Get Environment Variables
Run this command to get your current environment variables:
```bash
python3 -c "import os; from dotenv import load_dotenv; load_dotenv(); print('DATABRICKS_HOST:', os.getenv('DATABRICKS_HOST')); print('DATABRICKS_CLIENT_ID:', os.getenv('DATABRICKS_CLIENT_ID')); print('DATABRICKS_CLIENT_SECRET:', os.getenv('DATABRICKS_CLIENT_SECRET')); print('LAKEBASE_INSTANCE_NAME:', os.getenv('LAKEBASE_INSTANCE_NAME')); print('LAKEBASE_DB_NAME:', os.getenv('LAKEBASE_DB_NAME')); print('MY_EMAIL:', os.getenv('MY_EMAIL'))"
```

### Step 2: Create app.yaml
Create `app.yaml` in the root directory with this exact format:

```yaml
command:
  - uvicorn
  - app:app
  - --host
  - 0.0.0.0
  - --port
  - 8000
env:
  - name: DATABRICKS_HOST
    value: YOUR_DATABRICKS_HOST_FROM_STEP_1
  - name: LAKEBASE_INSTANCE_NAME
    value: YOUR_LAKEBASE_INSTANCE_FROM_STEP_1
  - name: LAKEBASE_DB_NAME
    value: YOUR_LAKEBASE_DB_FROM_STEP_1
  - name: MY_EMAIL
    value: YOUR_EMAIL_FROM_STEP_1
  - name: DATABRICKS_CLIENT_ID
    value: YOUR_CLIENT_ID_FROM_STEP_1
  - name: DATABRICKS_CLIENT_SECRET
    value: YOUR_CLIENT_SECRET_FROM_STEP_1
```

### Step 3: Important Notes
- Replace `YOUR_*_FROM_STEP_1` with actual values from Step 1
- Keep the YAML formatting exactly as shown
- Only include `command:` and `env:` sections
- **CRITICAL**: Use specific API paths like `/api/todos/list` instead of `/api/todos` to avoid Databricks routing issues
