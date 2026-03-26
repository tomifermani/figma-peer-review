# Skill — 🔘 Border Radius

## Datos de entrada
Provistos por el orquestador via mcp__figma-remote-mcp__use_figma (Llamada 1 — nodes).

## Tokens esperados por contexto
- Botones (button, btn, cta) → 100px
- Badges/tags (badge, tag, chip, pill) → 9999px
- Cards/contenedores (card, container, surface) → 8px
- Inputs (input, field, text-field) → 8px

## Qué reportar
Todo nodo cuyo cornerRadius no coincida con el token esperado.

## Output esperado
{
  elementId: "id del nodo",
  elementName: "nombre del nodo",
  description: "Border radius incorrecto para [tipo]",
  found: "100px",
  expected: "9999px (token pill)"
}
