# Vision-assistant

Vision-assistant is a small demo voice + vision assistant built on top of the LiveKit agents framework. It demonstrates how to combine real-time audio (STT/TTS), voice activity detection (VAD), an LLM backend, and a live video track to create an assistant that can speak, listen, and reference the current camera/video frames when answering user questions.

This repository contains a minimal example implementation in `assistant.py` that wires together:
- LiveKit RTC for real-time audio/video and chat
- A Speech-To-Text provider (Deepgram)
- A Voice Activity Detector (Silero)
- An LLM backend (OpenAI GPT family)
- Text-To-Speech (OpenAI TTS used via a StreamAdapter)
- A small function API for calling "vision" capabilities and returning the latest video frame to the LLM

Use this project as a reference for integrating vision with conversational voice assistants in real-time.

## Features
- Real-time voice interaction with VAD + STT
- LLM-driven responses and function-calling to request vision checks
- Optionally attach a latest video frame to the chat context so the assistant can "see"
- Demonstrates LiveKit agents patterns and VoiceAssistant helper

## Requirements
- Python 3.10+
- Access to a LiveKit room (server or hosted service)
- API credentials for the services you want to use (OpenAI, Deepgram, etc.)
- Recommended: virtual environment (venv)

Note: This project uses provider SDKs that may require separate installation and configuration. Replace providers with your preferred ones as needed.

## Quickstart

1. Clone the repository
   ```
   git clone https://github.com/Ayu369-gen/Vision-assistant.git
   cd Vision-assistant
   ```

2. Create and activate a virtual environment
   ```
   python -m venv .venv
   source .venv/bin/activate   # macOS / Linux
   .venv\Scripts\activate      # Windows (PowerShell)
   ```

3. Install dependencies
   If a requirements file exists:
   ```
   pip install -r requirements.txt
   ```
   Otherwise, install the libraries you need (example):
   ```
   pip install livekit-agents openai deepgram-sdk silero
   ```
   (Adjust package names to match the actual SDKs in your environment.)

4. Configure environment variables
   The assistant expects credentials and config for LiveKit and the providers. Typical environment variables:

   - LIVEKIT_URL — LiveKit server URL (wss/http)
   - LIVEKIT_API_KEY — LiveKit API key (if creating/joining rooms server-side)
   - LIVEKIT_API_SECRET — LiveKit API secret (if needed)
   - OPENAI_API_KEY — OpenAI API key (for LLM and TTS)
   - DEEPGRAM_API_KEY — Deepgram API key (STT)
   - Other provider-specific env vars as required by your SDKs

   Example (Linux/macOS):
   ```
   export LIVEKIT_URL="wss://your-livekit.example"
   export LIVEKIT_API_KEY="..."
   export LIVEKIT_API_SECRET="..."
   export OPENAI_API_KEY="..."
   export DEEPGRAM_API_KEY="..."
   ```

5. Run the assistant
   ```
   python assistant.py
   ```

   The assistant registers as a LiveKit client, joins a room, and begins listening for voice/chat activity. When a user sends chat messages or speaks (VAD detects voice), the assistant will transcribe, call the LLM, and respond using TTS. If the LLM triggers the "vision" function, the assistant will attach the most recent video frame captured from the room.

## How it works (file overview)
- assistant.py
  - entrypoint(ctx: JobContext): main entry function used by the LiveKit agents CLI.
  - ChatContext: initializes the system prompt used by the LLM.
  - openai.LLM: configured GPT model (example uses "gpt-4o" in the script).
  - tts.StreamAdapter + openai.TTS: used so TTS can be streamed into the voice assistant.
  - AssistantFunction: demonstrates a small ai_callable function named `image` that would be invoked by the LLM when a vision-related evaluation is requested.
  - VoiceAssistant: orchestrates VAD (Silero), STT (Deepgram), LLM, and TTS.
  - get_video_track(room): helper to find the first remote video track and expose the latest frame to chat messages.
  - ChatManager handlers: respond to chat messages and function call completions.

Key flow:
1. Assistant connects to a LiveKit room.
2. Listens for chat messages and speech.
3. When speech is detected, STT produces text; the LLM generates a response.
4. If the model calls the `image` function (vision request), the assistant re-sends the user message with the latest captured video frame attached so the model can reason about the image.

## Customization
- System prompt: change the ChatContext messages to tailor assistant persona and behavior.
- LLM model: swap `gpt-4o` for another model supported by your OpenAI client.
- TTS voice: change the voice name passed to `openai.TTS(voice="Hanu")`.
- Vision handling: implement the `AssistantFunction.image` method to store/forward/process images, run an image model, or call an external vision API. Currently it only prints the triggering message.

## Troubleshooting
- No video frames found: ensure your room has at least one participant publishing a video track. The helper looks for the first remote video track and stores frames continuously.
- STT/TTS not working: confirm provider keys, network connectivity, and that the provider SDK versions are compatible.
- LiveKit connection issues: verify LiveKit URL, API keys, and room permissions.

## Security & Privacy
- Video frames captured by the assistant may contain sensitive visual information. Only deploy this code in environments where you have user consent to capture and process video/audio.
- Keep API keys and secrets out of source control. Use environment variables or secret stores.

## Contributing
Contributions and improvements are welcome. If you'd like to add features (e.g., improved vision function, additional STT/TTS providers, or deployment scripts), please open an issue or submit a PR.

When opening an issue, include:
- A concise description of the problem or feature
- Steps to reproduce (if applicable)
- Any relevant logs / stack traces


## Acknowledgements
- LiveKit agents and rtc libraries for real-time infrastructure primitives
- Deepgram, Silero, and OpenAI used as example providers in the sample code
