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

### Step 4: Databricks Apps Routing Requirements

**CRITICAL FOR DATABRICKS APPS**: The following changes are REQUIRED to make the app work in Databricks Apps:

#### Frontend Changes (frontend/index.html):
```javascript
// WRONG - Will cause ERR_CONNECTION_REFUSED in Databricks Apps:
fetch('http://localhost:8000/api/todos/')

// CORRECT - Use relative URLs:
fetch('/api/todos/list')
```

#### API Route Changes (routers/todos.py):
```python
# WRONG - Generic routes cause routing conflicts:
@router.get("/")
@router.post("/")
@router.put("/{todo_id}")
@router.delete("/{todo_id}")

# CORRECT - Specific routes for Databricks Apps:
@router.get("/list")
@router.post("/create")
@router.put("/{todo_id}/update")
@router.put("/{todo_id}/update-status")
@router.delete("/{todo_id}/delete")
```

#### Required API Endpoint Mappings:
- `GET /api/todos/list` - List todos (with optional `include_completed` parameter)
- `POST /api/todos/create` - Create new todo
- `PUT /api/todos/{id}/update` - Update todo content
- `PUT /api/todos/{id}/update-status` - Change todo status
- `DELETE /api/todos/{id}/delete` - Delete todo

#### Why These Changes Are Required:
1. **Relative URLs**: Databricks Apps don't use localhost - they use the app's domain
2. **Specific Paths**: Generic routes like `/api/todos/` conflict with Databricks routing
3. **No Trailing Slashes**: Avoid 307 redirects that can cause issues
4. **Explicit Actions**: Clear action names prevent routing ambiguity

#### Testing Checklist:
- [ ] All frontend API calls use relative URLs (no localhost)
- [ ] All API routes have specific action names
- [ ] No generic routes that could conflict with Databricks
- [ ] App works in both local development and Databricks Apps
