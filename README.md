# Ecosistema de Interoperabilidad para SIMSADI® 

Este repositorio contiene los archivos necesarios para poder desplegar el servicio Mirth Connect de la empresa NextGen y un servidor HAPI FHIR para el Proyecto SIMSADI UV: Ecosistemas de Interoperabilidad

Cabe destacar que este repositorio esta enfocado para el usuario final del módulo de los servicios (en otras palabras, te ayuda a implementar estos servicios, por lo tanto ciertas librerías y extensiones pueden que no estén incluídas en esta instalación). Si estás en busca del proyecto fuente de cada servicio,  puedes consultar en los sitios oficiales. Mirth connect en Github: https://github.com/nextgenhealthcare/connect. HAPI-FHIR en Github: https://github.com/hapifhir/hapi-fhir-jpaserver-starter


## Pre-requisitos de instalación.
- Ordenador con los siguientes especificaciones:
  - Hardware x86, los modelos actuales de Intel/AMD deberían soportarlo.
  - RAM: 8 GB mínimo
  - Almacenamiento: 30 GB mínimo dedicado
  - Sistema Operativo Ordenador: Windows 10
- Maquina Virtual con SO `CentOS Linux 7.6.1810 (x86_64)` http://ftp.iij.ad.jp/pub/linux/centos-vault/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
- Docker Engine

## Guía de implementación
`docker-compose.yml`

```javascript
version: "3.1"
services:
  mc:
    image: nextgenhealthcare/connect
    environment:
      - DATABASE=postgres
      - DATABASE_URL=jdbc:postgresql://172.16.255.61:5433/mirthdb
      - DATABASE_MAX_CONNECTIONS=20
      - DATABASE_USERNAME=mirthdb
      - DATABASE_PASSWORD=mirthdb
      - DATABASE_MAX_RETRY=2
      - DATABASE_RETRY_WAIT=10000
      - KEYSTORE_STOREPASS=Mati.123
      - KEYSTORE_KEYPASS=Mati.123
      - VMOPTIONS=-Xmx512m
    ports:
      - 8180:8080/tcp
      - 8443:8443/tcp
    depends_on:
      - db
  db:
    image: postgres
    environment:
      - POSTGRES_USER=mirthdb
      - POSTGRES_PASSWORD=mirthdb
      - POSTGRES_DB=mirthdb
    ports:
      - 5433:5432
```
