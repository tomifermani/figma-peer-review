# Skill — 🧩 Componentes

## Datos de entrada
Provistos por el orquestador via mcp__figma-remote-mcp__use_figma (Llamada 1 — nodes).

## Qué detectar
Nodos que cumplan AMBAS condiciones:
1. Nombre empieza con "🧩"
2. Tienen type: "FRAME" (no "INSTANCE")

Un componente vinculado tiene type: "INSTANCE".
Un componente desvinculado fue convertido por Figma a type: "FRAME".
NO usar componentId — su ausencia genera falsos positivos.

## EXCEPCIÓN — falsos positivos por anatomía interna
Si el nodo 🧩 FRAME tiene un ancestro con type: "INSTANCE" en el árbol,
NO está detached — es estructura interna del componente maestro.
Solo reportar como detached si el nodo es hijo directo del canvas
o de un FRAME que NO sea INSTANCE.

## Importante
- NO usar get_remote_components (causa timeouts)

## Output esperado
{
  elementId: "id del nodo",
  elementName: "nombre del nodo",
  description: "Componente desvinculado de la librería",
  found: "type: FRAME (debería ser INSTANCE)",
  expected: "reconectar al componente maestro 🧩"
}
