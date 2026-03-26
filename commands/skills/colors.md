# Skill — 🎨 Colores

## Datos de entrada
Provistos por el orquestador via mcp__figma-remote-mcp__use_figma (Llamada 1 — nodes).

## Qué detectar
Colores hardcodeados — fills que NO tienen una variable del DS vinculada.

## Cómo detectar

Para cada nodo del árbol (incluyendo nodos dentro de instancias):

1. Ignorar nodos con visible: false — no reportarlos
2. Revisar el campo fills de cada nodo
3. Para cada fill SOLID, verificar si el nodo tiene alguna variable de color vinculada:

```
Un nodo tiene color vinculado al DS si boundVariables contiene CUALQUIER clave
(fills, color, u otra) cuyo valor sea un VARIABLE_ALIAS o un array con VARIABLE_ALIAS.

hasColorVariable(boundVariables):
  para cada clave en boundVariables:
    si el valor es array → algún elemento tiene type === "VARIABLE_ALIAS" → vinculado
    si el valor es objeto → tiene type === "VARIABLE_ALIAS" → vinculado
  si ninguna clave cumple → NO vinculado → REPORTAR
```

NO limitarse a verificar solo `boundVariables.fills` — Figma usa claves distintas
según el tipo de nodo (`fills` para shapes, `color` para textos, etc.).

## Validación secundaria con get_variable_defs
Si `boundVariables` está vacío `{}` pero el nodo es hijo de una INSTANCE
(posible falso positivo — la variable puede estar definida en el componente maestro):
- Llamar: mcp__figma-desktop__get_variable_defs con el elementId del nodo
- Si la respuesta incluye una variable de color → NO reportar (está vinculado)
- Si está vacía o sin variables de color → reportar como hardcodeado

Usar esta validación especialmente en vectores e iconos dentro de instancias.

## Qué NO hacer
- No comparar valores hex contra una lista de colores del DS
- No reportar strokes (solo fills)
- No reportar nodos sin fills definidos o con fills vacíos
- No reportar fills con type: "IMAGE" o type: "GRADIENT"
  (solo fills de tipo "SOLID")

## Qué reportar
Solo fills de tipo SOLID donde `hasColorVariable(boundVariables)` sea false.
Extraer el hex del color para incluirlo en el issue:
  r, g, b (0-1) → convertir a hex (#RRGGBB)

## Output esperado
{
  elementId: "id del nodo",
  elementName: "nombre del nodo",
  description: "Color hardcodeado sin variable del DS",
  found: "#3A86FF",
  expected: "vincular a variable de color del DS"
}
