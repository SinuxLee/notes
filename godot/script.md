

### 内置帮助文档

- @GDScript  内置 GDScript 常量、函数、注解
- @GlobalScope 全局范围的常量和函数
- @ 符合开头的，一般为注解



### 工具

[Flint Texture Packer](https://texturepacker.flintbyte.com/)

[Free texture packer - sprite sheets for games and sites](https://free-tex-packer.com/app/)



### 资源网站

1. [2D Game Assets Store & Free - CraftPix.net](https://craftpix.net/)
2. [Home · Kenney](https://kenney.nl/)
3. [OpenGameArt.org](https://opengameart.org/)
4. [Top game assets - itch.io](https://itch.io/game-assets)
5. [4149 free SVG and PNG icons for your games or apps | Game-icons.net](https://game-icons.net/)
6. [免费正版高清图片素材库 超过5.4百万张优质图片和视频素材可供免费使用和下载 - Pixabay](https://pixabay.com/zh/)
7. [Freesound](https://freesound.org/)
8. [Welcome to the Free Music Archive - Free Music Archive](https://freemusicarchive.org/home)



### 脚本属性定义

1. 添加脚本可导出属性

   ```python
   ## 玩家等级，只能是正整数
   @export var level: int = 1
   
   ## 经验值
   @export_range(1, 100, 1, '123')
   var exp: int = 1
   
   # 普通分组，不能折叠
   @export_category("基础属性")
   @export var hp = 30
   @export var speed = 1.25
   
   # 前缀分组，可以折叠
   @export_group("战斗属性", "battle_")
   ## 角色的基础攻击力
   @export var battle_attack: int = 10
   ## 角色的防御值
   @export var battle_defense: int = 5
   
   # 导出颜色
   @export_group("颜色值","dye_")
   @export_color_no_alpha
   var dye_color: Color
   @export_color_no_alpha
   var dye_colors: Array[Color]
   ```

2. 格式化字符串

   ```python
   var name = 'libz'
   print("Hello %s!" % name)
   
   # 模板格式化对象
   var p = {id=65535, name="Epsilon", age=14,}
   var template = "ID: {id}, 姓名:{name}"
   print(template.format(p))
   
   # 重载基类方法，实现对象格式化输出
   func _to_string() -> String:
       return "[道具](ID:%s 名称:%s)" % [id, name]
   ```

   

3. 多样注释

   ```python
   #region 演示函数
   func f() -> void:
       print('我是区域注释，可以折叠哦')
   #endregion
   
   #### 以下不同关键字开头注释会显示不同颜色
   # NOTE 提示
   
   # TODO 待实现
   
   # CAUTION 警告
   
   # WARNING 还是警告
   ```

   

4. 一等公民函数

   ```python
   # 绑定对象方法
   func print_args(arg1, arg2, arg3 = ""):
       prints(arg1, arg2, arg3)
   
   func test():
       var callable = Callable(self, "print_args")
       callable.call("hello", "world")  # 输出“hello world ”。
   # 绑定普通函数
   var fp = print_args
   print(fp.call(2))
       
   # Lambda 表达式
   func _init():
   	var my_lambda = func (message):
   		print(message)
   
   	my_lambda.call("大家好呀！")
   
   
   # 匿名函数
   button_pressed.connect(func(): print("全军出击！"))
   
   ```

   

5. 警告消除

   ```py
   func test():
   	var a = 1 # Warning (if enabled in the Project Settings).
   	@warning_ignore_start("unused_variable")
   	var b = 2 # No warning.
   	var c = 3 # No warning.
   	@warning_ignore_restore("unused_variable")
   	var d = 4 # Warning (if enabled in the Project Settings).
   ```

   

6. 生命函数

   ```python
   extends Node2D
   
   # 解析脚本时创建资源实列
   var diamond = preload("res://diamond.tscn").instantiate()
   
   # 在 _init 之后，_ready 之前被初始化
   @onready var player : CharacterBody2D = $Player
   
   # 当构造函数使用
   func _init() -> void:
       pass
   
   # 界面初始化，只会被调用一次
   func _ready() -> void:
       pass
   
   # 每帧调用一次
   func _process(delta: float) -> void:
       pass
   ```

   

7. 类型转换

   ```py
   # 数字转字符串
   var speed: int = 5
   var string: String = String.num(speed)
   
   
   ```

   

8. 同时使用多个版本的 Godot

   在每个编辑器的同级目录下创建 .\_sc\_ 就可以强制编辑将配置文件存放在当前目录下。

9. 低处理器模式

   GUI 应用可以保持低帧率运行。项目->常规(打开高级设置)->应用->运行->开启低处理器模式

10. 设置运行时帧率

    ```py
    Engine.max_fps = 120
    ```

11. Texture/Material/Shader 关系

    - Texture是静态数据文件，Material是参数集合，Shader是动态计算程序
    - *类比*：Texture=颜料，Material=调色板配方，Shader=绘画技法
    - Texture、Material和Shader在Godot中构成**数据→配置→算法**的完整渲染链条

12. GUI 中内嵌 Node2D

    ```py
    Control
    └── ViewportContainer
        └── Viewport
            └── Node2D
    ```

    

13. 