# CarreñoJ-Post2-U12

**Patrones de Diseño de Software — Unidad 12: Integración de Patrones y Arquitecturas**
Universidad de Santander (UDES) · Ingeniería de Sistemas · 2026

---

## Objetivo

Implementar un conjunto de reglas de validación arquitectónica con **ArchUnit** sobre el sistema de pedidos del Post-Contenido 1, documentar tres decisiones de diseño clave en formato **ADR**, y verificar que las reglas se ejecutan automáticamente en un pipeline de **GitHub Actions**.

---

## Estructura del proyecto

```
.
├── .github/
│   └── workflows/
│       └── arquitectura.yml
├── docs/
│   └── adr/
│       ├── ADR-001.md
│       ├── ADR-002.md
│       └── ADR-003.md
├── src/
│   ├── main/java/com/empresa/pedidos/
│   │   ├── adaptadores/
│   │   │   ├── facade/FachadaPedidos.java
│   │   │   ├── procesadores/
│   │   │   │   ├── ProcesadorPedidoEstandar.java
│   │   │   │   ├── ProcesadorPedidoExpress.java
│   │   │   │   ├── ProcesadorPedidoFactory.java
│   │   │   │   └── ProcesadorPedidoInternacional.java
│   │   │   └── rest/PedidoController.java
│   │   ├── aplicacion/ServicioPedidos.java
│   │   ├── dominio/
│   │   │   ├── puertos/
│   │   │   │   ├── ProcesadorPedido.java
│   │   │   │   ├── RepositorioPedidos.java
│   │   │   │   └── ServicioNotificacion.java
│   │   │   ├── EstadoPedido.java
│   │   │   ├── Pedido.java
│   │   │   ├── PedidoId.java
│   │   │   ├── PedidoProcesadoEvent.java
│   │   │   └── TipoPedido.java
│   │   ├── infraestructura/
│   │   │   ├── notificaciones/
│   │   │   │   ├── NotificacionEmail.java
│   │   │   │   └── NotificacionLog.java
│   │   │   └── persistencia/
│   │   │       ├── PedidoJpaRepository.java
│   │   │       └── RepositorioPedidosJpa.java
│   │   └── PedidosApplication.java
│   └── test/java/com/empresa/pedidos/
│       ├── adaptadores/procesadores/ProcesadorPedidoFactoryTest.java
│       ├── PedidoProcesadoEventTests.java
│       ├── PedidosApplicationTests.java
│       └── ReglasArquitectura.java
└── README.md
```

---

## Validación Arquitectónica con ArchUnit

Se implementaron **5 reglas ArchUnit** en la clase `ReglasArquitectura` que codifican las intenciones de diseño como restricciones verificables automáticamente en cada build:

### Regla 1 — `dominioAislado`
El dominio no debe depender de infraestructura, adaptadores, JPA ni Spring Mail. Garantiza que el núcleo del negocio sea independiente de frameworks y detalles técnicos.

### Regla 2 — `controladorSoloFacade`
Las clases del paquete `adaptadores.rest` solo pueden acceder a `adaptadores.facade`, `dominio` y clases de Spring Web y Java estándar. Impide que el controlador REST acceda directamente a repositorios o servicios internos.

### Regla 3 — `puertosComoInterfaces`
Todas las clases del paquete `dominio.puertos` deben ser interfaces. Hace cumplir el principio de inversión de dependencias: el dominio define contratos, no implementaciones.

### Regla 4 — `procesadoresImplementanPuerto`
Todas las clases de `adaptadores.procesadores` deben implementar `ProcesadorPedido`. Asegura que cualquier nuevo procesador cumpla el contrato del puerto antes de compilar.

### Regla 5 — `infraNoAccedeRest`
Ninguna clase de infraestructura puede acceder a los adaptadores REST. Evita dependencias circulares entre capas.

```bash
# Ejecutar solo las reglas de arquitectura
mvn test -Dtest=ReglasArquitectura

# Salida esperada
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
```

---

## Pipeline GitHub Actions

El archivo `.github/workflows/arquitectura.yml` ejecuta las reglas ArchUnit automáticamente en cada push a `main` o `develop` y en cada Pull Request a `main`.

El historial del repositorio incluye:
- Un commit con una **violación intencional** en `Pedido.java` (import de infraestructura) que provocó el pipeline en rojo.
- El `git revert` que corrigió la violación y volvió el pipeline a verde.

> 📸 **Pipeline verde (reglas pasando):** ver `img/captura1.png`  
> 📸 **Pipeline rojo (violación detectada):** ver `img/captura2.png`  
> 📸 **Pipeline verde tras revert:** ver `img/captura3.png`

---

## Decisiones de Diseño (ADRs)

Los tres ADRs del sistema se encuentran en [`docs/adr/`](docs/adr/):

| ADR | Título | Estado |
|-----|--------|--------|
| [ADR-001](docs/adr/ADR-001.md) | Arquitectura Hexagonal para aislar el dominio | Aceptado |
| [ADR-002](docs/adr/ADR-002.md) | Factory + Strategy para selección de procesador | Aceptado |
| [ADR-003](docs/adr/ADR-003.md) | Spring Events (Observer) para notificaciones | Aceptado |

---

## Prerrequisitos

```xml
<!-- pom.xml: dependencia ArchUnit -->
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <version>1.2.1</version>
    <scope>test</scope>
</dependency>
```

- Java 17+
- Maven 3.9+
- Cuenta GitHub con Actions habilitado

---