TEN Framework Quick Start Guide:

  Overview

  This guide helps you set up your development environment and run your first TEN app in ~5 minutes.

---

  System Requirements

| Requirement | Details                                      |
| ----------- | -------------------------------------------- |
| OS          | Linux (x64/arm64), macOS Intel/Apple Silicon |
| Python      | 3.10 only (strict requirement)               |
| Go          | 1.20+                                        |
| Node.js     | Any recent version with npm                  |

  Important: Python 3.10 is required - not 3.11 or 3.12. Use pyenv or venv to manage this.

---

  Step-by-Step Process

1. Install tman (TEN Manager)

  macOS:
  brew install TEN-framework/ten-framework/tman

  Linux:
  sudo add-apt-repository ppa:ten-framework/ten-framework
  sudo apt update && sudo apt install tman

  Or via script:
  bash <(curl -fsSL https://raw.githubusercontent.com/TEN-framework/ten-framework/main/tools/tman/install_tman.sh)

2. Create & Run Demo App

  tman install app transcriber_demo
  cd transcriber_demo
  tman install
  tman run install_deps
  tman run build

3. Configure Azure ASR

  Create .env file with:
  AZURE_STT_KEY=your_azure_speech_api_key
  AZURE_STT_REGION=eastus
  AZURE_STT_LANGUAGE=en-US

4. Run & Test

  tman run start

  Then open http://localhost:8080 to:

- Use microphone for real-time transcription
- Upload audio files
- View subtitles

---

  Demo App Architecture

  The transcriber_demo showcases multi-language extensions:

| Language   | Extension            | Purpose             |
| ---------- | -------------------- | ------------------- |
| Go         | web_audio_control_go | WebSocket server    |
| Python     | azure_asr_python     | Speech recognition  |
| TypeScript | vtt_nodejs           | Subtitle generation |

---

  Advanced: C++ Extensions

  For C++ development, install tgn:

  curl -fsSL https://raw.githubusercontent.com/TEN-framework/ten-framework/main/tools/tgn/install_tgn.sh | bash
  export PATH="/usr/local/ten_gn:$PATH"

  Then add C++ extensions like VAD:
  tman install extension webrtc_vad_cpp
  tman run build
  tman run start_with_vad

---

  Common Troubleshooting

| Problem                               | Solution                                                |
| ------------------------------------- | ------------------------------------------------------- |
| libpython3.10.dylib not found (macOS) | export DYLD_LIBRARY_PATH=/usr/local/opt/python@3.10/... |
| Port 8080 in use                      | lsof -i :8080 then kill -9`<PID>`                     |
| Go build failed                       | go clean -modcache then rebuild                         |
| tman not found                        | Add /usr/local/bin to PATH                              |

---

  Next Steps After Quick Start

1. Explore extensions: tman designer (visual graph editor)
2. Develop your own: Follow the https://theten.ai/docs/ten_framework/development/how_to_develop_with_ext


Access the app at: http://localhost:8001

  The app includes:

- Web Audio Control (Go) - serves the web interface on port 8001
- OpenAI ASR (Python) - transcribes audio using OpenAI Whisper
- Audio File Player (Python) - plays audio files for transcription
- VTT Generator (Node.js) - generates VTT subtitle files

  What you can do:

1. Open http://localhost:8001 in your browser
2. Use the web interface to:
   - Record audio from your microphone for real-time transcription
   - Play audio files for transcription
   - View transcription results

  Note: There was a warning about ffmpeg not being found. If you want audio file playback to work, install ffmpeg:
  brew install ffmpeg



Voice Agent is running!

  I've created a full voice conversational agent using only OpenAI services:

  Pipeline Configuration

- ASR (Speech-to-Text): OpenAI Whisper (whisper-1)
- LLM: OpenAI GPT-4o with 512 max tokens
- TTS (Text-to-Speech): OpenAI TTS (tts-1 with alloy voice)

  Access the Voice Agent

  Open in your browser: http://localhost:8001/static/microphone.html

  This will allow you to:

1. Click the microphone button to start speaking
2. Your speech is transcribed by OpenAI Whisper
3. GPT-4o processes and generates a response
4. OpenAI TTS converts the response to speech

  Project Location

  /Users/rupeshpanwar/Documents/AI-Projects/ai-pack/ten-framework/voice_agent/

  Key Files

- property.json - Pipeline configuration (ASR→LLM→TTS)
- manifest.json - Extension dependencies
- .env - OpenAI API key
- logs/voice_agent.log - Runtime logs

  To Restart

  cd /Users/rupeshpanwar/Documents/AI-Projects/ai-pack/ten-framework/voice_agent
  bin/start

  The agent greets you with: "Hello! I'm your AI assistant. How can I help you today?"

Yes, this transcriber demo is only for speech-to-text. But the TEN Framework has everything to build a full voice agent like ElevenLabs!

  Available Components for Voice Agent

  LLM (Language Models)

- openai_llm2_python - GPT-4, GPT-4o
- gemini_llm2_python - Google Gemini
- bedrock_llm_python - AWS Bedrock (Claude)
- And 8+ more providers

  TTS (Text-to-Speech)

- elevenlabs_tts2_python - ElevenLabs (premium voices)
- openai_tts2_python - OpenAI TTS
- azure_tts_python - Azure TTS
- playht_tts_python - PlayHT
- And 15+ more providers

  ASR (Speech-to-Text)

- deepgram_asr_python - Fast, real-time
- openai_asr_python - Whisper (we're using this)
- And 20+ more providers

  Pre-built Voice Agent Examples

  The framework includes 12+ complete voice assistant examples in /ai_agents/agents/examples/:

| Example                  | Description                                |
| ------------------------ | ------------------------------------------ |
| voice-assistant          | Basic voice agent (STT → LLM → TTS)      |
| voice-assistant-realtime | Ultra-low latency with OpenAI Realtime API |
| voice-assistant-live2d   | With animated avatar                       |
| voice-assistant-advanced | Full-featured agent                        |

  Want me to set up a full voice conversational agent?

  I can create a new app that:

1. Listens to your voice (ASR)
2. Sends to OpenAI GPT-4 for response (LLM)
3. Speaks the response back (TTS with ElevenLabs or OpenAI)

  Would you like me to set this up? You'll need to choose:

- TTS Provider: ElevenLabs (requires API key) or OpenAI TTS (uses your existing key)
