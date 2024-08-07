`AES`（高级加密标准，`Advanced Encryption Standard`）是一种对称密钥加密标准，用于确保电子数据的安全。`AES`使用固定块大小的加密块（128位），并支持多种长度的密钥（128位、192位和256位）。

在使用`AES`时，通常需要指定一个**操作模式`Operation Mode`**，并且有时还需要指定一个**填充方案`Padding Scheme`**。`AES`的大多数操作模式（如`ECB`和`CBC`）仅提供数据的保密性。但是，当`AES`在`CCM`、`GCM`或`EAX`模式下运行时，这些模式不仅提供数据的保密性，还提供数据的真实性验证。

- 操作模式
	`AES`本身是一个块加密算法，它需要一个操作模式来处理大于一个块大小的数据。常见的操作模式包括`ECB`（电子密码本模式）、`CBC`（密码块链接模式）、`CCM`（计数器模式和`CBC-MAC`）、`GCM`（伽罗瓦/计数器模式）和`EAX`（一种结合了加密和认证的模式）；

- 填充方案
	由于`AES`是块加密，当数据长度不是块大小的整数倍时，需要填充数据以使其长度符合块大小的要求。填充方案如`PKCS#7`常用于这种情况；
# 软件实现

[[crytopp#AES加密]]

