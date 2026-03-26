# Skill — 📐 Espaciado

## Datos de entrada
Provistos por el orquestador via mcp__figma-remote-mcp__use_figma (Llamada 1 — nodes).

## IMPORTANTE
El script de nodes puede no retornar paddingLeft/Right/Top/Bottom ni itemSpacing
para todos los nodos. No interpretar su ausencia como "sin padding".

## Regla fundamental
Ignorar SIEMPRE nodos con visible: false antes de calcular cualquier gap o padding.
Un nodo oculto no ocupa espacio visual y genera falsos positivos si se incluye.

## Método correcto — gaps reales via absoluteBoundingBox
Para cada par de elementos hermanos **visibles**, calcular la distancia real:
  gap horizontal = elemento_B.absoluteBoundingBox.x
                   - (elemento_A.absoluteBoundingBox.x + elemento_A.absoluteBoundingBox.width)
  gap vertical   = elemento_B.absoluteBoundingBox.y
                   - (elemento_A.absoluteBoundingBox.y + elemento_A.absoluteBoundingBox.height)

Ordenar los hijos por posición (x o y) antes de calcular.
Filtrar: solo nodos con visible !== false y con absoluteBoundingBox definido.
Solo reportar gaps > 0 (gaps negativos = superposición, ignorar).

## Si paddingLeft/Right/Top/Bottom o itemSpacing SÍ están presentes
Verificarlos también contra la escala válida.

## Escala válida (sistema 8pt)
4, 8, 12, 16, 24, 32, 40, 48, 64, 72, 80, 88, 96px

## Output esperado
{
  elementId: "id del nodo",
  elementName: "nombre del nodo",
  description: "Gap fuera de escala entre [A] y [B]",
  found: "18px",
  expected: "16px o 24px"
}
