# Configuración del flujo

## Correo

| Campo | Valor |
|-------|-------|
| Remitente | Dominio **`@intelideck.com`** |
| Ejemplo | `ivonne.casillas@intelideck.com` |
| Carpeta Outlook | Bandeja de entrada |
| Solo con adjuntos | Sí |
| Nombre PDF | Indistinto |

## Excel de control

| Campo | Valor |
|-------|-------|
| Archivo | `CONTROL PEDIMENTOS.xlsx` |
| Ruta | `Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/CONTROL/` |
| Tabla | `tblPedimentos` |
| Clave anti-duplicado | `MessageID` + `NombrePDF` |

Columnas:

```text
FechaCorreo | Remitente | MessageID | NombrePDF | TamanoBytes | RutaDestino | FechaGuardado | Estado
```

## Destino OneDrive

| Campo | Valor |
|-------|-------|
| Base | `/Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES` |
| Año | `EXTRACCIONES {yyyy}` |
| Mes | `MM MES AÑO` (ej. `10 OCTUBRE 2026`) |

## Orden de acciones

1. Cuando llega un nuevo correo (V3)
2. Condición `@intelideck.com`
3. Variables `Anio`, `CarpetaMes`, `RutaAnio`, `RutaMes`
4. Crear carpeta año (run after error)
5. Crear carpeta mes (run after error)
6. Scope `ProcesarAdjuntos`
7. Aplicar a cada adjunto → ¿`.pdf`?
8. Listar filas Excel (OData o Filtrar matriz)
9. Si no existe → Get attachment → Crear archivo → Add row (`TamanoBytes`, `Estado=Guardado`)
10. Scope fallo → Add row (`Estado=Error`)
