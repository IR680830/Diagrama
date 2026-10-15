# Configuración del flujo (rellenar)

Copia estos valores al crear el flujo en make.powerautomate.com.

## Correo

| Campo | Valor |
|-------|-------|
| Remitente | Dominio **`@intelideck.com`** |
| Carpeta Outlook | Bandeja de entrada |
| Asunto contiene | *(dejar vacío)* |
| Solo correos con adjuntos | Sí |

## Adjuntos PDF (pedimentos)

| Campo | Valor |
|-------|-------|
| Ejemplo de nombre | `1422CEX0126026494.pdf` (el nombre no importa) |
| Regla | Cualquier archivo con extensión `.pdf` |
| Expresión recomendada | ver `EXPRESIONES.md` → *PDF de Intelideck* |

## Destino OneDrive

| Campo | Valor |
|-------|-------|
| Ruta base 2026 | `Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026` |
| Patrón carpeta mes | `MM MES AÑO` (ej. `10 OCTUBRE 2026`) |
| Tipos de archivo | Todos los `.pdf` |

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
7. Condición — es `.pdf` (cualquier nombre)
8. Crear archivo — OneDrive en carpeta del mes
9. (Opcional) Marcar como leído / Mover correo
