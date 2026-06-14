# Sin etiquetas no hay FinOps: el tagging es el paso cero que nadie quiere dar

Todos quieren optimizar costes en cloud. Rightsizing, reservas, CUDs, apagar entornos de noche. Son los temas que llenan las conversaciones de FinOps.

Pero hay un problema previo a todo eso. Un problema que no es técnicamente difícil, que no requiere herramientas caras y que, sin embargo, la mayoría de equipos no resuelve.

El tagging.

Sin etiquetas en tus recursos, no sabes quién gasta, en qué, ni por qué. No puedes hacer showback. No puedes hacer chargeback. No puedes medir si el rightsizing funcionó. Todo lo que viene después del tagging — optimización, reservas, negociación con el proveedor — es adivinar sin datos.

## El problema real

Conoces la situación. Llega fin de mes. La factura de cloud sube. Alguien pregunta: "¿Por qué subimos un 20%?" Y nadie puede responder con precisión porque los recursos no tienen etiquetas. O las tienen inconsistentes. O las pusieron una vez y nadie las mantuvo.

Lo que hacemos entonces es abrir Cost Explorer (o Cost Analysis, o Billing Reports) y filtrar por servicio o por cuenta. Vemos que EC2 subió, o que Compute Engine tiene más gasto. Pero no podemos responder la pregunta que importa: ¿qué equipo, qué aplicación, qué entorno generó ese incremento?

Esa es la pregunta que el tagging responde. Y sin esa respuesta, toda discusión de optimización es especulación.

## El giro: el tagging no es un proyecto técnico, es un proyecto organizativo

La documentación de AWS lo dice con claridad: implementar una estrategia de tagging es un proceso de iteración y mejora continua. No es algo que se configure una vez y se olvide. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Building your tagging strategy"]

El ciclo tiene cuatro fases: identificar necesidades y casos de uso, definir y publicar el esquema, implementar y enforcar, medir y mejorar. Luego se repite. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Tagging strategy iteration and improvement cycle"]

Y aquí está la parte que muchos ignoran: los equipos de ingeniería a menudo no tienen experiencia en la gestión financiera de sus recursos. Necesitan formación. Necesitan contexto. Necesitan entender por qué les pides que pongan un tag `cost-center:123` en cada instancia que desplieguen. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Tags for cost allocation and financial management"]

El tagging no falla por falta de herramientas. Falla porque nadie lo trata como lo que es: un cambio cultural que requiere ownership, comunicación y gobernanza.

## Idea 1: Showback y chargeback empiezan con un tag

En la documentación de AWS se distinguen dos conceptos fundamentales de cost allocation: showback y chargeback. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Cost allocation tags"]

El showback presenta el gasto a los equipos para que tengan visibilidad: "tu equipo de infraestructura fue responsable de X dólares el mes pasado." El chargeback va un paso más allá: ese coste se deduce realmente del presupuesto del equipo.

En ambos casos, sin tags no hay atribución posible. Los cost allocation tags te dicen quién es dueño del gasto y responsable de optimizarlo, qué workload o aplicación lo genera, en qué entorno ocurre, y qué áreas crecen más rápido. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Cost allocation tags"]

Un detalle que se pasa por alto: en AWS, los cost allocation tags no son retrospectivos. Solo aparecen en los informes de billing después de activarlos. Todo lo que no etiquetaste antes de la activación queda en un agujero negro de visibilidad. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Building a cost allocation strategy"]

## Idea 2: Cada cloud tiene su mecanismo, pero el problema es el mismo

El tagging no es un problema de AWS. Es un problema multicloud.

En **Azure**, la documentación de FinOps define allocation como el proceso de atribuir, asignar y redistribuir costes compartidos usando cuentas, tags y otros metadatos para establecer accountability entre equipos y proyectos. Azure ofrece tag inheritance en Cost Management, que aplica tags de subscripción y resource group a los recursos hijos automáticamente en los datos de coste, sin modificar los tags reales de los recursos. También permite usar Azure Policy para enforcar tags a escala y medir compliance como KPI. [Fuente: Azure_FinOps_Documentation.md, "Allocation" + azure-cost-management-billing-costs, "Tags / Tag inheritance"]

En **GCP**, se llaman labels en vez de tags, pero cumplen la misma función. El FinOps Score de Google Cloud — el indicador de madurez FinOps — incluye explícitamente el uso de tags y labels para asignar costes como uno de los factores de evaluación. Si no etiquetas, tu score baja. Así de directo. [Fuente: GCP_FinOps_Hub.md, "FinOps Score"]

En **OCI**, el FinOps Hub incluye tags como herramienta de gestión de metadata de recursos, integrada en la fase de Operate del ciclo FinOps (Inform, Optimize, Operate). [Fuente: OCI_FinOps_Documentation.md, "FinOps Hub" + "FinOps en OCI — Fases"]

El patrón es universal: los cuatro proveedores principales asumen que tienes tus recursos etiquetados antes de poder hacer cualquier cosa útil con los datos de coste.

## Idea 3: El enforcement es lo que separa la intención del resultado

Definir un esquema de tags es la parte fácil. Que se cumpla es otra historia.

En AWS, las herramientas de enforcement operan a varios niveles. Tag Policies en AWS Organizations permiten definir qué valores son aceptables para cada clave de tag y reportar — u opcionalmente enforcar — el cumplimiento. AWS Config con la regla `required_tags` identifica recursos que no tienen tags con claves específicas y los marca como no-compliant. Y las Service Control Policies (SCPs) pueden bloquear directamente el lanzamiento de instancias que no incluyan tags obligatorios. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Enforcement"]

En Azure, el enfoque equivalente es Azure Policy. La documentación de FinOps recomienda empezar con políticas de auditoría e ir expandiendo la cobertura lentamente para no impactar negativamente a los equipos de ingeniería. [Fuente: Azure_FinOps_Documentation.md, "Cloud policy and governance"]

Hay una frase en la documentación de Azure FinOps que me parece clave: el tagging toma tiempo para aplicar, revisar y limpiar. Hay que esperar múltiples ciclos de limpieza después de que todo el mundo tenga visibilidad de los datos de coste. Muchas personas no se dan cuenta de que hay un problema hasta que tienen visibilidad. [Fuente: Azure_FinOps_Documentation.md, "Allocation / Building on the basics"]

Eso resume el dilema: la gente no etiqueta porque no ve el impacto. Y no ve el impacto porque no etiqueta.

## Idea 4: Empieza con lo mínimo, mide, y crece

No necesitas un esquema de 20 tags el primer día. La documentación de AWS recomienda empezar por la prioridad inmediata y crecer el esquema según la necesidad. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Building your tagging strategy"]

Un punto de partida pragmático para cost allocation: cuatro tags obligatorios.

| Tag | Pregunta que responde |
|-----|----------------------|
| `CostCenter` o equivalente | ¿De qué presupuesto sale esto? |
| `ApplicationId` | ¿Qué workload genera este coste? |
| `Owner` | ¿Quién es responsable? |
| `Environment` | ¿Es producción, staging, desarrollo? |

Este esquema mínimo ya te permite filtrar en Cost Explorer por equipo, por aplicación y por entorno. Es suficiente para responder la pregunta del fin de mes.

Después, mide la efectividad. Puedes trackear el porcentaje de gasto atribuido vs. no atribuido usando las herramientas nativas: Cost Explorer en AWS, Cost Analysis en Azure, Billing Reports en GCP. Si tienes un 40% de gasto sin tags, sabes exactamente dónde enfocar la siguiente iteración. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Measuring tagging effectiveness and driving improvements"]

Y gestiona el esquema en un repositorio con control de versiones accesible a todos los stakeholders. Nada de esquemas en wikis que nadie actualiza. [Fuente: Best_Practices_for_Tagging_AWS_Resources_AWS_Whitepaper.pdf, "Defining and publishing a tagging schema"]

## Cierre

No me interesa vender el tagging como algo excitante. No lo es. Es trabajo repetitivo, requiere consenso entre equipos, y los resultados no son instantáneos.

Pero sin tags, FinOps es un ejercicio teórico. Puedes tener los mejores dashboards, los recommenders más avanzados, las reservas mejor negociadas. Si no sabes a quién atribuir el coste, no puedes responsabilizar. Si no puedes responsabilizar, no puedes optimizar. Y si no puedes optimizar con datos, estás adivinando.

El tagging es el paso cero. No es el más vistoso, pero es el que hace posible todo lo demás.

La próxima vez que alguien en tu organización proponga un proyecto de optimización de costes cloud, hazle una sola pregunta: ¿qué porcentaje de nuestro gasto tiene tags de cost allocation?

Si no lo sabe, ya tienes el primer proyecto.

---

## Fuentes

- **Best Practices for Tagging AWS Resources** — AWS Whitepaper (proyecto base)
- **Tagging AWS Resources and Tag Editor** — AWS User Guide (proyecto base)
- **Azure FinOps Documentation** — Microsoft Cloud, FinOps Framework implementation (proyecto base)
- **Azure Cost Management and Billing** — Microsoft Docs, Tags and cost allocation (proyecto base)
- **GCP FinOps Hub** — Google Cloud Billing Docs (proyecto base)
- **GCP What is Cloud FinOps** — Google Cloud Learn (proyecto base)
- **GCP Well-Architected Cost Optimization** — Google Cloud Architecture Framework (proyecto base)
- **OCI FinOps Documentation** — Oracle Cloud Infrastructure Docs (proyecto base)

---

*Serie: FinOps a lo LenOps*
*Licencia: CC BY-NC-SA 4.0*
