# Build 41 PWA prerequisite

Before the Build 41 native Siri/App Intents build ships, the PWA voice-command matcher must recognize the canonical commands `notes` and `study tour` through the existing `isBibleVoiceCommand` / normal command-handler path.

Do not bypass `isBibleVoiceCommand` from the native shell and do not call PWA internals directly from the shell.

This prerequisite is intentionally limited to the two command strings. Help-screen work remains separate.
