# demo2024
| Имя устройства | Итерфейс |  IPv4/IPv6   | Маска/Префикс |       Шлюз       |
| -------------- | -------- | ------------ | ------------- |    ----------    |
|                |  Gi1     | 1.1.1.4      |      /25      | 1.1.1.1          |
| ISP            |  Gi2     | 2.2.2.4      |      /25      | 2.2.2.2          |
|                |  Gi3     | DHCP         |      /24      |                  |
| HQ-R           |  Gi1     | 192.168.5.6  |      /24      | 192.168.5.100    |
|                |  Gi2     | 1.1.1.5      |      /25      |                  |
| BR-R           |  Gi1     | 172.16.5.6   |      /24      | 172.16.5.100     |
|                |  Gi2     | 2.2.2.5      |      /25      |                  |
| HQ-SRV         |  Gi1     | 192.168.5.5  |      /24      | 192.168.5.1      |
| BR-SRV         |  Gi1     | 172.16.5.5   |      /24      | 172.16.5.1       |
