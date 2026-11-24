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
