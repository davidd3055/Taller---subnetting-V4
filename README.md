# Taller---subnetting-V4

# Taller práctico de subnetting en Ubuntu

## 1. Objetivo
Diseñar dos subredes IPv4 (`/26`) a partir de `192.168.10.0/24`, representarlas mediante tres Network Namespaces en Ubuntu (`aula-a`, `aula-b` y `aula-r`) conectados con parejas `veth` y permitir la comunicación entre ambas subredes a través de un router virtual con reenvío IPv4 (`ip_forward`) habilitado.

## 2. Diseño de subredes (/26)
* **Máscara:** `255.255.255.192` (`/26`)
* **Direcciones por bloque:** 64 totales (62 utilizables)
* **Subred A:** `192.168.10.0/26` | Rango útil: `192.168.10.1` a `192.168.10.62` | Broadcast: `192.168.10.63`
* **Subred B:** `192.168.10.64/26` | Rango útil: `192.168.10.65` a `192.168.10.126` | Broadcast: `192.168.10.127`

| Dispositivo | Namespace | Interfaz | IP / Prefijo | Función |
| :--- | :--- | :--- | :--- | :--- |
| PC-A | `aula-a` | `veth-a` | `192.168.10.10/26` | Host de la subred A |
| Router, lado A | `aula-r` | `veth-ra` | `192.168.10.1/26` | Gateway de la subred A |
| PC-B | `aula-b` | `veth-b` | `192.168.10.70/26` | Host de la subred B |
| Router, lado B | `aula-r` | `veth-rb` | `192.168.10.65/26` | Gateway de la subred B |

## 3. Topología
```text
+-----------------------+                      +-----------------------+
| PC-A                  |                      | PC-B                  |
| Namespace: aula-a     |                      | Namespace: aula-b     |
| Interfaz: veth-a      |                      | Interfaz: veth-b      |
| IP: 192.168.10.10/26  |                      | IP: 192.168.10.70/26  |
+-----------+-----------+                      +-----------+-----------+
            |                                              |
            | enlace veth 1                                | enlace veth 2
            |                                              |
+-----------+----------------------------------------------+-----------+
| ROUTER: aula-r (IPv4 forwarding)                                   |
| Interfaz veth-ra: 192.168.10.1/26 (Gateway A)                        |
| Interfaz veth-rb: 192.168.10.65/26 (Gateway B)                       |
+----------------------------------------------------------------------+

4. ProcedimientoPreparación en Ubuntu: Se verificó Ubuntu 26.04.1 LTS (resolute) y iproute2 (6.19.0), confirmando iproute2, iputils-ping y procps, y revisando las interfaces reales (enp0s3 con 10.0.2.15/24) y rutas sin modificarlas.   Namespaces y enlaces veth: Se crearon los namespaces aula-a, aula-b y aula-r y dos parejas virtuales (veth-a con veth-ra, y veth-b con veth-rb), moviendo cada extremo a su namespace correspondiente.   Direcciones IP y activación: Se asignaron las IP /26 a cada interfaz y se levantaron (up) las interfaces lo y veth.   Gateways y reenvío IPv4: Se configuraron las rutas por defecto hacia 192.168.10.1 en aula-a y 192.168.10.65 en aula-b, y se habilitó net.ipv4.ip_forward=1 en aula-r.   Escenario /27 (Reto 3): Tras limpiar la topología /26, se recrearon los entornos con las subredes 10.20.30.0/27 (10.20.30.10/27 y gateway 10.20.30.1/27) y 10.20.30.32/27 (10.20.30.40/27 y gateway 10.20.30.33/27), verificando rutas y conectividad.

## 5. Pruebas y resultados
* **Antes de rutas y `ip_forward` (`/26`):** Los pings hacia los gateways locales (`192.168.10.1` desde `aula-a` y `192.168.10.65` desde `aula-b`) respondieron con 0% de pérdida, pero el ping de `aula-a` a `192.168.10.70` falló por no existir ruta hacia la otra subred[cite: 1, 28].
* **Con gateways y `net.ipv4.ip_forward=1` (`/26`):** Las pruebas entre `192.168.10.10` y `192.168.10.70` funcionaron en ambos sentidos con 0% de pérdida y TTL `63`[cite: 1, 29]. Con `ip -n aula-a route get 192.168.10.70` e `ip -n aula-a neigh` se verificó el siguiente salto (`192.168.10.1`) y la MAC del gateway (`8e:65:00:57:ec:5e`)[cite: 1, 30].
* **Escenario `/27`:** Se configuraron las redes `10.20.30.0/27` (hosts útiles `10.20.30.1` a `10.20.30.30`, broadcast `10.20.30.31`) y `10.20.30.32/27` (hosts útiles `10.20.30.33` a `10.20.30.62`, broadcast `10.20.30.63`), obteniendo conectividad bidireccional entre `10.20.30.10` y `10.20.30.40` con 0% de pérdida y TTL `63`[cite: 1, 34].

## 6. Diagnóstico de falla (`net.ipv4.ip_forward=0`)
* **Qué dejó de funcionar:** El ping entre subredes (de `192.168.10.10` a `192.168.10.70`) presentó 100% de pérdida de paquetes[cite: 1, 31].
* **Qué siguió funcionando:** El ping de `aula-a` hacia su gateway local (`192.168.10.1`) siguió respondiendo con 0% de pérdida porque está en la misma subred y enlace directo[cite: 1, 12, 31].
* **Por qué ocurrió y cómo se solucionó:** Al poner `net.ipv4.ip_forward=0` en `aula-r`, el router dejó de reenviar paquetes IPv4 entre sus interfaces[cite: 1, 18]. Se corrigió ejecutando `sudo ip netns exec aula-r sysctl -w net.ipv4.ip_forward=1`[cite: 1, 18].

## 7. Preguntas del Reto 1 y Conclusiones
* **¿Por qué `192.168.10.10/26` pertenece a `192.168.10.0/26`?** Porque entra en el rango de esa subred (`192.168.10.0` a `192.168.10.63`)[cite: 1, 4].
* **¿Por qué `192.168.10.70/26` pertenece a `192.168.10.64/26`?** Porque entra en el rango de ese bloque (`192.168.10.64` a `192.168.10.127`)[cite: 1, 4].
* **Máscara decimal de `/26` y broadcasts:** La máscara es `255.255.255.192`; el broadcast de A es `192.168.10.63` y el de B es `192.168.10.127`[cite: 1, 4].
* **Elección de gateways (`192.168.10.1` y `192.168.10.65`):** Son direcciones IP utilizables dentro de cada subred asignadas a las interfaces del router (`aula-r`) conectadas a cada red[cite: 1, 4, 5].
* **Conclusiones:** Dividir una red en subredes separa rangos de direcciones, los enlaces `veth` y las IP permiten hablar dentro de cada red local, y las rutas por defecto junto con `ip_forward=1` hacen posible que el router comunique subredes distintas[cite: 1, 14, 15, 22].
