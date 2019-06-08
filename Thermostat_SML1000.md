# SML1000 modbus protocol

This protocol takes standard MODBUS as a reference, mainly use for communication between thermostat and upper computer. This protocol doesn’t describe the MODBUS. As to standard MODBUS, please refer to the relevant standard documents.

## 1.  Basic description


| **No** | **Parameter**        | **Protocol provision**                               |
| ------ | -------------------- | ---------------------------------------------------- |
| 1      | Operating mode       | RS-485,master-slave；thermostat is the slave machine |
| 2      | Physical interface   | A(+),B(-)  two-wire system                           |
| 3      | Baud rate            | 9600 bps for standard                                |
| 4      | Byte format          | 9 format (8 data bits +1 stop bit)                   |
| 5      | Modbus               | RTU                                                  |
| 6      | Transmission mode    | RTU format (Please refer to standard MODBUS)         |
| 7      | Thermostat address   | 1－255                                               |
| 8      | Command code         | 03，06(03—read thermostat，  06—set thermostat)      |
| 9      | CRC check code       | CRC－16  (Please refer to standard MODBUS)           |
| 10     | CRC veriﬁcation mode | CRC－16  (Please refer to standard MODBUS)           |

## 

| Регистр | Тип     | Чтение / запись | Значение по умолчанию | Формат                                    | Назначение                        |
| ------- | ------- | --------------- | --------------------- | ----------------------------------------- | --------------------------------- |
| 1       | holding | RW              |                       | 0x5A–means close, 0xA5–means open         | Setting Power On/off              |
| 2       | holding | RW              |                       | remark 1                                  | High Temperature Protection Range |
| 3       | holding | RW              |                       | remark 1                                  | Temperature for external sensor   |
| 4       | holding | RW              |                       | remark 1                                  | Temperature for internal sensor   |
| 5       | holding | RW              |                       | remark 1                                  | Setting Temperature               |
| 6       | holding | W               |                       |                                           | Setting Minute                    |
| 7       | holding | W               |                       |                                           | Setting Hour                      |
| 8       | holding | W               |                       | 00 means Weekly program, 01 means  Manual | Setting  Mode                     |


Remark:

1. Точность термостата составляет 0,5 ℃, поэтому термостат отправляет данные о температуре в виде целового числа - значение температуры умноженое на 2. Например: когда собранная температура составляет 25,5 ℃, значение, посылаемое термостатом на компьютер, будет 33H (числовое значение равно 51). Точно так же, когда верхний компьютер отправляет заданные данные о температуре в термостат, значение заданной температуры должно быть умножено на 2 и полностью отправлено по формату HEX, так как точность составляет 0,5 ℃.

2. How to change thermostat IP address?

During power off, press button M and button Clock for 5 seconds at the same time into high senior options.

Press M to item 2.

Then press up and down to change the relative value. The default is 0x01.  

All data will be saved only after a restart.