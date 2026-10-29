# Konrad Gourmet — Sistema de Abastecimiento

Sistema de abastecimiento para restaurantes, construido con **arquitectura hexagonal**
(puertos y adaptadores) y organizado en módulos que corresponden a microservicios.

## Arquitectura

Cada módulo se divide siempre en las mismas capas:

| Capa | Contenido | Regla |
|---|---|---|
| `dominio/` | Entidades, objetos de valor y reglas de negocio | No depende de nada externo |
| `eventos/` | Eventos de dominio que publica el módulo | Extienden `EventoDominio` |
| `puertos/entrada/` | Casos de uso que el módulo ofrece | Interfaces |
| `puertos/salida/` | Lo que el módulo necesita del exterior | Interfaces |
| `aplicacion/` | Servicios que implementan los casos de uso | Orquesta el dominio |
| `adaptadores/entrada/` | Controladores REST, listeners, jobs | Llaman a los puertos de entrada |
| `adaptadores/salida/` | Persistencia, clientes HTTP/SOAP, correo | Implementan los puertos de salida |

La dependencia siempre apunta hacia adentro: los adaptadores conocen los puertos,
nunca al revés. El dominio no conoce a nadie.

## Módulos

| Módulo | Microservicio | Historias | Responsabilidad |
|---|---|---|---|
| `compartido` | — | — | Librería común: publicación de eventos y auditoría |
| `identidad` | ms-identidad | V01 | Usuarios, roles, sucursales y autenticación |
| `pedidos` | ms-pedidos | V02–V04 | Registro y seguimiento de pedidos |
| `menu` | ms-menu | V06 | Platos, ingredientes y categorías |
| `inventario` | ms-inventario | V05, V07 | Stock, umbrales y consumo diario |
| `solicitudes` | ms-solicitudes | V08–V12 | Solicitudes de alimentos |
| `cotizaciones` | ms-cotizaciones | V13–V20 | Cotizaciones, proveedores y reglas de validación |
| `ordenes` | ms-ordenes | V21–V23 | Órdenes de compra |
| `notificaciones` | ms-notificaciones | — | Correos y alarmas en la aplicación |
| `integracion` | ms-integracion | — | Frontera SOAP con el Ministerio de Comercio |
| `administracion` | ms-administracion | — | Parámetros del sistema y auditoría |
| `analitica` | ms-analitica | — | Tablero BAM e indicadores |
| `demo` | — | — | Ejecuta los patrones sin base de datos ni Kafka |

## Patrones de diseño

| Patrón | Dónde | Clases |
|---|---|---|
| Factory Method | `notificaciones/dominio` | `CreadorNotificacion`, `CreadorCorreo`, `CreadorAlarmaApp` |
| Decorator | `notificaciones/dominio` | `CorreoDecorator`, `CorreoCertificadoDecorator`, `EstampaCronologicaDecorator` |
| Strategy | `cotizaciones/dominio` | `ReglaValidacion`, `ReglaRechazada`, `ReglaSospechosa`, `ReglaOpcionada` |
| Facade | `cotizaciones/aplicacion` | `GenerarCotizacionFacade` |
| Proxy | `integracion/adaptadores/salida` | `PrecioReferenciaCacheProxy` |
| Observer | `compartido/eventos` | `EventoDominio`, `EventListener` |

## Estructura

```
KonradGourmet/
├── README.md
└── src/co/edu/konradlorenz/
    ├── compartido/
    ├── identidad/
    ├── pedidos/
    ├── menu/
    ├── inventario/
    ├── solicitudes/
    ├── cotizaciones/
    ├── ordenes/
    ├── notificaciones/
    ├── integracion/
    ├── administracion/
    ├── analitica/
    └── demo/
```

## Ejecutar la demo

```bash
javac -d out $(find src -name "*.java")
java -cp out co.edu.konradlorenz.demo.DemoArquitectura
```

## Interfaz web

La pantalla de inicio de sesión está en la raíz del proyecto:

```
index.html      maquetado
styles.css      estilos
assets/logo/    logo.png
assets/video/   3DVideo.mp4
```

Se abre directamente en el navegador. Si el video no arranca por las
restricciones de `file://`, levanta un servidor local:

```bash
python -m http.server 8000
```
