Gestión de notificaciones en ConstanciaUV

Status: proposed

Date: 2026-11-02

Context and Problem Statement

La Universidad requiere un sistema (ConstanciaUV) que permita a los estudiantes autorizados solicitar y obtener constancias digitales. El servicio de notificaciones por correo electrónico es un sistema externo a la aplicación (RT1). Debemos decidir cómo organizar arquitectónicamente la comunicación entre el flujo de generación de constancias y el servicio de notificaciones de modo que, si el proveedor externo no está disponible, la generación y almacenamiento de constancias continúen sin interrupción (RT2), las notificaciones pendientes se conserven para reintentarse posteriormente (RT3), y se prevengan solicitudes de notificación duplicadas para una misma constancia sin presuponer que el proveedor garantice una entrega única (RT4).

Decision Drivers

Resiliencia operativa y disponibilidad del núcleo (RT2, RT3): Garantizar que la indisponibilidad o latencia del proveedor externo de correo no interrumpa ni bloquee la emisión y el almacenamiento de constancias digitales.

Integridad transaccional e idempotencia (RT1, RT4): Mantener un control estricto de las solicitudes de notificación para conservar los pendientes y prevenir envíos duplicados hacia el proveedor externo.

Considered Options

Alternativa A: Comunicación directa con recuperación de fallos (solicitud síncrona inicial y registro de reintento directo ante fallos).

Alternativa B: Procesamiento asíncrono mediante eventos persistidos (registro persistente inmediato y componente procesador independiente).

Decision Outcome

Chosen option: "Alternativa B", porque desacopla por completo el flujo principal de generación y almacenamiento de constancias respecto al estado del servicio externo de correo, cumpliendo de forma nativa con las restricciones RT2 y RT3 al tiempo que permite un control riguroso de concurrencia para evitar notificaciones duplicadas (RT4).

Consequences

Good, porque la generación y almacenamiento de constancias nunca se detienen ni se ven afectados por caídas o tiempos de espera agotados del proveedor de correo.

Good, porque las solicitudes de notificación pendientes quedan registradas en almacenamiento persistente de forma segura para reintentos posteriores.

Bad, porque incrementa la complejidad arquitectónica al requerir un componente independiente o trabajador en segundo plano para procesar la cola de eventos.

Trade-off: Sacrificamos la simplicidad de implementación inicial de la Alternativa A a cambio de ganar resiliencia, alta disponibilidad y desacoplamiento estructural.

Confirmation

La correcta implementación de esta decisión se comprobará mediante:

Pruebas de integración con inyección de fallos (fault injection), simulando que el servicio externo de correo responde con errores 500 o tiempos de espera agotados (timeouts), para verificar que la generación y persistencia de la constancia concluyen con éxito.

Pruebas unitarias y de concurrencia sobre la base de datos de notificaciones pendientes para validar que las restricciones de unicidad e idempotencia impiden procesar o insertar solicitudes duplicadas para una misma constancia.

Pros and Cons of the Options

Alternativa A: Comunicación directa con recuperación de fallos

Good, porque presenta una menor complejidad de diseño e implementación inicial al no requerir procesadores en segundo plano.

Bad, porque genera un acoplamiento temporal fuerte que arriesga bloquear el hilo principal o interrumpir el flujo si el proveedor externo responde con lentitud o falla masivamente.

Alternativa B: Procesamiento asíncrono mediante eventos persistidos

Good, porque aísla al sistema central de los fallos de terceros y asegura la conservación de notificaciones pendientes de forma transparente.

Bad, porque agrega sobrecarga operacional y mayor complejidad en la gestión de estados y tareas asíncronas.

More Information


Revisión 1. Cambio en el proveedor de notificaciones

Cambio recibido: El proveedor externo de notificaciones modifica su interfaz de comunicación, alterando la estructura de las peticiones y los códigos de error con los que reporta los fallos de envío.
Impacto sobre la alternativa seleccionada: El impacto se confina de manera estricta al adaptador de integración dentro del componente procesador independiente, evitando cualquier modificación o riesgo en el flujo central de generación y almacenamiento de constancias.
Vigencia de la decisión: La decisión original (Alternativa B) se mantiene, ya que el desacoplamiento estructural basado en eventos persistidos aísla eficazmente al núcleo del sistema frente a cambios en contratos de terceros.
Riesgo o consecuencia a revisar: Se incrementa ligeramente el acoplamiento con el contrato del proveedor en la capa del adaptador, requiriendo pruebas de integración específicas para dicha interfaz.
Comprobación actualizada: Se incorpora en la sección de `Confirmation` la validación mediante pruebas de contrato y adaptador para asegurar que las nuevas respuestas de error del proveedor se mapean correctamente hacia los estados de reintento.
