# 掉落物配方

**请注意，在阅读本章节之前，请先学习数据包的制作与使用，不懂的话请阅读WIKI中相关的内容**

同样的，我们先创建目录，这里我们以方块的掉落举例。

```
resources
├── META-INF
│   └── mods.toml
├── assets
├── data
│   ├── minecraft
│   └── neutrino
│       ├── loot_tables
│       │   └── blocks
│       └── recipes
│           ├── obsidian_block.json
│           └── obsidian_ingot.json
└── pack.mcmeta
```

在`loot_tables/blocks`下创建和你方块注册名相同的Json文件。这里我们以我们创建的`obsidian_block`为例。

`obsidian_block.json`：

```json
{
  "type": "minecraft:block",
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "neutrino:obsidian_block"
        }
      ],
      "conditions": [
        {
          "condition": "minecraft:survives_explosion"
        }
      ]
    }
  ]
}
```

同样的，这里你可以通过`neutrino:obsidian_block`来指定你的物品。

可能很多读者，对于原版的`Loot_table`很难理解，这里我可以举个例子，你就把`Loot_table`当作是在赌场里抽奖，里面配置的信息不过就是奖品是什么，可以抽几次，什么条件下可以抽，什么情况下可以多抽几次。