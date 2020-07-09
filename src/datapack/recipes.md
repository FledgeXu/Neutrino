# 配方

**请注意，在阅读本章节之前，请先学习数据包的制作与使用，不懂的话请阅读WIKI中相关的内容**

在这节中，我们将要来学习如何添加配方，这里我们将以合成配方，和熔炼配方举例。

正如原版的配方是放在数据包内，我们Mod的配方也应该放在我们的自己的数据包内，首先第一步就是创建我们Mod的数据包。请按照下面的结构性在你的`resources`文件夹下创建目录。

```
resources
├── META-INF
│   └── mods.toml
├── assets
├── data
│   ├── minecraft
│   └── neutrino
│       └── recipes
└── pack.mcmeta
```

其中的`data/neutrino`就是你自己的数据包了。

接下在`recipes`文件夹下创建内容。这里你创建的文件名随意，一般情况下我都会写成和物品注册名同名。

接下来举两个例子。

`obsidian_block.json`:

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "###",
    "###",
    "###"
  ],
  "key": {
    "#": {
      "item": "neutrino:obsidian_ingot"
    }
  },
  "result": {
    "item": "neutrino:obsidian_block",
    "count": 1
  }
}
```

`obsidian_ingot.json`

```json
{
  "type": "minecraft:smelting",
  "ingredient": {
    "item": "minecraft:obsidian"
  },
  "result": "neutrino:obsidian_ingot",
  "experience": 0.35,
  "cookingtime": 200
}
```

可以看到，这里和原版的数据包最大的不同就是我们通过类似`neutrino:obsidian_ingot`指定了，Mod中的物品，而通过`minecraft:obsidian`指定了原版的物品。

创建完成以后目录结构如下:

```
resources
├── META-INF
│   └── mods.toml
├── assets
├── data
│   ├── minecraft
│   └── neutrino
│       └── recipes
│           ├── obsidian_block.json
│           └── obsidian_ingot.json
└── pack.mcmeta
```

