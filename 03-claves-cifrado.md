# Claves de cifrado

Para cifrar los datos sensibles del clúster

## Generación de la clave:

En nuestro equipo cliente generamos la clave y la codificamos en base64

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## Fichero de configuración

```
cat > Config/encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

## Distribución en los nodos controladores

```
for instance in controller{1..3}; do
  scp Config/encryption-config.yaml root@${instance}:~/
done
```
