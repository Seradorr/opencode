# Closed-Network Assumptions

- Assume this installation is used in a closed or restricted network.
- Do not rely on public package registries, SaaS tools, or outbound internet access unless the user explicitly asks for it and the environment supports it.
- Keep endpoint details, API keys, and credentials out of generated code unless the user is explicitly editing local configuration.
- When discussing configuration, prefer local paths and OpenAI-compatible self-hosted endpoints.
- Favor solutions that still work when external services are unavailable.
