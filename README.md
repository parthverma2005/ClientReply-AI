# ClientReply AI — runnable MVP

A local-first writing assistant for freelancers. Paste a client message, choose the scenario and tone, optionally add your goal/context, and receive an editable reply plus a short rationale and next step.

## Included
- Responsive browser UI with editable subject and reply.
- Scenarios: general, scope change, revisions, overdue payment, price negotiation, deadline changes, boundaries, new project inquiries.
- Tone choices: professional, friendly, firm, concise, diplomatic.
- Copy subject/reply/full draft and download a `.txt` file.
- FastAPI backend with request validation.
- Optional open-weight model inference via Ollama.
- Scenario-specific fallback templates when Ollama is unavailable.
- No database, accounts, analytics, or automatic sending.

## Run on Windows (PowerShell)

1. Extract this ZIP.
2. Open PowerShell in the extracted `clientreply-ai-mvp` folder.
3. Run:

   ```powershell
   py -m venv .venv
   .\.venv\Scripts\Activate.ps1
   pip install -r requirements.txt
   uvicorn app:app --reload
   ```

4. Visit http://127.0.0.1:8000

If activation is blocked for this terminal, run:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

## Enable open-weight local AI

1. Install Ollama: https://ollama.com/
2. In a terminal, download the model:

   ```powershell
   ollama pull qwen2.5:3b
   ```

3. Leave Ollama running. The backend sends prompts to `http://localhost:11434/api/generate`.

The default model tag is `qwen2.5:3b`. To change it:

```powershell
$env:OLLAMA_MODEL="your-model-tag"
uvicorn app:app --reload
```

To use a different Ollama host, set `OLLAMA_URL` before starting the server. If Ollama is unavailable or returns invalid output, the app uses its built-in templates.

## Quick smoke tests

Open `http://127.0.0.1:8000/api/health`, then try:

```powershell
$body = @{
  client_message = "Can you add three extra pages for the same price?"
  scenario = "scope"
  tone = "firm"
  goal = "keep the agreed price and clarify the extra cost"
  context = "The original project included five pages."
} | ConvertTo-Json

Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/api/draft -ContentType "application/json" -Body $body
```

## API
- `GET /api/health`
- `POST /api/draft`

Example JSON body:

```json
{
  "client_message": "Could you do this for less?",
  "scenario": "negotiation",
  "tone": "diplomatic",
  "goal": "preserve the value of the work",
  "context": "The current quote is for the full scope.",
  "currency": "INR (₹)"
}
```

## Safety, privacy, limitations
- The app only drafts; it does not send email or message clients.
- Verify all names, money, dates, deadlines, and contractual statements before sending.
- The fallback is template-based, not generative AI. The UI indicates which mode produced the result.
- No drafts are persisted by this app. The browser sends the message to the FastAPI backend; when Ollama is enabled, the backend sends it to the configured Ollama host. For local-only processing, run both services on your own computer.
- Do not deploy publicly without adding authentication, rate limiting, transport security, and a privacy review.
- The model can make mistakes. Treat it as a writing assistant, not legal or financial advice.
