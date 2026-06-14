# Savings Plans vs Reserved Instances en AWS: el error que nadie te cuenta hasta que lo ves en la factura

La escena se repite. Un equipo de infraestructura revisa el uso de EC2 en Cost Explorer, ve que tiene cargas estables corriendo 24/7, y toma la decisión "obvia": comprar Reserved Instances. Tres meses después, alguien migra una carga a Fargate, otro equipo cambia de familia de instancias, y la RI sigue ahí, facturando puntualmente cada hora, sin que nadie la use.

No es un problema de descuento. Es un problema de rigidez.

Y lo peor: nadie te avisa antes de firmar.

---

## Lo que una Reserved Instance realmente compromete

Una RI no es una máquina virtual. Es un compromiso de pago por un tipo de instancia específico, en una región (o zona) específica, por un plazo de uno o tres años. A cambio, obtienes una tarifa horaria descontada frente a On-Demand. [Fuente: AWS_Billing_User_Guide.pdf, sección Reserved Instances, p.392]

Hasta ahí, la teoría. Ahora la letra pequeña.

### Zonal RI: descuento atado a una Availability Zone

Cuando compras una Zonal RI, el descuento se aplica solo al uso del mismo tipo de instancia en la misma Availability Zone donde compraste la reserva. Si tu equipo lanza instancias en us-west-2a y la RI fue comprada en us-west-2b, no aplica. Y hay un matiz que mucha gente pasa por alto: la ubicación física real de una AZ puede variar entre cuentas. Lo que es us-west-2a para la cuenta A puede ser una ubicación diferente para la cuenta B. [Fuente: AWS_Billing_User_Guide.pdf, sección Billing examples for specific services, p.381]

A cambio de esa rigidez, las Zonal RIs reservan capacidad. Si necesitas garantía de que tu instancia va a tener slot disponible, esto tiene valor. Pero si no necesitas esa garantía, estás pagando por una restricción que no te sirve.

### Regional RI: más flexibilidad, menos garantías

Las Regional RIs no reservan capacidad. A cambio, ofrecen dos cosas: flexibilidad de Availability Zone (el descuento aplica en cualquier AZ de la región) y flexibilidad de tamaño de instancia dentro de la misma familia, pero solo en Linux/Unix con tenencia por defecto. [Fuente: AWS_Billing_User_Guide.pdf, sección Regional Reserved Instances, p.392-393]

Esto es mejor que la Zonal, pero sigue anclado a una familia de instancias y a una región concreta. Si cambias de m5 a m6i, la RI no te sigue.

---

## Savings Plans: comprometer gasto, no instancias

Aquí está el cambio de modelo mental. Un Savings Plan no te compromete con un tipo de instancia. Te compromete con un nivel de gasto por hora. Tú dices "voy a gastar X dólares por hora en compute" y AWS te aplica tarifas descontadas automáticamente. [Fuente: AWS_Billing_User_Guide.pdf, sección Savings Plans, p.394]

La diferencia fundamental es lo que cubre. Un Compute Savings Plan aplica a EC2, Fargate y Lambda. No importa la familia de instancias, la región, el sistema operativo ni la tenencia. Si mañana migras de EC2 a contenedores en Fargate, el descuento se mueve contigo. [Fuente: AWS_Billing_User_Guide.pdf, sección Savings Plans, p.394]

### Cómo aplica AWS el descuento

Este detalle importa y casi nadie lo menciona: AWS calcula los cargos de EC2, Fargate y Lambda agregando todo el uso que no está cubierto por Reserved Instances, y luego aplica las tarifas de Savings Plans empezando por el descuento más alto. [Fuente: AWS_Billing_User_Guide.pdf, sección Calculating Costs with Savings Plans, p.394]

Es decir, AWS optimiza automáticamente a tu favor. No tienes que asignar manualmente el SP a un recurso. El sistema busca dónde puede ahorrarte más y aplica ahí primero.

---

## La trampa de la asignación en organizaciones multi-cuenta

Tanto las RIs como los SPs se aplican primero a la cuenta que los compró. Solo después de satisfacer el uso de esa cuenta, el descuento se comparte con otras cuentas de la organización. [Fuente: AWS_Billing_User_Guide.pdf, sección RI/SP allocation, pp.393-394]

Aquí es donde muchos equipos pierden dinero sin saberlo. AWS ofrece tres modos de compartir descuentos:

| Modo | Comportamiento | Cuándo usarlo |
|------|----------------|---------------|
| Open sharing | Descuento disponible para todas las cuentas activas en la org (por defecto) | Optimización de costes a nivel org |
| Prioritized group | Primero cuenta propietaria, luego grupo definido, luego resto de la org | Balance entre control de grupo y optimización global |
| Restricted group | Descuentos solo dentro de grupos definidos, sin compartir fuera | Asignación estricta por unidad de negocio |

[Fuente: AWS_Billing_User_Guide.pdf, sección Sharing Modes, pp.383-384]

El modo Restricted suena bien para chargeback limpio, pero tiene un coste directo: si el grupo no consume toda la reserva, esas horas se pierden. No se redistribuyen. Es una decisión de negocio, no solo técnica. [Fuente: AWS_Billing_User_Guide.pdf, sección Important Considerations, p.386]

---

## El error más caro: comprar sin analizar primero

El Well-Architected Framework de AWS lo dice directamente en COST 7: antes de comprometer, analiza cada componente de la carga de trabajo. Determina si va a funcionar durante periodos extendidos (candidato a compromiso) o si es dinámica y de corta duración (candidata a Spot u On-Demand). Usa las recomendaciones de Cost Explorer para este análisis. [Fuente: AWS_WellArchitected_Framework.pdf, COST 7, p.91]

El AWS FinOps Agent va un paso más allá, agrupando las oportunidades de optimización en categorías: rightsizing, recursos idle, Savings Plans y Reserved Instances, y migración de arquitectura. Las recomendaciones de SP/RI se basan en tu cobertura actual y el ahorro estimado de compromisos adicionales. [Fuente: AWS_FinOps_Agent_preview_User_Guide.pdf, sección Optimization recommendations, p.29]

Pero hay una advertencia que pocos leen: las recomendaciones de ahorro se basan en precios públicos. Los montos de compra recomendados son precisos, pero los ahorros proyectados pueden ser mayores que los ahorros reales. [Fuente: AWS_Billing_User_Guide.pdf, sección RI/SP recommendations, p.336]

Traducción: la calculadora te dice cuánto comprar, pero el ahorro que muestra es optimista. Si tu uso real varía, el ahorro real será menor.

---

## Un detalle que nadie menciona: la factura después de cerrar la cuenta

Si cierras una cuenta de AWS que tiene Reserved Instances activas, las facturas de la RI continúan hasta que expire el periodo de reserva. Si tienes Savings Plans, las facturas continúan hasta que se complete el plazo del plan. [Fuente: AWS_Billing_User_Guide.pdf, sección Charges received after account closure, p.71]

Esto no es un bug. Es el contrato. Y es un riesgo que hay que incluir en cualquier decisión de compra de compromisos.

---

## Entonces, qué usar y cuándo

No voy a darte una tabla mágica con porcentajes de descuento. Lo que sí puedo decirte, basándome en cómo funcionan estos instrumentos según la documentación:

**Savings Plans (Compute)** son la opción por defecto para la mayoría de escenarios. Si tu organización usa EC2, y además tiene cargas en Fargate o Lambda, el SP cubre todo. Si cambias de familia de instancias o de región, el descuento se adapta. Si no tienes certeza absoluta de qué instancias vas a correr dentro de un año, el SP te da margen.

**Reserved Instances** tienen sentido en dos casos específicos: cuando necesitas reserva de capacidad garantizada (Zonal RI), o cuando tu carga es tan estable y predecible que puedes asumir la rigidez a cambio de un mayor descuento potencial.

Lo que no tiene sentido, en ningún caso, es comprar compromisos sin haber respondido estas preguntas:

1. ¿Tengo al menos 30 días de datos de uso estable en Cost Explorer?
2. ¿He analizado si mis cargas están en la familia de instancias correcta (rightsizing)?
3. ¿Mi organización tiene una política clara de cómo compartir descuentos entre cuentas?
4. ¿Quién monitoriza la utilización de los compromisos después de la compra?

Si la respuesta a cualquiera de esas preguntas es "no sé", no compres todavía. Primero optimiza el uso. Después compromete el gasto.

Ese es el orden. No al revés.

---

## Fuentes

- AWS_Billing_User_Guide.pdf — Secciones: Reserved Instances (pp. 380-394), Savings Plans (pp. 394-395), RI/SP discount sharing (pp. 383-386), Charges after account closure (p. 71), RI/SP recommendations (p. 336)
- AWS_WellArchitected_Framework.pdf — COST 7: Modelos de precios (p. 91)
- AWS_FinOps_Agent_preview_User_Guide.pdf — Optimization recommendations (p. 29)

---

*Serie "FinOps a lo LenOps" — Mi forma de entender FinOps: clara, práctica y sin humo.*

*Licencia: CC BY-NC-SA 4.0*
