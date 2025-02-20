
| 客户端                           | 服务器                                         |
| ----------------------------- | ------------------------------------------- |
| 无                             | `Game Mode`仅存在于服务器，负责控制游戏进程、生成与销毁`Actor`等功能 |
| 仅持有自己的`Player Controller`     | 持有所有客户端的`Player Controller`                 |
| 持有所有客户端的`Player State`与`Pawn` | 持有所有客户端的`Player State`与`Pawn`               |
| 持有自身的`Hud`与`Widgets`界面        | 一般来说没有UI界面                                  |
|                               |                                             |

- `Replication`是从服务器单向同步到每个客户端，如果在客户端修改了并不会同步到服务器，因此标记为`Replicated=True`的变量不应该被客户端修改；

- 客户端同步到服务器，需要依赖`RPC`，即**远程过程调用**`Remote Procedure Call`；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250212230848.png)
