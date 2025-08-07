# 多人联机DEMO

## 1.使用说明

### 编辑器版本

​	UE4.25

### 支持联机模式

- 本地联机
- 同一局域网内联机（能相互ping通）【需打包后在同一局域网内的两台不同的电脑运行】

### 游戏人数修改

![image-20250807140312028](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807140312028.png)

修改参与的玩家数量即可

### 演示

#### 1.二人联机演示



#### 2.多人联机演示



## 2.开发过程（仅记录，无学习需求可不阅读）

### 1.多人联机调试设置

![image-20250807140654466](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807140654466.png)

【新建编辑器窗口】是一种**游戏运行模式**，指的是当你点击编辑器中的 “play”时，游戏会在当前 UE4 编辑器内**新建一个独立的窗口**来运行，而不是全屏占据整个编辑器界面，也不是启动一个完全脱离编辑器的独立游戏进程。

新建编辑器窗口的数量会依据【多玩家选项】中【玩家数量】的设置新建。如此处玩家数量设置为2，则新建窗口数量为2。

### 2.联机大厅界面

多人游戏中，玩家进入游戏之后并不会直接开始游戏，而是会先进入联机大厅界面。

#### 2.1.新建联机大厅地图

区别于游戏地图，联机大厅地图一般不会有角色形象，也不支持角色移动，仅仅支持一些按键选择。

![image-20250807142524480](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807142524480.png)

新建一个关卡，任意选择，自己觉得适合当大厅就行。笔者选择的是TimeOfDay关卡。

#### 2.2.大厅功能核心组件

`BaseGameMode`（基础游戏模式）和`BP_Lobby`（大厅角色）是实现 “大厅功能” 的核心组件，主要用于解决多人游戏中**玩家进入正式游戏前的准备阶段**的逻辑管理。

**具体操作：**

##### 1.创建两个蓝图，分别为`BP_Lobby`（大厅角色蓝图）【继承自character】和`BaseGameMode`（基础游戏模式）【继承自游戏模式】

2.打开“BaseGameMode”，默认Pawn类选择“BP_Lobby”

3.在世界场景设置中设置游戏模式重载为“BaseGameMode”

**详细解释**： 

###### 1. `BP_Lobby`（大厅角色蓝图）

![image-20250807143158086](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807143158086.png)

`BP_Lobby`是继承自`Character`的蓝图类，专门用于玩家在 “大厅地图（Lobby_Time_Map）” 中的角色表现。
与正式游戏中的战斗 / 探索角色不同，大厅角色通常具有简化的功能：
当玩家刚加入游戏时，先进入`LobbyMap`并生成`BP_Lobby`角色，表明当前处于 “准备阶段”；当所有玩家准备完成后，再切换到正式关卡并替换为正式游戏角色。

###### 2.`BaseGameMode`（基础游戏模式）

![image-20250807143331107](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807143331107.png)

游戏模式（GameMode）是 UE4 中控制 “游戏规则” 的核心类，`BaseGameMode`专门用于定义`Lobby_Time_Map`中的规则：

- 通过设置`Default Pawn Class = BP_Lobby`，确保玩家进入`Lobby_Time_Map`时自动生成`BP_Lobby`角色（而非正式游戏角色）。

- **隔离不同阶段的逻辑**：
  正式游戏关卡通常会使用另一个游戏模式（如`GameplayGameMode`），而`BaseGameMode`仅作用于`LobbyMap`。通过在 “世界场景设置” 中为`LobbyMap`指定`BaseGameMode`，可实现 “大厅阶段” 与 “正式游戏阶段” 的规则隔离，避免逻辑冲突。

###### 2.打开“BaseGameMode”，默认Pawn类选择“BP_Lobby”

![image-20250807143527637](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807143527637.png)

#### 2.3.设计**大厅交互的用户界面（UI）控件**

新建一个蓝图，命名为UMG_Lobby【继承自控件】。

![image-20250807143900499](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807143900499.png)

`UMG_Lobby` 作为大厅地图（`Lobby_Time_Map`）的交互界面，为玩家提供了进入正式游戏前的关键操作入口，结合控件组成（两个按钮 + 可编辑文本），包含这些功能：

- **按钮 1（“创建服务器” ）**：点击后由当前玩家作为主机创建服务器，直接进入正式关卡（适合双人联机中 “主机启动游戏” 的场景）。
- **按钮 2（ “连接服务器”）**：点击后触发连接逻辑，读取可编辑文本中的 IP 地址，通过网络模块连接到指定服务器。
- **可编辑文本控件**：用于输入或显示服务器 IP 地址，方便玩家手动输入目标服务器地址

打开“UMG_Lobby”，参照左侧层级设计界面

![image-20250807144206544](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807144206544.png)

其中“服务器IP"为提示文字，需要玩家手动输入。

具体设计无需完全参照，依据自身审美设计即可

#### 2.4 将UI控件连接到于大厅地图

打开大厅地图Lobby_Time_Map -> 打开关卡蓝图

![image-20250807144733724](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807144733724.png)

该关卡原始存在蓝图设计，无需更改。找到事件开始运行的节点，如下编辑节点。

![image-20250807144939380](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807144939380.png)

【注意】如果找不到对应节点，检查是否取消勾选情景关联。该选项勾选后会自动过滤一些节点

![image-20250807145213895](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807145213895.png)

该蓝图的逻辑较为易懂，此处仅作一些简单解释。

**目标：**在关卡开始时，为拥有 UMG 控件的玩家显示 UI 界面，并仅为该玩家启用鼠标光标（确保光标与玩家正确绑定）。

**具体步骤：**

事件开始时，创建UMG控件，并且显示在窗口上。（添加到视口）

在此基础上还要显示玩家的鼠标，玩家的鼠标通过set设置。

- 如何将光标和玩家对应起来？通过刚刚的创建设置，返回拥有UMG控件的玩家，拥有这些控件的玩家即为要显示光标的玩家。
- 获取完成之后，连接到set的目标处

至此，UI已经可以正确显示在界面上。

![image-20250807150113790](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807150113790.png)

完整的关卡蓝图设计如下

![image-20250807150309470](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807150309470.png)

### 3.处理大厅控件逻辑

UMG控件中一共有三个需要处理逻辑的控件，分别为

- 创建服务器
- 加入服务器
- 可编辑输入框：输入服务器IP

#### 3.1.创建服务器

在UMG_Lobby中，选中创建服务器按钮，在事件中新增点击事件，会自动创建蓝图节点。

![image-20250807150549810](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807150549810.png)

由于笔者编写时已完成创建，故显示的是“查看”，原始状态是和底下一样的加号

![image-20250807150634440](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807150634440.png)

在该节点后面，新增按钮逻辑

![image-20250807150704462](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807150704462.png)

逻辑较为简单，此处仅解释最核心的“创建会话”和“打开关卡”。

**“创建会话（Create Session）”** 节点是多人游戏开发中用于**创建一个可供其他玩家加入的游戏会话（Session）** 的核心网络节点，主要作用是在本地或专用服务器上初始化一个游戏房间，让其他客户端可以通过 “查找会话” 和 “加入会话” 节点连接进来。

- **“Player Controller（玩家控制器）” 参数**用于指定**发起创建会话的玩家所对应的玩家控制器**。
- **公共连接数（Public Connections）**是会话允许的最大玩家数量（包括创建者自己）。例如，双人游戏设置为 2，意味着除主机外还可加入 1 名玩家。

**“打开关卡（Open Level）”** 是用于**加载新关卡（Level）并切换当前游戏场景**的核心节点。

1. **关卡名称（Level Name）**
   需加载的目标关卡名称（必须与项目中 “Content” 文件夹下的关卡文件名一致，区分大小写）。例如，若要加载 “LobbyMap.umap”，则填写 “LobbyMap”。

2. **选项（Options）**
   可选参数，用于传递字符串格式的额外信息（如 “PlayerName=John?Difficulty=Hard”），新关卡可通过 “Get Game Mode” 或 “世界设置” 中的 “关卡选项” 获取这些参数，实现关卡间数据传递。

   此处设置为listen，是一个专门用于**多人游戏场景**的参数，其作用是让当前关卡以 **“Listen Server（监听服务器）” 模式 ** 加载。

   ​	“Listen Server” 是 UE 中一种多人游戏服务器模式，指的是 **“既是客户端又是服务器” 的角色 **：

   - 启动该模式的玩家（主机）自身可以操控角色进行游戏（作为客户端）。
   - 同时，该玩家的游戏进程会作为服务器，允许其他客户端（玩家）连接并加入当前关卡。

#### 3.2处理输入的服务器IP

在设计器中，选中可编辑TEXT文本框，添加事件。原始是显示一个加号，因为笔者已设置过，所以显示的是查看。自动打开对应的事件节点。

![image-20250807152029269](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807152029269.png)

![image-20250807152141689](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807152141689.png)

为了将玩家输入的IP地址进行传递，需要设计一个变量专门存储该IP地址。

在该控件蓝图中，打开【我的蓝图】窗口

![image-20250807151804909](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807151804909.png)

在我的蓝图中，添加新变量serverAddress，用于存储玩家输入的IP。

![image-20250807151846387](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807151846387.png)

在此处，笔者设置的变量为文本类型；未尝试过字符串设计是否可行，有兴趣的读者可自行尝试。

需要用到该变量时，将变量拖动到蓝图的空白处，选择获取/设置即可。

![image-20250807152503071](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807152503071.png)

回到上文存储玩家输入IP的逻辑，在TEXT被编辑过后，需要存储编辑后的TEXT到该变量中，则选择的是设置变量。

![image-20250807152537693](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807152537693.png)

连接完成后如图。

#### 3.3 加入服务器

加入服务器的逻辑处理同样是在点击后进行，创建事件参照创建服务器。

整体处理逻辑如下：![image-20250807152713790](C:\Users\Galaxy\AppData\Roaming\Typora\typora-user-images\image-20250807152713790.png)

此处的实现逻辑同样较为简单，仅解释部分较为重要的节点。

**“执行控制台命令（Execute Console Command）”** 节点用于在运行时**执行 UE 编辑器或游戏中的控制台命令**，通过代码逻辑触发原本需要手动在控制台输入的指令，实现各种调试、配置或功能控制。

**命令（Command）**
输入需要执行的控制台命令字符串（与在游戏内按`~`键调出控制台后输入的命令完全一致）。
例如：

- `Open Level BattleMap`：切换到名为 “BattleMap” 的关卡（等效于 “打开关卡” 节点）。
- `Show FPS`：显示帧率信息。

此处执行的是连接服务器的命令，参数有两个，所以用的是附加。连接起来的命令是open IP。

【注意】因为附加不会自动在两者之间加空格，A处输入的是【open 】（末尾处有空格），否则无法识别为两个参数。

“延迟”节点是为了给连接服务器一些缓冲时间，2s之后再跳出连接失败。

至此，整个DEMO已可实现多人联机，效果请参照演示。

感谢您的阅读，最后请您欣赏UE4内置的日出日落。