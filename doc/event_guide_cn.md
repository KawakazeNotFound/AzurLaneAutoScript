# 活动识别素材位置及添加新活动指南

## 识别素材位置

每次新活动的识别素材分布在以下位置：

### 1. 活动识别素材（Assets）

活动的识别素材（图片资源）存储在 `assets` 目录下，按服务器分类：

```
assets/
├── cn/          # 国服素材
│   ├── event/   # 通用活动识别素材（如护航活动等）
│   ├── campaign/# 关卡入口识别素材
│   └── ...
├── en/          # 国际服素材
│   ├── event/
│   ├── campaign/
│   └── ...
├── jp/          # 日服素材
│   ├── event/
│   ├── campaign/
│   └── ...
└── tw/          # 台服素材
    ├── event/
    ├── campaign/
    └── ...
```

#### 识别素材说明：

- **通用活动素材** (`assets/[server]/event/`)：
  - 用于识别活动入口、活动UI元素等
  - 例如：`ESCORT_CHECK.png`（护航活动检测）
  - 例如：`MAIN_GOTO_ESCORT.png`（主界面进入护航）

- **关卡识别素材** (`assets/[server]/campaign/`)：
  - 用于识别关卡名称、关卡按钮等
  - 这些素材由 `dev_tools/button_extract.py` 自动生成

### 2. 活动地图文件（Campaign Maps）

活动的地图文件存储在 `campaign` 目录下：

```
campaign/
├── event_20200227_cn/    # 活动目录（格式：event_日期_服务器）
│   ├── campaign_base.py  # 活动基类
│   ├── sp.py            # SP关卡配置
│   ├── sp1.py           # SP1关卡地图
│   ├── sp2.py           # SP2关卡地图
│   └── ...
├── event_20220818_cn/
├── raid_20200624/        # 共斗活动
└── war_archives_xxxxx/   # 作战档案（常驻活动）
```

#### 地图文件说明：

每个活动目录包含：
- `campaign_base.py`：活动的基础配置类
- `sp.py`, `sp1.py`, `sp2.py` 等：各个关卡的地图数据
  - 地图形状 (`MAP.shape`)
  - 地图数据 (`MAP.map_data`)：敌人分布、障碍物等
  - 相机数据 (`MAP.camera_data`)：视角切换点
  - 刷新数据 (`MAP.spawn_data`)：敌人刷新规则
  - 网格定义：A1, B1, C1 等地图坐标

### 3. 活动模块（Module）

活动相关的逻辑代码在 `module` 目录下：

```
module/
├── event/              # 通用活动模块
│   ├── assets.py      # 活动识别素材定义（自动生成）
│   ├── campaign_sp.py # SP关卡逻辑
│   ├── campaign_abcd.py # ABCD关卡逻辑
│   └── maritime_escort.py # 护航活动
├── event_hospital/    # 医院活动模块
├── eventstory/        # 活动剧情模块
└── campaign/          # 关卡通用模块
    ├── assets.py      # 关卡识别素材定义
    ├── campaign_ui.py # 关卡UI交互
    └── run.py         # 关卡运行逻辑
```

## 如何添加新活动

### 步骤 1：准备识别素材

1. **截取游戏截图**：
   - 分辨率必须为 **1280x720**
   - 使用模拟器或游戏截图工具
   - 截图应包含需要识别的UI元素

2. **保存素材到对应位置**：
   ```
   # 如果是通用活动元素（如新的活动类型入口）
   assets/cn/event/NEW_BUTTON.png
   assets/en/event/NEW_BUTTON.png
   assets/jp/event/NEW_BUTTON.png
   assets/tw/event/NEW_BUTTON.png
   
   # 如果是关卡相关
   assets/cn/campaign/STAGE_ENTRANCE.png
   # ...其他服务器
   ```

3. **运行按钮提取工具**（可选，用于自动生成 assets.py）：
   ```bash
   cd /path/to/AzurLaneAutoScript
   python -m dev_tools.button_extract
   ```
   这会自动扫描 `assets` 目录并生成 `module/*/assets.py` 文件。

### 步骤 2：创建活动目录和地图文件

1. **创建活动目录**：
   ```bash
   mkdir campaign/event_YYYYMMDD_[server]
   ```
   - `YYYYMMDD`：活动首次上线的日期（格式：年月日）
   - `[server]`：服务器标识（cn/en/jp/tw）

2. **创建地图文件**：
   
   参考现有活动目录（如 `campaign/event_20220818_cn/`），创建以下文件：
   
   - `campaign_base.py`：继承基类，定义活动特有配置
   - `sp1.py`, `sp2.py` 等：各关卡地图数据

   **地图文件模板** (`sp1.py`)：
   ```python
   from .campaign_base import CampaignBase
   from module.map.map_base import CampaignMap
   from module.map.map_grids import SelectedGrids, RoadGrids
   from module.logger import logger

   MAP = CampaignMap('SP1')
   MAP.shape = 'I7'  # 地图大小（列x行）
   MAP.camera_data = ['D4', 'E5', 'F4']  # 相机视角位置
   MAP.camera_data_spawn_point = ['D3']  # 出生点相机位置
   MAP.map_data = """
       -- ++ -- SP -- SP -- ++ ++
       -- ++ -- -- -- -- -- ME --
       ME -- -- -- MS -- -- -- ME
       -- -- Me ++ Me ++ Me -- --
       ME -- -- -- __ -- -- -- ME
       -- ++ ++ ME -- ME -- ME --
       -- -- ++ -- MB -- ++ ++ ++
   """
   # 图例：
   # -- : 空地
   # ++ : 障碍物
   # ME : 敌方舰队
   # MB : Boss
   # MS : 塞壬精英
   # SP : 特殊点位
   # __ : 起始点

   MAP.weight_data = """
       50 50 50 50 50 50 50 50 50
       50 50 50 50 50 50 50 50 50
       ...
   """
   # 权重数据：用于路径规划，数值越小越优先

   MAP.spawn_data = [
       {'battle': 0, 'enemy': 2, 'siren': 1},
       {'battle': 1, 'enemy': 2},
       {'battle': 2, 'enemy': 1},
       {'battle': 3, 'enemy': 1},
       {'battle': 4, 'boss': 1},
   ]
   # 刷新规则：每场战斗后的敌人刷新情况

   # 网格定义（自动生成）
   A1, B1, C1, D1, E1, F1, G1, H1, I1, \
   A2, B2, C2, D2, E2, F2, G2, H2, I2, \
   ...
       = MAP.flatten()

   class Config:
       # 活动特殊配置
       MAP_SIREN_TEMPLATE = ['ShipName1', 'ShipName2']  # 塞壬模板
       MOVABLE_ENEMY_TURN = (2,)  # 可移动敌人回合
       MAP_HAS_SIREN = True
       MAP_HAS_MOVABLE_ENEMY = True
   ```

### 步骤 3：更新活动列表

编辑 `campaign/Readme.md`，在活动列表表格中添加新行：

```markdown
| Aired Date | Directory                | Event Name           | CN         | EN         | JP         | TW         |
| :--------- | :----------------------- | :------------------- | :--------- | :--------- | :--------- | :--------- |
| YYYYMMDD   | event_YYYYMMDD_cn        | Event English Name   | 活动中文名 | Event Name | イベント名 | 活動中文名 |
```

**字段说明**：
- **Aired Date**：活动首次上线日期（YYYYMMDD格式）
- **Directory**：活动目录名（使用下划线 `_` 而非空格）
- **Event Name**：官方英文名称（如未在EN服上线，使用CN名称）
- **CN/EN/JP/TW**：各服务器GUI显示名称（未上线的服务器填 `-`）
  - 如果是复刻活动，可以在名称前加"复刻"
  - 如果是作战档案，会自动添加"档案"前缀

### 步骤 4：运行配置更新工具

```bash
cd /path/to/AzurLaneAutoScript
python -m module.config.config_updater
```

这个工具会：
- 读取 `campaign/Readme.md` 中的活动列表
- 自动生成 `module/config/config_generated.py`
- 更新GUI配置，使新活动在界面中可选

### 步骤 5：测试活动

1. **启动 Alas**：
   ```bash
   python gui.py
   ```

2. **在GUI中选择新活动**：
   - 进入"出击"页面
   - 在活动下拉列表中选择新添加的活动
   - 选择对应的关卡（SP1, SP2等）

3. **运行并验证**：
   - 观察脚本能否正确识别活动入口
   - 观察脚本能否正确导航地图
   - 观察脚本能否正确战斗

4. **查看日志**：
   - 正常日志：`log/YYYY-MM-DD.txt`
   - 错误日志：`log/error/` 目录

### 步骤 6：提交代码

确认测试无误后，提交你的更改：

```bash
git add campaign/event_YYYYMMDD_*/
git add campaign/Readme.md
git add assets/*/event/*.png  # 如果有新素材
git add assets/*/campaign/*.png  # 如果有新素材
git commit -m "Add event: [活动名称]"
git push
```

## 常见问题

### Q1: 识别素材的分辨率必须是 1280x720 吗？
**A**: 是的。Alas 设计为使用 1280x720 分辨率，所有识别素材都必须是这个分辨率。

### Q2: 如果活动在不同服务器有不同的UI怎么办？
**A**: 为每个服务器分别准备素材，放在对应的 `assets/[server]/` 目录下。

### Q3: 如何知道地图数据中的符号含义？
**A**: 常见符号：
- `--`: 可通行的空地
- `++`: 障碍物（不可通行）
- `==`: 海洋（特殊地形）
- `ME`: 普通敌方舰队
- `MB`: Boss 舰队
- `MS`: 塞壬精英舰队
- `MA`: 弹药补给点
- `SP`: 特殊点位（如谜题、机关）
- `__`: 舰队起始点
- `FL`: 舰队当前位置

### Q4: 什么时候需要运行 `button_extract.py`？
**A**: 当你添加新的识别素材（PNG图片）到 `assets` 目录后，运行此工具会自动生成或更新 `assets.py` 文件，该文件包含了按钮的位置和颜色信息。

### Q5: 复刻活动需要创建新目录吗？
**A**: 通常不需要。如果复刻活动的地图与原活动完全相同，可以复用原活动的目录。在 `campaign/Readme.md` 中添加新行，指向相同的 Directory 即可。

### Q6: 如何调试地图识别问题？
**A**: 
1. 查看 `log/error/` 目录下的错误日志
2. 检查截图，确认识别素材是否匹配
3. 使用 `dev_tools/map_extractor.py` 辅助提取地图信息
4. 启用调试模式，查看详细的地图识别日志

## 开发工具

### button_extract.py
自动提取按钮位置和颜色信息，生成 `assets.py` 文件。

**使用方法**：
```bash
python -m dev_tools.button_extract
```

### map_extractor.py
辅助提取地图信息的工具（需要手动运行和调整）。

### config_updater.py
更新配置文件，读取 `campaign/Readme.md` 并生成 GUI 配置。

**使用方法**：
```bash
python -m module.config.config_updater
```

## 参考资源

- [Wiki 首页](https://github.com/LmeSzinc/AzurLaneAutoScript/wiki)
- [开发文档](https://github.com/LmeSzinc/AzurLaneAutoScript/wiki/1.-Start)
- [海图识别文档](https://github.com/LmeSzinc/AzurLaneAutoScript/wiki/perspective)
- [FAQ](https://github.com/LmeSzinc/AzurLaneAutoScript/wiki/FAQ_en_cn)

## 贡献指南

如果你添加了新活动并希望贡献给项目：

1. Fork 项目仓库
2. 创建新分支：`git checkout -b add-event-xxxx`
3. 按照上述步骤添加活动
4. 提交 Pull Request
5. 在 PR 中说明：
   - 活动名称和服务器
   - 测试情况
   - 已知问题（如果有）

感谢你为 Alas 项目做出贡献！
