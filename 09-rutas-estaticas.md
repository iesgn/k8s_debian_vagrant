# Rutas estáticas

En este despliegue utilizamos el plugin básico de red de kubernetes,
mediante el cual los contenedores se ejecutan simplemente en un puente
de red en cada nodo. Para que se puedan comunicar entre sí, añadimos
rutas estáticas al resto de los contenedores de los otros nodos:

## Nodo1

```
ip r add 10.200.13.0/24 via 10.0.10.13 
ip r add 10.200.14.0/24 via 10.0.10.14
```

## Nodo2

```
ip r add 10.200.12.0/24 via 10.0.10.12
ip r add 10.200.14.0/24 via 10.0.10.14
```

## Nodo3

```
ip r add 10.200.12.0/24 via 10.0.10.12
ip r add 10.200.13.0/24 via 10.0.10.13
```
