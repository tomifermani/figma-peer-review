# Figma Peer Review

Sistema de auditoría automática de diseño que analiza archivos de Figma con Claude Code y detecta inconsistencias antes de que el diseño pase a desarrollo.

## ¿Qué analiza?

Por cada frame de una página revisa seis dimensiones:

| Módulo | Qué detecta |
|--------|-------------|
| 📐 Espaciado | Gaps y paddings fuera de la escala 8pt |
| 🔘 Border Radius | Valores que no corresponden al token del componente |
| 🎨 Colores | Fills hardcodeados sin variable del DS vinculada |
| ✏️ Copy | Errores ortográficos, tildes faltantes, lorem ipsum |
| 📦 Contenido | Datos placeholder poco realistas |
| 🧩 Componentes | Instancias desvinculadas de la librería (detached) |

Las observaciones se escriben directamente en el canvas de Figma como notas posicionadas a la derecha de cada frame.

---

## Instalación

### 1. Requisitos

- [Claude Code](https://claude.ai/code) instalado
- Figma Desktop abierto con el archivo a revisar

### 2. Habilitar el plugin oficial de Figma

En Claude Code ejecutar:

```
/plugin install figma@claude-plugins-official
```

```
/reload-plugins
```

> Alternativamente, editar `~/.claude/settings.json` y agregar:
> ```json
> {
>   "enabledPlugins": {
>     "figma@claude-plugins-official": true
>   }
> }
> ```

### 3. Copiar los archivos de comandos

```bash
# Clonar este repo
git clone https://github.com/tomifermani/figma-peer-review

# Copiar los archivos a Claude Code
cp -r figma-peer-review/commands/peer-review.md ~/.claude/commands/
cp -r figma-peer-review/commands/skills ~/.claude/commands/
```

### 4. Verificar

En Claude Code escribir `/peer-review` — debería responder pidiendo el link de Figma y el nombre de la página.

---

## Uso

1. Abrir Claude Code
2. Escribir `/peer-review`
3. Compartir el link del archivo de Figma y el nombre exacto de la página
4. Esperar el análisis — las notas aparecen automáticamente en el canvas

---

## Estructura del repo

```
figma-peer-review/
├── README.md
└── commands/
    ├── peer-review.md          ← comando principal
    └── skills/
        ├── spacing.md
        ├── border-radius.md
        ├── components.md
        ├── colors.md
        ├── copy.md
        ├── content.md
        └── notes.md
```

---

## Actualizar

Cuando haya cambios en los skills, cada integrante corre:

```bash
cd figma-peer-review
git pull
cp -r commands/peer-review.md ~/.claude/commands/
cp -r commands/skills ~/.claude/commands/
```
