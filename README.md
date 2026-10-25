# Reto 3 — Facilidad de Integración y Modificación (Holding XYZ)

Curso **ARTI4109 – Arquitectura de Software**, Maestría en Arquitecturas de TI (MATI), Universidad de los Andes.

## Contexto

La holding de salud XYZ (1 EPS, 5 IPS, 1 aseguradora de medicina prepagada y 1 empresa de seguros de vida) está formada por empresas adquiridas en momentos distintos, con sistemas heterogéneos (ERP, callcenter, CRM) que hoy se integran manualmente mediante archivos CSV.

El reto es diseñar una arquitectura que integre clientes, contratos, proveedores, ventas, ingresos, egresos y productos, que sea **modificable** (algoritmos de interpretación e informes cambian con frecuencia) y **extensible** (futuras adquisiciones en turismo y salud, reemplazo de componentes en producción).

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| [ARTI4109-reto3-v2.pdf](ARTI4109-reto3-v2.pdf) | Enunciado del reto |
| [Arquitectura-XYZ-documento.html](Arquitectura-XYZ-documento.html) | Documento de arquitectura (abrir en el navegador) |
| [MEMORIA.md](MEMORIA.md) | Memoria de trabajo: decisiones, ASRs/QAS, vistas y pendientes |
| [Arquitectura-Reto3.drawio](Arquitectura-Reto3.drawio) | Diagramas fuente (draw.io), una pestaña por vista |
| `Arquitectura-Reto3_*.svg` | Exportaciones de cada vista: funcional, módulos, zona cruda, zona estructurada, INT-1 (facturación y pagos), INT-2 (consolidación de personas) |

## Atributos de calidad (ASRs)

- **ASR-MOD-1 / ASR-MOD-2**: modificabilidad de algoritmos de interpretación y de presentación de informes.
- **ASR-INT-1 / ASR-INT-2**: integrabilidad (facturación y pagos; consolidación de personas).

El detalle de cada escenario está en [MEMORIA.md](MEMORIA.md).

## Cómo ver el documento

Abrir `Arquitectura-XYZ-documento.html` en cualquier navegador (no requiere servidor ni dependencias). Los diagramas se editan con [draw.io](https://www.drawio.com/) abriendo el archivo `.drawio`.

## Autor

Julian Builes — Universidad de los Andes, MATI.
