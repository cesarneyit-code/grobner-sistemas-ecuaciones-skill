# grobner-sistemas-ecuaciones-skill

Skill y documentación académica-operativa para resolver sistemas de ecuaciones polinomiales con Gröbner usando `oscar-mcp`.

Repositorio: https://github.com/cesarneyit-code/grobner-sistemas-ecuaciones-skill

## Contenido

- `SKILL.md`: activación y flujo oficial de la skill.
- `references/master_guide.md`: guía maestra completa (académica + operativa).
- `references/quickstart_cheatsheet.md`: guía rápida para uso diario.
- `references/decision_tree.md`: árbol de decisión método/fallback.
- `references/cookbook.md`: recetas reproducibles por técnica.

## Requisitos

- `julia` + `Oscar.jl`
- `python3`
- `uv`
- servidor MCP de OSCAR (`mcp-oscar-server`)
- cliente con soporte MCP (Codex, Claude Desktop, Cursor, etc.)

## 1) Usarlo en Codex (MCP + Skill)

### 1.1 Clonar

```bash
git clone https://github.com/cesarneyit-code/grobner-sistemas-ecuaciones-skill.git
```

### 1.2 Instalar la skill en Codex

Opción recomendada (symlink):

```bash
ln -s /ABS/PATH/grobner-sistemas-ecuaciones-skill ~/.codex/skills/grobner-sistemas-ecuaciones
```

Opción alternativa (copia):

```bash
cp -R /ABS/PATH/grobner-sistemas-ecuaciones-skill ~/.codex/skills/grobner-sistemas-ecuaciones
```

### 1.3 Registrar MCP de OSCAR en Codex

```bash
codex mcp add oscar-mcp -- \
  /Users/cesargalindo/.local/bin/uv run --directory \
  /Users/cesargalindo/Documents/docencia/presentaciones/mcp-oscar-server server.py
```

Si usas otra máquina, reemplaza rutas por las tuyas.

### 1.4 Variables de entorno recomendadas para `oscar-mcp`

```toml
[mcp_servers.oscar-mcp.env]
OSCAR_JULIA_CMD = "/Users/TU_USUARIO/.local/bin/oscar-julia"
OSCAR_PROJECT = "/Users/TU_USUARIO/opt/oscar/project"
OSCAR_DEPOT = "/Users/TU_USUARIO/opt/oscar/depot"
```

### 1.5 Verificación rápida

```bash
codex mcp list
```

Debe aparecer `oscar-mcp` en estado `enabled`.

Prompt de prueba:

```text
Usa grobner-sistemas-ecuaciones para eliminar x del sistema y-x^2, z-x^3 y valida con normal form.
```

## 2) ¿Qué hacer en otro cliente?

Sí, puedes usar **MCP + skill** también fuera de Codex, con esta estrategia:

1. Configura `oscar-mcp` en el cliente (sección MCP).
2. Reutiliza esta skill como guía/prompt library.
3. Usa prompts estructurados (`ProblemSpec`, `RunSpec`, `ReportSpec`) del `master_guide.md`.

## 3) Configuración MCP en Claude/Cursor (ejemplo JSON)

```json
{
  "mcpServers": {
    "oscar-mcp": {
      "command": "/Users/TU_USUARIO/.local/bin/uv",
      "args": [
        "run",
        "--directory",
        "/ABS/PATH/mcp-oscar-server",
        "server.py"
      ],
      "env": {
        "OSCAR_JULIA_CMD": "/Users/TU_USUARIO/.local/bin/oscar-julia",
        "OSCAR_PROJECT": "/Users/TU_USUARIO/opt/oscar/project",
        "OSCAR_DEPOT": "/Users/TU_USUARIO/opt/oscar/depot"
      }
    }
  }
}
```

## 4) ¿Cómo usar la skill si el cliente no tiene sistema de skills nativo?

Si el cliente no soporta skills como Codex:

1. Abre `references/master_guide.md`.
2. Copia la plantilla de solicitud del Apéndice A.
3. Pide explícitamente:
- objetivo,
- toolchain,
- validación,
- `ReportSpec` con citas.

Prompt base recomendado:

```text
Resuelve este sistema con enfoque Groebner usando ProblemSpec/RunSpec,
con validación explícita y salida en ReportSpec con citas BIB/DOC.
```

## 5) Flujo recomendado de uso (MCP + Skill)

1. Define `ProblemSpec`.
2. Elige ruta con `decision_tree.md` (simbólica o híbrida).
3. Ejecuta tools `grobner_*` vía MCP.
4. Valida (`basis_check`, `normal_form`, o residual si híbrido).
5. Reporta con `ReportSpec` + citas (`bibliography_primary.md`).

## 6) Ejemplos de prompts útiles

1. `Usa grobner-sistemas-ecuaciones y resuelve un caso 0-dimensional con FGLM.`
2. `Usa grobner-sistemas-ecuaciones para eliminación de parámetros y dame la relación implícita.`
3. `Usa grobner-sistemas-ecuaciones para radical + primaria y explica las ramas.`
4. `Usa grobner-sistemas-ecuaciones y decide si conviene puente numérico.`

## 7) Troubleshooting

1. `oscar-mcp` no aparece:
- revisa `codex mcp list`
- vuelve a registrar con `codex mcp add ...`

2. `oscar-mcp` aparece pero falla:
- verifica `julia --version`
- verifica `julia -e 'using Oscar; println("OSCAR_OK")'`
- revisa `OSCAR_JULIA_CMD`, `OSCAR_PROJECT`, `OSCAR_DEPOT`

3. Skill no activa en Codex:
- confirma symlink/copia en `~/.codex/skills/grobner-sistemas-ecuaciones`
- menciona explícitamente el nombre de la skill en el prompt

4. FGLM falla:
- revisar precondición 0-dimensional (ver `decision_tree.md`)

## 8) Actualización

```bash
cd /ABS/PATH/grobner-sistemas-ecuaciones-skill
git pull
```

Si instalaste por symlink en `~/.codex/skills`, queda actualizado automáticamente.

## 9) Referencias internas clave

- `SKILL.md`
- `references/master_guide.md`
- `references/quickstart_cheatsheet.md`
- `references/decision_tree.md`
- `references/cookbook.md`
- `references/bibliography_primary.md`

