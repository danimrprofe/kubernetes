# kubernetes
Es un sistema de gestión y orquestación de conenedores. Ejecuta y gestiona aplicaciones
aplicaciones containerizadas sobre un cluster.
# Cluster
Un cluster es un conjunto de nodos. cada uno de estos nodos puede ser:
* Una máquina real física
* Una máquina virtual
# Nodos worker
Los nodos worker son máquinas que ejecutan aplicaciones dentro de contenedores. 
Ejecutan, monitorizan y proveen de servicios a las aplicaciones a través de diferentes componentes:
* Docker (u otro sistema) ejecuta los contenedores
* Los kubeletes se comunican con la API del servidor y gestionan los contenedores en su propio nodo
* Un proxy de red balancea el tráfico entre los diferente componentes
