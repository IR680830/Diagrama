# Automatización de pedimentos (Power Automate)

Guía para guardar automáticamente los PDF de pedimentos que envía el agente aduanal por Outlook en la carpeta mensual de Extracciones en OneDrive.

## Documentación

- [Guía completa del flujo](power-automate-pedimentos/GUIA-FLUJO.md)
- [Expresiones para pegar](power-automate-pedimentos/EXPRESIONES.md)
- [Plantilla de configuración](power-automate-pedimentos/CONFIG.md)

## Resumen

| Qué | Cómo |
|-----|------|
| Origen | Correos Outlook del agente aduanal con PDF |
| Destino | OneDrive → `Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026` |
| Carpeta del mes | Se crea sola, ej. `10 OCTUBRE 2026` |
| Equivalente en PC | `C:\Users\ISMAEL RAMIREZ\OneDrive - andujar\Documentos\...` (sincronizado) |

Sigue la guía en `power-automate-pedimentos/GUIA-FLUJO.md` dentro de [make.powerautomate.com](https://make.powerautomate.com).
