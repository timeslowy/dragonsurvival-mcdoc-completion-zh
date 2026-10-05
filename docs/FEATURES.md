
# 功能特性详情

## 数据包JSON补全

其中`资源数据映射`由于Spyglass的`identifier`功能仍未完善，所以无法精确匹配文件

| 支持的功能        | 功能对应的路径                  |
| ----------------- | ------------------------------- |
| 资源数据映射      | data/<命名空间>/dragon_ability  |
| 能力/技能         | data/<命名空间>/dragon_ability  |
| 身体类型/子物种   | data/<命名空间>/dragon_body     |
| 龙表情            | data/<命名空间>/dragon_emote_set|
| 物种缺陷/缺陷     | data/<命名空间>/dragon_penalty  |
| 物种数据          | data/<命名空间>/dragon_species  |
| 成长阶段          | data/<命名空间>/dragon_stage    |
| 自定义弹射物      | data/<命名空间>/projectile_data |
| 龙之生存自定义标签| data/<命名空间>/tags/dragonsurvival |

## 资源包JSON补全

其中`生物模型`仅添加引用

| 支持的功能        | 功能对应的路径                     |
| ----------------- | ---------------------------------- |
| 自定义龙魂图标    | assets/<命名空间>/custom_soul_icons|
| 生物模型          | assets/<命名空间>/geo              |
| 默认组件          | assets/<命名空间>/skin/default_parts|
| 皮肤组件          | assets/<命名空间>/skin/parts       |

## 龙生Mod新增实体NBT

限于精力，只支持用于数据包制作的自定义弹射物

| 支持的实体          | 实体对应的ID                        |
| ------------------- | ----------------------------------- |
| 自定义箭类弹射物    | dragonsurvival:generic_arrow_entity |
| 自定义火球类弹射物  | dragonsurvival:generic_ball_entity  |

## 龙生Mod新增/修改的游戏内容

其中`生物ID`仅添加用于数据包制作的自定义弹射物

| 支持的功能      | 功能对应的ID              |
| --------------- | ------------------------- |
| 生物属性        | attribute                 |
| 生物ID          | entity_type               |
| 实体子谓词      | entity_sub_predicate_type |
| 药水效果        | mob_effect                |
| 粒子效果        | particle_type             |
| 成就触发器      | trigger_type              |

## 附属模组扩展点：Additional Abilities for DS

本包为 [`Additional Abilities for DS`](https://github.com/Dragon-LinFeng) 开放了扩展点。原理是 mcdoc 的
`dispatch` 分派表**是全局的**（存于 `mcdoc/dispatcher` 类别、按资源位置索引），任何文件都能给同一张表加分支，
因此**各组件自身的字段定义放在附属模组自己的 mcdoc 文件里，不需要改动本包**。

本包只负责登记「取值」——因为 `effect_type` / `target_type` / `activation_type` / `trigger_type`
四个索引字段写的是**封闭枚举**，新值不登记就会报 `type mismatch` 且不进补全列表：

| 索引字段 | 新增取值 |
| -------- | -------- |
| `activation_type` | `additional_abilities:charged`、`additional_abilities:optional_charged` |
| `trigger_type` | `additional_abilities:on_block_placed`、`additional_abilities:on_item_consumed`、`additional_abilities:on_ability_cast` |
| `target_type` | `additional_abilities:anti_dragon_breath`、`additional_abilities:annulus`、`additional_abilities:domain` |
| `effect_type`（实体） | `additional_abilities:damage_reflection`、`percentaged_damage`、`simple_screen_vision`、`enchantment_bonus`、`durability` |
| `effect_type`（方块） | `additional_abilities:block_quake`、`extinguish`、`glow` |
| `attribute`（注册表） | `additional_abilities:dragon_breath_restriction` |

> ⚠️ **长期维护**：该枚举是一份「注册表镜像」。附属模组新增 `AAAbility*` 注册项时，
> 需要回来补本包对应的枚举之一，否则新组件仍会报 `type mismatch`。

配套的附属模组侧文件（含 16 个组件的完整字段定义）位于
`Additional Abilities/mcdoc/data/additional_abilities/dragon_ability.mcdoc`。

### 为附属模组挂载本包

> ⚠️ **重要：`env.dependencies` 外链本包**时，本包的 mcdoc 只会被注册成**模块名**，
> **不会**贡献任何结构/枚举/分派（实测：`::mcdoc::data::dragonsurvival::dragon_ability` 下
> 0 个子节点）。因此外链**无法**让附属模组用上本包的类型定义，字段补全不会生效。
>
> **可行做法：把本包的 `mcdoc/` 复制进附属模组自己的项目里**（`Additional Abilities/mcdoc/`），
> 并让 `mcdoc` 成为工作区文件夹（该模组的 `additional_abilities.code-workspace` 已如此配置）。
> 这样它走的是「工作区根」路径，`::data::dragonsurvival::dragon_ability::*` 才会真正有 141 个子节点。

`mcmetaSummaryOverrides` 仍建议指向本包（**属性/指令注册表**走的是另一条路，外链有效）：

```jsonc
"dependencies": [
    "@vanilla-mcdoc",
    "file:///D:/Minecraft_Mod/Dragonsurvival/"   // 龙生本体（data / assets）
],
"mcmetaSummaryOverrides": {
    "registries": { "path": "file:///D:/Minecraft_Mod/dragonsurvival-mcdoc-completion-zh/spyglass/registries.json" },
    "commands":   { "path": "file:///D:/Minecraft_Mod/dragonsurvival-mcdoc-completion-zh/spyglass/commands.json" }
}
```

> ⚠️ **切勿**让附属模组用 `mcmetaSummaryOverrides.registries` 自己写一份覆盖文件：
> Spyglass 的 `core.merge()` 对**数组是整体替换**（只递归 POJO），写一份只会把原版 + 龙生的
> 44 条属性全部冲掉。要加属性请直接改本包的 `spyglass/registries.json`。

### 跨模块引用时的路径前缀

附属模组引用本包已定义的公共结构（如 `DurationInstanceBase` / `EntityTargeting`）时，
`use` 路径以 **`::data::`** 开头，**不带** `mcdoc::`：

```mcdoc
use ::data::dragonsurvival::dragon_ability::DurationInstanceBase
```

原因：附属模组的 `.code-workspace` 把 `mcdoc/` 目录**本身**注册成了一个工作区根
（`"name": "mcdoc", "path": "../../../mcdoc"`），而 Spyglass 的模块路径是相对
**最近的根**计算的，于是 `mcdoc/data/...` → `::data::...`。

实证（Spyglass 符号缓存，即本机 `%LOCALAPPDATA%\spyglassmc-nodejs\Cache\symbols`）：

| 模块名 | 是否存在 | 子节点 |
| ------ | -------- | ------ |
| `::data::dragonsurvival::dragon_ability` | ✅ | 141 |
| `::mcdoc::data::dragonsurvival::dragon_ability` | ✅（外链依赖产生） | **0** |

两者同名但是不同实体 —— 写错了不会报「模块不存在」，只会报
`Identifier "X" does not exist in module "::mcdoc::data::..."`，并连带让所有引用它的
字段（`applied_effects`/`animations`/`sound`/`base` 等）报 `Expected nothing`。


