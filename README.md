# 🛡️ Packet Tracer: Desafío de Implementación de ACL IPv4

<div align="center">

**Laboratorio CISCO - Access Control Lists (ACL) Avanzadas**

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet_Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)](https://www.netacad.com)
[![ACL Protocol](https://img.shields.io/badge/Protocol-ACL_IPv4-00A86B?style=for-the-badge)](https://www.cisco.com/)
[![CCNA](https://img.shields.io/badge/Certification-CCNA-blue?style=for-the-badge)](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/associate/ccna.html)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

[🎯 Objetivos](#-objetivos) • 
[📊 Tabla de Direcciones](#️-tabla-de-asignación-de-direcciones) • 
[📋 Escenario](#️-antecedentesescenario) • 
[⚙️ Configuración](#️-configuración-paso-a-paso) • 
[🔍 Verificación](#️-verificación-y-preguntas) • 
[👨‍💻 Autor](#️-autor)

</div>

---

## 📋 Descripción del Proyecto
Este laboratorio avanzado de Cisco Packet Tracer implementa **Access Control Lists (ACL) IPv4** para controlar el tráfico en una red empresarial compleja. Se configuran ACL estándar y extendidas, tanto numeradas como nombradas, para cumplir con requisitos específicos de seguridad y comunicación entre múltiples redes LAN y conexiones a Internet.

### 🎯 Objetivos del Laboratorio
- Configurar un router con ACL estándar nombrada
- Configurar un router con ACL extendida nombrada
- Implementar ACL extendidas para cumplir requisitos de comunicación específicos
- Configurar ACL para controlar acceso a líneas terminal VTY
- Aplicar ACL en interfaces con dirección apropiada
- Verificar el funcionamiento de todas las ACL configuradas

---

## 📊 Tabla de Asignación de Direcciones

| Dispositivo | Interfaz | Dirección IP | Máscara |
|-------------|----------|--------------|---------|
| **Branch** | G0/0/0 | 192.168.1.1 | /26 |
| **Branch** | G0/0/1 | 192.168.1.65 | /29 |
| **Branch** | S0/1/0 | 192.0.2.1 | /30 |
| **Branch** | S0/1/1 | 192.168.3.1 | /30 |
| **HQ** | G0/0/0 | 192.168.2.1 | /27 |
| **HQ** | G0/0/1 | 192.168.2.33 | /28 |
| **HQ** | S0/1/1 | 192.168.3.2 | /30 |
| **PC-1** | NIC | 192.168.1.10 | /26 |
| **PC-2** | NIC | 192.168.1.20 | /26 |
| **PC-3** | NIC | 192.168.1.30 | /26 |
| **Admin** | NIC | 192.168.1.67 | /29 |
| **Enterprise Web Server** | NIC | 192.168.1.70 | /29 |
| **Branch PC** | NIC | 192.168.2.17 | /27 |
| **Branch Server** | NIC | 192.168.2.45 | /28 |
| **Internet Users** | NIC | 198.51.100.218 | /24 |
| **External Web Server** | NIC | 203.0.113.73 | /24 |

---

## 📋 Antecedentes/Escenario
En este laboratorio, configurará ACL extendidas, con nombre estándar y con nombre extendido para cumplir los requisitos de comunicación especificados en una red empresarial que incluye oficinas centrales (HQ), sucursales (Branch) y conexión a Internet.

### 🌐 Topología de Red
```
               [Internet]
                   |
           198.51.100.0/24
                   |
               [ISP Router]
                   |
             192.0.2.0/30
                   |
            +------+------+
            |   Branch    |
            |   Router    |
            +------+------+
            |            |
    192.168.1.0/26  192.168.1.64/29
            |            |
        [PC-1,2,3]  [Admin, Enterprise WS]
                   |
             192.168.3.0/30
                   |
            +------+------+
            |     HQ      |
            |   Router    |
            +------+------+
            |            |
    192.168.2.0/27  192.168.2.32/28
            |            |
    [Branch PC]      [Branch Server]
```

---

## ⚙️ Configuración Paso a Paso

### Paso 1: Verificar Conectividad Inicial
Antes de configurar ACL, verificar que todos los hosts pueden hacer ping a todos los demás hosts en la topología.

```cisco
! Ejemplo de pruebas de conectividad iniciales
Branch_PC> ping 192.168.1.10    # Hacia PC-1 en Branch LAN
Branch_PC> ping 192.168.2.45    # Hacia Branch Server en HQ
Internet_User> ping 192.168.1.70 # Hacia Enterprise Web Server
```

### Paso 2: Configurar ACL Según Requisitos

#### 🔐 ACL 101 (Extendida Numerada)
**Requisitos:**
1. Bloquear explícitamente el acceso FTP a Enterprise Web Server desde Internet
2. No permitir tráfico ICMP desde Internet a ningún host en HQ LAN 1
3. Permitir todo el resto del tráfico

```cisco
! Configurar en router HQ (interfaz hacia Internet)
HQ(config)# access-list 101 deny tcp any host 192.168.1.70 eq ftp
HQ(config)# access-list 101 deny icmp any 192.168.2.0 0.0.0.31
HQ(config)# access-list 101 permit ip any any

! Aplicar ACL en interfaz S0/1/1 (hacia Branch/Internet)
HQ(config)# interface S0/1/1
HQ(config-if)# ip access-group 101 in
```

#### 🔐 ACL 111 (Extendida Numerada)
**Requisitos:**
- Ningún host de HQ LAN 1 debería poder acceder al servidor de Branch
- Permitir todo el resto del tráfico

```cisco
! Configurar en router HQ
HQ(config)# access-list 111 deny ip 192.168.2.0 0.0.0.31 host 192.168.2.45
HQ(config)# access-list 111 permit ip any any

! Aplicar ACL en interfaz G0/0/0 (HQ LAN 1)
HQ(config)# interface G0/0/0
HQ(config-if)# ip access-group 111 out
```

#### 🔐 ACL vty_block (Estándar Nombrada)
**Requisitos:**
- Solo direcciones de HQ LAN 2 pueden acceder a líneas VTY del router HQ
- Nombre exacto: vty_block

```cisco
! Configurar ACL estándar nombrada
HQ(config)# ip access-list standard vty_block
HQ(config-std-nacl)# permit 192.168.2.32 0.0.0.15
HQ(config-std-nacl)# deny any

! Aplicar a líneas VTY
HQ(config)# line vty 0 4
HQ(config-line)# access-class vty_block in
HQ(config-line)# login
HQ(config-line)# password cisco
```

#### 🔐 ACL branch_to_hq (Extendida Nombrada)
**Requisitos:**
- Ningún host en Branch LANs debe acceder a HQ LAN 1
- Usar una instrucción por cada LAN de Branch
- Nombre exacto: branch_to_hq

```cisco
! Configurar ACL extendida nombrada
Branch(config)# ip access-list extended branch_to_hq
Branch(config-ext-nacl)# deny ip 192.168.1.0 0.0.0.63 192.168.2.0 0.0.0.31
Branch(config-ext-nacl)# deny ip 192.168.1.64 0.0.0.7 192.168.2.0 0.0.0.31
Branch(config-ext-nacl)# permit ip any any

! Aplicar en interfaz hacia HQ
Branch(config)# interface S0/1/1
Branch(config-if)# ip access-group branch_to_hq out
```

---

## 🔍 Verificación y Preguntas

### Pruebas de Conectividad y Análisis

#### Pregunta 1: Ping desde Branch PC a Enterprise Web Server
**Test:** `Branch_PC> ping 192.168.1.70`

**Resultado:** ❌ NO exitoso

**Explicación:** El tráfico desde Branch PC (192.168.2.17) hacia Enterprise Web Server (192.168.1.70) debe pasar por la ACL `branch_to_hq` en el router Branch. Dado que Branch PC está en la red HQ LAN 1 (192.168.2.0/27) y Enterprise Web Server está en Branch LAN (192.168.1.64/29), la ACL `branch_to_hq` niega específicamente el tráfico desde las redes Branch hacia HQ LAN 1, pero no afecta el tráfico en la dirección inversa (de HQ a Branch). Sin embargo, puede haber otras ACL o problemas de ruteo.

**ACL Involucrada:** 
- Router: Branch
- ACL: `branch_to_hq` (extendida nombrada)
- Línea: `deny ip 192.168.1.64 0.0.0.7 192.168.2.0 0.0.0.31` (para tráfico desde Branch LAN a HQ LAN 1, pero esto es en dirección opuesta)

#### Pregunta 2: Ping desde PC-1 (HQ LAN 1) a Branch Server
**Test:** `PC-1> ping 192.168.2.45`

**Resultado:** ❌ NO exitoso

**Explicación:** PC-1 (192.168.1.10) en Branch LAN intenta alcanzar Branch Server (192.168.2.45) en HQ LAN 2. La ACL 111 en el router HQ deniega específicamente el tráfico desde HQ LAN 1 (192.168.2.0/27) hacia Branch Server (192.168.2.45).

**ACL Involucrada:**
- Router: HQ
- ACL: 111 (extendida numerada)
- Línea: `deny ip 192.168.2.0 0.0.0.31 host 192.168.2.45`

#### Pregunta 3: Acceso Web desde External Server a Enterprise Web Server
**Test:** Navegador web en External Server a `http://192.168.1.70`

**Resultado:** ✅ Exitoso (para HTTP, no para FTP)

**Explicación:** La ACL 101 en el router HQ bloquea FTP desde Internet hacia Enterprise Web Server, pero permite otros protocolos como HTTP. Dado que la ACL 101 tiene:
1. `deny tcp any host 192.168.1.70 eq ftp` → bloquea FTP
2. `deny icmp any 192.168.2.0 0.0.0.31` → bloquea ping a HQ LAN 1
3. `permit ip any any` → permite todo lo demás (incluyendo HTTP)

**ACL Involucrada:**
- Router: HQ
- ACL: 101 (extendida numerada)
- Línea: `permit ip any any` (para tráfico HTTP)

### Pruebas de Conexiones desde Internet

#### Pregunta 4: Conexión FTP desde Internet Users a Branch Server
**Test:** `Internet_Users> ftp 192.168.2.45`

**Resultado:** ✅ Conexión FTP exitosa (inicialmente)

**ACL a Modificar:** ACL 101 en el router HQ necesita ser modificada para bloquear FTP al Branch Server también.

**Instrucciones a Agregar:**
```cisco
! Agregar a ACL 101 en HQ
HQ(config)# access-list 101 deny tcp any host 192.168.2.45 eq ftp
! Nota: Esta línea debe ir antes del 'permit ip any any'
```

#### Verificación de Contadores ACL
```cisco
! Verificar operación de ACLs
HQ# show ip access-lists
HQ# show ip interface S0/1/1
Branch# show ip access-lists extended branch_to_hq

! Restablecer contadores para pruebas
HQ# clear access-list counters
```

---

## 💡 Conceptos Clave Aprendidos

### 🎯 Tipos de ACL Configuradas
| Tipo | Nombre/Número | Ubicación | Dirección | Propósito |
|------|---------------|-----------|-----------|-----------|
| Extendida Numerada | 101 | HQ S0/1/1 | IN | Filtrado Internet→Red Interna |
| Extendida Numerada | 111 | HQ G0/0/0 | OUT | Restringir HQ LAN 1 |
| Estándar Nombrada | vty_block | HQ VTY Lines | IN | Control acceso administrativo |
| Extendida Nombrada | branch_to_hq | Branch S0/1/1 | OUT | Restringir Branch→HQ LAN 1 |

### 🔧 Comandos ACL Esenciales
```cisco
! Crear ACL estándar numerada
access-list 1 permit 192.168.1.0 0.0.0.255

! Crear ACL extendida numerada
access-list 101 deny tcp any any eq 23

! Crear ACL estándar nombrada
ip access-list standard NOMBRE
 permit 192.168.1.0 0.0.0.255

! Crear ACL extendida nombrada
ip access-list extended NOMBRE
 deny tcp any any eq ftp

! Aplicar ACL a interfaz
interface GigabitEthernet0/0
 ip access-group 101 in

! Aplicar ACL a líneas VTY
line vty 0 4
 access-class 1 in

! Verificar ACL
show ip access-lists
show ip interface [interfaz]
show running-config | include access-list
```

### 📊 Mejores Prácticas Implementadas
1. **Orden de reglas:** Específicas antes de generales
2. **Ubicación eficiente:** ACL aplicadas cerca del origen del tráfico
3. **Nomenclatura:** ACL nombradas para mejor documentación
4. **Verificación:** Uso de contadores para monitoreo
5. **Documentación:** Comentarios en configuración

---

## 🚀 Solución de Problemas de ACL

### Síntomas Comunes y Soluciones

#### ❌ ACL no está filtrando tráfico
```cisco
! Verificar si ACL está aplicada
show ip interface [interfaz]
show running-config interface [interfaz]

! Verificar dirección de aplicación (in/out)
show ip access-lists
clear access-list counters
debug ip packet
```

#### ❌ Tráfico legítimo está siendo bloqueado
```cisco
! Verificar orden de reglas ACL
show ip access-lists [número/nombre]

! Verificar contadores para ver qué regla coincide
show ip access-lists | include matches

! Probar con tráfico específico
ping [destino] source [origen]
telnet [destino] [puerto]
```

#### ❌ Problemas con ACL nombradas
```cisco
! Verificar sintaxis de ACL nombrada
show run | section ip access-list

! Verificar aplicación correcta
show ip access-lists [nombre]
show ip interface brief | include [interfaz]
```

### 📋 Checklist de Verificación ACL
- [ ] Reglas ACL en orden lógico (específicas → generales)
- [ ] ACL aplicada en interfaz correcta
- [ ] Dirección correcta (in/out)
- [ ] Contadores incrementando en reglas apropiadas
- [ ] No hay reglas redundantes
- [ ] Regla 'permit any any' al final si es necesario
- [ ] ACL no bloquea tráfico de routing (EIGRP, OSPF, etc.)

---

## 📚 Recursos Adicionales

### Documentación Oficial Cisco
- [Cisco ACL Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_acl/configuration/15-mt/sec-acl-15-mt-book.html)
- [ACL Best Practices](https://www.cisco.com/c/en/us/support/docs/security/ios-firewall/23602-confaccesslists.html)
- [Named ACL Configuration](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3560/software/release/12-2_55_se/configuration/guide/3560_scg/swacl.html)

### Libros Recomendados
- "CCNA 200-301 Official Cert Guide" - ACL Chapter
- "Cisco IOS Access Lists" - O. Held
- "Network Security with ACLs" - Cisco Press

### Laboratorios Relacionados
- **ACL Estándar:** Filtrado básico por dirección fuente
- **ACL Extendida:** Filtrado por dirección, protocolo, puerto
- **ACL Reflexivas:** Sesiones stateful
- **ACL Basadas en Tiempo:** Filtrado por horario

### 🎓 Preguntas de Práctica CCNA
1. ¿Cuál es la diferencia entre ACL estándar y extendida?
2. ¿Por qué se recomienda colocar ACL estándar cerca del destino?
3. ¿Cómo afecta el orden de las reglas en una ACL?
4. ¿Qué comando muestra cuántos paquetes han coincidido con cada regla ACL?

---

## 👨‍💻 Autor

<div align="center">

**Darwin Manuel Ovalles Cesar**

<p align="center">
<a href="https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg" alt="LinkedIn - Darwin Ovalles" height="40" width="50" />
</a>
<a href="https://github.com/dovalless" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/github.svg" alt="GitHub - Darwin Ovalles" height="40" width="50" />
</a>
</p>

💼 **LinkedIn**: [darwin-manuel-ovalles-cesar-dev](https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev/)  
🌐 **GitHub**: [@dovalless](https://github.com/dovalless)  
🎓 **Certificaciones**: CCNA, Network+, A+  

*"Las ACL son los guardianes de la red, determinando qué tráfico puede pasar y qué tráfico debe ser detenido. Este laboratorio demuestra cómo implementar controles granulares para proteger una red empresarial compleja."*

**#Cisco #PacketTracer #ACL #NetworkSecurity #CCNA #IPv4 #AccessControl**

</div>

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.

```
MIT License
Copyright (c) 2024 Darwin Manuel Ovalles Cesar
```

---

## 🙏 Agradecimientos

- **Cisco Networking Academy** - Por proporcionar Packet Tracer y recursos educativos
- **Comunidad de Seguridad de Redes** - Por compartir mejores prácticas de ACL
- **Ingenieros de Red** - Por su expertise en diseño de políticas de acceso

<div align="center">

### ⭐ Si este laboratorio te ayudó a entender ACL IPv4, compártelo con otros ⭐

### 🔄 **Reflexión Final:**
*"Configurar ACL es como ser el portero de un club exclusivo: debes conocer exactamente quién puede entrar, por qué puerta, y a qué horas. Cada regla es una instrucción para el router sobre cómo manejar el tráfico de red."*

**Desarrollado con 💙 para profesionales de seguridad de red**

---
*Laboratorio completado: Packet Tracer - Desafío de implementación de ACL IPv4*  
*Habilidades demostradas: ACL Estándar, ACL Extendida, ACL Nombradas, Filtrado por Protocolo, Seguridad Perimetral*

</div>
```
