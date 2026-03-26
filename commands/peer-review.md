# Peer Review de Figma

## Modo de ejecución
- No expliques lo que vas a hacer, hacelo directamente
- No resumas los datos obtenidos de Figma, procesalos de inmediato
- No pidas confirmación entre frames
- Output mínimo en el chat — solo progreso esencial

## Inicio
Al arrancar pedí únicamente:

"Compartime el link del archivo de Figma y el nombre exacto de la página a revisar."

No empieces el análisis hasta tenerlo.

Extraer el fileKey de la URL:
https://figma.com/design/:fileKey/:nombre?... → usar :fileKey en todas las llamadas a use_figma.

---

## Inicialización

### CICLO 1 — Info + frames + skills (todo en un solo mensaje, en paralelo)

Ejecutar SIMULTÁNEAMENTE:

**1a — use_figma** (info del archivo + tokens DS + lista de frames):
```js
const PAGE_NAME = 'NOMBRE_DE_LA_PAGINA'; // reemplazar con el nombre real
const fileName = figma.root.name;
const page = figma.root.children.find(p => p.name === PAGE_NAME)
          || figma.root.children.find(p => p.name.trim() === PAGE_NAME.trim());
if (!page) return {
  error: `Página "${PAGE_NAME}" no encontrada`,
  pages: figma.root.children.map(p => p.name)
};
await figma.setCurrentPageAsync(page);
const allVars = await figma.variables.getLocalVariablesAsync();
const colorVarIds = allVars.filter(v => v.resolvedType === 'COLOR').map(v => v.id);
const textStyles = figma.getLocalTextStyles().map(s => ({ id: s.id, name: s.name }));

const SKIP_NAMES = new Set(['Guía', 'Notas', 'Doc', 'Changelog', 'Cover', 'Instrucciones']);
const SKIP_TYPES = new Set(['CONNECTOR', 'SECTION']);
const SKIP_PREFIXES = ['⚙️', '📝'];
const frames = page.children
  .filter(n => {
    if (SKIP_TYPES.has(n.type)) return false;
    if (SKIP_NAMES.has(n.name)) return false;
    if (SKIP_PREFIXES.some(p => n.name.startsWith(p))) return false;
    return true;
  })
  .map(n => ({ id: n.id, name: n.name, type: n.type }));

return { fileName, pageId: page.id, colorVarIds, textStyles, frames };
```

**1b — Leer los 7 skills en paralelo** (en el mismo mensaje que 1a):
- Read file: /Users/tomasfermani/.claude/commands/skills/spacing.md
- Read file: /Users/tomasfermani/.claude/commands/skills/border-radius.md
- Read file: /Users/tomasfermani/.claude/commands/skills/components.md
- Read file: /Users/tomasfermani/.claude/commands/skills/colors.md
- Read file: /Users/tomasfermani/.claude/commands/skills/copy.md
- Read file: /Users/tomasfermani/.claude/commands/skills/content.md
- Read file: /Users/tomasfermani/.claude/commands/skills/notes.md

Registrá internamente la hora de inicio: startTime = Date.now()

Si colorVarIds y textStyles están vacíos: continuar igual.
Los skills reportarán todos los fills SOLID como hardcodeados (comportamiento esperado cuando el DS no usa variables locales).

Guardar en memoria: { colorVariableIds, textStyleIds, spacingValues: [4,8,12,16,24,32,40,48,64,72,80,88,96] }

### CICLO 2 — Fetch batch — datos de todos los frames (una sola llamada)

**Una sola llamada use_figma** que recolecta nodos+textos de TODOS los frames a revisar.
Incluye detección de señales visuales para decidir si se necesita screenshot:

```js
const PAGE_ID = 'PAGE_ID'; // usar el pageId obtenido en paso 1
const FRAME_IDS = ['id1', 'id2', ...]; // lista completa de frames a revisar

const page = figma.root.children.find(p => p.id === PAGE_ID);
await figma.setCurrentPageAsync(page);

function serializeBoundVars(bv) {
  if (!bv) return {};
  const result = {};
  for (const key of Object.keys(bv)) {
    const val = bv[key];
    if (Array.isArray(val)) result[key] = val.map(v => ({ type: v.type, id: v.id }));
    else if (val && val.type) result[key] = { type: val.type, id: val.id };
  }
  return result;
}

function collectNodes(node, depth, insideInstance) {
  depth = depth || 0;
  // Saltear nodos ocultos y todos sus hijos — no ocupan espacio visual
  // y generan falsos positivos en spacing, colores y componentes.
  // Excepción: el frame raíz (depth === 0) siempre se incluye.
  if (depth > 0 && node.visible === false) return [];
  // Saltear vectores y boolean ops profundos dentro de instancias — son partes de íconos,
  // nunca tienen issues relevantes de spacing/radius/color y generan mucho payload
  if ((node.type === 'VECTOR' || node.type === 'BOOLEAN_OPERATION') && insideInstance && depth > 3) {
    return [];
  }
  const isInstance = node.type === 'INSTANCE';
  // Omitir absoluteBoundingBox en vectores (no se usa para ningún skill)
  const includeBBox = node.type !== 'VECTOR' && node.type !== 'BOOLEAN_OPERATION';
  // Omitir fills cuando ya tiene boundVariables (el color está vinculado, no es relevante analizarlo)
  const bv = serializeBoundVars(node.boundVariables);
  const hasBoundVars = Object.keys(bv).length > 0;
  const out = [{
    id: node.id, name: node.name, type: node.type,
    visible: node.visible,
    fills: hasBoundVars ? undefined : node.fills,
    cornerRadius: node.cornerRadius,
    absoluteBoundingBox: includeBBox ? node.absoluteBoundingBox : undefined,
    boundVariables: bv
  }];
  if (node.children) {
    node.children.forEach(c => out.push(...collectNodes(c, depth + 1, insideInstance || isInstance)));
  }
  return out;
}

function hasVisualAnomalySignals(frame, nodes) {
  const fb = frame.absoluteBoundingBox;
  for (const n of nodes) {
    if (!n.absoluteBoundingBox || n.visible === false) continue;
    const nb = n.absoluteBoundingBox;
    // Overflow: nodo se sale del frame
    if (nb.x + nb.width > fb.x + fb.width + 5) return true;
    if (nb.y + nb.height > fb.y + fb.height + 5) return true;
    // Imagen (posible placeholder)
    if (n.fills && n.fills.some(f => f.type === 'IMAGE')) return true;
    // Texto con altura sospechosamente pequeña (posible truncado)
    if (n.type === 'TEXT' && nb.height < 12) return true;
    // Nodo con dimensiones cero (posible problema de layout)
    if (nb.width === 0 || nb.height === 0) return true;
  }
  return false;
}

const results = [];
for (const frameId of FRAME_IDS) {
  const frame = page.findOne(n => n.id === frameId);
  if (!frame) { results.push({ frameId, empty: true }); continue; }
  const nodes = collectNodes(frame);
  // Recolectar textos con walker propio — frame.findAll() no respeta visibilidad
  // de ancestros: un TEXT hijo de un FRAME oculto igual aparecería en los resultados.
  // Este walker para la recursión en cualquier nodo con visible: false.
  function collectTexts(node) {
    const result = [];
    if (node.type === 'TEXT' && node.characters) {
      result.push({ id: node.id, name: node.name, text: node.characters });
    }
    if (node.children) {
      for (const child of node.children) {
        if (child.visible !== false) result.push(...collectTexts(child));
      }
    }
    return result;
  }
  const texts = collectTexts(frame);
  const needsScreenshot = hasVisualAnomalySignals(frame, nodes);
  results.push({ frameId, nodes, texts, needsScreenshot });
}
return results;
```

Guardar en memoria los resultados como `framesData = [{ frameId, nodes, texts, needsScreenshot }]`.

**Screenshots condicionales — solo para frames con `needsScreenshot: true`:**
En el MISMO mensaje que contiene la llamada use_figma de arriba (si se anticipa que habrá frames con señales), o en el siguiente mensaje tras recibir los resultados:
- Por cada frame donde `needsScreenshot === true`: llamar `mcp__figma-desktop__get_screenshot` con su nodeId.
- Frames con `needsScreenshot === false`: no tomar screenshot.
- Si algún screenshot falla: ignorar y continuar.

---

## Flujo por frame

Los datos (nodes, texts) ya están en memoria desde el paso 4.
Screenshots solo disponibles para frames que los necesitaban.
No hacer ningún fetch adicional. Solo analizar y escribir notas.

Para cada frame en `framesData`:

Si `frame.empty === true` → saltear silenciosamente.

### Análisis visual
Solo si el frame tiene screenshot disponible (needsScreenshot era true):
Revisar la imagen para detectar anomalías visuales:
- Texto truncado o con overflow
- Contenido desbordando contenedores
- Componentes superpuestos
- Imágenes placeholder
- Desalineaciones evidentes

Si no hay screenshot: omitir este paso y continuar con los skills.

### Ejecución de skills
Los skills están cargados en contexto desde la pre-carga inicial — no releer.

Para cada frame en framesData (análisis en memoria, sin llamadas externas):

PASO 1 — Ejecutar con frame.nodes:
spacing · border-radius · components · colors

PASO 2 — Ejecutar con frame.texts:
copy · content

Acumular todos los issues de todos los frames en memoria: `allFrameIssues = [{ frameId, frameName, issuesByModule }]`

### CICLO 3 — Escritura de notas (UN SOLO use_figma para todos los frames)

Tras analizar todos los frames, crear todas las notas en una única llamada use_figma.
El script itera `allFrameIssues` y genera los noteFrames de todos los frames en un solo bloque JS.
Ver notes.md para el patrón del script — aplicarlo en un loop sobre todos los frames con issues.

Estructura del script:
```js
// Para cada frame con issues, crear sus notas posicionadas a la derecha del frame
// Un solo forEach/for sobre allFrameIssues
// Todos los noteFrames creados en la misma ejecución
```

Progreso acumulado (una línea por frame, al escribir las notas):
"✅ [frame1] — [N] issues · ✅ [frame2] — [N] issues · ..."

Escribir el resumen final automáticamente tras crear las notas.

---

## Formato de output estandarizado entre skills
Todos los skills deben devolver issues con esta estructura exacta:
{
  elementId: "id del nodo",
  elementName: "nombre del nodo",
  description: "descripción del problema",
  found: "valor encontrado",
  expected: "corrección sugerida"
}

---

## Manejo de errores
Si falla cualquier llamada use_figma:
"⚠️ Error de conexión con Figma. Verificá que el archivo esté abierto en Figma Desktop y volvé a intentar."
No sigas hasta resolver.

---

## Resumen final

✅ Review completo — [nombre del archivo]
─────────────────────────────────────
Frames: X revisados · X salteados

📐 Espaciado:     X issues
🔘 Border Radius: X issues
🎨 Colores:       X issues
✏️  Copy:          X issues
📦 Contenido:     X issues
🧩 Componentes:   X issues
─────────────────────────────────────
Top 3 más repetidos:
· [issue] — X frames
· [issue] — X frames
· [issue] — X frames
─────────────────────────────────────
⏱️  Tiempo total: [calcular con Date.now() - startTime, mostrar en minutos y segundos]
Todos los issues están marcados en el canvas de Figma.
