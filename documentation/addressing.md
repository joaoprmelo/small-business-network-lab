# IP Addressing

Documentação do endereçamento IPv4 utilizado no laboratório.

## Redes

| Rede           | Máscara         | Finalidade            |
| -------------- | --------------- | --------------------- |
| 192.168.1.0/24 | 255.255.255.0   | Rede LAN 1            |
| 192.168.2.0/24 | 255.255.255.0   | Rede LAN 2            |
| 10.0.0.0/30    | 255.255.255.252 | Conexão R1 ↔ ISP      |
| 10.0.1.0/24    | 255.255.255.0   | Rede externa simulada |

## Dispositivos

| Dispositivo | Interface | IP          | Máscara         | Gateway     |
| ----------- | --------- | ----------- | --------------- | ----------- |
| R1          | G0/0      | 192.168.1.1 | 255.255.255.0   | —           |
| R1          | G0/1      | 192.168.2.1 | 255.255.255.0   | —           |
| R1          | G0/2      | 10.0.0.1    | 255.255.255.252 | —           |
| ISP         | G0/0      | 10.0.0.2    | 255.255.255.252 | —           |
| ISP         | G0/1      | 10.0.1.1    | 255.255.255.0   | —           |
| Server1     | —         | 192.168.1.2 | 255.255.255.0   | 192.168.1.1 |
| Server2     | —         | 192.168.2.2 | 255.255.255.0   | 192.168.2.1 |
| Server3     | —         | 10.0.1.2    | 255.255.255.0   | 10.0.1.1    |
| PC1         | —         | DHCP        | 255.255.255.0   | 192.168.1.1 |
| PC2         | —         | DHCP        | 255.255.255.0   | 192.168.1.1 |

## DHCP

O R1 fornece endereços IP automaticamente para os dispositivos da LAN 1.

### Configuração

```text
ip dhcp pool LAN
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
```

Os endereços `192.168.1.1` até `192.168.1.9` foram excluídos do DHCP:

```text
ip dhcp excluded-address 192.168.1.1 192.168.1.9
```

Dessa forma, o DHCP pode distribuir endereços a partir de `192.168.1.10`.

## Observação

O endereço `192.168.1.1` é utilizado como gateway da LAN 1, enquanto `192.168.2.1` é utilizado como gateway da LAN 2.

A rede `10.0.0.0/30` foi utilizada para a conexão ponto a ponto entre o R1 e o roteador ISP.
