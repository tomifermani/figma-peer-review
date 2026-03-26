# Skill — 📝 Notes

## Responsabilidad
Tomar todos los issues recolectados y escribirlos como notas en Figma.

## Reglas de agrupación
- UNA nota por módulo por frame
- Si no hay issues de un módulo → no crear nota
- Máximo 7 notas por frame

## Colores por módulo
📐 Espaciado     → verde menta (r:0.82, g:1, b:0.88)
🔘 Border Radius → celeste (r:0.78, g:0.93, b:1)
✏️  Copy          → amarillo (r:1, g:0.96, b:0.7)
📦 Contenido     → naranja claro (r:1, g:0.88, b:0.72)
🧩 Componentes   → rojo claro (r:1, g:0.82, b:0.82)
🎨 Colores       → azul claro (r:0.8, g:0.9, b:1)

## Cómo crear las notas
Una sola llamada mcp__figma-remote-mcp__use_figma por frame — crea todas las notas en un único script.

Construir el array NOTES con los módulos que tienen issues (omitir módulos sin issues).
Máximo 6 entradas (una por módulo activo).

Script JS (reemplazar PAGE_ID, FRAME_ID, FRAME_NAME y el array NOTES con los datos reales):
```js
const page = figma.root.children.find(p => p.id === 'PAGE_ID');
await figma.setCurrentPageAsync(page);

await figma.loadFontAsync({ family: 'Inter', style: 'Regular' });
await figma.loadFontAsync({ family: 'Inter', style: 'Bold' });

const frameNode = page.findOne(n => n.id === 'FRAME_ID');
const baseX = frameNode.absoluteBoundingBox.x + frameNode.absoluteBoundingBox.width + 32;
const baseY = frameNode.absoluteBoundingBox.y;

// Completar con los módulos que tienen issues — omitir los que no tienen
const NOTES = [
  { module: '📐 Espaciado',     color: { r: 0.82, g: 1,    b: 0.88 }, lines: ['🟡 ...'] },
  { module: '🔘 Border Radius', color: { r: 0.78, g: 0.93, b: 1    }, lines: ['🟡 ...'] },
  { module: '🎨 Colores',       color: { r: 0.8,  g: 0.9,  b: 1    }, lines: ['🔴 ...'] },
  { module: '✏️ Copy',          color: { r: 1,    g: 0.96, b: 0.7  }, lines: ['🟠 ...'] },
  { module: '📦 Contenido',     color: { r: 1,    g: 0.88, b: 0.72 }, lines: ['🟠 ...'] },
  { module: '🧩 Componentes',   color: { r: 1,    g: 0.82, b: 0.82 }, lines: ['🔴 ...'] },
];

const created = [];
NOTES.forEach((note, idx) => {
  const noteFrame = figma.createFrame();
  noteFrame.name = `📝 ${note.module} — FRAME_NAME`;
  noteFrame.fills = [{ type: 'SOLID', color: note.color }];
  noteFrame.cornerRadius = 8;
  noteFrame.layoutMode = 'VERTICAL';
  noteFrame.paddingTop = noteFrame.paddingBottom = 12;
  noteFrame.paddingLeft = noteFrame.paddingRight = 12;
  noteFrame.resize(272, 40); // resize ANTES de sizing modes
  noteFrame.primaryAxisSizingMode = 'AUTO';  // hug vertical
  noteFrame.counterAxisSizingMode = 'FIXED'; // ancho fijo 272px
  noteFrame.x = baseX;
  noteFrame.y = baseY + (idx * 116);

  // CRÍTICO: resize(248, 10) ANTES de appendChild
  // NO usar layoutSizingHorizontal='FILL' — causa width:0
  const t = figma.createText();
  t.resize(248, 10);
  t.textAutoResize = 'HEIGHT';
  noteFrame.appendChild(t);
  t.fontSize = 11;
  const titulo = note.module;
  const fullText = titulo + '\n' + note.lines.join('\n');
  t.characters = fullText;
  t.setRangeFontName(0, titulo.length + 1, { family: 'Inter', style: 'Bold' });
  t.setRangeFontName(titulo.length + 1, fullText.length, { family: 'Inter', style: 'Regular' });

  created.push(noteFrame.id);
});

return { created };
```

## Posicionamiento
- x = frameNode.absoluteBoundingBox.x + frameNode.absoluteBoundingBox.width + 32
- y = frameNode.absoluteBoundingBox.y + (índice * 116)
  - índice 0 = primera nota, 1 = segunda, etc.

## Formato del texto
Línea 1: título del módulo (Bold)
Líneas siguientes:
[emoji severidad] [elemento]: [descripción] ([encontrado] → [corrección])

Emoji de severidad:
🔴 crítico (colores hardcoded, font family sin asignar, detached)
🟡 advertencia (espaciado, border radius, tipografía)
🟠 copy/contenido

Ejemplo:
📐 Espaciado
🟡 Header / Body: gap 18px → 16px
🟡 Card / Button: padding 10px → 8px

Ejemplo:
🧩 Componentes
🔴 🧩 Tarjeta-Resumen: desvinculado de librería → reconectar
