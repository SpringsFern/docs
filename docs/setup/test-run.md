# Test Run the Application

Before setting up systemd or Nginx, verify that the application runs correctly.

This ensures that configuration and dependencies are working as expected.

---

## 1. Activate Virtual Environment

Inside the project directory:

```bash
source venv/bin/activate
```

Confirm that `(venv)` appears in your terminal.

---

## 2. Start the Application

Run the main application:

```bash
python -m tgfs
```

---

## 3. Verify Application Startup

If successful, you should see:

- No import errors
- No missing environment variable errors
- Server binding message (example):

```
Running on http://0.0.0.0:8080
```

---

## 4. Test Locally

From the same server:

```bash
curl http://localhost:8080
```

Or open in browser:

```
http://<server-ip>:8080
```

If running correctly, you should receive A Response

---

## 5. Common Startup Errors

### Missing Environment Variables

Error example:

```
KeyError: API_ID
```

Solution:
- Ensure `.env` file exists
- Verify all required variables are set

---

### Port Already in Use

Error example:

```
Address already in use
```

Check running processes:

```bash
sudo lsof -i :8080
```

Kill process if necessary:

```bash
sudo kill -9 <PID>
```

---

### Module Not Found

Error example:

```
ModuleNotFoundError
```

Solution:

```bash
pip install -r requirements.txt
```

---

## 6. Stop the Application

Press:

```
CTRL + C
```

The application should stop gracefully.

---

## Important

Only proceed to systemd setup after confirming:

- Application runs without errors
- Port binding works
- Local access works
