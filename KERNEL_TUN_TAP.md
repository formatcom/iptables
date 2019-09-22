REF: https://www.kernel.org/doc/Documentation/networking/tuntap.txt

TUN/TAP proporciona recepcion de paquetes y transmicion para programas
en espacio de usuario. Puedes ver esto como un simple punto a punto o
un dispositivo Ethernet, que, en lugar de recibir paquetes en un medio
fisico, los recibe un programa de espacio de usuario y en lugar de enviar
paquetes a traves de medios fisicos los escribe en el programa de espacio
de usuario.

Para IP Packets -> TUN.

Para Ethernet Frame -> TAP.

![packet](https://www.laneye.com/network/ethernet-network-packet-holding-an-ip-packet.gif)
