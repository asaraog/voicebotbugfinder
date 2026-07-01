# Setup

Use these commands to run the full pipeline from the command line:

Python version: 3.10+

```bash
pip install fastapi uvicorn websockets twilio openai requests python-dotenv
cp .env.example .env          # fill in SERVER_DOMAIN after tunnel starts
npx localtunnel --port 8080   # Terminal 1
jupyter nbconvert --to notebook --execute --inplace pgai-voice-bot.ipynb   # Terminal 2 -- runs notebook + bug report
```

# Voice Bot -- Architecture

The notebook orchestration functions trigger outbound calls through Twilio. When connected, Twilio opens a Media Stream WebSocket, and the in-notebook bridge proxies bidirectionally to the Azure OpenAI Realtime API in a single low-latency (~300ms) session.

| Layer                         | Technology                                      |
| ----------------------------- | ----------------------------------------------- |
| **Telephony**           | Twilio (outbound PSTN + dual-channel recording) |
| **Live Audio Bridge**   | FastAPI + WebSockets                            |
| **Speech-to-Speech AI** | Azure OpenAI `gpt-realtime-mini`              |
| **Transcription**       | Azure AI Speech (hi-IN + en-US)                 |
| **Bug Analysis**        | Azure OpenAI GPT-5-mini                         |

# Scenarios and Helper functions

| ID           | Scenario                    | Key Test                                          |
| ------------ | --------------------------- | ------------------------------------------------- |
| 01           | New patient scheduling      | Collects name, DOB, reason                        |
| 02           | Reschedule appointment      | Reschedule + policy clarification                 |
| 03           | Medication refill           | Lisinopril + atorvastatin                         |
| 04           | Insurance inquiry           | BCBS PPO, Medicare, UHC                           |
| 05           | Sunday booking [KEY BUG]    | **CRITICAL -- agent should refuse weekend** |
| **06** | **Multiple requests** | **Multi-intent handling in single call**    |
| 07           | Urgent same-day             | Chest tightness triage                            |
| 08           | Barge-in / interruption     | Mid-sentence interruption recovery                |
| 09           | Wrong department            | Patient thinks they called pharmacy               |
| **10** | **Hindi only**        | **Title VI language access compliance**     |
