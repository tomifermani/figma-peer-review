# Skill — ✏️ Copy

## Datos de entrada
Provistos por el orquestador via mcp__figma-remote-mcp__use_figma (Llamada 2 — texts).

## Qué revisar

### Errores ortográficos (español es-MX)
- No reportar camelCase, snake_case ni contenido dentro de {{}}
  salvo que la variable tenga error ortográfico

### Tildes faltantes
- Interrogativas: qué, cómo, cuándo, dónde, cuál, quién, cuánto
- Frecuentes: número, título, sección, diagnóstico, médico,
  teléfono, dirección, código, categoría, información, más, también

### Lorem ipsum y variables con errores
- Detectar lorem ipsum o variantes
- Variables: {{Diagnostico}} → {{Diagnóstico}}

## Output esperado
{
  elementId: "id del nodo",
  elementName: "nombre del nodo",
  description: "Tilde faltante / Error ortográfico / Lorem ipsum",
  found: "texto encontrado",
  expected: "corrección sugerida"
}
