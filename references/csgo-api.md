# CSGO-API — Static JSON Data

## CSGO-API — Static JSON Data (ByMykel/CSGO-API)

A free, public, static JSON API that maps CS2 internal numeric IDs to
human-readable names, rarities, images, and metadata. No auth required, no
CORS. Hosted on raw.githubusercontent.com. MIT license.

**Base URL:** `https://raw.githubusercontent.com/ByMykel/CSGO-API/main/public/api/{language}/`
**Languages hosted:** `en`, `zh-CN` (28 available locally)
**Image CDN:** `ByMykel/counter-strike-image-tracker`

### Why Use It

`globaloffensive` gives you raw numeric IDs — `def_index`, `paint_index`,
`sticker_id`, `rarity` — but not names, colors, images, or relationships.
CSGO-API bridges that gap:

```
globaloffensive item       CSGO-API lookup
─────────────────────      ──────────────────────────────
def_index: 1               "weapon_deagle" → "Desert Eagle"
paint_index: 44            "AK-47 | Redline"
rarity: 5                  "Covert" / color: #eb4b4b
paint_wear: 0.12           min_float: 0.00, max_float: 0.70
stickers[0].sticker_id: 75 "Sticker | Titan | Katowice 2014"
paint_index + weapon_id    "skin-{weapon_id}_{paint_index}"
```

### All 18 Endpoints

| Endpoint | ID Format | Purpose |
|----------|-----------|---------|
| `all.json` | `{id}` | All items keyed by ID |
| `skins.json` | `skin-{weapon_id}_{paint_index}` | Skins grouped by wear range (`wears[]`) |
| `skins_not_grouped.json` | `skin-{weapon_id}_{paint_index}_{wear}` | Individual skin+wear combos with `market_hash_name` |
| `stickers.json` | `sticker-{def_index}` | Stickers with tournament/team info |
| `sticker_slabs.json` | `sticker_slab-{def_index}` | Sticker slabs (charms) with `original{name,image_inventory}` |
| `keychains.json` | `keychain-{def_index}` | Weapon charms |
| `collections.json` | `collection-set-{name}` | Collections with `contains[]` and `crates[]` |
| `crates.json` | `crate-{def_index}` | Cases/capsules/boxes/souvenir packages with `contains[]`, `contains_rare[]` |
| `keys.json` | `key-{name}` | Case keys with `crates[]` (which they unlock) |
| `collectibles.json` | `collectible-{def_index}` | Service medals, veteran coins |
| `agents.json` | `agent-{def_index}` | Agent skins with `team` (terrorists/ct) |
| `patches.json` | `patch-{def_index}` | Agent patches |
| `graffiti.json` | `graffiti-{def_index}` | Sealed graffiti |
| `music_kits.json` | `music_kit-{def_index}` | Music kits with `exclusive` flag |
| `base_weapons.json` | `base_weapon-{name}` | Default weapon models |
| `highlights.json` | `aus2025_{name}` | Tournament highlights with `video` URL |
| `inventory.json` | `{skins: {weapon_id: {paint_index}}}` | Inventory lookup — skins by `weapon_id→paint_index`, other items by `def_index` |

### Common Object Shapes

Every endpoint returns arrays or objects with these recurring patterns:

**Rarity (always present on rarity-bearing items):**
```js
{ id: "rarity_ancient_weapon", name: "Covert", color: "#eb4b4b" }
```

**Rarity ID → Name / Color Table:**

| Rarity ID | Display Name | Color |
|-----------|-------------|-------|
| `rarity_common` | Base Grade | `#b0c3d9` |
| `rarity_uncommon` / `rarity_uncommon_weapon` | Industrial Grade | `#5e98d9` |
| `rarity_rare` / `rarity_rare_weapon` | Mil-Spec / High Grade | `#4b69ff` |
| `rarity_mythical` | Restricted / Remarkable | `#8847ff` |
| `rarity_legendary` / `rarity_legendary_character` | Classified / Superior | `#d32ce6` |
| `rarity_ancient` / `rarity_ancient_weapon` | Covert / Extraordinary | `#eb4b4b` |
| `rarity_immortal` | Gold | `#ffd700` |

**Cross-references (used in skins, stickers, keychains, agents, etc.):**
```js
collections: [{ id: "collection-set-overpass", name: "The Overpass Collection", image: "..." }]
crates: [{ id: "crate-4028", name: "ESL One Cologne 2014 Overpass Souvenir Package", image: "..." }]
```

### Skin Object (`skins.json`)
```js
{
  id: "skin-65604",              // skin-{weapon_id}_{paint_index}
  name: "Desert Eagle | Urban DDPAT",
  description: "...",
  weapon: { id: "weapon_deagle", weapon_id: 1, name: "Desert Eagle" },
  category: { id: "csgo_inventory_weapon_category_pistols", name: "Pistols" },
  pattern: { id: "hy_ddpat_urb", name: "Urban DDPAT" },
  min_float: 0.06, max_float: 0.8,  // Valid wear range
  rarity: { id: "rarity_uncommon_weapon", name: "Industrial Grade", color: "#5e98d9" },
  stattrak: false, souvenir: true,
  paint_index: "17",              // String! Maps to CSOEconItem attribute 6
  wears: [{ id: "SFUI_InvTooltip_Wear_Amount_0", name: "Factory New" }, ...],
  collections: [...], crates: [...],
  team: { id: "both", name: "Both Teams" },
  legacy_model: true,
  image: "https://raw.githubusercontent.com/ByMykel/counter-strike-image-tracker/main/..."
}
```

### Skin Not Grouped (`skins_not_grouped.json`)
Same as above plus:
```js
{
  id: "skin-65604_0",             // skin-{weapon_id}_{paint_index}_{wear_index}
  skin_id: "skin-65604",          // Parent grouped skin ID
  wear: { id: "SFUI_InvTooltip_Wear_Amount_0", name: "Factory New" },
  stattrak: false, souvenir: false,  // Explicit per-variant
  market_hash_name: "Desert Eagle | Urban DDPAT (Factory New)",
  style: { id: 2, name: "Hydrographic", url: "..." }
}
```

### Sticker Object (`stickers.json`)
```js
{
  id: "sticker-75",               // sticker-{def_index}
  name: "Sticker | Titan | Katowice 2014",
  description: "...",
  rarity: { id: "rarity_rare", name: "High Grade", color: "#4b69ff" },
  crates: [...],
  tournament_event: "Katowice 2014",
  tournament_team: "Titan",
  type: "Team",                   // Team, Player, Other
  market_hash_name: "Sticker | Titan | Katowice 2014",
  effect: "Other",
  image: "..."
}
```

### Sticker Slab Object (`sticker_slabs.json`)
```js
{
  id: "sticker_slab-75",
  name: "Sticker Slab | Titan | Katowice 2014",
  def_index: "75",                // String!
  rarity: { ... },
  crates: [...], collections: [],
  type: "Team",
  tournament: { id: 3, name: "2014 EMS One Katowice" },
  team: { id: 27, tag: "TIT", geo: "FR", name: "Titan" },
  original: { name: "kat2014_titan", image_inventory: "econ/stickers/emskatowice2014/titan_1355_37" },
  market_hash_name: "Sticker Slab | Titan | Katowice 2014",
  effect: "Other",
  image: "..."
}
```

### Crate Object (`crates.json`)
```js
{
  id: "crate-4904",
  name: "Kilowatt Case",
  type: "Case",                   // Case, Capsule, Graffiti Box, Music Kit Box, Souvenir Package
  first_sale_date: "2024-01-16",
  contains: [{ id, name, rarity, paint_index, image }, ...],
  contains_rare: [{ id, name, rarity, paint_index: null, phase: null, image }, ...],
  market_hash_name: "Kilowatt Case",
  rental: true,                   // CS2 rental feature
  image: "...",
  model_player: "models/props/crates/csgo_drop_crate_community_33.vmdl",
  loot_list: { name: "★ Kukri Knife ★", footer: "...", image: "..." }
}
```

### Collection Object (`collections.json`)
```js
{
  id: "collection-set-community-3",
  name: "The Huntsman Collection",
  crates: [{ id, name, image }, ...],
  contains: [{ id, name, rarity, paint_index, image }, ...],
  image: "..."
}
```

### Agent, Keychain, Patch, Music Kit, Graffiti, Collectible
All follow the same pattern: `{ id, name, description, rarity, market_hash_name, image }` plus type-specific fields:
- **Agents:** `team: { id: "terrorists"|"ct", name }`
- **Keychains:** `collections[]`
- **Patches:** rarity only
- **Music Kits:** `exclusive: true|false`
- **Graffiti:** `crates[]`
- **Collectibles:** `type`, `genuine`

### Inventory Lookup (`inventory.json`)

```js
{
  skins: {
    "1": {                        // weapon_id (from CSOEconItem.def_index → weapon mapping)
      "17": {                     // paint_index (from CSOEconItem attribute 6)
        name: "Desert Eagle | Urban DDPAT",
        rarity: { id: "rarity_uncommon_weapon", name: "Industrial Grade", color: "#5e98d9" },
        marketable: true,
        image: "..."
      }
    }
  },
  crates: { "1210": { name, rarity, marketable, image }, ... },
  stickers: { "75": { ... }, ... },
  keychains: { "1": { ... }, ... },
  music_kits: { "39": { ... }, ... },
  agents: { "4613": { ... }, ... },
  graffiti: { "1654": { ... }, ... },
  collectibles: { "874": { ... }, ... },
  patches: { "5126": { ... }, ... },
  highlights: { "aus2025_...": { ... }, ... },
  tools: { ... }                  // Name tags, storage units, etc.
}
```

### How to Use with globaloffensive

**Resolve a skin from inventory:**
```js
const API_BASE = 'https://raw.githubusercontent.com/ByMykel/CSGO-API/main/public/api/en';

async function resolveSkin(item, fetchFn = fetch) {
  const { paint_index, def_index } = item;  // from CSOEconItem
  // Look up by weapon_id (def_index) and paint_index
  // Option 1: Full skins_not_grouped.json scan
  const res = await fetchFn(`${API_BASE}/skins_not_grouped.json`);
  const skins = await res.json();
  return skins.find(s => s.weapon?.weapon_id === def_index && s.paint_index === String(paint_index));
}

async function resolveSticker(stickerId, fetchFn = fetch) {
  const res = await fetchFn(`${API_BASE}/stickers.json`);
  const stickers = await res.json();
  return stickers.find(s => s.id === `sticker-${stickerId}`);
}

async function resolveRarity(rarityNum) {
  // rarity 5 (from CEconItemPreviewDataBlock) → "Covert" + "#eb4b4b"
  const res = await fetchFn(`${API_BASE}/skins.json`);
  const skins = await res.json();
  const match = skins.find(s => s.rarity && s.weapon?.weapon_id === 1);
  // Better: use the rarity table built into this skill
}
```

**Build a local inventory resolver:**
```js
// Download once, use many times
async function buildResolver(fetchFn = fetch) {
  const [skinsNG, stickers, keychains, crates] = await Promise.all([
    fetchFn(`${API_BASE}/skins_not_grouped.json`).then(r => r.json()),
    fetchFn(`${API_BASE}/stickers.json`).then(r => r.json()),
    fetchFn(`${API_BASE}/keychains.json`).then(r => r.json()),
    fetchFn(`${API_BASE}/crates.json`).then(r => r.json()),
  ]);

  const skinByPaintIdx = {};
  for (const s of skinsNG) {
    if (!skinByPaintIdx[s.paint_index]) skinByPaintIdx[s.paint_index] = [];
    skinByPaintIdx[s.paint_index].push(s);
  }

  const stickerById = {};
  for (const s of stickers) stickerById[s.id] = s;

  return {
    resolveSkin(paintIndex, weaponId) {
      return (skinByPaintIdx[String(paintIndex)] || [])
        .find(s => s.weapon?.weapon_id === weaponId);
    },
    resolveSticker(stickerId) {
      return stickerById[`sticker-${stickerId}`];
    },
  };
}
```

### Important Details

- **`paint_index` is a STRING** in CSGO-API, not a number — always use `String()` when comparing.
- **`def_index`** in the API is also a string. Compare with care.
- **`skins.json` groups by wear ranges** — use `skins_not_grouped.json` for exact `wear+stattrak+souvenir` combos.
- **Images are in a separate repo** (`ByMykel/counter-strike-image-tracker`) — all URLs point there.
- **The `inventory.json` endpoint** is optimized for fast `weapon_id → paint_index → skin` lookup (no array scan needed).
- **Only `en` and `zh-CN` are hosted** — for other languages, clone and build locally with `node update.js --languages <codes>`.
- **28 language codes available:** `en, zh-CN, pt-BR, ru, es-ES, bg, cs, da, nl, fi, fr, de, el, hu, it, ja, ko, es-MX, no, pl, pt-PT, ro, sv, zh-TW, th, tr, uk, vi`

---


# 本地库存匹配方案 (cs2tradetool v2)

## 本地库存匹配方案 (cs2tradetool v2)

基于 `cs2tradetool/index.js` 验证过的生产级方案。核心思想：**`def_index` 是容器型号不是物品身份**，
真正的身份信息在 GC 返回的 `stickers[0].sticker_id` 和 `paint_index` 中。

### 数据源

```
all.json (71MB, 45,756条, 单一文件)
  ├─ skin-{hash}_{wear}  → paint_index + weapon_id → 皮肤名+稀有度+图片
  ├─ sticker-{id}        → O(1) 直接查找贴纸名
  ├─ crate-{id}          → O(1) 直接查找箱子名
  ├─ agent-{id}          → 探员
  ├─ keychain-{id}       → 挂件
  └─ collectible-{id}    → 收藏品
+
items_game.txt + csgo_english.txt  (VDF, 6.6MB)
  └─ 断层回退: CSGO-API 缺失的旧贴纸/涂鸦 (如 1695-1738 范围)
```

**`all.json` 下载:** `node generate_dict.js` (约 70MB, 从 raw.githubusercontent.com)

### 三路分派规则

```
CSOEconItem (raw GC data)
  │
  ├─ item.stickers[0] exists?
  │   YES → 贴纸/涂鸦容器。身份 = stickers[0].sticker_id
  │         查找: all.json "sticker-{sticker_id}"
  │         失败 → items_game.txt 断层回退 (英文名)
  │         ★ 绝不用 def_index 命名!
  │
  ├─ item.paint_index != null && item.paint_index !== 0?
  │   YES → 武器皮肤。Key = paint_index + "|" + weapon_id
  │         查找: all.json skin 条目, 获取中文名/稀有度/图片/min_float/max_float
  │         磨损: 依据 0.07/0.15/0.38/0.45 阈值分 5 档
  │
  ├─ isKnownWeapon(def_index)?
  │   YES → 原版武器 (无涂装, paint_index=0)
  │
  └─ 否则 → 箱子/收藏品/探员/涂鸦等
            从 all.json 按类别精度计分匹配
            (all.json 条目多的类别如 sticker=10441 权重极低)
```

### 类别精度计分

`def_index` 在所有物品类型间共享 (如 4236 同时是箱子+贴纸)。
用精度基数消歧义: `10000 / 该类别条目数`。越少条目的类别越精准:

| 类别 | 条目数 | 精度分 | 说明 |
|------|-------|--------|------|
| tool | 4 | 2500 | 最精准 |
| key | 39 | 256 | |
| agent | 63 | 159 | |
| keychain | 78 | 128 | |
| patch | 112 | 89 | |
| music_kit | 189 | 53 | |
| crate | 478 | 20.9 | |
| collectible | 627 | 15.9 | |
| graffiti | 2111 | 4.7 | |
| sticker_slab | 10441 | 0.96 | 权重最低, 永远不首选 |
| sticker | 10441 | 0.96 | 权重最低 |

### `def_index` 歧义验证 (实战数据)

以 GC 实际数据验证——所有有歧义的 def_index 案例分析:

```
def_index=1209  (514件, 全部有 sticker[0]):
  all.json def=1209 → "Zeus(金色)"          ← 错, def_index 查的
  all.json sticker_id=4531 → "滑翔高手"       ← 对, 真实身份
  all.json sticker_id=4683 → "远古保卫者"     ← 对
  → 102种不同sticker_id → def_index是容器, sticker_id是身份

def_index=1348  (14件, 全部有 sticker[0]):
  → 全部 sticker_id 在 CSGO-API 断层(1695-1738)
  → items_game.txt 回退: "Cocky", "Mr. Teeth", ...
  → 这些是旧版涂鸦/喷漆包

def_index=1349  (4件, 全部有 sticker[0]):
  → sticker_id 1737 = "GTG" (items_game.txt 回退)
  → sticker_id 1718 = "Sheriff"

def_index=4236  (49件, 全部无 sticker[0]):
  all.json crate-4236 → "伽玛武器箱"          ← 对, crate精度分20.9
  all.json sticker-4236 → "nitr0(金色)"        ← 噪音, 权重0.96
  → 无贴纸属性 → crate 优先
```

### 关键实现 (CsgoResolver 类)

```js
class CsgoResolver {
  constructor() {
    const all = JSON.parse(fs.readFileSync('data/all.json', 'utf8'));
    // 皮肤: "paint_index|weapon_id" → entry
    this._skinByKey = {};
    // 贴纸: sticker_id → entry
    this._stickerById = {};
    // 所有非皮肤类别: "type|def_index" → entry
    this._byTypeAndDef = {};
    // 已知武器 ID 集合
    this._weaponIds = new Set();

    for (const [key, item] of Object.entries(all)) {
      const type = key.split('-')[0];
      if (type === 'skin' && item.paint_index && item.weapon?.weapon_id) {
        this._skinByKey[item.paint_index + '|' + item.weapon.weapon_id] = item;
        this._weaponIds.add(item.weapon.weapon_id);
      }
      if (type === 'sticker') {
        this._stickerById[key.replace('sticker-', '')] = item;
      }
      if (item.def_index) {
        (this._byTypeAndDef[type] ??= {})[item.def_index] = item;
      }
    }
  }
}
```

### 断层回退

all.json 的贴纸数据在 1695-1738 等范围有断层 (CSGO-API 未收录的旧物品)。
这些由 `items_game.txt` (VDF格式, 从 Valve 官方游戏文件提取) 填补:

```js
// VDF 解析 → sticker_kits 段 → 取 item_name → 英文翻译 → 贴纸名
const gameVdf = VDF.parse(fs.readFileSync('data/game/items_game.txt'));
const stickerKits = gameVdf.items_game.sticker_kits;
const name = translate(stickerKits[stickerId].item_name); // e.g. "GTG", "Sheriff"
```

### 单复数陷阱

all.json 的 key 前缀使用**单数形式**: `crate`, `agent`, `graffiti`, `collectible` 等。
代码中的类别搜索必须匹配单数形式，否则 `resolveByDef('crates', 4236)` 永远找不到
索引中的 `byTypeAndDef['crate']`。这是从多文件迁移到 all.json 时最常见的 bug。

---
