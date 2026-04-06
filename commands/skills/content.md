# Skill — 📦 Contenido

## Datos de entrada
Provistos por el orquestador via mcp__figma-remote-mcp__use_figma (Llamada 2 — texts).

## Importante — variables {{}}
Este skill SÍ reporta placeholders dentro de `{{}}` si el contenido es irrealista.
La excepción de variables `{{}}` aplica SOLO al skill de Copy (para errores ortográficos).
Ejemplos a reportar: `{{Nombre Apellido}}`, `{{DD/MM/AAAA}}`, `{{Nombre del doctor}}`, `{{Especialidad}}`.

## Patrones a detectar

- Nombres genéricos: "Usuario 1", "John Doe", "Nombre Apellido", `{{Nombre Nombre Apellido}}`
- Fechas irreales: "01/01/2000", "DD/MM/AAAA", "00/00/0000", `{{DD/MM/AAAA}}`
- Montos placeholder: "$0.00", "$0", "MXN 0.00"
- Emails falsos: "test@test.com", "email@email.com"
- Teléfonos genéricos: "000-000-0000", "55 0000 0000"
- Descripciones de una sola línea en campos de notas/dirección
- Números redondos sospechosos: "100%", "$10,000", "10 resultados"

## Output esperado
{
  elementId: "id del nodo",
  elementName: "nombre del nodo",
  description: "Contenido de prueba poco realista",
  found: "texto encontrado",
  expected: "sugerencia de contenido real"
}
