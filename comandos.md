# Comandos utilizados en el taller de subnetting

## 1. Verificación del sistema y herramientas
* `lsb_release -a` y `ip -V`: muestran la versión de Ubuntu (`26.04.1 LTS`) y de `iproute2` (`6.19.0`)[cite: 1, 24].
* `sudo apt update` y `sudo apt install -y iproute2 iputils-ping procps`: actualizan la lista de paquetes e instalan/verifican las herramientas necesarias para redes, `ping` y `sysctl`[cite: 1, 5, 24, 25].
* `ip -br addr` e `ip -4 route`: muestran un resumen de las interfaces y direcciones IPv4 del equipo (como `enp0s3` con `10.0.2.15/24`) y su tabla de rutas IPv4[cite: 1, 5, 6, 25, 26].

## 2. Creación de namespaces y cables virtuales (veth)
* `sudo ip netns add aula-a` (y `aula-b`, `aula-r`): crea un entorno de red aislado en Linux con su propia red, rutas y vecinos[cite: 1, 7, 8, 19].
* `sudo ip netns list`: muestra los namespaces creados[cite: 1, 7, 8].
* `sudo ip link add veth-a type veth peer name veth-ra` (y `veth-b` con `veth-rb`): crea un par de interfaces Ethernet virtuales conectadas entre sí como los dos extremos de un cable virtual[cite: 1, 8, 9, 19].
* `sudo ip link set veth-a netns aula-a`: mueve la interfaz `veth-a` dentro del namespace `aula-a` (igual con las demás interfaces hacia su respectivo namespace)[cite: 1, 9, 19].

## 3. Direcciones IP, activar interfaces y pruebas previas
* `sudo ip -n aula-a addr add 192.168.10.10/26 dev veth-a`: asigna la IP `192.168.10.10/26` a `veth-a` dentro de `aula-a` (igual se asignaron `192.168.10.1/26` en `veth-ra`, `192.168.10.70/26` en `veth-b` y `192.168.10.65/26` en `veth-rb`)[cite: 1, 10, 11, 19].
* `sudo ip -n aula-a link set lo up` y `sudo ip -n aula-a link set veth-a up`: encienden la interfaz de loopback y la interfaz `veth` dentro del namespace (lo mismo se hizo con `lo` y las demás `veth`)[cite: 1, 11, 27].
* `sudo ip -n aula-a -br addr` e `ip -n aula-a route`: muestran las interfaces con sus IP y su estado (`UP`), y la tabla de rutas del namespace[cite: 1, 11, 13, 28].
* `sudo ip netns exec aula-a ping -c 3 192.168.10.1`: envía 3 pings desde `aula-a` hacia su gateway local (responde si están en la misma red y enlace activo)[cite: 1, 12, 28].

## 4. Gateways, reenvío IPv4, escenario /27 y limpieza
* `sudo ip -n aula-a route add default via 192.168.10.1` (y en `aula-b` hacia `192.168.10.65`): agrega la ruta por defecto para enviar hacia el router el tráfico de otras subredes[cite: 1, 14, 29].
* `sudo ip netns exec aula-r sysctl -w net.ipv4.ip_forward=1` (y `=0` para probar el fallo): activa o desactiva el reenvío de paquetes IPv4 entre interfaces dentro del router `aula-r`[cite: 1, 15, 18, 29].
* `sudo ip -n aula-a route get 192.168.10.70` y `sudo ip -n aula-a neigh`: muestran qué ruta y siguiente salto se usa hacia el destino y la dirección MAC del vecino (gateway) en la tabla ARP[cite: 1, 17, 30].
* **Escenario `/27`:** Se repitió el armado con `10.20.30.0/27` (`10.20.30.10/27` y gateway `10.20.30.1/27`) y `10.20.30.32/27` (`10.20.30.40/27` y gateway `10.20.30.33/27`), configurando rutas y `net.ipv4.ip_forward=1`[cite: 1, 20, 33, 34].
* `sudo ip netns del aula-a` (y `aula-b`, `aula-r`): elimina los namespaces creados al terminar la práctica y libera sus entornos virtuales[cite: 1, 19, 32].
