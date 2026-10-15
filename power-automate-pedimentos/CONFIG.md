# Configuración del flujo (rellenar)

Copia estos valores al crear el flujo en make.powerautomate.com.

## Correo

| Campo | Valor |
|-------|-------|
| Remitente | Cualquier correo que contenga `intelideck` |
| Carpeta Outlook | Bandeja de entrada |
| Asunto contiene | *(dejar vacío)* |
| Solo correos con adjuntos | Sí |

## Adjuntos PDF (pedimentos)

| Campo | Valor |
|-------|-------|
| Ejemplo de nombre | `1422CEX0126026494.pdf` |
| Regla | Extensión `.pdf` y nombre tipo código (suele llevar `CEX`) |
| Expresión recomendada | ver `EXPRESIONES.md` → *PDF pedimento Intelideck* |

## Destino OneDrive

| Campo | Valor |
|-------|-------|
| Ruta base 2026 | `Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026` |
| Patrón carpeta mes | `MM MES AÑO` (ej. `10 OCTUBRE 2026`) |
| Tipos de archivo | `.pdf` con nombre tipo `1422CEX0126026494` |

## Equivalencia PC ↔ OneDrive

| En el PC (sincronizado) | En Power Automate (OneDrive) |
|-------------------------|------------------------------|
| `C:\Users\ISMAEL RAMIREZ\OneDrive - andujar\Documentos\01 IMPORTACIONES ANDUJAR\IMPORTACIONES\EXTRACCIONES 2026\10 OCTUBRE 2026` | `Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026/10 OCTUBRE 2026` |

## Acciones del flujo (orden)

1. Cuando llega un nuevo correo electrónico (V3) — Outlook
2. Compose `MesEsp` — expresión switch
3. Inicializar / Compose `NombreCarpetaMes`
4. Inicializar `RutaBase`
5. Crear nueva carpeta — OneDrive (`RutaBase` + `NombreCarpetaMes`)
6. Aplicar a cada uno — Adjuntos
7. Condición — PDF pedimento (`.pdf` + nombre tipo `…CEX…`)
8. Crear archivo — OneDrive en carpeta del mes
9. (Opcional) Marcar como leído / Mover correo
