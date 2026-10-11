# Rule: Privacy & PII Sanitization Guardrail

## Context
This project produces educational and open laboratory guide materials. Under no circumstances should real personal identifying information (PII) of the user be hardcoded, generated, or committed into guides, configuration templates, or documentation.

## Invariant Directives
1. **Strict Username Sanitization:**
   - Never output or embed personal usernames (e.g., `rd908213` or personal account handles).
   - Always use the standardized default homelab username: `pi` (or `user` if on generic Linux distributions).
   - In SSH connection examples, chown commands, and path examples, strictly use `pi@homelab-pi.local`, `pi:pi`, and `/home/pi/`.

2. **Personal Domain, Email & IP Redaction:**
   - Never include personal email addresses, private WAN IPs, or unique personal domain names.
   - Use standard RFC placeholders: `user@example.com`, `192.168.1.X`, `homelab-pi.local`, or `homelab-nas`.

3. **Credential Masking:**
   - Never embed actual user passwords. Always use generic placeholders: `[Write your password here]` or `<secure-password>`.
   - Never include live API keys, Tailscale auth keys, or private SSH keys.

4. **Pre-Generation Audit:**
   - Before writing or updating any file in `modules/`, `modules_filled/`, or `README.md`, scan the content to verify no personal user identifiers are present.
