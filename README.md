# IPTABLES

REF: https://wiki.archlinux.org/index.php/Iptables_(Espa%C3%B1ol)
iptables se usa para inspeccionar, modificar, reenviar, redirigir
y/o eliminar paquetes de red de IP. El codigo para filtrar paquetes
de IP ya esta integrado en el kernel y esta organizado en una
coleccion de tablas, cada una con un proposito especifico. Las
tablas estan formadas por un conjuntos de cadenas predefinidas, y
las cadenas contienen reglas que se recorren en orden consecurivo.
Cada regla consiste en una condicion asociada a una aventual accion
(llamada target) que si se cumple, se ejecuta la regla; esto es, se
aplican las reglas si las condiciones coinciden. iptables es la
utilidad de usuario que le permite trabajar con estas cadenas/reglas.
La mayoria de los usuarios nuevos consideran que las complejidades del
enrutamiento IP de linux son bastantes desalentadoras, pero, en la
practica, los casos de uso mas comunes (NAT y/o cortafuegos basicos
de internet) son considerablemente menos complejos.

![tables traverse](https://www.frozentux.net/iptables-tutorial/images/tables_traverse.jpg)
