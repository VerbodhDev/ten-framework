# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

TEN Framework is an open-source framework for real-time multimodal conversational AI. The repository is organized into two main parts:

1. **Core Framework** (`core/`, `packages/`, `tools/`) - The C/C++/Rust runtime, language bindings, and build tooling
2. **AI Agents** (`ai_agents/`) - Python/Go/TypeScript extensions for building conversational AI agents

## High-Level Architecture

```
ten-framework/
├── core/                    # C/C++ runtime implementation
│   └── src/
│       ├── ten_runtime/     # Core message-passing runtime
│       ├── ten_manager/     # Package management (tman)
│       ├── ten_rust/        # Rust components
│       └── ten_utils/       # Shared utilities
├── packages/                # Extension templates and core packages
│   ├── core_extensions/     # Default extension templates (C++, Go, Python, Node.js)
│   ├── core_addon_loaders/  # Language addon loaders
│   ├── core_apps/           # Core application templates
│   └── example_extensions/  # Example extension implementations
├── ai_agents/               # AI agent implementations (see ai_agents/CLAUDE.md)
│   ├── agents/              # Agent examples and extensions
│   ├── server/              # Go API server
│   └── playground/          # Next.js frontend
├── tools/                   # Build and development tools
│   ├── tgn/                 # TEN GN build system
│   └── tman/                # TEN Manager install scripts
└── tests/                   # Core framework tests
```

## Build System

TEN uses a custom build system called `tgn` (based on Google's GN) for C/C++ code.

### Core Framework Build Commands

```bash
# Generate build configuration
tgn gen linux x64 debug   # or: darwin arm64 release

# Build everything
tgn build linux x64 debug

# Build specific targets
tgn build:"ten_manager_package tests/ten_manager" linux x64 debug

# Clean build
rm -rf out
```

### Using Taskfile

```bash
# Root-level tasks
task gen                  # Generate build config
task build                # Build framework
task clean                # Clean build artifacts
task gen-tman             # Generate tman for local development
task build-tman           # Build tman only

# AI Agents tasks (prefixed with ai_agents:)
task ai_agents:lint       # Lint Python extensions
task ai_agents:test       # Run all tests
task ai_agents:format     # Format code
```

## Extension Development

TEN extensions can be written in C++, Go, Python, or Node.js. Each language has a template.

### Creating New Extensions

```bash
# Create extension from template
tman create extension my_ext_name --template default_extension_cpp
tman create extension my_ext_name --template default_extension_go --template-data class_name_prefix=MyName
tman create extension my_ext_name --template default_async_extension_python --template-data class_name_prefix=MyName
tman create extension my_ext_name --template default_extension_nodejs --template-data class_name_prefix=MyName

# Install dependencies
tman install --standalone

# Build (for compiled languages)
tman run build

# Run tests
tman run test
```

### Extension Lifecycle

All extensions implement these lifecycle methods:
- `on_init` / `on_configure` - Initialization and configuration
- `on_start` - Start processing
- `on_cmd` - Handle command messages
- `on_data` - Handle data messages
- `on_audio_frame` / `on_video_frame` - Handle media streams
- `on_stop` / `on_deinit` - Cleanup

### Graph-Based Configuration

Extensions are connected via `property.json` using predefined graphs:

```json
{
  "ten": {
    "predefined_graphs": [{
      "name": "my_agent",
      "auto_start": true,
      "graph": {
        "nodes": [
          {"name": "stt", "addon": "my_asr_python", "property": {...}},
          {"name": "llm", "addon": "my_llm_python", "property": {...}}
        ],
        "connections": [
          {"extension": "stt", "cmd": [...], "data": [...], "audio_frame": [...]}
        ]
      }
    }]
  }
}
```

## Testing

### Core Framework Tests

```bash
# Build and run core tests
tgn gen linux x64 debug -- ten_enable_standalone_test=true
tgn build linux x64 debug
./out/linux/x64/tests/<test_binary>
```

### AI Agent Extension Tests

```bash
# Test all extensions
task ai_agents:test

# Test specific extension
task ai_agents:test-extension EXTENSION=agents/ten_packages/extension/elevenlabs_tts_python

# Test without reinstalling dependencies
task ai_agents:test-extension-no-install EXTENSION=agents/ten_packages/extension/elevenlabs_tts_python
```

Extension tests use a standalone test harness:
- Entry point: `tests/bin/start` (shell script)
- Sets PYTHONPATH to include ten_runtime and ten_ai_base
- Uses pytest for Python extensions

## Key Tools

### tman (TEN Manager)

Package manager for TEN:
- `tman install` - Install dependencies from manifest.json
- `tman install --standalone` - Install for standalone extension development
- `tman run <task>` - Run tasks defined in manifest.json
- `tman designer` - Launch visual graph designer (TMAN Designer)
- `tman create extension <name> --template <template>` - Create new extension

### tgn (TEN GN)

C/C++ build system:
- `tgn gen <os> <arch> <build_type>` - Generate build files
- `tgn build <os> <arch> <build_type>` - Build project
- Supports: linux/darwin, x64/arm64, debug/release

## Environment Configuration

### PYTHONPATH for Python Extensions

```bash
export PYTHONPATH="./agents/ten_packages/system/ten_runtime_python/lib:./agents/ten_packages/system/ten_runtime_python/interface:./agents/ten_packages/system/ten_ai_base/interface"
```

### Required Environment Variables

Extension-specific API keys go in `.env`:
- `AGORA_APP_ID`, `AGORA_APP_CERTIFICATE` - RTC
- `OPENAI_API_KEY` - LLM
- `DEEPGRAM_API_KEY`, `AZURE_ASR_KEY` - ASR
- `ELEVENLABS_TTS_KEY`, `AZURE_TTS_KEY` - TTS

## Git-Ignored Files (Do Not Modify)

Auto-generated files that should not be manually edited:
- `manifest-lock.json` - Generated by tman
- `compile_commands.json` - Generated by build system
- `BUILD.gn` (in extensions) - Generated build config
- `out/`, `.ten/`, `build/` - Build outputs
- `bin/main`, `bin/worker` - Compiled binaries

## Development Guidelines

### Code Style

- **Python**: Black formatter, PEP 8, type hints
- **TypeScript**: ESLint, TypeScript strict mode
- **Go**: gofmt, standard Go conventions
- **C++**: clang-format (see `.clang-format`)

### Commit Messages

Use conventional commits:
```
feat: add new ASR integration
fix: resolve memory leak in audio processing
docs: update installation instructions
test: add integration tests for TTS module
refactor: improve error handling in server
```

### Configuration in property.json

Use `${env:VAR_NAME}` for environment variable substitution:
```json
{"api_key": "${env:OPENAI_API_KEY}"}
```

Use `${env:VAR_NAME|}` for optional variables with empty fallback.

## AI Agents Specific Guidance

For detailed guidance on AI agent development, see `ai_agents/CLAUDE.md` which covers:
- Extension base classes (AsyncASRBaseExtension, AsyncTTSBaseExtension, etc.)
- API interface definitions in `agents/ten_packages/system/ten_ai_base/api/`
- HTTP client patterns for TTS/ASR services
- Integration test frameworks (ASR Guarder, TTS Guarder)
- AudioFrame creation and handling patterns

## Quick Reference

| Task | Command |
|------|---------|
| Build core framework | `tgn build linux x64 debug` |
| Build AI agents | `cd ai_agents && task build` |
| Lint Python | `task ai_agents:lint` |
| Test extension | `task ai_agents:test-extension EXTENSION=path/to/ext` |
| Create extension | `tman create extension name --template template_name` |
| Start TMAN Designer | `tman designer` |
| Run agent example | `cd ai_agents/agents/examples/voice-assistant && task run` |
