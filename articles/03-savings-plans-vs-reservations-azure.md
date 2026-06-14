# Reservas en Azure: el error que convierte un descuento en un coste

Comprar reservas en Azure es fácil. Demasiado fácil. Y ahí está el problema.

## El problema real

Conozco el patrón. Lo he visto en más de una organización. Alguien abre Azure Advisor, ve las recomendaciones de Reserved Instances con esos porcentajes de ahorro en verde, y compra. Sin analizar el uso real. Sin entender qué compromete exactamente. Sin saber siquiera que existe otra opción llamada Savings Plans.

El resultado: reservas infrautilizadas que no generan ahorro sino desperdicio. Capacidad reservada que no se transfiere de una hora a la siguiente. Dinero comprometido a uno o tres años sobre workloads que cambiaron de tamaño, de región o directamente dejaron de existir.

La propia documentación de Azure lo dice con claridad: comprar más capacidad que tu uso histórico resulta en una reserva infrautilizada, y la capacidad reservada no consumida no se arrastra a la hora siguiente. El uso que exceda la cantidad reservada se cobra a precio on-demand. [Fuente: azure-cost-management-billing-costs_compressed.pdf, sección "Determine what reservation to purchase"]

No es un problema de la herramienta. Es un problema de proceso.

## El giro: no es "reserva o savings plan", es "en qué orden"

Aquí es donde la mayoría de equipos se pierde. La decisión no empieza eligiendo entre Reservations y Savings Plans. Empieza mucho antes.

Azure Advisor lo documenta con un orden específico que pocos respetan: primero rightsizing y shutdown de recursos infrautilizados, después reservas, y finalmente savings plans. En esa secuencia. No al revés, no a la vez. [Fuente: azure-advisor.pdf, sección "Understand cost savings"]

La razón es mecánica, no filosófica. Cada paso invalida las recomendaciones del siguiente. Si haces rightsizing primero, reduces tu consumo on-demand. Eso cambia tu patrón de uso. Eso cambia las recomendaciones de reservas. Y eso cambia las recomendaciones de savings plans. Las recomendaciones actualizadas tardan al menos tres días en reflejarse después de cada compra. [Fuente: azure-cost-management-billing-costs_compressed.pdf, sección "Recommendations in the Azure portal"]

Comprar reservas sin haber limpiado primero es como medir para un traje antes de decidir si vas al gimnasio.

## Idea clave 1: Reservations y Savings Plans no son lo mismo — y la diferencia importa

Las Azure Reservations son un compromiso de uso. Te comprometes a usar un tipo específico de recurso, un SKU concreto, en una ubicación determinada, durante uno o tres años. A cambio, descuentos de hasta un 72% sobre el precio on-demand. [Fuente: cloud-computing-finops_compressed.pdf, sección "Rate optimization"]

Los Azure Savings Plans son un compromiso de gasto. Te comprometes a gastar una cantidad fija por hora en compute, independientemente del tipo de recurso, SKU o ubicación. A cambio, descuentos de hasta un 65% sobre el precio on-demand. [Fuente: cloud-computing-finops_compressed.pdf, sección "Rate optimization"]

La diferencia de descuento no es casual. La documentación lo explica directamente: la mayor especificidad de las reservas es lo que genera un descuento más favorable. [Fuente: cloud-computing-finops_compressed.pdf, sección "Rate optimization"]

Traducido: más riesgo, más descuento. Menos flexibilidad, mejor precio.

La decisión correcta depende de tu nivel de confianza en la estabilidad de tu uso. Si tienes alta confianza en que mantendrás un nivel específico de uso para un tipo, SKU y ubicación concretos, la documentación recomienda empezar con una reserva. Si necesitas flexibilidad porque tus workloads cambian de tipo, tamaño o región, savings plans cubren un rango amplio de compute con un compromiso de gasto por hora. [Fuente: cloud-computing-finops_compressed.pdf, sección "Rate optimization"]

## Idea clave 2: la reserva infrautilizada es un concepto medible, no una sensación

Azure no te deja a ciegas después de comprar. Lo que sí pasa es que la mayoría no mira.

Puedes ver la utilización directamente desde la página de reservas o savings plans en el portal. Puedes configurar alertas de utilización para que avisen si cae por debajo de un umbral. Puedes expandir el scope o habilitar instance size flexibility para aumentar la utilización de un compromiso existente. [Fuente: cloud-computing-finops_compressed.pdf, sección "Rate optimization" — Getting started]

Para los que necesitan datos más granulares, Azure separa los datos de coste en dos datasets: Actual Cost y Amortized Cost. El dataset de Amortized Cost es el que muestra las horas de reserva no utilizadas y el valor monetario del desperdicio. Se filtra con `ChargeType = "UnusedReservation"` usando el ReservationID o ReservationName para identificar qué reserva concreta se infrautilizó. [Fuente: azure-cost-management-billing-costs_compressed.pdf, sección "Reservation charges in Azure usage data"]

Si no estás midiendo la utilización de tus compromisos, no estás haciendo rate optimization. Estás haciendo compras.

## Idea clave 3: las políticas de intercambio y reembolso tienen límites reales

Cuando alguien compra una reserva equivocada, la primera pregunta es "¿puedo cambiarla?". La respuesta es: depende, y tiene letra pequeña.

Los intercambios de reservas de compute (VMs, Dedicated Host, App Service) siguen disponibles durante un periodo de gracia extendido que inicialmente iba a terminar en enero de 2024, pero se ha extendido hasta nuevo aviso. Dentro de ese periodo puedes cambiar entre series de instancias o regiones. Pero esa flexibilidad tiene fecha de caducidad. [Fuente: azure-cost-management-billing-costs_compressed.pdf, sección "Reservation exchange policy changes"]

Los reembolsos tienen un tope: 50.000 USD acumulados en una ventana móvil de 12 meses, a nivel de billing scope. Y hay reservas que directamente no admiten ni intercambio ni reembolso: planes de Red Hat, SUSE Linux, y todos los planes de pre-compra. [Fuente: azure-cost-management-billing-costs_compressed.pdf, sección "Exchange or refund"]

La alternativa que propone la propia documentación: convertir reservas de workloads dinámicos a savings plans, y mantener reservas solo para workloads estables donde la configuración específica es conocida. [Fuente: azure-cost-management-billing-costs_compressed.pdf, sección "Reservation exchange policy changes"]

## Idea clave 4: Azure Advisor tiene un sesgo que debes conocer

Las recomendaciones de reservas en Azure Advisor operan con scope de suscripción individual. Si quieres recomendaciones para todo el billing scope (cuenta de facturación o perfil), tienes que ir por otro camino: portal de Reservaciones, agregar el tipo que quieres evaluar. [Fuente: azure-cost-management-billing-costs_compressed.pdf, sección "Recommendations in Azure Advisor"]

Otro detalle que pocos conocen: las recomendaciones de Advisor son por defecto para reservas de tres años. Si el servicio no vende reservas de tres años, se calcula con precio de un año. Y si compras una reserva de scope compartido, las recomendaciones pueden tardar hasta cinco días en desaparecer. [Fuente: azure-cost-management-billing-costs_compressed.pdf, sección "Recommendations in Azure Advisor"]

Todo esto significa que si compras una reserva a las 10 de la mañana y otra a las 3 de la tarde basándote en las mismas recomendaciones, la segunda compra se hizo con datos que ya no reflejan tu situación real.

## Cierre

El error más caro con las reservas en Azure no es comprar la reserva equivocada. Es comprar antes de limpiar, medir antes de comprar nada, y monitorizar después de cada decisión.

La documentación de Azure describe un proceso claro: primero eliminar desperdicio (rightsize, shutdown), luego comprar reservas donde hay alta confianza y estabilidad, y cubrir el resto con savings plans para flexibilidad. No inventé nada. Solo estoy describiendo lo que ya está documentado pero que pocos ejecutan en ese orden.

Empieza con tres preguntas antes de tu próxima compra de compromisos:

1. ¿Ya hice rightsizing de mis recursos? Si no, cualquier recomendación de reserva está calculada sobre uso inflado.
2. ¿Cuánta confianza tengo en que este workload mantendrá el mismo tipo, SKU y región durante 1-3 años?
3. ¿Tengo un sistema de monitorización de utilización configurado, o voy a comprar y olvidar?

Si la respuesta a cualquiera de esas tres es "no" o "no sé", no compres todavía. El descuento de mañana sigue siendo descuento. La reserva infrautilizada de hoy es coste desde el minuto uno.

---

## Fuentes

| Documento | Contenido utilizado |
|-----------|-------------------|
| `cloud-computing-finops_compressed.pdf` | Definición de rate optimization, diferencias Reservations vs Savings Plans, porcentajes de descuento, recomendaciones de monitorización y cobertura |
| `azure-cost-management-billing-costs_compressed.pdf` | Mecánica de descuentos, política de intercambio/reembolso, instance size flexibility, datasets Actual vs Amortized Cost, recomendaciones del portal y Advisor |
| `azure-advisor.pdf` | Orden de ejecución (rightsizing → reservations → savings plans), criterios de VM underutilization, Cost Optimization workbook |

---

*Serie "FinOps a lo LenOps" — Mi forma de entender FinOps: clara, práctica y sin humo.*

*Licencia: CC BY-NC-SA 4.0*
