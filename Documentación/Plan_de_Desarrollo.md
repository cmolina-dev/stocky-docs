# Plan de Desarrollo - NexusInventory
## Roadmap de Implementación

**Proyecto:** NexusInventory  
**Objetivo:** Sistema de inventario multi-tenant offline-first  
**Última actualización:** 2026-02-07

---

## Filosofía de Desarrollo

### Principios Clave

1. **Incremental y Funcional**: Cada fase debe resultar en algo funcional que puedas probar
2. **Offline-First desde el inicio**: No dejes la funcionalidad offline para el final
3. **Backend y Frontend en paralelo**: Desarrolla ambos simultáneamente para validar la integración temprano
4. **Testing continuo**: Prueba cada módulo antes de avanzar al siguiente

### Estrategia de Priorización

```
Prioridad 1: Funcionalidad CORE offline (operación local de una sede)
Prioridad 2: Sincronización básica (push/pull)
Prioridad 3: Multi-tenant y coordinación inter-sede
Prioridad 4: Features avanzadas (IA, reportes complejos)
```

---

## Fase 0: Setup Inicial (1-2 días)

### Backend

- [ ] Inicializar proyecto NestJS con TypeScript
- [ ] Configurar PostgreSQL local (Docker recomendado)
- [ ] Configurar Redis local (Docker recomendado)
- [ ] Setup Prisma ORM
- [ ] Definir schema.prisma completo (basado en ERD_PostgreSQL.md)
- [ ] Ejecutar primera migración
- [ ] Configurar variables de entorno (.env)
- [ ] Setup ESLint y Prettier

### Frontend

- [ ] Inicializar proyecto React + Vite + TypeScript
- [ ] Configurar Tailwind CSS
- [ ] Instalar Shadcn/ui
- [ ] Instalar RxDB y dependencias
- [ ] Configurar estructura de carpetas
- [ ] Setup ESLint y Prettier
- [ ] Configurar Vite PWA Plugin

### Infraestructura

- [ ] Crear repositorio Git
- [ ] Configurar .gitignore
- [ ] Documentar README con instrucciones de setup
- [ ] Configurar scripts de desarrollo (package.json)

**Entregable:** Proyectos inicializados, bases de datos corriendo, estructura básica lista

---

## Fase 1: Autenticación y Multi-Tenancy (3-4 días)

### Backend

- [ ] Implementar AuthModule (NestJS)
  - [ ] Registro de usuarios
  - [ ] Login con JWT
  - [ ] Refresh tokens
  - [ ] Guards de autenticación
- [ ] Implementar TenantModule
  - [ ] CRUD de tenants
  - [ ] CRUD de sedes
  - [ ] Middleware de tenant isolation
- [ ] Configurar Row-Level Security en PostgreSQL
- [ ] Implementar RBAC (Role-Based Access Control)
  - [ ] Guards por rol
  - [ ] Decoradores @Roles()

### Frontend

- [ ] Crear páginas de autenticación
  - [ ] Login
  - [ ] Registro (solo para testing)
- [ ] Implementar AuthContext/Store (Zustand)
- [ ] Guardar JWT en localStorage
- [ ] Configurar axios/fetch con interceptors para JWT
- [ ] Crear ProtectedRoute component
- [ ] Implementar colección auth_session en RxDB

### Testing

- [ ] Login exitoso genera JWT válido
- [ ] JWT se guarda en localStorage
- [ ] Rutas protegidas redirigen si no hay sesión
- [ ] Roles se validan correctamente

**Entregable:** Sistema de login funcional, usuarios pueden autenticarse y acceder según su rol

---

## Fase 2: Base de Datos Local (RxDB) (4-5 días)

### Frontend

- [ ] Implementar schemas RxDB (basado en Schema_IndexedDB.md)
  - [ ] products
  - [ ] inventory
  - [ ] transactions
  - [ ] sync_queue
  - [ ] config
- [ ] Crear función de inicialización de RxDB
- [ ] Implementar ProductCache en memoria
- [ ] Crear hooks React para cada colección
  - [ ] useProducts()
  - [ ] useInventory()
  - [ ] useTransactions()
- [ ] Implementar observadores de cambios (RxDB subscriptions)
- [ ] Crear utilidades para batch queries

### Testing

- [ ] Crear productos en IndexedDB
- [ ] Consultar productos (individual y batch)
- [ ] Crear transacciones append-only
- [ ] Calcular stock desde transacciones
- [ ] Verificar que datos persisten al recargar página
- [ ] Probar multi-tab sync (abrir en 2 pestañas)

**Entregable:** Base de datos local funcional, operaciones CRUD offline, datos persisten

---

## Fase 3: Módulo de Productos e Inventario (Offline) (5-6 días)

### Frontend

- [ ] Crear UI para gestión de productos
  - [ ] Lista de productos
  - [ ] Crear producto
  - [ ] Editar producto
  - [ ] Eliminar producto (soft delete)
  - [ ] Búsqueda y filtros
- [ ] Crear UI para inventario
  - [ ] Vista de stock actual
  - [ ] Alertas de stock bajo
  - [ ] Ajustes manuales de inventario
- [ ] Implementar validaciones de formularios
- [ ] Agregar indicador visual de "Modo Offline"

### Lógica de Negocio

- [ ] Validar SKU único por tenant
- [ ] Calcular stock automáticamente desde transacciones
- [ ] Actualizar inventory cuando se crea una transacción
- [ ] Implementar control de versión para conflictos

### Testing

- [ ] Crear 20 productos offline
- [ ] Editar productos y verificar versión incrementa
- [ ] Crear transacciones y verificar stock se actualiza
- [ ] Cerrar navegador y verificar datos persisten
- [ ] Simular pérdida de conexión (DevTools offline)

**Entregable:** Gestión completa de productos e inventario funcionando 100% offline

---

## Fase 4: Módulo de Transacciones (POS Lite) (4-5 días)

### Frontend

- [ ] Crear UI para registro de transacciones
  - [ ] Venta (OUT)
  - [ ] Entrada (IN)
  - [ ] Ajuste manual (ADJUSTMENT)
- [ ] Implementar carrito de compra simple
- [ ] Validar stock disponible antes de venta
- [ ] Crear historial de transacciones
- [ ] Implementar filtros por fecha, tipo, producto

### Lógica de Negocio

- [ ] Validar stock antes de registrar venta
- [ ] Crear transacción append-only
- [ ] Actualizar inventory automáticamente
- [ ] Agregar transacción a sync_queue
- [ ] Implementar batch processing para ventas múltiples

### Testing

- [ ] Registrar venta de 1 producto
- [ ] Registrar venta de 50 productos (validar rendimiento)
- [ ] Intentar vender más stock del disponible (debe fallar)
- [ ] Verificar historial muestra todas las transacciones
- [ ] Verificar transacciones se agregan a sync_queue

**Entregable:** POS funcional offline, ventas y entradas de inventario operativas

---

## Fase 5: Backend - Endpoints de Sincronización (5-6 días)

### Backend

- [ ] Implementar SyncModule
  - [ ] POST /api/sync/push (recibir cambios del cliente)
  - [ ] GET /api/sync/pull (enviar cambios al cliente)
- [ ] Implementar lógica de validación de cambios
- [ ] Guardar transacciones en transactions_log
- [ ] Actualizar inventory_consolidated
- [ ] Actualizar products_consolidated
- [ ] Implementar detección de conflictos
- [ ] Aplicar política Last-Write-Wins
- [ ] Crear logs de auditoría

### Testing

- [ ] Push de 10 transacciones desde cliente
- [ ] Verificar datos en PostgreSQL
- [ ] Pull de configuraciones desde backend
- [ ] Simular conflicto (mismo producto editado en 2 sedes)
- [ ] Verificar Last-Write-Wins funciona correctamente

**Entregable:** Backend puede recibir y enviar cambios, sincronización básica funcional

---

## Fase 6: Motor de Sincronización (Frontend) (5-6 días)

### Frontend

- [ ] Implementar Network Detector
  - [ ] Monitorear navigator.onLine
  - [ ] Hacer pings periódicos al backend
- [ ] Implementar Sync Engine
  - [ ] Procesar sync_queue automáticamente
  - [ ] Push en lotes de 50 operaciones
  - [ ] Pull periódico (cada 5 minutos)
  - [ ] Retry logic con backoff exponencial
- [ ] Crear UI de estado de sincronización
  - [ ] Indicador online/offline
  - [ ] Contador de operaciones pendientes
  - [ ] Última sincronización exitosa
  - [ ] Progreso de sincronización
- [ ] Implementar sincronización manual (botón)

### Testing

- [ ] Crear 20 transacciones offline
- [ ] Reconectar y verificar sincronización automática
- [ ] Verificar datos en PostgreSQL
- [ ] Desconectar durante sincronización (debe reintentar)
- [ ] Simular error de backend (debe marcar como FAILED después de 5 intentos)

**Entregable:** Sincronización automática funcional, datos fluyen entre cliente y backend

---

## Fase 7: Gestión de Sedes (3-4 días)

### Backend

- [ ] Implementar SedesModule
  - [ ] CRUD de sedes
  - [ ] Asignar usuarios a sedes
  - [ ] Actualizar last_sync_at
  - [ ] Endpoint para listar sedes online/offline

### Frontend

- [ ] Crear UI de administración de sedes (Tenant Admin)
  - [ ] Lista de sedes
  - [ ] Crear/editar sede
  - [ ] Ver estado de sincronización por sede
  - [ ] Asignar usuarios a sedes

### Testing

- [ ] Crear 3 sedes para un tenant
- [ ] Asignar usuarios a cada sede
- [ ] Verificar aislamiento (Sede A no ve datos de Sede B)
- [ ] Verificar last_sync_at se actualiza correctamente

**Entregable:** Gestión de múltiples sedes, aislamiento de datos por sede

---

## Fase 8: Transferencias entre Sedes (7-8 días)

### Backend

- [ ] Implementar TransferModule
  - [ ] POST /transfers/request
  - [ ] POST /transfers/:id/approve
  - [ ] POST /transfers/:id/reject
  - [ ] GET /transfers (listar transferencias)
- [ ] Configurar BullMQ
- [ ] Crear TransferWorker
  - [ ] Validar stock en sede origen
  - [ ] Ejecutar transacción atómica
  - [ ] Crear TRANSFER_OUT y TRANSFER_IN
  - [ ] Actualizar inventory_consolidated
  - [ ] Notificar vía WebSocket
- [ ] Implementar WebSocket Gateway (Socket.io)
  - [ ] Rooms por tenant
  - [ ] Eventos de transferencia

### Frontend

- [ ] Crear UI para solicitar transferencia
  - [ ] Seleccionar sede destino
  - [ ] Seleccionar producto
  - [ ] Especificar cantidad
  - [ ] Validar que ambas sedes estén online
- [ ] Crear UI para aprobar/rechazar transferencias
- [ ] Implementar WebSocket client
  - [ ] Escuchar eventos de transferencia
  - [ ] Actualizar IndexedDB local al recibir TRANSFER_IN/OUT
- [ ] Crear lista de transferencias pendientes

### Testing

- [ ] Solicitar transferencia de Sede A a Sede B
- [ ] Aprobar desde Sede B
- [ ] Verificar stock se actualiza en ambas sedes
- [ ] Verificar transacciones se crean en ambas sedes
- [ ] Rechazar transferencia (debe cancelarse)
- [ ] Intentar transferir más stock del disponible (debe fallar)
- [ ] Simular desconexión durante transferencia (debe marcar como PENDING)

**Entregable:** Transferencias entre sedes funcionales, operación atómica garantizada

---

## Fase 9: Reportes Consolidados (4-5 días)

### Backend

- [ ] Implementar InventoryModule
  - [ ] GET /inventory/consolidated (inventario total)
  - [ ] GET /inventory/by-sede (inventario por sede)
  - [ ] GET /inventory/low-stock (productos con stock bajo)
- [ ] Implementar caché con Redis
  - [ ] Cachear inventario consolidado
  - [ ] Invalidar caché al sincronizar
- [ ] Crear queries SQL optimizadas con agregaciones

### Frontend

- [ ] Crear dashboard de Tenant Admin
  - [ ] Inventario total por producto
  - [ ] Inventario por sede
  - [ ] Productos con stock bajo
  - [ ] Indicador de última sincronización por sede
  - [ ] Advertencias de datos desactualizados
- [ ] Implementar gráficos (Chart.js o Recharts)
- [ ] Crear filtros y búsqueda

### Testing

- [ ] Verificar totales consolidados son correctos
- [ ] Verificar advertencias cuando sede está desincronizada
- [ ] Verificar caché de Redis funciona
- [ ] Probar con 3 sedes y 100+ productos

**Entregable:** Reportes consolidados funcionales, visibilidad multi-sede

---

## Fase 10: Módulo de IA (Normalizer) (5-6 días)

### Backend

- [ ] Implementar AIModule
  - [ ] POST /ai/normalize (normalizar productos)
  - [ ] Integración con OpenAI API
  - [ ] Rate limiting
  - [ ] Caché de resultados
- [ ] Crear lógica de detección de duplicados
- [ ] Implementar sugerencias de categorización

### Frontend

- [ ] Crear UI para importar productos (CSV/Excel)
- [ ] Crear UI para revisar sugerencias de IA
  - [ ] Lista de productos similares
  - [ ] Sugerencias de nombres normalizados
  - [ ] Aprobar/rechazar en lote
- [ ] Implementar preview antes de aplicar cambios

### Testing

- [ ] Importar 50 productos con nombres inconsistentes
- [ ] Verificar IA detecta duplicados
- [ ] Aprobar sugerencias y verificar productos se unifican
- [ ] Verificar rate limiting funciona

**Entregable:** IA normaliza catálogos, reduce duplicados, mejora consistencia

---

## Fase 11: Auditoría y Logs (3-4 días)

### Backend

- [ ] Implementar AuditModule
  - [ ] Interceptor global para capturar acciones
  - [ ] Guardar en audit_logs
  - [ ] GET /audit/logs (consultar logs)
- [ ] Implementar filtros por usuario, acción, entidad

### Frontend

- [ ] Crear UI de auditoría (solo admins)
  - [ ] Lista de logs
  - [ ] Filtros por fecha, usuario, acción
  - [ ] Detalles de cada acción (before/after)

### Testing

- [ ] Crear producto y verificar log se genera
- [ ] Editar producto y verificar before/after
- [ ] Filtrar logs por usuario
- [ ] Verificar logs de transferencias

**Entregable:** Sistema de auditoría completo, trazabilidad total

---

## Fase 12: PWA y Optimizaciones (4-5 días)

### Frontend

- [ ] Configurar Service Worker (Vite PWA Plugin)
- [ ] Definir estrategia de caché
  - [ ] Cache-first para assets estáticos
  - [ ] Network-first para API calls
- [ ] Crear manifest.json
- [ ] Agregar iconos PWA (diferentes tamaños)
- [ ] Implementar install prompt
- [ ] Optimizar bundle size
  - [ ] Code splitting
  - [ ] Lazy loading de rutas
  - [ ] Tree shaking
- [ ] Implementar limpieza de datos antiguos
- [ ] Monitorear uso de almacenamiento

### Testing

- [ ] Instalar PWA en desktop y mobile
- [ ] Verificar funciona completamente offline
- [ ] Verificar assets se cachean correctamente
- [ ] Probar en diferentes navegadores
- [ ] Verificar rendimiento (Lighthouse)

**Entregable:** PWA instalable, funciona offline, rendimiento optimizado

---

## Fase 13: Testing y QA (5-7 días)

### Testing Funcional

- [ ] Crear suite de tests E2E (Playwright o Cypress)
  - [ ] Flujo completo de venta offline
  - [ ] Sincronización después de offline
  - [ ] Transferencia entre sedes
  - [ ] Conflictos de sincronización
- [ ] Tests de integración backend (Jest)
  - [ ] Endpoints de sincronización
  - [ ] Transferencias atómicas
  - [ ] Multi-tenancy isolation
- [ ] Tests unitarios frontend (Vitest)
  - [ ] Hooks de RxDB
  - [ ] Lógica de negocio
  - [ ] Validaciones

### Testing de Escenarios Adversos

- [ ] Pérdida de conexión durante venta
- [ ] Pérdida de conexión durante sincronización
- [ ] Pérdida de conexión durante transferencia
- [ ] Edición concurrente del mismo producto
- [ ] Cierre inesperado del navegador
- [ ] Almacenamiento lleno
- [ ] Backend caído durante operación

### Testing de Rendimiento

- [ ] Venta de 100 productos simultáneos
- [ ] Sincronización de 1000 transacciones
- [ ] 10 usuarios concurrentes en misma sede
- [ ] 3 sedes sincronizando simultáneamente

**Entregable:** Suite de tests completa, bugs críticos resueltos

---

## Fase 14: Deployment y DevOps (3-4 días)

### Backend

- [ ] Configurar Railway (o alternativa)
- [ ] Configurar PostgreSQL managed
- [ ] Configurar Redis managed
- [ ] Variables de entorno en producción
- [ ] Configurar CI/CD (GitHub Actions)
- [ ] Setup de migraciones automáticas
- [ ] Configurar logs y monitoring

### Frontend

- [ ] Configurar Vercel (o alternativa)
- [ ] Configurar variables de entorno
- [ ] Configurar CI/CD
- [ ] Configurar dominio personalizado
- [ ] Setup de analytics (opcional)

### Testing en Producción

- [ ] Smoke tests en producción
- [ ] Verificar PWA se instala correctamente
- [ ] Verificar sincronización funciona
- [ ] Verificar WebSockets funcionan

**Entregable:** Aplicación desplegada en producción, CI/CD configurado

---

## Fase 15: Documentación y Pulido (2-3 días)

### Documentación

- [ ] README completo con instrucciones
- [ ] Documentación de API (Swagger/OpenAPI)
- [ ] Guía de usuario
- [ ] Guía de administrador
- [ ] Troubleshooting común

### Pulido de UI/UX

- [ ] Revisar consistencia de diseño
- [ ] Mejorar mensajes de error
- [ ] Agregar loading states
- [ ] Mejorar feedback visual
- [ ] Optimizar para mobile

**Entregable:** Aplicación pulida, documentación completa

