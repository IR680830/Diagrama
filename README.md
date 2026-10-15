# Pedimentos Intelideck → OneDrive + Excel

Flujo Power Automate para guardar PDF de `@intelideck.com` en la carpeta mensual de Extracciones, con bitácora en Excel y control de duplicados.

## Documentación

- [Guía completa del flujo](power-automate-pedimentos/GUIA-FLUJO.md)
- [Expresiones para pegar](power-automate-pedimentos/EXPRESIONES.md)
- [Plantilla de configuración](power-automate-pedimentos/CONFIG.md)

## Reglas

| Qué | Cómo |
|-----|------|
| Remitente | Dominio **`@intelideck.com`** |
| PDF | Todos los `.pdf` (nombre indistinto) |
| Destino | `.../EXTRACCIONES {año}/{MM MES AÑO}` |
| Anti-duplicado | Excel `MessageID` + `NombrePDF` |
| Auditoría | `CONTROL PEDIMENTOS.xlsx` con `TamanoBytes` y `Estado` |

Ármalo en [make.powerautomate.com](https://make.powerautomate.com).
