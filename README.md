# Examne_1-Avaliaci-n_IagoDaPonteGonz-lez

**1. Explica métodos para 'abrir' unha consola/shell a un contenedor en execución.**

Para poder abrir unha consola dun contenedor hai varios metodos. Se o facemos dende VSC, no apartado de docker previamente instalado faremos click dereito e podemos ver o apartado de `Attach Shell`. Se o facemos dende comandos na consola poderemos facer `docker exec -it (nome do contenedor) /bin/bash` en caso de ser ubuntu ou `/bin/sh` en caso de alpine.

**2. No contenedor anterior (en execución), qué opciones tes que ter usado ó arrincalo para poder interactuar coas súas entradas e salidas**

`i` para manter a terminal aberta e a `t` para crear unha terminal propia.
`docker run -it (nome do contenedor)`

**3. Cómo sería un ficheiro docker-compose para que dous contenedores se comuniquen entre si nunha rede só deles?**

```
version: '3'

services:
  asir_bind9:
    container_name: Exame_bind9
    image: ubuntu/bind9
    platform: linux/amd64
    ports:
      - "56:53"  # Exponer el puerto 56 para que otros puedan hacer consultas DNS
    networks:
      bind9_subnet:
        ipv4_address: 172.16.0.2  # IP estática del servidor DNS
    volumes:
      - ./conf:/etc/bind/  # Montar la configuración del servidor BIND
      - ./zonas:/var/lib/bind/  # Montar el directorio de zonas
    restart: unless-stopped  # Reiniciar el contenedor si falla o si Docker se reinicia

  cliente:
    container_name: Exame_alpine
    image: alpine
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 172.16.0.2  # El cliente usará el DNS del servidor asir_bind9
    networks:
      bind9_subnet:
        ipv4_address: 172.16.0.3  # IP estática del cliente en la misma subred
    restart: unless-stopped  # Asegurarse de que el cliente también se reinicie si falla

networks:
  bind9_subnet:
    driver: bridge  # Tipo de red 'bridge'
    ipam:
     
      config:
        - subnet: "172.16.0.0/16"  # Definir el rango de subred
          ip_range: "172.16.0.0/24"  # Rango de IPs para los contenedores
          gateway: "172.16.0.254"  # Puerta de enlace de la red
```

**4. Qué tes que engadir ó ficheiro anterior para que un contenedor teña unha IP fixa?**

No apartado de `networks` dentro dos nosos contenedores poderemos especificar unha IP estática.  

```
networks:
      bind9_subnet:
        ipv4_address: 192.28.5.1  # IP estática do servidor DNS
```

**5. Qué comando de consola podo usar para sabe-las ips dos contenedores anteriores?**

O comando para saber as IPs dos contenedores é `docker network inspect (nome do contenedor)`. E se queremos ver se están na mesma rede `docker network inspect (rede)`. 

**6. Cál é a funcionalidade do apartado "ports" en docker compose?**

O apartado de `ports` serve para mapear os portos do contenedor aos portos da máquina host.

```
ports:
      - "56:53"  # Expoñer o porto 56 para que outros poidan facer consultas DNS
```

**7. Para qué serve o rexistro CNAME? Pon un exemplo**

Serve para asignar un nome de dominio DNS con alias a outro con nome canónico principal  
`alias	IN CNAME	Exame`

**8. Cómo podo facer para que a configuración dun contenedor DNS non se borre se creo outro contenedor?**

No apartado de `volumes` se gardará a configuración que fixeramos nese contenedor.  

```
volumes:
      - ./conf:/etc/bind/  # Montar a configuración do servidor BIND
      - ./zonas:/var/lib/bind/  # Montar o directorio de zonas
```

**9. Engade unha zoa tendaelectronica.int no teu docker DNS que teña:**
- www á IP 172.16.0.1  
- owncloud sexa un CNAME de www  
- un rexistro de texto có contido "1234ASDF"  
**Comproba que todo funciona có comando "dig"**  
**Mostra nos logs que o servicio funciona ben usando a saída da terminal ó levantar o compose ou có comando "docker logs [nomeContenedorOuID]"**  
(o apartado 9 realízase na máquina virtual)


Aquí mostraremos a configuración da nosa zona tendaelectronica.in, situada dentro do directorio de zonas, cos párametros requeridos no enunciado:

```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.tendaelectronica.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.tendaelectronica.int.
ns		IN A		172.16.0.2
test	IN A		172.16.0.4
www		IN A		172.16.0.1
owncloud	IN CNAME	www
@	IN TXT		1234ASDF
``` 

Facendo o comando `dig ns.tendaelectronica.int` aparece o seguinte: 

```
; <<>> DiG 9.18.27 <<>> ns.tendaelectronica.int
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47030
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: f2ad232d308793f70100000067378e98d2c3186ae1b0df0b (good)
;; QUESTION SECTION:
;ns.tendaelectronica.int.	IN	A

;; ANSWER SECTION:
ns.tendaelectronica.int. 38400	IN	A	172.16.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Fri Nov 15 18:10:32 UTC 2024
;; MSG SIZE  rcvd: 96
```

Posteriormente se facemos o comando `docker logs Exame_bind9` aparece o seguinte:

```
15-Nov-2024 18:09:09.607 BIND 9 is maintained by Internet Systems Consortium,
15-Nov-2024 18:09:09.607 Inc. (ISC), a non-profit 501(c)(3) public-benefit 
15-Nov-2024 18:09:09.607 corporation.  Support and training for BIND 9 are 
15-Nov-2024 18:09:09.607 available at https://www.isc.org/support
15-Nov-2024 18:09:09.607 ----------------------------------------------------
15-Nov-2024 18:09:09.607 found 1 CPU, using 1 worker thread
15-Nov-2024 18:09:09.607 using 1 UDP listener per interface
15-Nov-2024 18:09:09.612 DNSSEC algorithms: RSASHA1 NSEC3RSASHA1 RSASHA256 RSASHA512 ECDSAP256SHA256 ECDSAP384SHA384 ED25519 ED448
15-Nov-2024 18:09:09.612 DS algorithms: SHA-1 SHA-256 SHA-384
15-Nov-2024 18:09:09.612 HMAC algorithms: HMAC-MD5 HMAC-SHA1 HMAC-SHA224 HMAC-SHA256 HMAC-SHA384 HMAC-SHA512
15-Nov-2024 18:09:09.612 TKEY mode 2 support (Diffie-Hellman): yes
15-Nov-2024 18:09:09.612 TKEY mode 3 support (GSS-API): yes
15-Nov-2024 18:09:09.612 loading configuration from '/etc/bind/named.conf'
15-Nov-2024 18:09:09.616 unable to open '/etc/bind/bind.keys'; using built-in keys instead
15-Nov-2024 18:09:09.616 looking for GeoIP2 databases in '/usr/share/GeoIP'
15-Nov-2024 18:09:09.616 using default UDP/IPv4 port range: [32768, 60999]
15-Nov-2024 18:09:09.616 using default UDP/IPv6 port range: [32768, 60999]
15-Nov-2024 18:09:09.616 listening on IPv4 interface lo, 127.0.0.1#53
15-Nov-2024 18:09:09.616 listening on IPv4 interface eth0, 172.16.0.2#53
15-Nov-2024 18:09:09.616 IPv6 socket API is incomplete; explicitly binding to each IPv6 address separately
15-Nov-2024 18:09:09.616 listening on IPv6 interface lo, ::1#53
15-Nov-2024 18:09:09.621 generating session key for dynamic DNS
15-Nov-2024 18:09:09.621 sizing zone task pool based on 1 zones
15-Nov-2024 18:09:09.621 none:99: 'max-cache-size 90%' - setting to 7148MB (out of 7942MB)
15-Nov-2024 18:09:09.621 set up managed keys zone for view _default, file 'managed-keys.bind'
15-Nov-2024 18:09:09.621 configuring command channel from '/etc/bind/rndc.key'
15-Nov-2024 18:09:09.621 command channel listening on 127.0.0.1#953
15-Nov-2024 18:09:09.621 configuring command channel from '/etc/bind/rndc.key'
15-Nov-2024 18:09:09.621 command channel listening on ::1#953
15-Nov-2024 18:09:09.621 not using config file logging statement for logging due to -g option
15-Nov-2024 18:09:09.621 managed-keys-zone: loaded serial 0
15-Nov-2024 18:09:09.621 zone tendaelectronica.int/IN: loaded serial 10000002
15-Nov-2024 18:09:09.621 all zones loaded
15-Nov-2024 18:09:09.621 running
15-Nov-2024 18:09:33.343 missing expected cookie from 1.1.1.1#53
```
