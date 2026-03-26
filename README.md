# Skills

Colección de Agent Skills para AI agents (Claude, Cursor, Codex y otros).

## Skills disponibles

### commit-analyzer

Analiza cambios staged en git para detectar bugs, vulnerabilidades de seguridad, malas prácticas, y genera descripciones detalladas de commits con mensaje en formato Conventional Commits.

**Instalar:**
```bash
npx skillsadd jgiorgis/skills/commit-analyzer
```

**Uso:** Agregá tus cambios al staging area (`git add`) y pedile al agente que los analice antes de commitear.

**Qué hace:**
- Detecta bugs: off-by-one, null references, catch vacíos, promesas sin await
- Detecta vulnerabilidades: SQL injection, XSS, secrets hardcodeados, path traversal
- Evalúa prácticas de código: naming, SOLID, tipado, dead code
- Genera recomendaciones con severidad (CRITICO / ADVERTENCIA / SUGERENCIA)
- Produce descripción detallada del commit con alcance y beneficios
- Sugiere mensaje de commit en formato Conventional Commits

## Licencia

MIT
