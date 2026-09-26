# Blockman Common Lua Reference

Arguments and usage for the shared Lua layer of the Blockman engine. Entries give the signature, the role of each argument, and real call examples taken from the game script tree. Source locations are intentionally omitted.

- **Scope:** the common framework only (game-mode trees excluded)
- **Generated:** 2026-09-26 18:02

| Statistic | Count |
| --- | ---: |
| **Functions** | **402** |
| Unique signatures | 401 |
| Engine host methods | 22 |
| Engine events | 9 |
| Engine data classes | 2 |
| Entries with usage examples | 248 |

---

## Contents

- [1. Overview](#1-overview)
- [2. How to read the entries](#2-how-to-read-the-entries)
- [3. Engine Host API](#3-engine-host-api)
  - [3.1 HostApi](#31-hostapi)
  - [3.2 Events](#32-events)
  - [3.3 Data classes](#33-data-classes)
  - [3.4 Other host objects](#34-other-host-objects)
- [4. Function reference](#4-function-reference)
- [5. Constants and enums](#5-constants-and-enums)
- [6. Index](#6-index)

---

## 1. Overview

The common layer sits between the engine host and the individual game modes. It owns four concerns, and a mode normally uses all four without touching the host directly:

1. **Event plumbing.** Events are subscribed once and then dispatched to whatever mode is running, so a mode never calls `registerCallBack` itself.
2. **Client protocol.** Message construction and the HTTP calls that carry them to the game server.
3. **Persistence and world state.** Wrappers over the host database, player objects and world accessors.
4. **Cross-cutting rules.** Ranking, player attributes, progression, seasonal resets and the tick scheduler.

Because the common layer is a library rather than an application, most of it is called by game modes rather than by itself. Usage examples below are therefore harvested from across the script tree, not just from the common layer.

## 2. How to read the entries

Each entry shows the signature, an argument table, and how the function is actually called. Argument roles are inferred from the parameter name, so treat them as guidance rather than a type declaration - Lua is dynamically typed and enforces nothing.

A name written `Owner:name(...)` uses the colon form: `name` is the first parameter and holds the receiver. A name written `Owner.name(...)` uses the dot form and takes no receiver. `Owner` is normally a singleton table, so these are called without constructing anything.

| Inferred role | Meaning |
| --- | --- |
| `receiver` | the implicit `self` |
| `object` | a class or table |
| `identifier` | an id or key |
| `number` | a count, time or amount |
| `boolean` | a flag |
| `string` | text |
| `table` | a list or map |
| `function` | a callback |
| `varargs` | any number of extra values |
| `value` | anything else |

---

## 3. Engine Host API

Injected into the Lua VM by the host. Implemented natively inside the game binary, so these are not Lua you can read - they are the boundary the common layer is built on. Only symbols the common layer actually uses are listed.

### 3.1 HostApi

The primary engine bridge, a singleton. Call it directly; never construct it. 22 methods are used by the common layer.

<a id="hostapi-addcoinmapping"></a>

#### `HostApi.addCoinMapping (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.addCoinMapping(v.coinId, v.itemId)
```

<a id="hostapi-addcommodity"></a>

#### `HostApi.addCommodity (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.addCommodity(index, type, goods)
```

<a id="hostapi-broadcastmsg"></a>

#### `HostApi.broadCastMsg (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.broadCastMsg(rakssid, 0, 5, msg)
HostApi.broadCastMsg(rakssid, 1, time, msg)
HostApi.broadCastMsg(rakssid, 2, time, msg)
```

<a id="hostapi-broadcastmsgbytype"></a>

#### `HostApi.broadCastMsgByType (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.broadCastMsgByType(rakssid, 0, 5, msg, args or "")
HostApi.broadCastMsgByType(rakssid, 1, time, msg, args or "")
HostApi.broadCastMsgByType(rakssid, 2, time, msg, args or "")
```

<a id="hostapi-changecommoditylist"></a>

#### `HostApi.changeCommodityList (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.changeCommodityList(entityId, rakssId, index)
```

<a id="hostapi-getdbdata"></a>

#### `HostApi.getDBData (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.getDBData(userId, subKey)
```

<a id="hostapi-log"></a>

#### `HostApi.log (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.log(string.format("PlayerManager:addPlayer(PlayerCount=%d)", self:getPlayerCount()))
HostApi.log(string.format("PlayerManager:subPlayer(PlayerCount=%d)", self:getPlayerCount()))
HostApi.log(VarDump(ServerWorld))
```

<a id="hostapi-platformorderdispose"></a>

#### `HostApi.platformOrderDispose (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.platformOrderDispose(userId, WebService.BlockymodBaseUrl, orderId, isConsume or false)
```

<a id="hostapi-resetpos"></a>

#### `HostApi.resetPos (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.resetPos(self.rakssid, pos.x, pos.y + 0.5, pos.z)
HostApi.resetPos(self.rakssid, self.team.initPos.x, self.team.initPos.y + 0.5, self.team.initPos.z)
HostApi.resetPos(self.rakssid, GameConfig.initPos.x, GameConfig.initPos.y + 0.5, GameConfig.initPos.z)
```

<a id="hostapi-resetposwithyaw"></a>

#### `HostApi.resetPosWithYaw (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.resetPosWithYaw(self.rakssid, pos.x, pos.y + 0.5, pos.z, yaw)
HostApi.resetPosWithYaw(self.rakssid, unpack(self._rpos))
```

<a id="hostapi-sendappexpresult"></a>

#### `HostApi.sendAppExpResult (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.sendAppExpResult(userId, curLv, curLv, 0, curExp, curExp, upExp, status)
HostApi.sendAppExpResult(userId, expInfo.curLv, expInfo.toLv, expInfo.addExp, expInfo.curExp, expInfo.toExp, upExp, status)
```

<a id="hostapi-sendenterothergame"></a>

#### `HostApi.sendEnterOtherGame (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.sendEnterOtherGame(rakssid, gameType, targetId, mapId)
```

<a id="hostapi-sendgamestatus"></a>

#### `HostApi.sendGameStatus (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.sendGameStatus(clientPeer:getRakssid(), status)
HostApi.sendGameStatus(0, 1)
HostApi.sendGameStatus(0, 2)
```

<a id="hostapi-sendgameover"></a>

#### `HostApi.sendGameover (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.sendGameover(clientPeer:getRakssid(), IMessages:msgGamePlayerIsEnough(), GameOverCode.PlayerIsEnough)
HostApi.sendGameover(clientPeer:getRakssid(), IMessages:msgGameOver(), GameOverCode.GameOver)
HostApi.sendGameover(clientPeer:getRakssid(), IMessages:msgGameHasStarting(), GameOverCode.GameStarted)
```

<a id="hostapi-sendgotoothergame"></a>

#### `HostApi.sendGotoOtherGame (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.sendGotoOtherGame(rakssid, targetId, gameType, mapId)
```

<a id="hostapi-sendkillmsg"></a>

#### `HostApi.sendKillMsg (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.sendKillMsg(rakssid, json.encode(data))
```

<a id="hostapi-sendrankdata"></a>

#### `HostApi.sendRankData (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.sendRankData(rakssid, entityId, result)
HostApi.sendRankData(rakssid, v.entityId, data)
HostApi.sendRankData(self.rakssid, GameRank:fetch_rank_npc(tonumber(id)), json.encode(data))
```

<a id="hostapi-sendstartgame"></a>

#### `HostApi.sendStartGame (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.sendStartGame(PlayerManager:getPlayerCount())
HostApi.sendStartGame(playerCount)
HostApi.sendStartGame(self.prestartPlayerCount)
```

<a id="hostapi-setdbdata"></a>

#### `HostApi.setDBData (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.setDBData(userId, subKey, data, immediate or false)
HostApi.setDBData(player.userId, DBDataListener.TAG_RANCHER_EXPLORE_INFO, json.encode(data), immediate or false)
```

<a id="hostapi-setdblogdata"></a>

#### `HostApi.setDBLogData (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.setDBLogData(userId, type, action, data, immediate or true)
```

<a id="hostapi-setservergametype"></a>

#### `HostApi.setServerGameType (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.setServerGameType(GameType)
HostApi.setServerGameType(GameMatch.gameType)
```

<a id="hostapi-stopserver"></a>

#### `HostApi.stopServer (...)`

Singleton method on the engine bridge. Argument names are not visible from the call sites, so only the call shape is shown.

```lua
HostApi.stopServer()
```

### 3.2 Events

Subscribe with `registerCallBack`. The base listener wires the core set at start-up, so a mode normally does not subscribe itself.

| Argument | Role | Meaning |
| --- | --- | --- |
| `fn` | `function` | Handler invoked when the event fires. |

<a id="event-gameinitevent"></a>

#### `GameInitEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
GameInitEvent.registerCallBack(self.onGameInit)
```

<a id="event-getdatafromdbevent"></a>

#### `GetDataFromDBEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
GetDataFromDBEvent.registerCallBack(self.onGetDataFromDB)
```

<a id="event-httpresponseevent"></a>

#### `HttpResponseEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
HttpResponseEvent.registerCallBack(self.onHttpResponse)
```

<a id="event-playerdynamicattrevent"></a>

#### `PlayerDynamicAttrEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
PlayerDynamicAttrEvent.registerCallBack(self.onPlayerDynamicAttr)
```

<a id="event-playerfirstspawnevent"></a>

#### `PlayerFirstSpawnEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
PlayerFirstSpawnEvent.registerCallBack(self.onPlayerFirstSpawn)
```

<a id="event-playerloginevent"></a>

#### `PlayerLoginEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
PlayerLoginEvent.registerCallBack(self.onPlayerLogin)
```

<a id="event-playerlogoutevent"></a>

#### `PlayerLogoutEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
PlayerLogoutEvent.registerCallBack(self.onPlayerLogout)
```

<a id="event-playerreadyevent"></a>

#### `PlayerReadyEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
PlayerReadyEvent.registerCallBack(self.onPlayerReady)
```

<a id="event-playerrespawnevent"></a>

#### `PlayerRespawnEvent.registerCallBack (fn)`

Engine event. Fires from the host during the match lifecycle. The common layer subscribes the core set at start-up, so a mode does not normally subscribe itself.

```lua
PlayerRespawnEvent.registerCallBack(self.onPlayerRespawn)
```

### 3.3 Data classes

Constructed with `new`. Fields are filled in by the host, so read them but do not expect writes to persist.

<a id="class-vector3"></a>

#### `Vector3.new (...)`

Engine data class.

```lua
Vector3.new()
Vector3.new()
```

<a id="class-vector3i"></a>

#### `Vector3i.new (...)`

Engine data class.

```lua
Vector3i.new()
Vector3i.new()
```

### 3.4 Other host objects

| Object | Methods observed |
| --- | --- |
| `CommonTask` | `start` |
| `HttpRequest` | `asyncGet`, `asyncPost` |
| `Item` | `getItemById` |
| `SecTimer` | `createTimer`, `startTimer`, `stopTimer` |
| `Vector3` | `new` |
| `Vector3i` | `new` |

---

## 4. Function reference

All 402 functions, ordered alphabetically by name.


### A

<a id="fn-appprops-hasprop"></a>

#### `AppProps:hasProp (player, propId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |
| `propId` | `identifier` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-appprops-init"></a>

#### `AppProps:init (props)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `props` | `value` | The instance this method was called on. |

Used as:

```lua
AppProps:init(self:getCsvConfig(string.gsub(configPath, "config.yml", "AppProps.csv")))
AppProps:init(CsvUtil.loadCsvFile((string.gsub(configPath, "config.yml", "AppProps.csv")), 2))
AppProps:init(self:getCsvConfig(string.gsub(configPath, "config.yml", "XAppProps.csv")))
```

<a id="fn-appprops-initprops"></a>

#### `AppProps:initProps (props)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `props` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-appprops-useappprop"></a>

#### `AppProps:useAppProp (player, prop)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |
| `prop` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-appprops-useappprops"></a>

#### `AppProps:useAppProps (player, isFirst)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |
| `isFirst` | `boolean` |  |

Used as:

```lua
AppProps:useAppProps(self, self.respawnTimes == 0)
```


### B

<a id="fn-bitwise"></a>

#### `BITWISE (x, y, matrix)`

> performs the bitwise operation specified by truth matrix on two numbers.

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `x` | `value` |  |
| `y` | `value` |  |
| `matrix` | `value` |  |

Used as:

```lua
BITWISE(bytes[7], 0x0f, MATRIX_AND)
BITWISE(bytes[7], 0x40, MATRIX_OR)
BITWISE(bytes[7], 0x3f, MATRIX_AND)
```

<a id="fn-baselistener-ongameinit"></a>

#### `BaseListener.onGameInit (GamePath, ver, serverworld, config, initPos)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `GamePath` | `object` | Pass a table or class value. |
| `ver` | `value` |  |
| `serverworld` | `value` |  |
| `config` | `value` |  |
| `initPos` | `value` |  |

Never called directly. It is registered as a callback, so the host invokes it:

```lua
BaseListener.registerCallBack (self.onGameInit)
GameInitEvent.registerCallBack (self.onGameInit)
```

<a id="fn-baselistener-ongetdatafromdb"></a>

#### `BaseListener.onGetDataFromDB (userId, subKey, data)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` |  |
| `subKey` | `identifier` |  |
| `data` | `table` |  |

Never called directly. It is registered as a callback, so the host invokes it:

```lua
BaseListener.registerCallBack (self.onGetDataFromDB)
GetDataFromDBEvent.registerCallBack (self.onGetDataFromDB)
```

<a id="fn-baselistener-onplayerdynamicattr"></a>

#### `BaseListener.onPlayerDynamicAttr (platformId, class, team, region, pioneer, vip)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `platformId` | `identifier` |  |
| `class` | `value` |  |
| `team` | `value` |  |
| `region` | `value` |  |
| `pioneer` | `value` |  |
| `vip` | `value` |  |

Never called directly. It is registered as a callback, so the host invokes it:

```lua
PlayerDynamicAttrEvent.registerCallBack (self.onPlayerDynamicAttr)
```

<a id="fn-baselistener-onplayerfirstspawn"></a>

#### `BaseListener.onPlayerFirstSpawn (clientPeer)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `clientPeer` | `value` |  |

Never called directly. It is registered as a callback, so the host invokes it:

```lua
PlayerFirstSpawnEvent.registerCallBack (self.onPlayerFirstSpawn)
```

<a id="fn-baselistener-onplayerlogin"></a>

#### `BaseListener.onPlayerLogin (clientPeer)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `clientPeer` | `value` |  |

Never called directly. It is registered as a callback, so the host invokes it:

```lua
BaseListener.registerCallBack (self.onPlayerLogin)
PlayerLoginEvent.registerCallBack (self.onPlayerLogin)
```

<a id="fn-baselistener-onplayerlogout"></a>

#### `BaseListener.onPlayerLogout (userId)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` |  |

Used as:

```lua
BaseListener.onPlayerLogout(self.userId)
```

<a id="fn-baselistener-onplayerready"></a>

#### `BaseListener.onPlayerReady (rakssid)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |

Never called directly. It is registered as a callback, so the host invokes it:

```lua
BaseListener.registerCallBack (self.onPlayerReady)
PlayerReadyEvent.registerCallBack (self.onPlayerReady)
```

<a id="fn-baselistener-onplayerrespawn"></a>

#### `BaseListener.onPlayerRespawn (clientPeer)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `clientPeer` | `value` |  |

Never called directly. It is registered as a callback, so the host invokes it:

```lua
BaseListener.registerCallBack (self.onPlayerRespawn)
PlayerRespawnEvent.registerCallBack (self.onPlayerRespawn)
```

<a id="fn-baselistener-registercallback"></a>

#### `BaseListener.registerCallBack (event, func)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `event` | `value` |  |
| `func` | `function` |  |

Used as:

```lua
BaseListener.registerCallBack(GameInitEvent, self.onGameInit)
BaseListener.registerCallBack(PlayerLoginEvent, self.onPlayerLogin)
BaseListener.registerCallBack(PlayerLogoutEvent, self.onPlayerLogout)
```

<a id="fn-baselistener-init"></a>

#### `BaseListener:init ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
BaseListener:init()
```

<a id="fn-basemain-getgametype"></a>

#### `BaseMain:getGameType ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
BaseMain:getGameType()
```

<a id="fn-basemain-setgametype"></a>

#### `BaseMain:setGameType (GameType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `GameType` | `object` | The instance this method was called on. |

Used as:

```lua
BaseMain:setGameType(GameMatch.gameType)
BaseMain:setGameType(define.gameType)
BaseMain:setGameType(Match.gameType)
```


### C

<a id="fn-cbaseplayer-addbackpackcapacity"></a>

#### `CBasePlayer:addBackpackCapacity (capacity)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `capacity` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addcurrency"></a>

#### `CBasePlayer:addCurrency (currency)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `currency` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addcustomeffect"></a>

#### `CBasePlayer:addCustomEffect (name, effectName, duration)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |
| `effectName` | `value` |  |
| `duration` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addechantmentitem"></a>

#### `CBasePlayer:addEchantmentItem (id, num, damage, enchantmentId, enchantmentLevel)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |
| `num` | `number` |  |
| `damage` | `value` |  |
| `enchantmentId` | `identifier` |  |
| `enchantmentLevel` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addechantmentsitem"></a>

#### `CBasePlayer:addEchantmentsItem (id, num, damage, enchantments)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |
| `num` | `number` |  |
| `damage` | `value` |  |
| `enchantments` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addeffect"></a>

#### `CBasePlayer:addEffect (id, time, level)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |
| `time` | `number` |  |
| `level` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addgunbulletnum"></a>

#### `CBasePlayer:addGunBulletNum (gunId, bulletNum)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `gunId` | `identifier` | The instance this method was called on. |
| `bulletNum` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addgundamage"></a>

#### `CBasePlayer:addGunDamage (gunId, damage)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `gunId` | `identifier` | The instance this method was called on. |
| `damage` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addgunitem"></a>

#### `CBasePlayer:addGunItem (id, num, damage, bullets, isReverse)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |
| `num` | `number` |  |
| `damage` | `value` |  |
| `bullets` | `boolean` |  |
| `isReverse` | `boolean` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addgunitemtoenderchest"></a>

#### `CBasePlayer:addGunItemToEnderChest (slot, id, num, damage, bullets)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `slot` | `value` | The instance this method was called on. |
| `id` | `identifier` |  |
| `num` | `number` |  |
| `damage` | `value` |  |
| `bullets` | `boolean` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addhealth"></a>

#### `CBasePlayer:addHealth (hp)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `hp` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-additem"></a>

#### `CBasePlayer:addItem (id, num, damage, isReverse)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |
| `num` | `number` |  |
| `damage` | `value` |  |
| `isReverse` | `boolean` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-additemtoenderchest"></a>

#### `CBasePlayer:addItemToEnderChest (slot, id, num, damage)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `slot` | `value` | The instance this method was called on. |
| `id` | `identifier` |  |
| `num` | `number` |  |
| `damage` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addnightvisionpotioneffect"></a>

#### `CBasePlayer:addNightVisionPotionEffect (seconds)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `seconds` | `number` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-addvehicles"></a>

#### `CBasePlayer:addVehicles (vehicles)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `vehicles` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-buildshowname"></a>

#### `CBasePlayer:buildShowName ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-changeclothes"></a>

#### `CBasePlayer:changeClothes (partName, actorId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `partName` | `value` | The instance this method was called on. |
| `actorId` | `identifier` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-changeheart"></a>

#### `CBasePlayer:changeHeart (hp, maxHp)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `hp` | `value` | The instance this method was called on. |
| `maxHp` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-changemaxhealth"></a>

#### `CBasePlayer:changeMaxHealth (health)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `health` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-cleareffects"></a>

#### `CBasePlayer:clearEffects ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-clearinv"></a>

#### `CBasePlayer:clearInv ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-consumediamonds"></a>

#### `CBasePlayer:consumeDiamonds (uniqueId, diamonds, remark, isConsume)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `uniqueId` | `identifier` | The instance this method was called on. |
| `diamonds` | `value` |  |
| `remark` | `value` |  |
| `isConsume` | `boolean` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-ctor"></a>

#### `CBasePlayer:ctor (clientPeer)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `clientPeer` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-equiparmor"></a>

#### `CBasePlayer:equipArmor (id, damage)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |
| `damage` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-fillenderinvetory"></a>

#### `CBasePlayer:fillEnderInvetory (enderInv)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `enderInv` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-fillinvetory"></a>

#### `CBasePlayer:fillInvetory (inventory)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `inventory` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getattack"></a>

#### `CBasePlayer:getAttack ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getbirdsimulator"></a>

#### `CBasePlayer:getBirdSimulator ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getcurrency"></a>

#### `CBasePlayer:getCurrency ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getdefence"></a>

#### `CBasePlayer:getDefence ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getdiamond"></a>

#### `CBasePlayer:getDiamond ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getdisplayname"></a>

#### `CBasePlayer:getDisplayName ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getenderinventory"></a>

#### `CBasePlayer:getEnderInventory ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getentityid"></a>

#### `CBasePlayer:getEntityId ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getentityplayer"></a>

#### `CBasePlayer:getEntityPlayer ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getfoodlevel"></a>

#### `CBasePlayer:getFoodLevel ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getgold"></a>

#### `CBasePlayer:getGold ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-gethealth"></a>

#### `CBasePlayer:getHealth ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-gethelditemid"></a>

#### `CBasePlayer:getHeldItemId ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getinventory"></a>

#### `CBasePlayer:getInventory ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getitemnumbyid"></a>

#### `CBasePlayer:getItemNumById (id)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getmaxhealth"></a>

#### `CBasePlayer:getMaxHealth ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getposition"></a>

#### `CBasePlayer:getPosition ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getranch"></a>

#### `CBasePlayer:getRanch ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getsex"></a>

#### `CBasePlayer:getSex ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-getyaw"></a>

#### `CBasePlayer:getYaw ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-initdynamicattr"></a>

#### `CBasePlayer:initDynamicAttr ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-initmembervariables"></a>

#### `CBasePlayer:initMemberVariables ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-initstaticattr"></a>

#### `CBasePlayer:initStaticAttr (teamId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `teamId` | `identifier` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-isvisitor"></a>

#### `CBasePlayer:isVisitor ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-iswatchmode"></a>

#### `CBasePlayer:isWatchMode ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-leavevehicle"></a>

#### `CBasePlayer:leaveVehicle ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-onplayerlogout"></a>

#### `CBasePlayer:onPlayerLogout ()`

Method. Colon form: the first argument is the receiver.

Never called directly. It is registered as a callback, so the host invokes it:

```lua
BaseListener.registerCallBack (self.onPlayerLogout)
PlayerLogoutEvent.registerCallBack (self.onPlayerLogout)
```

<a id="fn-cbaseplayer-onusecustomprop"></a>

#### `CBasePlayer:onUseCustomProp (propId, itemId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `propId` | `identifier` | The instance this method was called on. |
| `itemId` | `identifier` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-playskilleffect"></a>

#### `CBasePlayer:playSkillEffect (name, duration, range, color)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |
| `duration` | `value` |  |
| `range` | `number` |  |
| `color` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-removeeffect"></a>

#### `CBasePlayer:removeEffect (id)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-removeitem"></a>

#### `CBasePlayer:removeItem (id, num)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |
| `num` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-replaceitem"></a>

#### `CBasePlayer:replaceItem (id, num, damage, stackIndex, clipBullet)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `id` | `identifier` | The instance this method was called on. |
| `num` | `number` |  |
| `damage` | `value` |  |
| `stackIndex` | `number` |  |
| `clipBullet` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-resetbackpack"></a>

#### `CBasePlayer:resetBackpack (capacity, maxCapacity)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `capacity` | `value` | The instance this method was called on. |
| `maxCapacity` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-resetclothes"></a>

#### `CBasePlayer:resetClothes ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-respawn"></a>

#### `CBasePlayer:respawn (clientPeer)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `clientPeer` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-sendloginresult"></a>

#### `CBasePlayer:sendLoginResult (isSuccess, teamId, teamName, playerNum, maxPlayer)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `isSuccess` | `boolean` | The instance this method was called on. |
| `teamId` | `identifier` |  |
| `teamName` | `value` |  |
| `playerNum` | `number` |  |
| `maxPlayer` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-sendskilleffect"></a>

#### `CBasePlayer:sendSkillEffect (effectId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `effectId` | `identifier` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setallowflying"></a>

#### `CBasePlayer:setAllowFlying (flying)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `flying` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setarmitem"></a>

#### `CBasePlayer:setArmItem (itemId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `itemId` | `identifier` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setbaginfo"></a>

#### `CBasePlayer:setBagInfo (curCapacity, maxCapacity)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `curCapacity` | `value` | The instance this method was called on. |
| `maxCapacity` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setbirdconvert"></a>

#### `CBasePlayer:setBirdConvert (isConvert)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `isConvert` | `boolean` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setcurrency"></a>

#### `CBasePlayer:setCurrency (currency)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `currency` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setfoodlevel"></a>

#### `CBasePlayer:setFoodLevel (value)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `value` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-sethealth"></a>

#### `CBasePlayer:setHealth (health)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `health` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setonfire"></a>

#### `CBasePlayer:setOnFire (fireTime)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `fireTime` | `number` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setonfrozen"></a>

#### `CBasePlayer:setOnFrozen (frozenTime)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `frozenTime` | `number` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setpersonalshoparea"></a>

#### `CBasePlayer:setPersonalShopArea (startPos, endPos)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `startPos` | `value` | The instance this method was called on. |
| `endPos` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setpositionandrotation"></a>

#### `CBasePlayer:setPositionAndRotation (vec, yaw, pitch)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `vec` | `value` | The instance this method was called on. |
| `yaw` | `value` |  |
| `pitch` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setrespawnpos"></a>

#### `CBasePlayer:setRespawnPos (pos)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `pos` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setshowname"></a>

#### `CBasePlayer:setShowName (name, targetId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |
| `targetId` | `identifier` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setspeedaddition"></a>

#### `CBasePlayer:setSpeedAddition (level)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `level` | `number` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setteam"></a>

#### `CBasePlayer:setTeam (team)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `team` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-setunlockedcommodities"></a>

#### `CBasePlayer:setUnlockedCommodities (unlockedCommodities)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `unlockedCommodities` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-subbackpackcapacity"></a>

#### `CBasePlayer:subBackpackCapacity (capacity)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `capacity` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-subcurrency"></a>

#### `CBasePlayer:subCurrency (currency)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `currency` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-subgunrecoil"></a>

#### `CBasePlayer:subGunRecoil (gunId, recoil)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `gunId` | `identifier` | The instance this method was called on. |
| `recoil` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-subhealth"></a>

#### `CBasePlayer:subHealth (hp)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `hp` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-teleportpos"></a>

#### `CBasePlayer:teleportPos (pos)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `pos` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-teleportposwithyaw"></a>

#### `CBasePlayer:teleportPosWithYaw (pos, yaw)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `pos` | `value` | The instance this method was called on. |
| `yaw` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-updateexp"></a>

#### `CBasePlayer:updateExp (level, exp, maxExp)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `level` | `number` | The instance this method was called on. |
| `exp` | `number` |  |
| `maxExp` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cbaseplayer-useappprops"></a>

#### `CBasePlayer:useAppProps ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-csvutil-getrowcontent"></a>

#### `CsvUtil.getRowContent (file, filePath)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `file` | `value` |  |
| `filePath` | `value` |  |

Used as:

```lua
CsvUtil.getRowContent(file, filePath)
```

<a id="fn-csvutil-loadcsvfile"></a>

#### `CsvUtil.loadCsvFile (filePath, tRow)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `filePath` | `value` |  |
| `tRow` | `value` |  |

Used as:

```lua
CsvUtil.loadCsvFile(path, 2)
CsvUtil.loadCsvFile((string.gsub(configPath, "config.yml", "AppProps.csv")), 2)
CsvUtil.loadCsvFile((string.gsub(configPath, "config.yml", "SownBallPromote.csv")), 2)
```

<a id="fn-class"></a>

#### `class (classname, super)`

Global function. Plain global function.

| Argument | Role | Notes |
| --- | --- | --- |
| `classname` | `value` |  |
| `super` | `value` |  |

Used as:

```lua
class("BasePlayer")
class("CommonTask", ITask)
class()
```

<a id="fn-class-meta-call"></a>

#### `class.__meta.__call (cls, ...)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `cls` | `value` |  |
| `...` | `varargs` | Any number of extra values. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-class-def"></a>

#### `class.def (base, type, cls)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `base` | `boolean` |  |
| `type` | `value` |  |
| `cls` | `value` |  |

Used as:

```lua
class:def('null')
class:def('map')
class:def('omap')
```

<a id="fn-clone"></a>

#### `clone (object)`

Global function. Plain global function.

| Argument | Role | Notes |
| --- | --- | --- |
| `object` | `value` |  |

Used as:

```lua
clone(super)
```

<a id="fn-cls-ctor"></a>

#### `cls.ctor ()`

Assigned function. Assigned to an existing table field.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-cls-new"></a>

#### `cls.new (...)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `...` | `varargs` | Any number of extra values. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-codepoint-to-utf8"></a>

#### `codepoint_to_utf8 (n)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `n` | `value` |  |

Used as:

```lua
codepoint_to_utf8((n1 - 0xd800) * 0x400 + (n2 - 0xdc00) + 0x10000)
codepoint_to_utf8(n1)
```

<a id="fn-countindent"></a>

#### `countindent (line)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |

Used as:

```lua
countindent(ln)
countindent(line)
countindent(nextline)
```

<a id="fn-create-set"></a>

#### `create_set (...)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `...` | `varargs` | Any number of extra values. |

Used as:

```lua
create_set(" ", "\t", "\r", "\n")
create_set(" ", "\t", "\r", "\n", "]", "}", ",")
create_set("\\", "/", '"', "b", "f", "n", "r", "t", "u")
```


### D

<a id="fn-dbmanager-addcache"></a>

#### `DBManager:addCache (key, data)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `key` | `identifier` | The instance this method was called on. |
| `data` | `table` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-dbmanager-getcache"></a>

#### `DBManager:getCache (key)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `key` | `identifier` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-dbmanager-getplayerdata"></a>

#### `DBManager:getPlayerData (userId, subKey)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `subKey` | `identifier` |  |

Used as:

```lua
DBManager:getPlayerData(player.userId, 1)
DBManager:getPlayerData(player.userId, GamePlayer.ROLE_POLICE)
DBManager:getPlayerData(player.userId, GamePlayer.ROLE_CRIMINAL)
```

<a id="fn-dbmanager-initplayerdata"></a>

#### `DBManager:initPlayerData (userId, subKey, data)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `subKey` | `identifier` |  |
| `data` | `table` |  |

Used as:

```lua
DBManager:initPlayerData(userId, subKey, data)
DBManager:initPlayerData(userId, tag, data)
```

<a id="fn-dbmanager-removecache"></a>

#### `DBManager:removeCache (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
DBManager:removeCache(player.userId)
```

<a id="fn-dbmanager-saveplayerdata"></a>

#### `DBManager:savePlayerData (userId, subKey, data, immediate)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `subKey` | `identifier` |  |
| `data` | `table` |  |
| `immediate` | `value` |  |

Used as:

```lua
DBManager:savePlayerData(player.userId, 1, json.encode(data), immediate)
DBManager:savePlayerData(player.userId, GamePlayer.ROLE_CRIMINAL, json.encode(data), immediate)
DBManager:savePlayerData(player.userId, GamePlayer.ROLE_POLICE, json.encode(data), immediate)
```

<a id="fn-dateutil-getdate"></a>

#### `DateUtil.getDate (time)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |

Used as:

```lua
DateUtil.getDate()
```

<a id="fn-dateutil-getdatestring"></a>

#### `DateUtil.getDateString (time)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-dateutil-getdaylasttime"></a>

#### `DateUtil.getDayLastTime (time)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |

Used as:

```lua
DateUtil.getDayLastTime()
```

<a id="fn-dateutil-getweeklasttime"></a>

#### `DateUtil.getWeekLastTime (time)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |

Used as:

```lua
DateUtil.getWeekLastTime()
```

<a id="fn-dateutil-getweekseconds"></a>

#### `DateUtil.getWeekSeconds ()`

Namespaced function. Dot form on a table; no receiver.

Used as:

```lua
DateUtil.getWeekSeconds()
```

<a id="fn-dateutil-getweeksofyear"></a>

#### `DateUtil.getWeeksOfYear (time)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |

Used as:

```lua
DateUtil.getWeeksOfYear(time)
```

<a id="fn-dateutil-getyearday"></a>

#### `DateUtil.getYearDay (time)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |

Used as:

```lua
DateUtil.getYearDay()
```

<a id="fn-dateutil-getyearweek"></a>

#### `DateUtil.getYearWeek (time)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |

Used as:

```lua
DateUtil.getYearWeek()
DateUtil.getYearWeek(os.time() - DateUtil.getWeekSeconds())
```

<a id="fn-decode-error"></a>

#### `decode_error (str, idx, msg)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `idx` | `identifier` |  |
| `msg` | `string` |  |

Used as:

```lua
decode_error(str, j, "control character in string")
decode_error(str, j, "invalid unicode escape in string")
decode_error(str, j, "invalid escape char '" .. c .. "' in string")
```

<a id="fn-dot2s"></a>

#### `doT2S (_i, _v)`

Global function. Plain global function.

| Argument | Role | Notes |
| --- | --- | --- |
| `_i` | `value` |  |
| `_v` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._


### E

<a id="fn-engineutil-disposeplatformorder"></a>

#### `EngineUtil.disposePlatformOrder (userId, orderId, isConsume)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` |  |
| `orderId` | `identifier` |  |
| `isConsume` | `boolean` |  |

Used as:

```lua
EngineUtil.disposePlatformOrder(0, payId, false)
EngineUtil.disposePlatformOrder(player.userId, payId, false)
EngineUtil.disposePlatformOrder(player.userId, payId, true)
```

<a id="fn-engineutil-sendenterothergame"></a>

#### `EngineUtil.sendEnterOtherGame (rakssid, gameType, targetId, mapId)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |
| `gameType` | `value` |  |
| `targetId` | `identifier` |  |
| `mapId` | `identifier` |  |

Used as:

```lua
EngineUtil.sendEnterOtherGame(player.rakssid, "g1032", player.userId)
EngineUtil.sendEnterOtherGame(c_player.rakssid, "g1032", c_player.userId)
EngineUtil.sendEnterOtherGame(c_player.rakssid, c_player.gameType, c_player.userId)
```

<a id="fn-engineutil-sendgotoothergame"></a>

#### `EngineUtil.sendGotoOtherGame (rakssid, gameType, targetId, mapId)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |
| `gameType` | `value` |  |
| `targetId` | `identifier` |  |
| `mapId` | `identifier` |  |

Used as:

```lua
EngineUtil.sendGotoOtherGame(callBackPlayer.rakssid, targetId, "g1031")
EngineUtil.sendGotoOtherGame(rakssid, player.userId, "g1035")
EngineUtil.sendGotoOtherGame(v.rakssid, v.userId, "g1031")
```

<a id="fn-engineworld-addactornpc"></a>

#### `EngineWorld:addActorNpc (position, yaw, actor, name, skill, effect, isCanObstruct, isCollided)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `position` | `value` | The instance this method was called on. |
| `yaw` | `value` |  |
| `actor` | `value` |  |
| `name` | `string` |  |
| `skill` | `value` |  |
| `effect` | `value` |  |
| `isCanObstruct` | `boolean` |  |
| `isCollided` | `boolean` |  |

Used as:

```lua
EngineWorld:addActorNpc(pos, v.yaw, "ranchers_door.actor", "", "", "", true, false)
EngineWorld:addActorNpc(v.position, v.yaw, v.actorName, v.name)
EngineWorld:addActorNpc(v.npcPosition, v.npcYaw, v.actorName, v.name, "idle")
```

<a id="fn-engineworld-addcreaturenpc"></a>

#### `EngineWorld:addCreatureNpc (position, id, yaw, actor, name, userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `position` | `value` | The instance this method was called on. |
| `id` | `identifier` |  |
| `yaw` | `value` |  |
| `actor` | `value` |  |
| `name` | `string` |  |
| `userId` | `identifier` |  |

Used as:

```lua
EngineWorld:addCreatureNpc(pos.position, self.config.id, pos.yaw, "basement.actor")
EngineWorld:addCreatureNpc(postion.initPos, self.config.id, postion.yaw, self.config.actor)
EngineWorld:addCreatureNpc(VectorUtil.newVector3(unpack(self.initPos)), 1001, self.yaw, self.actor, self.name)
```

<a id="fn-engineworld-addentityitem"></a>

#### `EngineWorld:addEntityItem (itemId, count, meta, life, pos, motion, isCombine, isEverLife)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `itemId` | `identifier` | The instance this method was called on. |
| `count` | `number` |  |
| `meta` | `value` |  |
| `life` | `value` |  |
| `pos` | `value` |  |
| `motion` | `value` |  |
| `isCombine` | `boolean` |  |
| `isEverLife` | `boolean` |  |

Used as:

```lua
EngineWorld:addEntityItem(iron.id, iron.num, 0, iron.life, v.ironPos)
EngineWorld:addEntityItem(gold.id, gold.num, 0, gold.life, v.goldPos)
EngineWorld:addEntityItem(money.id, money.num, 0, money.life, v.pos)
```

<a id="fn-engineworld-addgoodsgrouptoshop"></a>

#### `EngineWorld:addGoodsGroupToShop (type, icon, name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `type` | `value` | The instance this method was called on. |
| `icon` | `value` |  |
| `name` | `string` |  |

Used as:

```lua
EngineWorld:addGoodsGroupToShop(group.type, group.icon, group.name)
EngineWorld:addGoodsGroupToShop(shop.type, shop.icon, shop.name)
EngineWorld:addGoodsGroupToShop(element.type, element.icon, element.name)
```

<a id="fn-engineworld-addgoodstoshop"></a>

#### `EngineWorld:addGoodsToShop (type, goods)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `type` | `value` | The instance this method was called on. |
| `goods` | `value` |  |

Used as:

```lua
EngineWorld:addGoodsToShop(group.type, self.newGoods(goods))
EngineWorld:addGoodsToShop(shop.type, goods)
EngineWorld:addGoodsToShop(element.type, self.newGoods(goodsItem))
```

<a id="fn-engineworld-addmerchantnpc"></a>

#### `EngineWorld:addMerchantNpc (position, yaw, name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `position` | `value` | The instance this method was called on. |
| `yaw` | `value` |  |
| `name` | `string` |  |

Used as:

```lua
EngineWorld:addMerchantNpc(pos, yaw, name)
EngineWorld:addMerchantNpc(config.initPos, config.yaw, config.name)
EngineWorld:addMerchantNpc(config.MerchantPos, config.MerchantYaw, config.MerchantName)
```

<a id="fn-engineworld-addranknpc"></a>

#### `EngineWorld:addRankNpc (position, yaw, name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `position` | `value` | The instance this method was called on. |
| `yaw` | `value` |  |
| `name` | `string` |  |

Used as:

```lua
EngineWorld:addRankNpc(self.initPos, self.yaw, self.name)
EngineWorld:addRankNpc(v.pos, v.yaw, v.name)
```

<a id="fn-engineworld-addsessionnpc"></a>

#### `EngineWorld:addSessionNpc (position, yaw, type, name, actor, body, bodyId, content, isPerson, isCanObstruct)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `position` | `value` | The instance this method was called on. |
| `yaw` | `value` |  |
| `type` | `value` |  |
| `name` | `string` |  |
| `actor` | `value` |  |
| `body` | `boolean` |  |
| `bodyId` | `identifier` |  |
| `content` | `string` |  |
| `isPerson` | `boolean` |  |
| `isCanObstruct` | `boolean` |  |

Used as:

```lua
EngineWorld:addSessionNpc(v.position, v.yaw, v.type, v.name, v.actorName, "body")
EngineWorld:addSessionNpc(self.position, self.yaw, self.type, self.name, self.actor, "body", "", self.content)
EngineWorld:addSessionNpc(self.position, self.yaw, self.type, self.name, self.actor, "body", "", json.encode(data))
```

<a id="fn-engineworld-addsimpleeffect"></a>

#### `EngineWorld:addSimpleEffect (name, position, yaw, duration, targetId, scale)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |
| `position` | `value` |  |
| `yaw` | `value` |  |
| `duration` | `value` |  |
| `targetId` | `identifier` |  |
| `scale` | `value` |  |

Used as:

```lua
EngineWorld:addSimpleEffect(v.effect, v.pos, v.yaw)
EngineWorld:addSimpleEffect("ender_pearl.effect", VectorUtil.toVector3(tpPos), player:getYaw(), 1000)
EngineWorld:addSimpleEffect("explosion.effect", pos, player:getYaw(), 1000)
```

<a id="fn-engineworld-addvehiclenpc"></a>

#### `EngineWorld:addVehicleNpc (position, type, yaw)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `position` | `value` | The instance this method was called on. |
| `type` | `value` |  |
| `yaw` | `value` |  |

Used as:

```lua
EngineWorld:addVehicleNpc(vehicle.pos, vehicle.id, vehicle.yaw)
EngineWorld:addVehicleNpc(vehicle.pos, vehicle.type, vehicle.yaw)
```

<a id="fn-engineworld-changeplayeractor"></a>

#### `EngineWorld:changePlayerActor (player, actor, bodyId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |
| `actor` | `value` |  |
| `bodyId` | `identifier` |  |

Used as:

```lua
EngineWorld:changePlayerActor(self, self.model.actor)
EngineWorld:changePlayerActor(player, furniture.name, furniture.bodyId)
EngineWorld:changePlayerActor(player, clothes.name)
```

<a id="fn-engineworld-getblockid"></a>

#### `EngineWorld:getBlockId (blockPos)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `blockPos` | `boolean` | The instance this method was called on. |

Used as:

```lua
EngineWorld:getBlockId(blockPos)
EngineWorld:getBlockId(first)
EngineWorld:getBlockId(second)
```

<a id="fn-engineworld-getentity"></a>

#### `EngineWorld:getEntity (entityId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `entityId` | `identifier` | The instance this method was called on. |

Used as:

```lua
EngineWorld:getEntity(itemEntityId)
```

<a id="fn-engineworld-getworld"></a>

#### `EngineWorld:getWorld ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
EngineWorld:getWorld()
```

<a id="fn-engineworld-removeentity"></a>

#### `EngineWorld:removeEntity (entityId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `entityId` | `identifier` | The instance this method was called on. |

Used as:

```lua
EngineWorld:removeEntity(self.entityId)
EngineWorld:removeEntity(egg.entityId)
EngineWorld:removeEntity(npc.entityId)
```

<a id="fn-engineworld-resetmap"></a>

#### `EngineWorld:resetMap ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-engineworld-restoreplayeractor"></a>

#### `EngineWorld:restorePlayerActor (player)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |

Used as:

```lua
EngineWorld:restorePlayerActor(self)
```

<a id="fn-engineworld-setblock"></a>

#### `EngineWorld:setBlock (blockPos, id, meta, flag, immediate)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `blockPos` | `boolean` | The instance this method was called on. |
| `id` | `identifier` |  |
| `meta` | `value` |  |
| `flag` | `value` |  |
| `immediate` | `value` |  |

Used as:

```lua
EngineWorld:setBlock(vc, 54)
EngineWorld:setBlock(blockPos, bridge.blockId)
EngineWorld:setBlock(pos, BlockID.CHEST)
```

<a id="fn-engineworld-setblocktoair"></a>

#### `EngineWorld:setBlockToAir (blockPos)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `blockPos` | `boolean` | The instance this method was called on. |

Used as:

```lua
EngineWorld:setBlockToAir(VectorUtil.newVector3i(vec3.x, vec3.y, vec3.z))
EngineWorld:setBlockToAir(self.position)
EngineWorld:setBlockToAir(vec)
```

<a id="fn-engineworld-setworld"></a>

#### `EngineWorld:setWorld (world)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `world` | `value` | The instance this method was called on. |

Used as:

```lua
EngineWorld:setWorld(serverworld)
```

<a id="fn-engineworld-stopworldtime"></a>

#### `EngineWorld:stopWorldTime (time)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |

Used as:

```lua
EngineWorld:stopWorldTime()
EngineWorld:stopWorldTime(GameConfig.worldTime)
```

<a id="fn-exprule-checklvup"></a>

#### `ExpRule:checkLvUp (curLv, curExp, addExp)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `curLv` | `value` | The instance this method was called on. |
| `curExp` | `number` |  |
| `addExp` | `number` |  |

Used as:

```lua
ExpRule:checkLvUp(expInfo.curLv, expInfo.curExp, expInfo.addExp)
```

<a id="fn-exprule-checktodayexpfull"></a>

#### `ExpRule:checkTodayExpFull (hasGet, addExp)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `hasGet` | `boolean` | The instance this method was called on. |
| `addExp` | `number` |  |

Used as:

```lua
ExpRule:checkTodayExpFull(expInfo.hasGet, addExp)
```

<a id="fn-exprule-correctaddexp"></a>

#### `ExpRule:correctAddExp (hasGet, addExp)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `hasGet` | `boolean` | The instance this method was called on. |
| `addExp` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-exprule-disable"></a>

#### `ExpRule:disable ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
ExpRule:disable()
```

<a id="fn-exprule-getaddexp"></a>

#### `ExpRule:getAddExp (time, win, camps)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |
| `win` | `value` |  |
| `camps` | `value` |  |

Used as:

```lua
ExpRule:getAddExp(expInfo.time, win, camps)
```

<a id="fn-exprule-getaverageupexp"></a>

#### `ExpRule:getAverageUpExp (curLv, toLv)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `curLv` | `value` | The instance this method was called on. |
| `toLv` | `value` |  |

Used as:

```lua
ExpRule:getAverageUpExp(curLv, curLv)
ExpRule:getAverageUpExp(expInfo.curLv, expInfo.toLv)
```

<a id="fn-exprule-initrole"></a>

#### `ExpRule:initRole (data)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `data` | `table` | The instance this method was called on. |

Used as:

```lua
ExpRule:initRole(data)
```

<a id="fn-exprule-isenable"></a>

#### `ExpRule:isEnable ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
ExpRule:isEnable()
```

<a id="fn-exprule-isexpmaxtoday"></a>

#### `ExpRule:isExpMaxToday (hasGet)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `hasGet` | `boolean` | The instance this method was called on. |

Used as:

```lua
ExpRule:isExpMaxToday(hasGet)
```

<a id="fn-encode"></a>

#### `encode (val, stack)`

Assigned function. Assigned to an existing table field.

| Argument | Role | Notes |
| --- | --- | --- |
| `val` | `value` |  |
| `stack` | `value` |  |

Used as:

```lua
encode(v, stack)
encode(k, stack)
encode(val)
```

<a id="fn-encode-nil"></a>

#### `encode_nil (val)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `val` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-encode-number"></a>

#### `encode_number (val)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `val` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-encode-string"></a>

#### `encode_string (val)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `val` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-encode-table"></a>

#### `encode_table (val, stack)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `val` | `value` |  |
| `stack` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-escape-char"></a>

#### `escape_char (c)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `c` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._


### G

<a id="fn-gamelog-log"></a>

#### `GameLog.log (userId, type, action, data, immediate)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` |  |
| `type` | `value` |  |
| `action` | `value` |  |
| `data` | `table` |  |
| `immediate` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._


### H

<a id="fn-httplistener-onhttpresponse"></a>

#### `HttpListener.onHttpResponse (result, marked)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` |  |
| `marked` | `value` |  |

Never called directly. It is registered as a callback, so the host invokes it:

```lua
HttpResponseEvent.registerCallBack (self.onHttpResponse)
```

<a id="fn-httplistener-init"></a>

#### `HttpListener:init ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
HttpListener:init()
```


### I

<a id="fn-icommontask-ontick"></a>

#### `ICommonTask:onTick (ticks)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `ticks` | `number` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-imessages-longwaitinghint"></a>

#### `IMessages:longWaitingHint ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:longWaitingHint()
```

<a id="fn-imessages-msgcanstartgame"></a>

#### `IMessages:msgCanStartGame ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgCanStartGame()
```

<a id="fn-imessages-msgcloseservertimehint"></a>

#### `IMessages:msgCloseServerTimeHint (time, unit, ifHalf)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |
| `unit` | `value` |  |
| `ifHalf` | `value` |  |

Used as:

```lua
IMessages:msgCloseServerTimeHint(seconds / 60, IMessages.UNIT_MINUTE, false)
IMessages:msgCloseServerTimeHint(seconds / 60, IMessages.UNIT_MINUTE, true)
IMessages:msgCloseServerTimeHint(seconds, IMessages.UNIT_SECOND, false)
```

<a id="fn-imessages-msgdontbuygoodsbynostart"></a>

#### `IMessages:msgDontBuyGoodsByNoStart ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgDontBuyGoodsByNoStart()
```

<a id="fn-imessages-msgdoublekill"></a>

#### `IMessages:msgDoubleKill (name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |

Used as:

```lua
IMessages:msgDoubleKill(self:getDisplayName())
IMessages:msgDoubleKill(killer:getDisplayName())
```

<a id="fn-imessages-msgfightgamestart"></a>

#### `IMessages:msgFightGameStart ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgFightGameStart()
```

<a id="fn-imessages-msgfightgamestarttimehint"></a>

#### `IMessages:msgFightGameStartTimeHint (time, unit, ifHalf)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |
| `unit` | `value` |  |
| `ifHalf` | `value` |  |

Used as:

```lua
IMessages:msgFightGameStartTimeHint(seconds / 60, IMessages.UNIT_MINUTE, false)
IMessages:msgFightGameStartTimeHint(seconds / 60, IMessages.UNIT_MINUTE, true)
IMessages:msgFightGameStartTimeHint(seconds, IMessages.UNIT_SECOND, false)
```

<a id="fn-imessages-msgfirstkill"></a>

#### `IMessages:msgFirstKill (name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |

Used as:

```lua
IMessages:msgFirstKill(killer.name)
IMessages:msgFirstKill(killer:getDisplayName())
```

<a id="fn-imessages-msgfivekill"></a>

#### `IMessages:msgFiveKill (name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |

Used as:

```lua
IMessages:msgFiveKill(self:getDisplayName())
IMessages:msgFiveKill(killer:getDisplayName())
```

<a id="fn-imessages-msggameendtimehint"></a>

#### `IMessages:msgGameEndTimeHint (time, unit, ifHalf)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |
| `unit` | `value` |  |
| `ifHalf` | `value` |  |

Used as:

```lua
IMessages:msgGameEndTimeHint(seconds / 60, IMessages.UNIT_MINUTE, false)
IMessages:msgGameEndTimeHint(seconds / 60, IMessages.UNIT_MINUTE, true)
IMessages:msgGameEndTimeHint(seconds)
```

<a id="fn-imessages-msggamehasstarting"></a>

#### `IMessages:msgGameHasStarting ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameHasStarting()
```

<a id="fn-imessages-msggameover"></a>

#### `IMessages:msgGameOver ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameOver()
```

<a id="fn-imessages-msggameoverbydbdataerror"></a>

#### `IMessages:msgGameOverByDBDataError ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameOverByDBDataError()
```

<a id="fn-imessages-msggameoverbylittle"></a>

#### `IMessages:msgGameOverByLittle ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameOverByLittle()
```

<a id="fn-imessages-msggameoverdeath"></a>

#### `IMessages:msgGameOverDeath ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameOverDeath()
```

<a id="fn-imessages-msggameoverhint"></a>

#### `IMessages:msgGameOverHint (seconds)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `seconds` | `number` | The instance this method was called on. |

Used as:

```lua
IMessages:msgGameOverHint(seconds)
```

<a id="fn-imessages-msggameoverwin"></a>

#### `IMessages:msgGameOverWin ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameOverWin()
```

<a id="fn-imessages-msggameplayerisenough"></a>

#### `IMessages:msgGamePlayerIsEnough ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGamePlayerIsEnough()
```

<a id="fn-imessages-msggameplayernum"></a>

#### `IMessages:msgGamePlayerNum (curPlayers, startPlayers)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `curPlayers` | `value` | The instance this method was called on. |
| `startPlayers` | `value` |  |

Used as:

```lua
IMessages:msgGamePlayerNum(PlayerManager:getPlayerCount(), GameConfig.startPlayers)
IMessages:msgGamePlayerNum(PlayerManager:getPlayerCount(), GameConfig.minPlayers)
IMessages:msgGamePlayerNum(self.site.playernum, GameConfig.startPlayers)
```

<a id="fn-imessages-msggameresethint"></a>

#### `IMessages:msgGameResetHint ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameResetHint()
```

<a id="fn-imessages-msggamerunning"></a>

#### `IMessages:msgGameRunning ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameRunning()
```

<a id="fn-imessages-msggamescore"></a>

#### `IMessages:msgGameScore (score)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `score` | `number` | The instance this method was called on. |

Used as:

```lua
IMessages:msgGameScore(v.score)
IMessages:msgGameScore(self._score)
```

<a id="fn-imessages-msggamestart"></a>

#### `IMessages:msgGameStart ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgGameStart()
```

<a id="fn-imessages-msggamestarttimehint"></a>

#### `IMessages:msgGameStartTimeHint (time, unit, ifHalf)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |
| `unit` | `value` |  |
| `ifHalf` | `value` |  |

Used as:

```lua
IMessages:msgGameStartTimeHint(seconds, IMessages.UNIT_SECOND, false)
IMessages:msgGameStartTimeHint(GameConfig.prepareTime, IMessages.UNIT_SECOND, false)
IMessages:msgGameStartTimeHint(self.curRound.prepare, IMessages.UNIT_SECOND, false)
```

<a id="fn-imessages-msgnowinner"></a>

#### `IMessages:msgNoWinner (type)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `type` | `value` | The instance this method was called on. |

Used as:

```lua
IMessages:msgNoWinner(1)
IMessages:msgNoWinner(0)
```

<a id="fn-imessages-msgplayercount"></a>

#### `IMessages:msgPlayerCount (num)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `num` | `number` | The instance this method was called on. |

Used as:

```lua
IMessages:msgPlayerCount(self:getPlayerCount())
```

<a id="fn-imessages-msgplayerkillplayer"></a>

#### `IMessages:msgPlayerKillPlayer (deceased, killer)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `deceased` | `value` | The instance this method was called on. |
| `killer` | `value` |  |

Used as:

```lua
IMessages:msgPlayerKillPlayer(dier.name, killer.name)
IMessages:msgPlayerKillPlayer(dier:getDisplayName(), killer:getDisplayName())
```

<a id="fn-imessages-msgplayerleft"></a>

#### `IMessages:msgPlayerLeft (num)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `num` | `number` | The instance this method was called on. |

Used as:

```lua
IMessages:msgPlayerLeft(self:getPlayerCount())
IMessages:msgPlayerLeft(PlayerManager:getPlayerCount())
```

<a id="fn-imessages-msgplayerquit"></a>

#### `IMessages:msgPlayerQuit (name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |

Used as:

```lua
IMessages:msgPlayerQuit(player:getDisplayName())
```

<a id="fn-imessages-msgpvpenableafter"></a>

#### `IMessages:msgPvpEnableAfter (time, unit, ifHalf)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |
| `unit` | `value` |  |
| `ifHalf` | `value` |  |

Used as:

```lua
IMessages:msgPvpEnableAfter(seconds / 60, IMessages.UNIT_MINUTE, false)
IMessages:msgPvpEnableAfter(seconds / 60, IMessages.UNIT_MINUTE, true)
IMessages:msgPvpEnableAfter(seconds, IMessages.UNIT_SECOND, false)
```

<a id="fn-imessages-msgreadyhint"></a>

#### `IMessages:msgReadyHint (time)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |

Used as:

```lua
IMessages:msgReadyHint(GameConfig.prepareTime)
```

<a id="fn-imessages-msgrespawn"></a>

#### `IMessages:msgRespawn (name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |

Used as:

```lua
IMessages:msgRespawn(player:getDisplayName())
```

<a id="fn-imessages-msgteleporttodeadplace"></a>

#### `IMessages:msgTeleportToDeadPlace (second)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `second` | `number` | The instance this method was called on. |

Used as:

```lua
IMessages:msgTeleportToDeadPlace(seconds)
```

<a id="fn-imessages-msgthirdkill"></a>

#### `IMessages:msgThirdKill (name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |

Used as:

```lua
IMessages:msgThirdKill(self:getDisplayName())
IMessages:msgThirdKill(killer:getDisplayName())
```

<a id="fn-imessages-msgtimetick"></a>

#### `IMessages:msgTimeTick (time, unit, ifHalf)`

> unit 0:minute 1:second

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |
| `unit` | `value` |  |
| `ifHalf` | `value` |  |

Used as:

```lua
IMessages:msgTimeTick(time, unit, ifHalf)
```

<a id="fn-imessages-msgwaitplayer"></a>

#### `IMessages:msgWaitPlayer ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
IMessages:msgWaitPlayer()
```

<a id="fn-imessages-msgwaitplayerenough"></a>

#### `IMessages:msgWaitPlayerEnough (time, unit, ifHalf)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` | The instance this method was called on. |
| `unit` | `value` |  |
| `ifHalf` | `value` |  |

Used as:

```lua
IMessages:msgWaitPlayerEnough(seconds / 60, IMessages.UNIT_MINUTE, false)
IMessages:msgWaitPlayerEnough(seconds / 60, IMessages.UNIT_MINUTE, true)
IMessages:msgWaitPlayerEnough(seconds, IMessages.UNIT_SECOND, false)
```

<a id="fn-imessages-msgwelcomeplayer"></a>

#### `IMessages:msgWelcomePlayer (game)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `game` | `value` | The instance this method was called on. |

Used as:

```lua
IMessages:msgWelcomePlayer(Messages:gamename())
IMessages:msgWelcomePlayer(message:gamename())
IMessages:msgWelcomePlayer(TextFormat:getTipType(55))
```

<a id="fn-imessages-msgwinnerinfo"></a>

#### `IMessages:msgWinnerInfo (name)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `name` | `string` | The instance this method was called on. |

Used as:

```lua
IMessages:msgWinnerInfo(winPlayerName)
IMessages:msgWinnerInfo(win:getDisplayName())
IMessages:msgWinnerInfo(Messages:getRoleName(GamePlayer.ROLE_SEEK))
```

<a id="fn-int2hex"></a>

#### `INT2HEX (x)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `x` | `value` |  |

Used as:

```lua
INT2HEX(bytes[1])
INT2HEX(bytes[5])
INT2HEX(bytes[7])
```

<a id="fn-itask-ctor"></a>

#### `ITask:ctor (status)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `status` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-itask-oncreate"></a>

#### `ITask:onCreate ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-itask-ontick"></a>

#### `ITask:onTick (ticks)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `ticks` | `number` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-itask-purestop"></a>

#### `ITask:pureStop ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-itask-start"></a>

#### `ITask:start ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-itask-stop"></a>

#### `ITask:stop ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-inventoryutil-fillitem"></a>

#### `InventoryUtil:fillItem (inv, config)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `inv` | `value` | The instance this method was called on. |
| `config` | `value` |  |

Used as:

```lua
InventoryUtil:fillItem(inv, GameConfig.inventory[1])
```

<a id="fn-inventoryutil-fillitembytype"></a>

#### `InventoryUtil:fillItemByType (inv, slotBegin, config, rangeLow, rangeUpper)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `inv` | `value` | The instance this method was called on. |
| `slotBegin` | `value` |  |
| `config` | `value` |  |
| `rangeLow` | `number` |  |
| `rangeUpper` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-inventoryutil-getplayerenderinventoryinfo"></a>

#### `InventoryUtil:getPlayerEnderInventoryInfo (player)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |

Used as:

```lua
InventoryUtil:getPlayerEnderInventoryInfo(player)
```

<a id="fn-inventoryutil-getplayerinventoryinfo"></a>

#### `InventoryUtil:getPlayerInventoryInfo (player)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |

Used as:

```lua
InventoryUtil:getPlayerInventoryInfo(self)
InventoryUtil:getPlayerInventoryInfo(player)
InventoryUtil:getPlayerInventoryInfo(dier)
```

<a id="fn-inventoryutil-preparechest"></a>

#### `InventoryUtil:prepareChest (config)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `config` | `value` | The instance this method was called on. |

Used as:

```lua
InventoryUtil:prepareChest(GameConfig.inventory[1].pos)
InventoryUtil:prepareChest(GameMatch:getCurGameScene().inventoryConfig.pos)
```

<a id="fn-inventoryutil-randomitem"></a>

#### `InventoryUtil:randomItem (subconfig, numMin, numMax)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `subconfig` | `value` | The instance this method was called on. |
| `numMin` | `number` |  |
| `numMax` | `number` |  |

Used as:

```lua
InventoryUtil:randomItem(config, rangeLow, rangeUpper)
```

<a id="fn-inventoryutil-randomitemnum"></a>

#### `InventoryUtil:randomItemNum (item)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `item` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-isemptyline"></a>

#### `isemptyline (line)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |

Used as:

```lua
isemptyline(ssub(line, #needle + 1))
isemptyline(rest)
isemptyline(line)
```


### J

<a id="fn-json-decode"></a>

#### `json.decode (str)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |

Used as:

```lua
json.decode(result)
json.decode(data)
json.decode(resJson)
```

<a id="fn-json-encode"></a>

#### `json.encode (val)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `val` | `value` |  |

Used as:

```lua
json.encode(data)
json.encode(result)
json.encode(content)
```


### L

<a id="fn-luatimer-cancel"></a>

#### `LuaTimer:cancel (key)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `key` | `identifier` | The instance this method was called on. |

Used as:

```lua
LuaTimer:cancel(key)
LuaTimer:cancel(self.respawnTaskKey)
LuaTimer:cancel(player.respawnTaskKey)
```

<a id="fn-luatimer-ontick"></a>

#### `LuaTimer:onTick ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
LuaTimer:onTick()
```

<a id="fn-luatimer-schedule"></a>

#### `LuaTimer:schedule (action, delay, period, ...)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `action` | `value` | The instance this method was called on. |
| `delay` | `number` |  |
| `period` | `value` |  |
| `...` | `varargs` | Any number of extra values. |

Used as:

```lua
LuaTimer:schedule(self.Update, nil, 1, self)
LuaTimer:schedule(self.RandomSupplyItem, delayTime, nil, self)
```

<a id="fn-luatimer-scheduletimer"></a>

#### `LuaTimer:scheduleTimer (action, period, times, ...)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `action` | `value` | The instance this method was called on. |
| `period` | `value` |  |
| `times` | `number` |  |
| `...` | `varargs` | Any number of extra values. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-ltrim"></a>

#### `ltrim (str)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |

Used as:

```lua
ltrim(line)
ltrim(ssub(line, 3))
ltrim(ssub(rh, #typename + 1))
```


### M

<a id="fn-m-new"></a>

#### `M.new (hwaddr)`

> Creates a new uuid. Either provide a unique hex string, or make sure the random seed is properly set. The module table itself is a shortcut to this function, so `my_uuid = uuid.new()` equals `my_uuid = uuid()`.  For proper use there are 3 options;  1. first require `luasocket`, then call `uuid.seed()`, and request a uuid using no parameter, eg. `my_uuid = uuid()` 2. use `uuid` without `luasocket`, set a random seed using `uuid.randomseed(some_good_seed)`, and request a uuid using no parameter, eg. `my_uuid = uuid()` 3. use `uuid` without `luasocket`, and request a uuid using an unique hex string, eg. `my_uuid = uuid(my_networkcard_macaddress)`  @return a properly formatted uuid string @param hwaddr (optional) string containing a unique hex value (e.g.: `00:0c:29:69:41:c6`), to be used to compensate for the lesser `math_random()` function. Use a mac address for solid results. If omitted, a fully randomized uuid will be generated, but then you must ensure that the random seed is set properly! @usage local uuid = require("uuid") print("here's a new uuid: ",uuid())

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `hwaddr` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-m-randomseed"></a>

#### `M.randomseed (seed)`

> Improved randomseed function. Lua 5.1 and 5.2 both truncate the seed given if it exceeds the integer range. If this happens, the seed will be 0 or 1 and all randomness will be gone (each application run will generate the same sequence of random numbers in that case). This improved version drops the most significant bits in those cases to get the seed within the proper range again. @param seed the random seed to set (integer from 0 - 2^32, negative values will be made positive) @return the (potentially modified) seed used @usage local socket = require("socket")  -- gettime() has higher precision than os.time() local uuid = require("uuid") -- see also example at uuid.seed() uuid.randomseed(socket.gettime()*10000) print("here's a new uuid: ",uuid())

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `seed` | `value` |  |

Used as:

```lua
M.randomseed(package.loaded["socket"].gettime() * 10000)
M.randomseed(os.time())
```

<a id="fn-m-seed"></a>

#### `M.seed ()`

> Seeds the random generator. It does so in 2 possible ways;  1. use `os.time()`: this only offers resolution to one second (used when LuaSocket hasn't been loaded yet 2. use luasocket `gettime()` function, but it only does so when LuaSocket has been required already. @usage local socket = require("socket")  -- gettime() has higher precision than os.time() -- LuaSocket loaded, so below line does the same as the example from randomseed() uuid.seed() print("here's a new uuid: ",uuid())

Namespaced function. Dot form on a table; no receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-merchantutil-addcommodity"></a>

#### `MerchantUtil:addCommodity (index, type, goods)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `index` | `number` | The instance this method was called on. |
| `type` | `value` |  |
| `goods` | `value` |  |

Used as:

```lua
MerchantUtil:addCommodity(index, type, self:newCommodity(item))
MerchantUtil:addCommodity((team - 1) * GameConfig.maxLevel + level, itemtype, self:newCommodity(item))
MerchantUtil:addCommodity(1, commodity.type, self:newCommodity(commodity))
```

<a id="fn-merchantutil-changecommoditylist"></a>

#### `MerchantUtil:changeCommodityList (entityId, rakssId, index)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `entityId` | `identifier` | The instance this method was called on. |
| `rakssId` | `identifier` |  |
| `index` | `number` |  |

Used as:

```lua
MerchantUtil:changeCommodityList(self.id, rakssid, self.begin)
MerchantUtil:changeCommodityList(self.id, v.rakssid, (self.team - 1) * GameConfig.maxLevel + self.level)
MerchantUtil:changeCommodityList(self.id, rakssid, (self.team - 1) * GameConfig.maxLevel + self.level)
```

<a id="fn-msgsender-sendbottomtips"></a>

#### `MsgSender.sendBottomTips (time, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendBottomTips(3, IMessages:msgCloseServerTimeHint(seconds, IMessages.UNIT_SECOND, false))
MsgSender.sendBottomTips(3, IMessages:msgPvpEnableAfter(seconds, IMessages.UNIT_SECOND, false))
MsgSender.sendBottomTips(3, IMessages:msgGameEndTimeHint(seconds))
```

<a id="fn-msgsender-sendbottomtipstotarget"></a>

#### `MsgSender.sendBottomTipsToTarget (rakssid, time, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |
| `time` | `number` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendBottomTipsToTarget(0, time, msg, args)
MsgSender.sendBottomTipsToTarget(rakssid, 3, Messages:EnchantMentLeftTime(left_time))
MsgSender.sendBottomTipsToTarget(v.rakssid, 3, pmsg)
```

<a id="fn-msgsender-sendcentertips"></a>

#### `MsgSender.sendCenterTips (time, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendCenterTips(3, Messages:EnchantMentOpen())
MsgSender.sendCenterTips(3, self:getQuestionId())
MsgSender.sendCenterTips(3, Messages:msgBuildEndTimeHint(seconds))
```

<a id="fn-msgsender-sendcentertipstotarget"></a>

#### `MsgSender.sendCenterTipsToTarget (rakssid, time, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |
| `time` | `number` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendCenterTipsToTarget(0, time, msg, args)
MsgSender.sendCenterTipsToTarget(self.rakssid, 3, Messages:msgWinTip())
MsgSender.sendCenterTipsToTarget(self.rakssid, 3, Messages:msgLoseTip())
```

<a id="fn-msgsender-sendkillmsg"></a>

#### `MsgSender.sendKillMsg (killer, dead, armsId, kills, head, color)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `killer` | `value` |  |
| `dead` | `value` |  |
| `armsId` | `identifier` |  |
| `kills` | `value` |  |
| `head` | `value` |  |
| `color` | `value` |  |

Used as:

```lua
MsgSender.sendKillMsg(self.name, dier.name, self:getHeldItemId(), self.evenKills, self.headshot, color)
MsgSender.sendKillMsg(self.name, dier.name, self:getHeldItemId(), self.evenKills, headshot, TextFormat.colorAqua)
```

<a id="fn-msgsender-sendkillmsgtotarget"></a>

#### `MsgSender.sendKillMsgToTarget (rakssid, killer, dead, armsId, kills, head, color)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |
| `killer` | `value` |  |
| `dead` | `value` |  |
| `armsId` | `identifier` |  |
| `kills` | `value` |  |
| `head` | `value` |  |
| `color` | `value` |  |

Used as:

```lua
MsgSender.sendKillMsgToTarget(0, killer, dead, armsId, kills, head, color)
```

<a id="fn-msgsender-sendmsg"></a>

#### `MsgSender.sendMsg (msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendMsg(IMessages:msgPlayerCount(self:getPlayerCount()))
MsgSender.sendMsg(IMessages:msgPlayerQuit(player:getDisplayName()))
MsgSender.sendMsg(IMessages:msgPlayerLeft(self:getPlayerCount()))
```

<a id="fn-msgsender-sendmsgtotarget"></a>

#### `MsgSender.sendMsgToTarget (rakssid, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendMsgToTarget(0, msg, args)
MsgSender.sendMsgToTarget(player.rakssid, IMessages:msgWelcomePlayer(Messages:gamename()))
MsgSender.sendMsgToTarget(player.rakssid, Messages:msgGamePrepareTip(self.curRound.roundTip))
```

<a id="fn-msgsender-sendothertips"></a>

#### `MsgSender.sendOtherTips (time, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendOtherTips(GameConfig.gameTime, Messages:jewelRecreate())
MsgSender.sendOtherTips(GameConfig.gameTime, Messages:JewelDiscover())
MsgSender.sendOtherTips(GameConfig.gameTime, Messages:playerWin(player.team:getDisplayName(), player.name))
```

<a id="fn-msgsender-sendothertipstotarget"></a>

#### `MsgSender.sendOtherTipsToTarget (rakssid, time, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |
| `time` | `number` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendOtherTipsToTarget(0, time, msg, args)
MsgSender.sendOtherTipsToTarget(v.rakssid, GameConfig.gameTime, Messages:placeStatue())
```

<a id="fn-msgsender-sendtoptips"></a>

#### `MsgSender.sendTopTips (time, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `time` | `number` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendTopTips(1, " ")
MsgSender.sendTopTips(3, Messages:gamePrepareTimeHint(seconds))
MsgSender.sendTopTips(GameConfig.gameTime, Messages:msgScore(GameMatch.Teams))
```

<a id="fn-msgsender-sendtoptipstotarget"></a>

#### `MsgSender.sendTopTipsToTarget (rakssid, time, msg, args)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` |  |
| `time` | `number` |  |
| `msg` | `string` |  |
| `args` | `table` |  |

Used as:

```lua
MsgSender.sendTopTipsToTarget(0, time, msg, args)
MsgSender.sendTopTipsToTarget(player.rakssid, 60, Messages:msgGameBuyTip())
MsgSender.sendTopTipsToTarget(player.rakssid, 300, Messages:msgWatchModeTip())
```


### N

<a id="fn-null-tostring"></a>

#### `Null.__tostring ()`

Namespaced function. Dot form on a table; no receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-null-isnull"></a>

#### `Null.isnull (v)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `v` | `value` |  |

Used as:

```lua
Null.isnull(root[1])
```

<a id="fn-numberutil-getintpart"></a>

#### `NumberUtil.getIntPart (num)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `num` | `number` |  |

Used as:

```lua
NumberUtil.getIntPart(time)
NumberUtil.getIntPart((start_wday + yday) / 7)
NumberUtil.getIntPart(money)
```

<a id="fn-next-char"></a>

#### `next_char (str, idx, set, negate)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `idx` | `identifier` |  |
| `set` | `value` |  |
| `negate` | `value` |  |

Used as:

```lua
next_char(str, i, delim_chars)
next_char(str, i, space_chars, true)
next_char(str, i + 1, space_chars, true)
```


### P

<a id="fn-playerclassinfo-addclasses"></a>

#### `PlayerClassInfo:addClasses (platformId, classInfo)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `platformId` | `identifier` | The instance this method was called on. |
| `classInfo` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-playerclassinfo-getclasses"></a>

#### `PlayerClassInfo:getClasses (platformId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `platformId` | `identifier` | The instance this method was called on. |

Used as:

```lua
PlayerClassInfo:getClasses(self.userId)
```

<a id="fn-playerclassinfo-removeclasses"></a>

#### `PlayerClassInfo:removeClasses (platformId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `platformId` | `identifier` | The instance this method was called on. |

Used as:

```lua
PlayerClassInfo:removeClasses(player.userId)
```

<a id="fn-playermanager-addplayer"></a>

#### `PlayerManager:addPlayer (player)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |

Used as:

```lua
PlayerManager:addPlayer(PlayerCount=%d)
PlayerManager:addPlayer(player)
```

<a id="fn-playermanager-copyplayers"></a>

#### `PlayerManager:copyPlayers ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
PlayerManager:copyPlayers()
```

<a id="fn-playermanager-getmaxplayer"></a>

#### `PlayerManager:getMaxPlayer ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
PlayerManager:getMaxPlayer()
```

<a id="fn-playermanager-getplayerbyentityid"></a>

#### `PlayerManager:getPlayerByEntityId (entityId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `entityId` | `identifier` | The instance this method was called on. |

Used as:

```lua
PlayerManager:getPlayerByEntityId(entityId)
PlayerManager:getPlayerByEntityId(targetId)
PlayerManager:getPlayerByEntityId(throwerId)
```

<a id="fn-playermanager-getplayerbyplayermp"></a>

#### `PlayerManager:getPlayerByPlayerMP (playerMP)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `playerMP` | `value` | The instance this method was called on. |

Used as:

```lua
PlayerManager:getPlayerByPlayerMP(hurtPlayer)
PlayerManager:getPlayerByPlayerMP(hurtFrom)
PlayerManager:getPlayerByPlayerMP(diePlayer)
```

<a id="fn-playermanager-getplayerbyrakssid"></a>

#### `PlayerManager:getPlayerByRakssid (rakssId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssId` | `identifier` | The instance this method was called on. |

Used as:

```lua
PlayerManager:getPlayerByRakssid(rakssid)
PlayerManager:getPlayerByRakssid(clientPeer:getRakssid())
PlayerManager:getPlayerByRakssid(rakssId)
```

<a id="fn-playermanager-getplayerbyuserid"></a>

#### `PlayerManager:getPlayerByUserId (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
PlayerManager:getPlayerByUserId(userId)
PlayerManager:getPlayerByUserId(data.id)
PlayerManager:getPlayerByUserId(data.userId)
```

<a id="fn-playermanager-getplayercount"></a>

#### `PlayerManager:getPlayerCount ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
PlayerManager:getPlayerCount()
```

<a id="fn-playermanager-getplayers"></a>

#### `PlayerManager:getPlayers ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
PlayerManager:getPlayers()
```

<a id="fn-playermanager-isplayerempty"></a>

#### `PlayerManager:isPlayerEmpty ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
PlayerManager:isPlayerEmpty()
```

<a id="fn-playermanager-isplayerenough"></a>

#### `PlayerManager:isPlayerEnough (players)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `players` | `value` | The instance this method was called on. |

Used as:

```lua
PlayerManager:isPlayerEnough(GameConfig.startPlayers)
PlayerManager:isPlayerEnough(GameConfig.minPlayers)
```

<a id="fn-playermanager-isplayerexists"></a>

#### `PlayerManager:isPlayerExists ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
PlayerManager:isPlayerExists()
```

<a id="fn-playermanager-isplayerfull"></a>

#### `PlayerManager:isPlayerFull ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
PlayerManager:isPlayerFull()
```

<a id="fn-playermanager-setmaxplayer"></a>

#### `PlayerManager:setMaxPlayer (maxPlayer)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `maxPlayer` | `value` | The instance this method was called on. |

Used as:

```lua
PlayerManager:setMaxPlayer(config.maxPlayers)
PlayerManager:setMaxPlayer(100)
```

<a id="fn-playermanager-subplayer"></a>

#### `PlayerManager:subPlayer (player)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |

Used as:

```lua
PlayerManager:subPlayer(PlayerCount=%d)
PlayerManager:subPlayer(player)
```

<a id="fn-parse"></a>

#### `parse (str, idx)`

Assigned function. Assigned to an existing table field.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `idx` | `identifier` |  |

Used as:

```lua
parse(str, i)
parse(str, next_char(str, 1, space_chars, true))
parse(str, i)
```

<a id="fn-parse-array"></a>

#### `parse_array (str, i)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `i` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-parse-literal"></a>

#### `parse_literal (str, i)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `i` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-parse-number"></a>

#### `parse_number (str, i)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `i` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-parse-object"></a>

#### `parse_object (str, i)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `i` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-parse-string"></a>

#### `parse_string (str, i)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `i` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-parse-unicode-escape"></a>

#### `parse_unicode_escape (s)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `s` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-parseblockstylestring"></a>

#### `parseblockstylestring (line, lines, indent)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |
| `lines` | `value` |  |
| `indent` | `value` |  |

Used as:

```lua
parseblockstylestring(line, lines, indent)
```

<a id="fn-parsedocuments"></a>

#### `parsedocuments (lines)`

> : (list<str>)->dict

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `lines` | `value` |  |

Used as:

```lua
parsedocuments(lines)
```

<a id="fn-parseflowstyle"></a>

#### `parseflowstyle (line, lines)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |
| `lines` | `value` |  |

Used as:

```lua
parseflowstyle(line, lines)
```

<a id="fn-parsemap"></a>

#### `parsemap (line, lines, indent)`

Global function. Plain global function.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |
| `lines` | `value` |  |
| `indent` | `value` |  |

Used as:

```lua
parsemap('', lines, indent2)
parsemap('', lines, level)
```

<a id="fn-parsescalar"></a>

#### `parsescalar (line, lines, indent)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |
| `lines` | `value` |  |
| `indent` | `value` |  |

Used as:

```lua
parsescalar(rest, lines)
parsescalar(s, {}, 0)
parsescalar(line, lines, indent)
```

<a id="fn-parseseq"></a>

#### `parseseq (line, lines, indent)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |
| `lines` | `value` |  |
| `indent` | `value` |  |

Used as:

```lua
parseseq('', lines, indent2)
parseseq('', lines, 0)
```

<a id="fn-parseset"></a>

#### `parseset (line, lines, indent)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |
| `lines` | `value` |  |
| `indent` | `value` |  |

Used as:

```lua
parseset('', lines, indent2)
```

<a id="fn-parsestring"></a>

#### `parsestring (line, stopper)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |
| `stopper` | `value` |  |

Used as:

```lua
parsestring(ssub(line, 2))
parsestring(line, ',{}[]')
parsestring(line)
```

<a id="fn-parsetimestamp"></a>

#### `parsetimestamp (line)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |

Used as:

```lua
parsetimestamp(line)
```


### R

<a id="fn-rankmanager-addplayerdayrank"></a>

#### `RankManager:addPlayerDayRank (player, key, rank, score, func, ...)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |
| `key` | `identifier` |  |
| `rank` | `value` |  |
| `score` | `number` |  |
| `func` | `function` |  |
| `...` | `varargs` | Any number of extra values. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-rankmanager-addplayerweekrank"></a>

#### `RankManager:addPlayerWeekRank (player, key, rank, score, func, ...)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |
| `key` | `identifier` |  |
| `rank` | `value` |  |
| `score` | `number` |  |
| `func` | `function` |  |
| `...` | `varargs` | Any number of extra values. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-rankmanager-buildplayerrank"></a>

#### `RankManager:buildPlayerRank (rakssid, key)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` | The instance this method was called on. |
| `key` | `identifier` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-rankmanager-clearranks"></a>

#### `RankManager:clearRanks ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
RankManager:clearRanks()
```

<a id="fn-rankmanager-removeplayerrank"></a>

#### `RankManager:removePlayerRank (rakssid)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` | The instance this method was called on. |

Used as:

```lua
RankManager:removePlayerRank(player.rakssid)
```

<a id="fn-rankmanager-sendplayerrank"></a>

#### `RankManager:sendPlayerRank (rakssid, key, data, entityId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `rakssid` | `identifier` | The instance this method was called on. |
| `key` | `identifier` |  |
| `data` | `table` |  |
| `entityId` | `identifier` |  |

Used as:

```lua
RankManager:sendPlayerRank(player.rakssid, self.key, data, self.entityId)
RankManager:sendPlayerRank(rakssid, "digger", data, v.entityId)
RankManager:sendPlayerRank(player.rakssid, self.key, data, npc)
```

<a id="fn-reportmanager-closeserver"></a>

#### `ReportManager:closeServer ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
ReportManager:closeServer()
```

<a id="fn-reportmanager-getuserreportcache"></a>

#### `ReportManager:getUserReportCache (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
ReportManager:getUserReportCache(userId)
```

<a id="fn-reportmanager-onplayerlogout"></a>

#### `ReportManager:onPlayerLogout (player)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `player` | `value` | The instance this method was called on. |

Used as:

```lua
ReportManager:onPlayerLogout(player)
ReportManager:onPlayerLogout(self)
```

<a id="fn-reportmanager-reportusercount"></a>

#### `ReportManager:reportUserCount (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
ReportManager:reportUserCount(userId)
```

<a id="fn-reportmanager-reportuserdata"></a>

#### `ReportManager:reportUserData (userId, kills, rank, isCount)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `kills` | `value` |  |
| `rank` | `value` |  |
| `isCount` | `number` |  |

Used as:

```lua
ReportManager:reportUserData(player.userId, player.kills, rank, 1)
ReportManager:reportUserData(self.userId, self.kills, GameMatch:getPlayerRank(self), isCount)
ReportManager:reportUserData(self.userId, 0, self.rank, 1)
```

<a id="fn-reportmanager-reportuserintegral"></a>

#### `ReportManager:reportUserIntegral (userId, integral)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `integral` | `value` |  |

Used as:

```lua
ReportManager:reportUserIntegral(player.userId, player.appIntegral)
```

<a id="fn-reportmanager-reportuserkill"></a>

#### `ReportManager:reportUserKill (userId, kill)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `kill` | `value` |  |

Used as:

```lua
ReportManager:reportUserKill(userId, kills)
```

<a id="fn-reportmanager-reportuserrank"></a>

#### `ReportManager:reportUserRank (userId, rank)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `rank` | `value` |  |

Used as:

```lua
ReportManager:reportUserRank(userId, rank)
```

<a id="fn-reportmanager-reportusertime"></a>

#### `ReportManager:reportUserTime (userId, time)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `time` | `number` |  |

Used as:

```lua
ReportManager:reportUserTime(player.userId, os.time() - player.inGameTime)
```

<a id="fn-reportmanager-setgametype"></a>

#### `ReportManager:setGameType (GameType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `GameType` | `object` | The instance this method was called on. |

Used as:

```lua
ReportManager:setGameType(self.GameType)
ReportManager:setGameType(GameMatch.gameType)
```

<a id="fn-reportmanager-setranktype"></a>

#### `ReportManager:setRankType (RankType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `RankType` | `object` | The instance this method was called on. |

Used as:

```lua
ReportManager:setRankType(ReportManager.RankType.Max)
```

<a id="fn-reportmanager-trycloseserver"></a>

#### `ReportManager:tryCloseServer ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
ReportManager:tryCloseServer()
```

<a id="fn-reportmanager-tryreportgamedata"></a>

#### `ReportManager:tryReportGameData ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
ReportManager:tryReportGameData()
```

<a id="fn-rewardmanager-addrewardqueue"></a>

#### `RewardManager:addRewardQueue (userId, rank)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `rank` | `value` |  |

Used as:

```lua
RewardManager:addRewardQueue(player.userId, rank)
```

<a id="fn-rewardmanager-getqueuereward"></a>

#### `RewardManager:getQueueReward (func, ...)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `func` | `function` | The instance this method was called on. |
| `...` | `varargs` | Any number of extra values. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-rewardmanager-getstartgametime"></a>

#### `RewardManager:getStartGameTime ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
RewardManager:getStartGameTime()
```

<a id="fn-rewardmanager-getstartplayercount"></a>

#### `RewardManager:getStartPlayerCount ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
RewardManager:getStartPlayerCount()
```

<a id="fn-rewardmanager-getusergoldreward"></a>

#### `RewardManager:getUserGoldReward (userId, gold, func)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `gold` | `value` |  |
| `func` | `function` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-rewardmanager-getuserreward"></a>

#### `RewardManager:getUserReward (userId, rank, func)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `rank` | `value` |  |
| `func` | `function` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-rewardmanager-isuserrewardfinish"></a>

#### `RewardManager:isUserRewardFinish (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
RewardManager:isUserRewardFinish(self.userId)
```

<a id="fn-rewardmanager-setgametype"></a>

#### `RewardManager:setGameType (GameType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `GameType` | `object` | The instance this method was called on. |

Used as:

```lua
RewardManager:setGameType(self.GameType)
RewardManager:setGameType("g1032")
RewardManager:setGameType("g1037")
```

<a id="fn-rewardmanager-startgame"></a>

#### `RewardManager:startGame ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
RewardManager:startGame()
```

<a id="fn-rtrim"></a>

#### `rtrim (str)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |

Used as:

```lua
rtrim(buf)
```


### S

<a id="fn-seasonmanager-adduserhonor"></a>

#### `SeasonManager:addUserHonor (userId, honor, level)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `honor` | `value` |  |
| `level` | `number` |  |

Used as:

```lua
SeasonManager:addUserHonor(self.userId, reward.integral, self.level)
SeasonManager:addUserHonor(player.userId, GameConfig.quitSubHonor, player.level)
```

<a id="fn-seasonmanager-trysavehonorresult"></a>

#### `SeasonManager:trySaveHonorResult ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
SeasonManager:trySaveHonorResult()
```

<a id="fn-stringutil-split"></a>

#### `StringUtil.split (str, reps, keep_blank)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `str` | `string` |  |
| `reps` | `value` |  |
| `keep_blank` | `value` |  |

Used as:

```lua
StringUtil.split(key, "#")
StringUtil.split(prop.enchants, "#")
StringUtil.split(enchant, "=")
```

<a id="fn-stringutil-v2s"></a>

#### `StringUtil.v2s (value, maxLevel)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `value` | `value` |  |
| `maxLevel` | `number` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-select"></a>

#### `select (list, pred)`

> utils

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `list` | `table` |  |
| `pred` | `value` |  |

Used as:

```lua
select("#", ...)
select(i, ...)
```

<a id="fn-startswith"></a>

#### `startswith (haystack, needle)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `haystack` | `value` |  |
| `needle` | `boolean` |  |

Used as:

```lua
startswith(line, needle)
startswith(line, '{')
startswith(line, '|')
```

<a id="fn-startswithline"></a>

#### `startswithline (line, needle)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `line` | `value` |  |
| `needle` | `boolean` |  |

Used as:

```lua
startswithline(line, '{}')
startswithline(line, '[]')
```


### T

<a id="fn-textformat-gettiptype"></a>

#### `TextFormat:getTipType (num)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `num` | `number` | The instance this method was called on. |

Used as:

```lua
TextFormat:getTipType(1)
TextFormat:getTipType(2)
TextFormat:getTipType(31)
```

<a id="fn-types-timestamp-init"></a>

#### `types.timestamp:__init (y, m, d, h, i, s, f, z)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `y` | `value` | The instance this method was called on. |
| `m` | `value` |  |
| `d` | `value` |  |
| `h` | `value` |  |
| `i` | `value` |  |
| `s` | `value` |  |
| `f` | `value` |  |
| `z` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-types-timestamp-tostring"></a>

#### `types.timestamp:__tostring ()`

Method. Colon form: the first argument is the receiver.

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-types-timestamp-gettz"></a>

#### `types.timestamp:gettz ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
gettz()
```


### U

<a id="fn-userexpmanager-addexpcache"></a>

#### `UserExpManager:addExpCache (userId, curLv, curExp, hasGet)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `curLv` | `value` |  |
| `curExp` | `number` |  |
| `hasGet` | `boolean` |  |

Used as:

```lua
UserExpManager:addExpCache(expInfo.userId, expInfo.level, expInfo.experience, expInfo.expGetToday)
```

<a id="fn-userexpmanager-adduserexp"></a>

#### `UserExpManager:addUserExp (userId, win, camps, global)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `win` | `value` |  |
| `camps` | `value` |  |
| `global` | `value` |  |

Used as:

```lua
UserExpManager:addUserExp(userId, false, 0, true)
UserExpManager:addUserExp(player.userId, winner == player:getTeamId(), 4)
UserExpManager:addUserExp(self.userId, isWin, 4)
```

<a id="fn-userexpmanager-getuserexpcache"></a>

#### `UserExpManager:getUserExpCache (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
UserExpManager:getUserExpCache(self.userId)
```

<a id="fn-userexpmanager-removeexpcache"></a>

#### `UserExpManager:removeExpCache (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
UserExpManager:removeExpCache(player.userId)
```

<a id="fn-userexpmanager-sendexpresult"></a>

#### `UserExpManager:sendExpResult (userId, status)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `status` | `value` |  |

Used as:

```lua
UserExpManager:sendExpResult(userId, status)
```

<a id="fn-userexpmanager-trycalculationexp"></a>

#### `UserExpManager:tryCalculationExp ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
UserExpManager:tryCalculationExp()
```

<a id="fn-userexpmanager-trysaveexpresult"></a>

#### `UserExpManager:trySaveExpResult ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
UserExpManager:trySaveExpResult()
```

<a id="fn-userinfocache-filterexisteduserid"></a>

#### `UserInfoCache:FilterExistedUserId (userIds)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userIds` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-userinfocache-getcache"></a>

#### `UserInfoCache:GetCache (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
UserInfoCache:GetCache(v.userId)
UserInfoCache:GetCache(rank.userId)
```

<a id="fn-userinfocache-getcachebyuserids"></a>

#### `UserInfoCache:GetCacheByUserIds (userIds, func, key, ...)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userIds` | `value` | The instance this method was called on. |
| `func` | `function` |  |
| `key` | `identifier` |  |
| `...` | `varargs` | Any number of extra values. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._


### V

<a id="fn-vectorutil-equal"></a>

#### `VectorUtil.equal (vec3_1, vec3_2)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `vec3_1` | `value` |  |
| `vec3_2` | `value` |  |

Used as:

```lua
VectorUtil.equal(player.lastPosition, currentPosition)
VectorUtil.equal(player.lastPos, curPos)
VectorUtil.equal(position, self.endPos)
```

<a id="fn-vectorutil-hashvector3"></a>

#### `VectorUtil.hashVector3 (vec3)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `vec3` | `value` |  |

Used as:

```lua
VectorUtil.hashVector3(blockPos)
VectorUtil.hashVector3(vec3)
VectorUtil.hashVector3(pos)
```

<a id="fn-vectorutil-newvector3"></a>

#### `VectorUtil.newVector3 (x, y, z)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `x` | `value` |  |
| `y` | `value` |  |
| `z` | `value` |  |

Used as:

```lua
VectorUtil.newVector3(0, 0, 0)
VectorUtil.newVector3(tonumber(v.initPos[1]), tonumber(v.initPos[2]), tonumber(v.initPos[3]))
VectorUtil.newVector3(tonumber(v.PosX), tonumber(v.PosY), tonumber(v.PosZ))
```

<a id="fn-vectorutil-newvector3i"></a>

#### `VectorUtil.newVector3i (x, y, z)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `x` | `value` |  |
| `y` | `value` |  |
| `z` | `value` |  |

Used as:

```lua
VectorUtil.newVector3i(pos.x, pos.y + 0.5, pos.z)
VectorUtil.newVector3i(x, y, z)
VectorUtil.newVector3i(pos.x + dx * i, pos.y, pos.z + dz * i)
```

<a id="fn-vectorutil-toblockvector3"></a>

#### `VectorUtil.toBlockVector3 (x, y, z)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `x` | `value` |  |
| `y` | `value` |  |
| `z` | `value` |  |

Used as:

```lua
VectorUtil.toBlockVector3(pos.x, pos.y - 1, pos.z)
VectorUtil.toBlockVector3(x, 0, z)
VectorUtil.toBlockVector3(dier:getPosition().x, dier:getPosition().y, dier:getPosition().z)
```

<a id="fn-vectorutil-tovector3"></a>

#### `VectorUtil.toVector3 (vec3i)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `vec3i` | `value` |  |

Used as:

```lua
VectorUtil.toVector3(tpPos)
```

<a id="fn-vectorutil-tovector3i"></a>

#### `VectorUtil.toVector3i (vec3)`

Namespaced function. Dot form on a table; no receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `vec3` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-v2s"></a>

#### `v2s (value, sp, restLevel)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `value` | `value` |  |
| `sp` | `value` |  |
| `restLevel` | `number` |  |

Used as:

```lua
v2s(v, nsp, restLevel)
v2s(value, "", maxLevel or 3)
```

<a id="fn-var-dump"></a>

#### `var_dump (_t)`

Global function. Plain global function.

| Argument | Role | Notes |
| --- | --- | --- |
| `_t` | `value` |  |

Used as:

```lua
var_dump(_v)
```


### W

<a id="fn-walletutils-addcoinmappings"></a>

#### `WalletUtils:addCoinMappings (coinMapping)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `coinMapping` | `value` | The instance this method was called on. |

Used as:

```lua
WalletUtils:addCoinMappings(MoneyConfig.coinMapping)
WalletUtils:addCoinMappings(GameConfig.coinMapping)
WalletUtils:addCoinMappings(MerchantConfig.coinMapping)
```

<a id="fn-webresponse-formatdata"></a>

#### `WebResponse:formatData (result)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |

Used as:

```lua
WebResponse:formatData(result)
```

<a id="fn-webresponse-formatresult"></a>

#### `WebResponse:formatResult (result)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |

Used as:

```lua
WebResponse:formatResult(result)
```

<a id="fn-webresponse-formatresultcode"></a>

#### `WebResponse:formatResultCode (result)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |

Used as:

```lua
WebResponse:formatResultCode(result)
```

<a id="fn-webresponse-oncallfunc"></a>

#### `WebResponse:onCallFunc (data, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `data` | `table` | The instance this method was called on. |
| `marked` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-webresponse-oncheckmails"></a>

#### `WebResponse:onCheckMails (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onCheckMails(result, marked)
```

<a id="fn-webresponse-ongetblockymodsexp"></a>

#### `WebResponse:onGetBlockymodsExp (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetBlockymodsExp(result, marked)
```

<a id="fn-webresponse-ongetblockymodsexprole"></a>

#### `WebResponse:onGetBlockymodsExpRole (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetBlockymodsExpRole(result, marked)
```

<a id="fn-webresponse-ongetblockymodsgoldreward"></a>

#### `WebResponse:onGetBlockymodsGoldReward (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetBlockymodsGoldReward(result, marked)
```

<a id="fn-webresponse-ongetblockymodsreward"></a>

#### `WebResponse:onGetBlockymodsReward (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetBlockymodsReward(result, marked)
```

<a id="fn-webresponse-ongetblockymodsrewardlist"></a>

#### `WebResponse:onGetBlockymodsRewardList (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetBlockymodsRewardList(result, marked)
```

<a id="fn-webresponse-ongetblockymodsuserattr"></a>

#### `WebResponse:onGetBlockymodsUserAttr (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetBlockymodsUserAttr(result, marked)
```

<a id="fn-webresponse-ongetblockymodsuserinfos"></a>

#### `WebResponse:onGetBlockymodsUserInfos (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetBlockymodsUserInfos(result, marked)
```

<a id="fn-webresponse-ongetblockymodsuserseasoninfo"></a>

#### `WebResponse:onGetBlockymodsUserSeasonInfo (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetBlockymodsUserSeasonInfo(result, marked)
```

<a id="fn-webresponse-ongetmcpeonlinereport"></a>

#### `WebResponse:onGetMcpeonlineReport (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetMcpeonlineReport(result, marked)
```

<a id="fn-webresponse-ongetmcpeonlinereward"></a>

#### `WebResponse:onGetMcpeonlineReward (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetMcpeonlineReward(result, marked)
```

<a id="fn-webresponse-ongetmcpeonlineuserattr"></a>

#### `WebResponse:onGetMcpeonlineUserAttr (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onGetMcpeonlineUserAttr(result, marked)
```

<a id="fn-webresponse-onreportblockymodsdata"></a>

#### `WebResponse:onReportBlockymodsData (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onReportBlockymodsData(result, marked)
```

<a id="fn-webresponse-onreportblockymodslist"></a>

#### `WebResponse:onReportBlockymodsList (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onReportBlockymodsList(result, marked)
```

<a id="fn-webresponse-onsaveblockymodsexp"></a>

#### `WebResponse:onSaveBlockymodsExp (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onSaveBlockymodsExp(result, marked)
```

<a id="fn-webresponse-onsaveblockymodshonor"></a>

#### `WebResponse:onSaveBlockymodsHonor (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onSaveBlockymodsHonor(result, marked)
```

<a id="fn-webresponse-onsendmails"></a>

#### `WebResponse:onSendMails (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onSendMails(result, marked)
```

<a id="fn-webresponse-onupdateblockymodsuserseasonreward"></a>

#### `WebResponse:onUpdateBlockymodsUserSeasonReward (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onUpdateBlockymodsUserSeasonReward(result, marked)
```

<a id="fn-webresponse-onupdatemails"></a>

#### `WebResponse:onUpdateMails (result, marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `result` | `value` | The instance this method was called on. |
| `marked` | `value` |  |

Used as:

```lua
WebResponse:onUpdateMails(result, marked)
```

<a id="fn-webresponse-registercallback"></a>

#### `WebResponse:registerCallBack (func, marked, ...)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `func` | `function` | The instance this method was called on. |
| `marked` | `value` |  |
| `...` | `varargs` | Any number of extra values. |

Used as:

```lua
WebResponse:registerCallBack(func, WebService:GetBlockymodsRewardList(data), ...)
WebResponse:registerCallBack(func, WebService:GetBlockymodsReward(userId, self.GameType, data))
WebResponse:registerCallBack(func, WebService:GetBlockymodsGoldReward(userId, self.GameType, gold))
```

<a id="fn-webresponse-unregistercallback"></a>

#### `WebResponse:unregisterCallBack (marked)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `marked` | `value` | The instance this method was called on. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-webservice-getblockymodsexprule"></a>

#### `WebService:GetBlockymodsExpRule ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
WebService:GetBlockymodsExpRule()
```

<a id="fn-webservice-getblockymodsgoldreward"></a>

#### `WebService:GetBlockymodsGoldReward (userId, gameType, golds)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `gameType` | `value` |  |
| `golds` | `value` |  |

Used as:

```lua
WebService:GetBlockymodsGoldReward(userId, self.GameType, gold)
```

<a id="fn-webservice-getblockymodsreward"></a>

#### `WebService:GetBlockymodsReward (userId, gameType, data)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `gameType` | `value` |  |
| `data` | `table` |  |

Used as:

```lua
WebService:GetBlockymodsReward(userId, self.GameType, data)
```

<a id="fn-webservice-getblockymodsrewardlist"></a>

#### `WebService:GetBlockymodsRewardList (data)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `data` | `table` | The instance this method was called on. |

Used as:

```lua
WebService:GetBlockymodsRewardList(data)
```

<a id="fn-webservice-getblockymodsuserattr"></a>

#### `WebService:GetBlockymodsUserAttr (userId, gameType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `gameType` | `value` |  |

Used as:

```lua
WebService:GetBlockymodsUserAttr(self.userId, BaseMain:getGameType())
```

<a id="fn-webservice-getblockymodsuserexp"></a>

#### `WebService:GetBlockymodsUserExp (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
WebService:GetBlockymodsUserExp(userId)
```

<a id="fn-webservice-getblockymodsuserinfos"></a>

#### `WebService:GetBlockymodsUserInfos (userIds, key)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userIds` | `value` | The instance this method was called on. |
| `key` | `identifier` |  |

Used as:

```lua
WebService:GetBlockymodsUserInfos(ids, key)
```

<a id="fn-webservice-getblockymodsuserseasoninfo"></a>

#### `WebService:GetBlockymodsUserSeasonInfo (userId, isLast)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `isLast` | `boolean` |  |

Used as:

```lua
WebService:GetBlockymodsUserSeasonInfo(self.userId, false)
WebService:GetBlockymodsUserSeasonInfo(userId, isLast)
```

<a id="fn-webservice-getplayerreward"></a>

#### `WebService:GetPlayerReward (userId, gametype, kills, iswin, gameId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `gametype` | `value` |  |
| `kills` | `value` |  |
| `iswin` | `boolean` |  |
| `gameId` | `identifier` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-webservice-getuserattr"></a>

#### `WebService:GetUserAttr (userId, gameType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `gameType` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-webservice-postgameresult"></a>

#### `WebService:PostGameResult (gameType, resultJson)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `gameType` | `value` | The instance this method was called on. |
| `resultJson` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-webservice-reportblockymodsdata"></a>

#### `WebService:ReportBlockymodsData (userId, gameType, time, kills, rank, isCount, integral, type)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `gameType` | `value` |  |
| `time` | `number` |  |
| `kills` | `value` |  |
| `rank` | `value` |  |
| `isCount` | `number` |  |
| `integral` | `value` |  |
| `type` | `value` |  |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-webservice-reportblockymodslist"></a>

#### `WebService:ReportBlockymodsList (data)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `data` | `table` | The instance this method was called on. |

Used as:

```lua
WebService:ReportBlockymodsList(self.WaitReportCache)
```

<a id="fn-webservice-saveblockymodsusersexp"></a>

#### `WebService:SaveBlockymodsUsersExp (data)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `data` | `table` | The instance this method was called on. |

Used as:

```lua
WebService:SaveBlockymodsUsersExp(data)
```

<a id="fn-webservice-saveblockymodsusershonor"></a>

#### `WebService:SaveBlockymodsUsersHonor (data)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `data` | `table` | The instance this method was called on. |

Used as:

```lua
WebService:SaveBlockymodsUsersHonor(data)
```

<a id="fn-webservice-updateblockymodsuserseasonreward"></a>

#### `WebService:UpdateBlockymodsUserSeasonReward (userId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |

Used as:

```lua
WebService:UpdateBlockymodsUserSeasonReward(player.userId)
```

<a id="fn-webservice-useprops"></a>

#### `WebService:UseProps (userId, PropId)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `PropId` | `object` | Pass a table or class value. |

_Not called from the Lua sources; invoked by the engine host or passed as a callback._

<a id="fn-webservice-checkmails"></a>

#### `WebService:checkMails (userId, gameType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `gameType` | `value` |  |

Used as:

```lua
WebService:checkMails(player.userId, "g1031")
```

<a id="fn-webservice-init"></a>

#### `WebService:init (version, config)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `version` | `value` | The instance this method was called on. |
| `config` | `value` |  |

Used as:

```lua
WebService:init(ver, config)
```

<a id="fn-webservice-initgameid"></a>

#### `WebService:initGameId ()`

Method. Colon form: the first argument is the receiver.

Used as:

```lua
WebService:initGameId()
```

<a id="fn-webservice-sendmails"></a>

#### `WebService:sendMails (fromUser, toUser, mailType, title, content, gameType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `fromUser` | `value` | The instance this method was called on. |
| `toUser` | `value` |  |
| `mailType` | `value` |  |
| `title` | `value` |  |
| `content` | `string` |  |
| `gameType` | `value` |  |

Used as:

```lua
WebService:sendMails(0, self.userId, MailType.LevelUp, title, data, "g1031")
WebService:sendMails(player.userId, helperId, MailType.Task, title, data, "g1031")
WebService:sendMails(0, player.userId, MailType.Explore, GameConfig.itemMailTilte, data, "g1031")
```

<a id="fn-webservice-sendmsg"></a>

#### `WebService:sendMsg (userId, targetIds, content, msgType, scope, gameType)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `targetIds` | `value` |  |
| `content` | `string` |  |
| `msgType` | `string` |  |
| `scope` | `value` |  |
| `gameType` | `value` |  |

Used as:

```lua
WebService:sendMsg(self.userId, 0, json_content, 3, 'game', "g1031")
```

<a id="fn-webservice-updatemails"></a>

#### `WebService:updateMails (userId, id, status)`

Method. Colon form: the first argument is the receiver.

| Argument | Role | Notes |
| --- | --- | --- |
| `userId` | `identifier` | The instance this method was called on. |
| `id` | `identifier` |  |
| `status` | `value` |  |

Used as:

```lua
WebService:updateMails(callBackPlayer.userId, v.id, WebRequestType.EMAIL_STATUS_RECEIVED)
```


### #

<a id="fn-copy"></a>

#### `_copy (object)`

File-local function. Declared `local`, so it is private to its file.

| Argument | Role | Notes |
| --- | --- | --- |
| `object` | `value` |  |

Used as:

```lua
_copy(key)
_copy(object)
```

---

## 5. Constants and enums

Named tables exposed by the common layer. Read them; they are intended as shared configuration rather than mutable state. Function assignments and values longer than 60 characters are omitted.

<a id="const-appprops"></a>

### `AppProps`

| Member | Value |
| --- | --- |
| `TYPE_BUFF` | `2` |
| `TYPE_CUSTOM` | `3` |
| `TYPE_ITEM` | `1` |

<a id="const-blockid"></a>

### `BlockID`

| Member | Value |
| --- | --- |
| `AIR` | `0` |
| `ANVIL` | `145` |
| `BEACON` | `138` |
| `BED` | `26` |
| `BEDROCK` | `7` |
| `BLOCK_CLAY` | `82` |
| `BLOCK_COAL` | `173` |
| `BLOCK_DIAMOND` | `57` |
| `BLOCK_EMERALD` | `133` |
| `BLOCK_GOLD` | `41` |
| `BLOCK_IRON` | `42` |
| `BLOCK_LAPIS` | `22` |
| `BLOCK_NETHER_QUARTZ` | `155` |
| `BLOCK_REDSTONE` | `152` |
| `BLOCK_SNOW` | `80` |
| `BOOK_SHELF` | `47` |
| `BREWING_STAND` | `117` |
| `BRICK` | `45` |
| `CACTUS` | `81` |
| `CAKE` | `92` |
| `CARROT` | `141` |
| `CAULDRON` | `118` |
| `CHEST` | `54` |
| `CHEST_TRAPPED` | `146` |
| `CLAY_HARDENED` | `172` |
| `CLAY_HARDENED_STAINED` | `159` |
| `CLOTH` | `35` |
| `COBBLESTONE` | `4` |
| `COBBLESTONE_MOSSY` | `48` |
| `COBBLESTONE_WALL` | `139` |
| `COCOA_PLANT` | `127` |
| `COMMAND_BLOCK` | `137` |
| `CROPS` | `59` |
| `DAYLIGHT_SENSOR` | `151` |
| `DEAD_BUSH` | `32` |
| `DIRT` | `3` |
| `DISPENSER` | `23` |
| `DOOR_IRON` | `71` |
| `DOOR_WOOD` | `64` |
| `DRAGON_EGG` | `122` |
| `DROPPER` | `158` |
| `ENCHANTMENT_TABLE` | `116` |
| `ENDER_CHEST` | `130` |
| `END_PORTAL` | `119` |
| `END_PORTAL_FRAME` | `120` |
| `FENCE` | `85` |
| `FENCE_GATE` | `107` |
| `FENCE_IRON` | `101` |
| `FIRE` | `51` |
| `FLOWER_POT` | `140` |
| `FURNACE_BURNING` | `62` |
| `FURNACE_IDLE` | `61` |
| `GLASS` | `20` |
| `GLOW_STONE` | `89` |
| `GRASS` | `2` |
| `GRAVEL` | `13` |
| `HAY_BLOCK` | `170` |
| `HOPPER_BLOCK` | `154` |
| `ICE` | `79` |
| `JUKEBOX` | `84` |
| `LADDER` | `65` |
| `LAVA_MOVING` | `10` |
| `LAVA_STILL` | `11` |
| `LEAVES` | `18` |
| `LEVER` | `69` |
| `LOCKED_CHEST` | `95` |
| `MELON` | `103` |
| `MELON_STEM` | `105` |
| `MOB_SPAWNER` | `52` |
| `MUSHROOM_BROWN` | `39` |
| `MUSHROOM_CAPBROWN` | `99` |
| `MUSHROOM_CAP_RED` | `100` |
| `MUSHROOM_RED` | `40` |
| `MUSIC` | `25` |
| `MYCELIUM` | `110` |
| `NETHERRACK` | `87` |
| `NETHER_BRICK` | `112` |
| `NETHER_FENCE` | `113` |
| `NETHER_STALK` | `115` |
| `OBSIDIAN` | `49` |
| `ORE_COAL` | `16` |
| `ORE_DIAMOND` | `56` |
| `ORE_EMERALD` | `129` |
| `ORE_GOLD` | `14` |
| `ORE_IRON` | `15` |
| `ORE_LAPIS` | `21` |
| `ORE_NETHER_QUARTZ` | `153` |
| `ORE_READSTONE_GLOWING` | `74` |
| `ORE_REDSTONE` | `73` |
| `PACKED_ICE` | `174` |
| `PISTON_BASE` | `33` |
| `PISTON_EXTENSION` | `34` |
| `PISTON_MOVING` | `36` |
| `PISTON_STICKY_BASE` | `29` |
| `PLANKS` | `5` |
| `PLANT_RED` | `38` |
| `PLANT_YELLOW` | `37` |
| `PORTAL` | `90` |
| `POTATO` | `142` |
| `PRESSURE_PLATE_GOLD` | `147` |
| `PRESSURE_PLATE_IRON` | `148` |
| `PRESSURE_PLATE_PLANKS` | `72` |
| `PRESSURE_PLATE_STONE` | `70` |
| `PUMPKIN` | `86` |
| `PUMPKIN_LANTERN` | `91` |
| `PUMPKIN_STEM` | `104` |
| `RAIL` | `66` |
| `RAIL_ACTIVATOR` | `157` |
| `RAIL_DETECTOR` | `28` |
| `RAIL_POWERED` | `27` |
| `RAIL_RECEIVE` | `175` |
| `REDSTONE_COMPARATOR_ACTIVE` | `150` |
| `REDSTONE_COMPARATOR_IDLE` | `149` |
| `REDSTONE_LAMP_ACTIVE` | `124` |
| `REDSTONE_LAMP_IDLE` | `123` |
| `REDSTONE_REPEATER_ACTIVE` | `94` |
| `REDSTONE_REPEATER_IDLE` | `93` |
| `REDSTONE_WIRE` | `55` |
| `REED` | `83` |
| `SAND` | `12` |
| `SAND_STONE` | `24` |
| `SAPLING` | `6` |
| `SIGN_POST` | `63` |
| `SIGN_WALL` | `68` |
| `SILVERFISH` | `97` |
| `SKULL` | `144` |
| `SLOW_SAND` | `88` |
| `SNOW` | `78` |
| `SPONGE` | `19` |
| `STAIRS_BRICK` | `108` |
| `STAIRS_COBBLESTONE` | `67` |
| `STAIRS_NETHER_BRICK` | `114` |
| `STAIRS_NETHER_QUARTZ` | `156` |
| `STAIRS_SAND_STONE` | `128` |
| `STAIRS_STONE_BRICK` | `109` |
| `STAIRS_WOOD_BIRCH` | `135` |
| `STAIRS_WOOD_JUNGLE` | `136` |
| `STAIRS_WOOD_OAK` | `53` |
| `STAIRS_WOOD_SPRUCE` | `134` |
| `STONE` | `1` |
| `STONE_BRICK` | `98` |
| `STONE_BUTTON` | `77` |
| `STONE_DOUBLE_SLAB` | `43` |
| `STONE_SINGLE_SLAB` | `44` |
| `TALL_GRASS` | `31` |
| `THIN_GLASS` | `102` |
| `TILLED_FIELD` | `60` |
| `TNT` | `46` |
| `TORCH_REDSTONE_ACTIVE` | `76` |
| `TORCH_REDSTONE_IDLE` | `75` |
| `TORCH_WOOD` | `50` |
| `TRAPDOOR` | `96` |
| `TRIP_WIRE` | `132` |
| `TRIP_WIRE_SOURCE` | `131` |
| `VINE` | `106` |
| `WATERLILY` | `111` |
| `WATER_MOVING` | `8` |
| `WATER_STILL` | `9` |
| `WEB` | `30` |
| `WHITE_STONE` | `121` |
| `WOOD` | `17` |
| `WOOD2` | `162` |
| `WOODEN_BUTTON` | `143` |
| `WOOD_DOUBLE_SLAB` | `125` |
| `WOOD_SINGLE_SLAB` | `126` |
| `WOOL_CARPET` | `171` |
| `WORK_BENCH` | `58` |

<a id="const-exprule"></a>

### `ExpRule`

| Member | Value |
| --- | --- |
| `DayMaxExp` | `1000` |
| `Enable` | `true` |
| `ExpCoefficient` | `1` |
| `MaxLv` | `0` |

<a id="const-gameovercode"></a>

### `GameOverCode`

| Member | Value |
| --- | --- |
| `GameOver` | `1` |
| `GameStarted` | `2` |
| `LoadDBDataError` | `6` |
| `NoThisTeam` | `5` |
| `PlayerIsEnough` | `3` |
| `ReadyNoPlayer` | `4` |
| `Success` | `0` |

<a id="const-imessages"></a>

### `IMessages`

| Member | Value |
| --- | --- |
| `TYPE_PLAYER` | `0` |
| `TYPE_TEAM` | `1` |
| `UNIT_MINUTE` | `0` |
| `UNIT_SECOND` | `1` |

<a id="const-itask"></a>

### `ITask`

| Member | Value |
| --- | --- |
| `created` | `false` |
| `status` | `0` |
| `timer` | `0` |

<a id="const-itemid"></a>

### `ItemID`

| Member | Value |
| --- | --- |
| `APPLE_GOLD` | `322` |
| `APPLE_RED` | `260` |
| `ARROW` | `262` |
| `AXE_DIAMOND` | `279` |
| `AXE_GOLD` | `286` |
| `AXE_IRON` | `258` |
| `AXE_STONE` | `275` |
| `AXE_WOOD` | `271` |
| `BAKED_POTATO` | `393` |
| `BED` | `355` |
| `BEEF_COOKED` | `364` |
| `BEEF_RAW` | `363` |
| `BLAZE_POWDER` | `377` |
| `BLAZE_ROD` | `369` |
| `BLOCKMAN_CRATE` | `342` |
| `BLOCKMAN_EMPTY` | `328` |
| `BLOCKMAN_HOPPER` | `408` |
| `BLOCKMAN_POWERED` | `343` |
| `BLOCKMAN_TNT` | `407` |
| `BOAT` | `333` |
| `BONE` | `352` |
| `BOOK` | `340` |
| `BOOTS_CHAIN` | `305` |
| `BOOTS_DIAMOND` | `313` |
| `BOOTS_GOLD` | `317` |
| `BOOTS_IRON` | `309` |
| `BOOTS_LEATHER` | `301` |
| `BOW` | `261` |
| `BOWL_EMPTY` | `281` |
| `BOWL_SOUP` | `282` |
| `BREAD` | `297` |
| `BREWING_STAND` | `379` |
| `BRICK` | `336` |
| `BUCKET_EMPTY` | `325` |
| `BUCKET_LAVA` | `327` |
| `BUCKET_MILK` | `335` |
| `BUCKET_WATER` | `326` |
| `CAKE` | `354` |
| `CARROT` | `391` |
| `CARROT_ON_STICK` | `398` |
| `CAULDRON` | `380` |
| `CHICKEN_COOKED` | `366` |
| `CHICKEN_RAW` | `365` |
| `CLAY` | `337` |
| `COAL` | `263` |
| `COMPARATOR` | `404` |
| `COMPASS` | `345` |
| `COOKIE` | `357` |
| `DIAMOND` | `264` |
| `DIAMOND_HORSE_ARMOR` | `419` |
| `DOOR_IRON` | `330` |
| `DOOR_WOOD` | `324` |
| `DYE_POWDER` | `351` |
| `EGG` | `344` |
| `EMERALD` | `388` |
| `EMPTY_MAP` | `395` |
| `ENCHANTED_BOOK` | `403` |
| `ENDER_PEARL` | `368` |
| `EXP_BOTTLE` | `384` |
| `EYE_OF_ENDER` | `381` |
| `FEATHER` | `288` |
| `FERMENTED_SPIDER_EYE` | `376` |
| `FIREBALL_CHARGE` | `385` |
| `FIREWORK` | `401` |
| `FIREWORK_CHARGE` | `402` |
| `FISHING_ROD` | `346` |
| `FISH_COOKED` | `350` |
| `FISH_RAW` | `349` |
| `FLINT` | `318` |
| `FLINT_AND_STEEL` | `259` |
| `FLOWER_POT` | `390` |
| `FROZEN_BALL` | `770` |
| `GHAST_TEAR` | `370` |
| `GLASS_BOTTLE` | `374` |
| `GLOW_STONE` | `348` |
| `GOLDEN_CARROT` | `396` |
| `GOLDEN_HORSE_ARMOR` | `418` |
| `GOLD_ARROW` | `500` |
| `GOLD_HEART` | `498` |
| `GOLD_NUGGET` | `371` |
| `GOLD_SHOES` | `499` |
| `GRENADE` | `425` |
| `GRENADE_TREMOR` | `769` |
| `GUN_POWDER` | `289` |
| `HAND_CUFFS` | `451` |
| `HELMET_CHAIN` | `302` |
| `HELMET_DIAMOND` | `310` |
| `HELMET_GOLD` | `314` |
| `HELMET_IRON` | `306` |
| `HELMET_LEATHER` | `298` |
| `HOE_DIAMOND` | `293` |
| `HOE_GOLD` | `294` |
| `HOE_IRON` | `292` |
| `HOE_STONE` | `291` |
| `HOE_WOOD` | `290` |
| `IMAP` | `358` |
| `INGOT_GOLD` | `266` |
| `INGOT_IRON` | `265` |
| `IRON_HORSE_ARMOR` | `417` |
| `ITEM_FRAME` | `389` |
| `LEAD` | `420` |
| `LEATHER` | `334` |
| `LEGS_CHAIN` | `304` |
| `LEGS_DIAMOND` | `312` |
| `LEGS_GOLD` | `316` |
| `LEGS_IRON` | `308` |
| `LEGS_LEATHER` | `300` |
| `MAGMA_CREAM` | `378` |
| `MELON` | `360` |
| `MELON_SEEDS` | `362` |
| `MONSTER_PLACER` | `383` |
| `NAME_TAG` | `421` |
| `NETHERRACK_BRICK` | `405` |
| `NETHER_QUARTZ` | `406` |
| `NETHER_STALK_SEEDS` | `372` |
| `NETHER_STAR` | `399` |
| `NULL` | `0` |
| `PAINTING` | `321` |
| `PAPER` | `339` |
| `PICKAXE_DIAMOND` | `278` |
| `PICKAXE_IRON` | `257` |
| `PICKAXE_STONE` | `274` |
| `PICKAXE_WOOD` | `270` |
| `PICK_AXE_GOLD` | `285` |
| `PLATE_CHAIN` | `303` |
| `PLATE_DIAMOND` | `311` |
| `PLATE_GOLD` | `315` |
| `PLATE_IRON` | `307` |
| `PLATE_LEATHER` | `299` |
| `POCKET_SUNDIAL` | `347` |
| `POISONOUS_POTATO` | `394` |
| `PORK_COOKED` | `320` |
| `PORK_RAW` | `319` |
| `POTATO` | `392` |
| `POTION` | `373` |
| `PROP_EXP` | `503` |
| `PROP_GOLD` | `502` |
| `PROP_IGNORE` | `505` |
| `PROP_RSTEP` | `504` |
| `PUMPKIN_PIE` | `400` |
| `PUMPKIN_SEEDS` | `361` |
| `QUESTION_MARK` | `501` |
| `RABBIT_COOKED` | `412` |
| `RED_STONE` | `331` |
| `RED_STONE_REPEATER` | `356` |
| `REED` | `338` |
| `ROTTEN_FLESH` | `367` |
| `SADDLE` | `329` |
| `SEEDS` | `295` |
| `SHEARS` | `359` |
| `SHOVEL_DIAMOND` | `277` |
| `SHOVEL_GOLD` | `284` |
| `SHOVEL_IRON` | `256` |
| `SHOVEL_STONE` | `273` |
| `SHOVEL_WOOD` | `269` |
| `SIGN` | `323` |
| `SILK` | `287` |
| `SKULL` | `397` |
| `SLIME_BALL` | `341` |
| `SNOWBALL` | `332` |
| `SPECKLED_MELON` | `382` |
| `SPIDER_EYE` | `375` |
| `STICK` | `280` |
| `SUGAR` | `353` |
| `SWORD_DIAMOND` | `276` |
| `SWORD_GOLD` | `283` |
| `SWORD_IRON` | `267` |
| `SWORD_STONE` | `272` |
| `SWORD_WOOD` | `268` |
| `TELEPORTHOME` | `462` |
| `WHEAT` | `296` |
| `WRITABLE_BOOK` | `386` |
| `WRITTEN_BOOK` | `387` |

<a id="const-luatimer"></a>

### `LuaTimer`

| Member | Value |
| --- | --- |
| `CurTaskKey` | `0` |
| `CurTaskKey` | `LuaTimer.CurTaskKey + 1` |
| `CurTaskKey` | `LuaTimer.CurTaskKey + 1` |

<a id="const-mailtype"></a>

### `MailType`

| Member | Value |
| --- | --- |
| `BeInvited` | `3     -- 被邀请人(接受邀请的人)奖励邮件` |
| `EveryDayBeSend` | `5     -- 每日互赠 被送者(接受的人)奖励邮件` |
| `EveryDaySend` | `4     -- 每日互赠 发送者(发送的人)奖励邮件` |
| `Explore` | `1     -- 探险奖励邮件` |
| `Invite` | `2     -- 邀请人(发送邀请的人)奖励邮件` |
| `LevelUp` | `7     -- 玩家等级提升奖励` |
| `Task` | `6     -- 任务奖励邮件` |

<a id="const-potionid"></a>

### `PotionID`

| Member | Value |
| --- | --- |
| `ABSORPTION` | `22` |
| `BLINDNESS` | `15` |
| `CONFUSION` | `9` |
| `DAMAGE_BOOST` | `5` |
| `DIG_SLOWDOWN` | `4` |
| `DIG_SPEED` | `3` |
| `FIRE_REGISTANCE` | `12` |
| `HARM` | `7` |
| `HEAL` | `6` |
| `HEALTH_BOOST` | `21` |
| `HUNGER` | `17` |
| `INVISIBILITY` | `14` |
| `JUMP` | `8` |
| `MOVE_SLOWDOWN` | `2` |
| `MOVE_SPEED` | `1` |
| `NIGHT_VISION` | `16` |
| `POISON` | `19` |
| `REGENERATION` | `10` |
| `RESISTANCE` | `11` |
| `SATURATION` | `23` |
| `WATER_BREATHING` | `13` |
| `WEAKNESS` | `18` |
| `WITHER` | `20` |

<a id="const-potionitemid"></a>

### `PotionItemID`

| Member | Value |
| --- | --- |
| `POTION_HEAL` | `517` |
| `POTION_INVISIBILITY` | `512` |
| `POTION_JUMP` | `514` |
| `POTION_LONG_INVISIBILITY` | `513` |
| `POTION_LONG_JUMP` | `515` |
| `POTION_LONG_MOVE_SPEED` | `520` |
| `POTION_LONG_NIGHT_VISION` | `526` |
| `POTION_LONG_POISON` | `523` |
| `POTION_MEDICINE_PACK` | `483` |
| `POTION_MEDICINE_POTION` | `484` |
| `POTION_MOVE_SPEED` | `519` |
| `POTION_NIGHT_VISION` | `525` |
| `POTION_POISON` | `522` |
| `POTION_SPLASH_HEAL` | `527` |
| `POTION_SPLASH_LONG_POISON` | `530` |
| `POTION_SPLASH_POISON` | `529` |
| `POTION_SPLASH_STRONG_HEAL` | `528` |
| `POTION_SPLASH_STRONG_POISON` | `531` |
| `POTION_STRONG_HEAL` | `518` |
| `POTION_STRONG_JUMP` | `516` |
| `POTION_STRONG_MOVE_SPEED` | `521` |
| `POTION_STRONG_POISON` | `524` |

<a id="const-reportmanager"></a>

### `ReportManager`

| Member | Value |
| --- | --- |
| `GameRankType` | `ReportManager.RankType.Inc` |
| `GameType` | `"g1001"` |
| `IsCloseServer` | `false` |
| `RankType` | `{ Inc = 0, Max = 1 }` |

<a id="const-rewardmanager"></a>

### `RewardManager`

| Member | Value |
| --- | --- |
| `GameType` | `"g1001"` |
| `StandardNum` | `10` |
| `StartPlayerCount` | `nil` |
| `StartPlayerTime` | `nil` |

<a id="const-scoreid"></a>

### `ScoreID`

| Member | Value |
| --- | --- |
| `DESTROY_BED` | `"5" -- 破坏床` |
| `DIE` | `"2" -- 死亡` |
| `FLAG` | `"6" -- 夺旗` |
| `KILL` | `"1"  -- 击杀` |
| `LIVE` | `"3" -- 生存` |
| `POINT` | `"7" -- 积分` |
| `SERIAL_KILL` | `"4" -- 连续击杀` |
| `WIN` | `"8" -- 胜局数` |

<a id="const-textformat"></a>

### `TextFormat`

| Member | Value |
| --- | --- |
| `argsSplit` | `"\t"` |
| `argsTipType` | `"#"` |
| `colorAqua` | `TextFormat.colorStart .. "FF00FFFF"` |
| `colorBlack` | `TextFormat.colorStart .. "FF000000"` |
| `colorBlue` | `TextFormat.colorStart .. "FF0000FF"` |
| `colorEnd` | `"▥"` |
| `colorGold` | `TextFormat.colorStart .. "FFFFD700"` |
| `colorGreen` | `TextFormat.colorStart .. "FF00FF00"` |
| `colorOrange` | `TextFormat.colorStart .. "FFFF8000"` |
| `colorPurple` | `TextFormat.colorStart .. "FF884898"` |
| `colorRed` | `TextFormat.colorStart .. "FFFF0000"` |
| `colorStart` | `"▢"` |
| `colorWrite` | `TextFormat.colorStart .. "FFFFFFFF"` |
| `colorYellow` | `TextFormat.colorStart .. "FFFFFF00"` |

<a id="const-webrequesttype"></a>

### `WebRequestType`

| Member | Value |
| --- | --- |
| `BLOCKYMODS_EXP_RULE` | `"blockymods.exp.rule"` |
| `BLOCKYMODS_GOLD_REWARD` | `"blockymods.user.gold.reward:"` |
| `BLOCKYMODS_REPORT` | `"blockymods.user.report:"` |
| `BLOCKYMODS_REPORT_LIST` | `"blockymods.report.list"` |
| `BLOCKYMODS_REWARD` | `"blockymods.user.reward:"` |
| `BLOCKYMODS_REWARD_LIST` | `"blockymods.reward.list"` |
| `BLOCKYMODS_SAVE_EXP` | `"blockymods.save.exp"` |
| `BLOCKYMODS_SAVE_HONOR` | `"blockymods.save.honor"` |
| `BLOCKYMODS_USER_ATTR` | `"blockymods.user.attr:"` |
| `BLOCKYMODS_USER_EXP` | `"blockymods.user.exp:"` |
| `BLOCKYMODS_USER_INFOS` | `"blockymods.user.infos:"` |
| `BLOCKYMODS_USER_SEASON_INFO` | `"blockymods.user.season.info:"` |
| `BLOCKYMODS_USER_SEASON_REWARD` | `"blockymods.user.season.reward:"` |
| `CHECK_MAILS` | `"blockymods.checkMails:"` |
| `EMAIL_STATUS_DELETED` | `"DELETED"` |
| `EMAIL_STATUS_READ` | `"READ"` |
| `EMAIL_STATUS_RECEIVED` | `"RECEIVED"` |
| `EMAIL_STATUS_UNREAD` | `"UNREAD"` |
| `MCPEONLINE_REPORT` | `"mcpeonline.report"` |
| `MCPEONLINE_REWARD` | `"mcpeonline.user.reward:"` |
| `MCPEONLINE_USER_ATTR` | `"mcpeonline.user.attr:"` |
| `SEND_MAILS` | `"blockymods.sendMails:"` |
| `SEND_MSG` | `"blockymods.sendMsg:"` |
| `UPDATE_MAILS` | `"blockymods.updateMails:"` |

<a id="const-webservice"></a>

### `WebService`

| Member | Value |
| --- | --- |
| `BlockymodBaseUrl` | `nil` |
| `BlockymodsExpRule` | `"/activity/api/v1/inner/activity/games/settlement/rule"` |
| `BlockymodsReportData` | `"/game/api/v1/inner/users/games/reporting"` |
| `BlockymodsReportList` | `"/game/api/v1/inner/users/games/reporting/list"` |
| `BlockymodsRewardAPI` | `"/game/api/v1/users/games/settlement"` |
| `BlockymodsRewardGoldAPI` | `"/game/api/v1/users/games/settlement/golds"` |
| `BlockymodsRewardList` | `"/game/api/v1/users/inner/games/settlement/list"` |
| `BlockymodsSaveExp` | `"/activity/api/v1/inner/activity/games/records"` |
| `BlockymodsSaveHonor` | `"/gameaide/api/v1/inner/user/segment/integral"` |
| `BlockymodsUserAttr` | `"/user/api/v1/inner/user/details"` |
| `BlockymodsUserExp` | `"/activity/api/v1/inner/activity/games/user/exp"` |
| `BlockymodsUserInfos` | `"/user/api/v1/inner/simple-info"` |
| `BlockymodsUserSeasonInfo` | `"/gameaide/api/v1/inner/user/segment/info"` |
| `BlockymodsUserSeasonReward` | `"/gameaide/api/v1/inner/segment/integral/reward"` |
| `ConsumeAPI` | `"/props/api/{version}/consume/{userid}"` |
| `MailsApi` | `"/gameaide/api/v1/inner/game/mails"` |
| `MailsStatusApi` | `"/gameaide/api/v1/inner/game/mails/{id}/status"` |
| `PropAddr` | `nil` |
| `RankAPI` | `"/minigame/api/{version}/ranks/{gametype}"` |
| `RankAddr` | `nil` |
| `RewardAPI` | `"/minigame/api/{version}/rewards/{gametype}/{userId}"` |
| `RewardAddr` | `nil` |
| `SendMsgApi` | `"/gameaide/api/v1/inner/msg/send"` |
| `UserAttrPath` | `"/minigame/i/api/{version}/user-details/{userId}"` |
| `ver` | `nil` |

<a id="const-cache"></a>

### `cache`

| Member | Value |
| --- | --- |
| `integral` | `cache.integral + (integral or 0)` |
| `isCount` | `1` |
| `kill` | `cache.kill + (kill or 0)` |
| `meanTime` | `cache.meanTime + (time or 0)` |
| `rank` | `rank or 0` |
| `type` | `self.GameRankType` |

<a id="const-cls"></a>

### `cls`

| Member | Value |
| --- | --- |
| `__cname` | `classname` |
| `__cname` | `classname` |
| `__create` | `super` |
| `__create` | `super.__create` |
| `__ctype` | `1` |
| `__ctype` | `2 -- lua` |
| `__index` | `cls` |
| `__index` | `cls` |
| `__meta` | `mt` |
| `__metatable` | `cls` |
| `__type` | `type` |
| `super` | `super` |
| `super` | `super` |

<a id="const-data"></a>

### `data`

| Member | Value |
| --- | --- |
| `ActorName` | `actor or ""` |
| `EffectName` | `effect or ""` |
| `HeadName` | `name or ""` |
| `IsCanObstruct` | `isCanObstruct` |
| `IsCanObstruct` | `true` |
| `IsCollided` | `isCollided` |
| `IsCollided` | `true` |
| `SkillName` | `skill or ""` |
| `armsId` | `armsId` |
| `color` | `color` |
| `dead` | `dead` |
| `duration` | `os.time() - startGameTime` |
| `gameId` | `gameType` |
| `head` | `head` |
| `integral` | `integral` |
| `isCount` | `isCount` |
| `kill` | `kills` |
| `killer` | `killer` |
| `kills` | `kills` |
| `meanTime` | `time` |
| `name` | `player.name` |
| `name` | `player.name` |
| `own` | `own` |
| `rank` | `rank` |
| `rank` | `rank` |
| `rank` | `rank` |
| `rank` | `rank or 10` |
| `recordId` | `WebService.gameId` |
| `score` | `tonumber(score)` |
| `score` | `tonumber(score)` |
| `standardNum` | `self.StandardNum` |
| `startTime` | `os.date("%Y-%m-%d %H:%M:%S", startGameTime)` |
| `totalNum` | `startPlayerCount` |
| `type` | `type` |
| `userId` | `tonumber(tostring(player.userId))` |
| `userId` | `tonumber(tostring(player.userId))` |
| `vip` | `player.vip` |
| `vip` | `player.vip` |

<a id="const-expinfo"></a>

### `expInfo`

| Member | Value |
| --- | --- |
| `addExp` | `addExp` |
| `addExp` | `expInfo.addExp + addExp` |
| `hasGet` | `expInfo.hasGet + addExp` |
| `isAdd` | `true` |
| `time` | `os.time()` |

<a id="const-item"></a>

### `item`

| Member | Value |
| --- | --- |
| `bullets` | `info.bullets` |
| `bullets` | `info.bullets` |
| `content` | `v.content` |
| `duration` | `os.time() - startGameTime` |
| `effects` | `effects` |
| `enchants` | `enchants` |
| `fromUser` | `v.fromUser` |
| `gameId` | `self.GameType` |
| `id` | `info.id` |
| `id` | `info.id` |
| `id` | `prop.id` |
| `id` | `v.id` |
| `isRepeat` | `tonumber(prop.isRepeat or "1") == 1` |
| `itemId` | `tonumber(prop.itemId)` |
| `itemMeta` | `tonumber(prop.itemMeta)` |
| `itemNum` | `tonumber(prop.itemNum)` |
| `mailType` | `v.mailType` |
| `meta` | `0` |
| `meta` | `0` |
| `meta` | `0` |
| `meta` | `info.meta` |
| `meta` | `info.meta` |
| `num` | `1` |
| `num` | `1` |
| `num` | `1` |
| `num` | `info.num` |
| `num` | `info.num` |
| `rank` | `cache.rank or 10` |
| `recordId` | `WebService.gameId` |
| `stackIndex` | `i - 1` |
| `standardNum` | `self.StandardNum` |
| `startTime` | `os.date("%Y-%m-%d %H:%M:%S", startGameTime)` |
| `status` | `v.status` |
| `title` | `v.title` |
| `toUser` | `v.toUser` |
| `totalNum` | `startPlayerCount` |
| `type` | `tonumber(prop.type)` |
| `userId` | `cache.userId` |

<a id="const-player"></a>

### `player`

| Member | Value |
| --- | --- |
| `adSwitch` | `reward.adSwitch or 0` |
| `adSwitch` | `reward.adSwitch or 0` |
| `appIntegral` | `0` |
| `available` | `reward.available or 0` |
| `available` | `reward.available or 0` |
| `available` | `reward.available or 0` |
| `gold` | `reward.golds or 0` |
| `gold` | `reward.golds or 0` |
| `gold` | `reward.golds or 0` |
| `hasGet` | `reward.hasGet or 0` |
| `hasGet` | `reward.hasGet or 0` |
| `hasGet` | `reward.hasGet or 0` |
| `vip` | `player.staticAttr.vip` |

<a id="const-player-staticattr"></a>

### `player.staticAttr`

| Member | Value |
| --- | --- |
| `attack` | `data.att` |
| `clanId` | `data.clanId` |
| `clanName` | `data.clanName` |
| `clanName` | `string.gsub(data.clanName or "", "\n", "")` |
| `classes` | `data.classes` |
| `defense` | `data.def` |
| `hasInit` | `true` |
| `hasInit` | `true` |
| `health` | `data.heal` |
| `lv` | `data.lv` |
| `props` | `data.propsId or {}` |
| `role` | `data.role` |
| `supperPlayer` | `data.spp` |
| `title` | `data.title` |
| `vip` | `data.vip` |

<a id="const-self"></a>

### `self`

| Member | Value |
| --- | --- |
| `BlockymodBaseUrl` | `config.blockymodsRewardAddr` |
| `DayMaxExp` | `data.maxExpPerDay` |
| `Enable` | `data.needSettle` |
| `Enable` | `false` |
| `ExpCoefficient` | `data.gameCoefficient / 100` |
| `GameRankType` | `RankType` |
| `GameType` | `GameType` |
| `GameType` | `GameType` |
| `GameType` | `GameType` |
| `IsCloseServer` | `true` |
| `MaxLv` | `self.MaxLv + 1` |
| `MaxPlayer` | `maxPlayer` |
| `PropAddr` | `config.propAddr` |
| `RankAddr` | `config.rankAddr` |
| `RewardAddr` | `config.rewardAddr` |
| `StartGameTime` | `os.time()` |
| `StartPlayerCount` | `PlayerManager:getPlayerCount()` |
| `UpgradeExpTable` | `data.upgradeRule` |
| `appIntegral` | `0` |
| `available` | `0` |
| `clientPeer` | `clientPeer` |
| `clientPeer` | `nil` |
| `clientPeer` | `nil` |
| `created` | `true` |
| `day` | `tonumber(d)` |
| `defaultRank` | `10` |
| `entityPlayerMP` | `clientPeer:getEntityPlayer()` |
| `entityPlayerMP` | `nil` |
| `entityPlayerMP` | `nil` |
| `fraction` | `0` |
| `fraction` | `f` |
| `fraction` | `tonumber(f) * math.pow(10, 3 - #f)` |
| `gameId` | `uuid()` |
| `gold` | `0` |
| `hasGet` | `0` |
| `hour` | `tonumber(h or 0)` |
| `inGameTime` | `os.time()` |
| `isReady` | `false` |
| `minute` | `tonumber(i or 0)` |
| `month` | `tonumber(m)` |
| `name` | `clientPeer:getName()` |
| `name` | `nil` |
| `rakssid` | `0` |
| `rakssid` | `clientPeer:getRakssid()` |
| `respawnTimes` | `-1` |
| `respawnTimes` | `self.respawnTimes + 1` |
| `second` | `tonumber(s or 0)` |
| `status` | `status` |
| `timer` | `SecTimer.createTimer(function(ticks)` |
| `timezone` | `z` |
| `userId` | `0` |
| `userId` | `clientPeer:getPlatformUserId()` |
| `ver` | `version` |
| `vip` | `0` |
| `vip` | `self.dynamicAttr.vip` |
| `world` | `world` |
| `year` | `tonumber(y)` |

<a id="const-self-dynamicattr"></a>

### `self.dynamicAttr`

| Member | Value |
| --- | --- |
| `clz` | `0` |
| `clz` | `attr.class` |
| `hasInit` | `false` |
| `hasInit` | `true` |
| `pioneer` | `attr.pioneer` |
| `pioneer` | `true` |
| `region` | `0` |
| `region` | `attr.region` |
| `team` | `0` |
| `team` | `attr.team` |
| `vip` | `0` |
| `vip` | `attr.vip` |

<a id="const-self-staticattr"></a>

### `self.staticAttr`

| Member | Value |
| --- | --- |
| `attack` | `0 -- att` |
| `clanId` | `0 -- clanId` |
| `clanName` | `"" -- clanName` |
| `classes` | `0 --classes` |
| `defense` | `0 -- def` |
| `hasInit` | `false` |
| `health` | `0 --heal` |
| `lv` | `0 -- lv` |
| `props` | `{} --props` |
| `role` | `-1 -- role` |
| `supperPlayer` | `0 -- spp` |
| `title` | `"" -- title` |
| `vip` | `0 -- vip` |

<a id="const-settlement"></a>

### `settlement`

| Member | Value |
| --- | --- |
| `activeValues` | `0` |
| `activeValues` | `data.use_energy` |
| `exp` | `0` |
| `exp` | `data.gain_exp` |
| `goldCoins` | `0` |
| `goldCoins` | `data.gain_gold` |
| `kills` | `0` |

<a id="const-task"></a>

### `task`

| Member | Value |
| --- | --- |
| `delay` | `-1` |
| `delay` | `delay` |
| `delay` | `delay` |
| `period` | `period` |
| `period` | `period` |
| `time` | `time` |
| `time` | `time` |
| `time` | `time` |
| `time` | `time` |
| `time` | `time` |
| `times` | `task.times - 1` |
| `type` | `TaskType.Delay` |
| `type` | `TaskType.DelayAndPeriod` |
| `type` | `TaskType.Period` |

<a id="const-v"></a>

### `v`

| Member | Value |
| --- | --- |
| `choose` | `false` |
| `choose` | `true` |
| `num` | `64` |
| `probUpper` | `total` |
| `probfloor` | `total` |

<a id="const-vec3"></a>

### `vec3`

| Member | Value |
| --- | --- |
| `x` | `vec3i.x` |
| `x` | `x` |
| `y` | `vec3i.y` |
| `y` | `y` |
| `z` | `vec3i.z` |
| `z` | `z` |

---

## 6. Index

All names, alphabetical. Entries link to their reference section.

**A** &middot; [`AppProps:hasProp`](#fn-appprops-hasprop) &middot; [`AppProps:init`](#fn-appprops-init) &middot; [`AppProps:initProps`](#fn-appprops-initprops) &middot; [`AppProps:useAppProp`](#fn-appprops-useappprop) &middot; [`AppProps:useAppProps`](#fn-appprops-useappprops)

**B** &middot; [`BITWISE`](#fn-bitwise) &middot; [`BaseListener.onGameInit`](#fn-baselistener-ongameinit) &middot; [`BaseListener.onGetDataFromDB`](#fn-baselistener-ongetdatafromdb) &middot; [`BaseListener.onPlayerDynamicAttr`](#fn-baselistener-onplayerdynamicattr) &middot; [`BaseListener.onPlayerFirstSpawn`](#fn-baselistener-onplayerfirstspawn) &middot; [`BaseListener.onPlayerLogin`](#fn-baselistener-onplayerlogin) &middot; [`BaseListener.onPlayerLogout`](#fn-baselistener-onplayerlogout) &middot; [`BaseListener.onPlayerReady`](#fn-baselistener-onplayerready) &middot; [`BaseListener.onPlayerRespawn`](#fn-baselistener-onplayerrespawn) &middot; [`BaseListener.registerCallBack`](#fn-baselistener-registercallback) &middot; [`BaseListener:init`](#fn-baselistener-init) &middot; [`BaseMain:getGameType`](#fn-basemain-getgametype) &middot; [`BaseMain:setGameType`](#fn-basemain-setgametype)

**C** &middot; [`CBasePlayer:addBackpackCapacity`](#fn-cbaseplayer-addbackpackcapacity) &middot; [`CBasePlayer:addCurrency`](#fn-cbaseplayer-addcurrency) &middot; [`CBasePlayer:addCustomEffect`](#fn-cbaseplayer-addcustomeffect) &middot; [`CBasePlayer:addEchantmentItem`](#fn-cbaseplayer-addechantmentitem) &middot; [`CBasePlayer:addEchantmentsItem`](#fn-cbaseplayer-addechantmentsitem) &middot; [`CBasePlayer:addEffect`](#fn-cbaseplayer-addeffect) &middot; [`CBasePlayer:addGunBulletNum`](#fn-cbaseplayer-addgunbulletnum) &middot; [`CBasePlayer:addGunDamage`](#fn-cbaseplayer-addgundamage) &middot; [`CBasePlayer:addGunItem`](#fn-cbaseplayer-addgunitem) &middot; [`CBasePlayer:addGunItemToEnderChest`](#fn-cbaseplayer-addgunitemtoenderchest) &middot; [`CBasePlayer:addHealth`](#fn-cbaseplayer-addhealth) &middot; [`CBasePlayer:addItem`](#fn-cbaseplayer-additem) &middot; [`CBasePlayer:addItemToEnderChest`](#fn-cbaseplayer-additemtoenderchest) &middot; [`CBasePlayer:addNightVisionPotionEffect`](#fn-cbaseplayer-addnightvisionpotioneffect) &middot; [`CBasePlayer:addVehicles`](#fn-cbaseplayer-addvehicles) &middot; [`CBasePlayer:buildShowName`](#fn-cbaseplayer-buildshowname) &middot; [`CBasePlayer:changeClothes`](#fn-cbaseplayer-changeclothes) &middot; [`CBasePlayer:changeHeart`](#fn-cbaseplayer-changeheart) &middot; [`CBasePlayer:changeMaxHealth`](#fn-cbaseplayer-changemaxhealth) &middot; [`CBasePlayer:clearEffects`](#fn-cbaseplayer-cleareffects) &middot; [`CBasePlayer:clearInv`](#fn-cbaseplayer-clearinv) &middot; [`CBasePlayer:consumeDiamonds`](#fn-cbaseplayer-consumediamonds) &middot; [`CBasePlayer:ctor`](#fn-cbaseplayer-ctor) &middot; [`CBasePlayer:equipArmor`](#fn-cbaseplayer-equiparmor) &middot; [`CBasePlayer:fillEnderInvetory`](#fn-cbaseplayer-fillenderinvetory) &middot; [`CBasePlayer:fillInvetory`](#fn-cbaseplayer-fillinvetory) &middot; [`CBasePlayer:getAttack`](#fn-cbaseplayer-getattack) &middot; [`CBasePlayer:getBirdSimulator`](#fn-cbaseplayer-getbirdsimulator) &middot; [`CBasePlayer:getCurrency`](#fn-cbaseplayer-getcurrency) &middot; [`CBasePlayer:getDefence`](#fn-cbaseplayer-getdefence) &middot; [`CBasePlayer:getDiamond`](#fn-cbaseplayer-getdiamond) &middot; [`CBasePlayer:getDisplayName`](#fn-cbaseplayer-getdisplayname) &middot; [`CBasePlayer:getEnderInventory`](#fn-cbaseplayer-getenderinventory) &middot; [`CBasePlayer:getEntityId`](#fn-cbaseplayer-getentityid) &middot; [`CBasePlayer:getEntityPlayer`](#fn-cbaseplayer-getentityplayer) &middot; [`CBasePlayer:getFoodLevel`](#fn-cbaseplayer-getfoodlevel) &middot; [`CBasePlayer:getGold`](#fn-cbaseplayer-getgold) &middot; [`CBasePlayer:getHealth`](#fn-cbaseplayer-gethealth) &middot; [`CBasePlayer:getHeldItemId`](#fn-cbaseplayer-gethelditemid) &middot; [`CBasePlayer:getInventory`](#fn-cbaseplayer-getinventory) &middot; [`CBasePlayer:getItemNumById`](#fn-cbaseplayer-getitemnumbyid) &middot; [`CBasePlayer:getMaxHealth`](#fn-cbaseplayer-getmaxhealth) &middot; [`CBasePlayer:getPosition`](#fn-cbaseplayer-getposition) &middot; [`CBasePlayer:getRanch`](#fn-cbaseplayer-getranch) &middot; [`CBasePlayer:getSex`](#fn-cbaseplayer-getsex) &middot; [`CBasePlayer:getYaw`](#fn-cbaseplayer-getyaw) &middot; [`CBasePlayer:initDynamicAttr`](#fn-cbaseplayer-initdynamicattr) &middot; [`CBasePlayer:initMemberVariables`](#fn-cbaseplayer-initmembervariables) &middot; [`CBasePlayer:initStaticAttr`](#fn-cbaseplayer-initstaticattr) &middot; [`CBasePlayer:isVisitor`](#fn-cbaseplayer-isvisitor) &middot; [`CBasePlayer:isWatchMode`](#fn-cbaseplayer-iswatchmode) &middot; [`CBasePlayer:leaveVehicle`](#fn-cbaseplayer-leavevehicle) &middot; [`CBasePlayer:onPlayerLogout`](#fn-cbaseplayer-onplayerlogout) &middot; [`CBasePlayer:onUseCustomProp`](#fn-cbaseplayer-onusecustomprop) &middot; [`CBasePlayer:playSkillEffect`](#fn-cbaseplayer-playskilleffect) &middot; [`CBasePlayer:removeEffect`](#fn-cbaseplayer-removeeffect) &middot; [`CBasePlayer:removeItem`](#fn-cbaseplayer-removeitem) &middot; [`CBasePlayer:replaceItem`](#fn-cbaseplayer-replaceitem) &middot; [`CBasePlayer:resetBackpack`](#fn-cbaseplayer-resetbackpack) &middot; [`CBasePlayer:resetClothes`](#fn-cbaseplayer-resetclothes) &middot; [`CBasePlayer:respawn`](#fn-cbaseplayer-respawn) &middot; [`CBasePlayer:sendLoginResult`](#fn-cbaseplayer-sendloginresult) &middot; [`CBasePlayer:sendSkillEffect`](#fn-cbaseplayer-sendskilleffect) &middot; [`CBasePlayer:setAllowFlying`](#fn-cbaseplayer-setallowflying) &middot; [`CBasePlayer:setArmItem`](#fn-cbaseplayer-setarmitem) &middot; [`CBasePlayer:setBagInfo`](#fn-cbaseplayer-setbaginfo) &middot; [`CBasePlayer:setBirdConvert`](#fn-cbaseplayer-setbirdconvert) &middot; [`CBasePlayer:setCurrency`](#fn-cbaseplayer-setcurrency) &middot; [`CBasePlayer:setFoodLevel`](#fn-cbaseplayer-setfoodlevel) &middot; [`CBasePlayer:setHealth`](#fn-cbaseplayer-sethealth) &middot; [`CBasePlayer:setOnFire`](#fn-cbaseplayer-setonfire) &middot; [`CBasePlayer:setOnFrozen`](#fn-cbaseplayer-setonfrozen) &middot; [`CBasePlayer:setPersonalShopArea`](#fn-cbaseplayer-setpersonalshoparea) &middot; [`CBasePlayer:setPositionAndRotation`](#fn-cbaseplayer-setpositionandrotation) &middot; [`CBasePlayer:setRespawnPos`](#fn-cbaseplayer-setrespawnpos) &middot; [`CBasePlayer:setShowName`](#fn-cbaseplayer-setshowname) &middot; [`CBasePlayer:setSpeedAddition`](#fn-cbaseplayer-setspeedaddition) &middot; [`CBasePlayer:setTeam`](#fn-cbaseplayer-setteam) &middot; [`CBasePlayer:setUnlockedCommodities`](#fn-cbaseplayer-setunlockedcommodities) &middot; [`CBasePlayer:subBackpackCapacity`](#fn-cbaseplayer-subbackpackcapacity) &middot; [`CBasePlayer:subCurrency`](#fn-cbaseplayer-subcurrency) &middot; [`CBasePlayer:subGunRecoil`](#fn-cbaseplayer-subgunrecoil) &middot; [`CBasePlayer:subHealth`](#fn-cbaseplayer-subhealth) &middot; [`CBasePlayer:teleportPos`](#fn-cbaseplayer-teleportpos) &middot; [`CBasePlayer:teleportPosWithYaw`](#fn-cbaseplayer-teleportposwithyaw) &middot; [`CBasePlayer:updateExp`](#fn-cbaseplayer-updateexp) &middot; [`CBasePlayer:useAppProps`](#fn-cbaseplayer-useappprops) &middot; `CommonTask` &middot; [`CsvUtil.getRowContent`](#fn-csvutil-getrowcontent) &middot; [`CsvUtil.loadCsvFile`](#fn-csvutil-loadcsvfile) &middot; [`class`](#fn-class) &middot; [`class.__meta.__call`](#fn-class-meta-call) &middot; [`class.def`](#fn-class-def) &middot; [`clone`](#fn-clone) &middot; [`cls.ctor`](#fn-cls-ctor) &middot; [`cls.new`](#fn-cls-new) &middot; [`codepoint_to_utf8`](#fn-codepoint-to-utf8) &middot; [`countindent`](#fn-countindent) &middot; [`create_set`](#fn-create-set)

**D** &middot; [`DBManager:addCache`](#fn-dbmanager-addcache) &middot; [`DBManager:getCache`](#fn-dbmanager-getcache) &middot; [`DBManager:getPlayerData`](#fn-dbmanager-getplayerdata) &middot; [`DBManager:initPlayerData`](#fn-dbmanager-initplayerdata) &middot; [`DBManager:removeCache`](#fn-dbmanager-removecache) &middot; [`DBManager:savePlayerData`](#fn-dbmanager-saveplayerdata) &middot; [`DateUtil.getDate`](#fn-dateutil-getdate) &middot; [`DateUtil.getDateString`](#fn-dateutil-getdatestring) &middot; [`DateUtil.getDayLastTime`](#fn-dateutil-getdaylasttime) &middot; [`DateUtil.getWeekLastTime`](#fn-dateutil-getweeklasttime) &middot; [`DateUtil.getWeekSeconds`](#fn-dateutil-getweekseconds) &middot; [`DateUtil.getWeeksOfYear`](#fn-dateutil-getweeksofyear) &middot; [`DateUtil.getYearDay`](#fn-dateutil-getyearday) &middot; [`DateUtil.getYearWeek`](#fn-dateutil-getyearweek) &middot; [`decode_error`](#fn-decode-error) &middot; [`doT2S`](#fn-dot2s)

**E** &middot; [`EngineUtil.disposePlatformOrder`](#fn-engineutil-disposeplatformorder) &middot; [`EngineUtil.sendEnterOtherGame`](#fn-engineutil-sendenterothergame) &middot; [`EngineUtil.sendGotoOtherGame`](#fn-engineutil-sendgotoothergame) &middot; [`EngineWorld:addActorNpc`](#fn-engineworld-addactornpc) &middot; [`EngineWorld:addCreatureNpc`](#fn-engineworld-addcreaturenpc) &middot; [`EngineWorld:addEntityItem`](#fn-engineworld-addentityitem) &middot; [`EngineWorld:addGoodsGroupToShop`](#fn-engineworld-addgoodsgrouptoshop) &middot; [`EngineWorld:addGoodsToShop`](#fn-engineworld-addgoodstoshop) &middot; [`EngineWorld:addMerchantNpc`](#fn-engineworld-addmerchantnpc) &middot; [`EngineWorld:addRankNpc`](#fn-engineworld-addranknpc) &middot; [`EngineWorld:addSessionNpc`](#fn-engineworld-addsessionnpc) &middot; [`EngineWorld:addSimpleEffect`](#fn-engineworld-addsimpleeffect) &middot; [`EngineWorld:addVehicleNpc`](#fn-engineworld-addvehiclenpc) &middot; [`EngineWorld:changePlayerActor`](#fn-engineworld-changeplayeractor) &middot; [`EngineWorld:getBlockId`](#fn-engineworld-getblockid) &middot; [`EngineWorld:getEntity`](#fn-engineworld-getentity) &middot; [`EngineWorld:getWorld`](#fn-engineworld-getworld) &middot; [`EngineWorld:removeEntity`](#fn-engineworld-removeentity) &middot; [`EngineWorld:resetMap`](#fn-engineworld-resetmap) &middot; [`EngineWorld:restorePlayerActor`](#fn-engineworld-restoreplayeractor) &middot; [`EngineWorld:setBlock`](#fn-engineworld-setblock) &middot; [`EngineWorld:setBlockToAir`](#fn-engineworld-setblocktoair) &middot; [`EngineWorld:setWorld`](#fn-engineworld-setworld) &middot; [`EngineWorld:stopWorldTime`](#fn-engineworld-stopworldtime) &middot; [`ExpRule:checkLvUp`](#fn-exprule-checklvup) &middot; [`ExpRule:checkTodayExpFull`](#fn-exprule-checktodayexpfull) &middot; [`ExpRule:correctAddExp`](#fn-exprule-correctaddexp) &middot; [`ExpRule:disable`](#fn-exprule-disable) &middot; [`ExpRule:getAddExp`](#fn-exprule-getaddexp) &middot; [`ExpRule:getAverageUpExp`](#fn-exprule-getaverageupexp) &middot; [`ExpRule:initRole`](#fn-exprule-initrole) &middot; [`ExpRule:isEnable`](#fn-exprule-isenable) &middot; [`ExpRule:isExpMaxToday`](#fn-exprule-isexpmaxtoday) &middot; [`encode`](#fn-encode) &middot; [`encode_nil`](#fn-encode-nil) &middot; [`encode_number`](#fn-encode-number) &middot; [`encode_string`](#fn-encode-string) &middot; [`encode_table`](#fn-encode-table) &middot; [`escape_char`](#fn-escape-char)

**G** &middot; [`GameInitEvent`](#event-gameinitevent) &middot; [`GameLog.log`](#fn-gamelog-log) &middot; [`GetDataFromDBEvent`](#event-getdatafromdbevent)

**H** &middot; [`HostApi.addCoinMapping`](#hostapi-addcoinmapping) &middot; [`HostApi.addCommodity`](#hostapi-addcommodity) &middot; [`HostApi.broadCastMsg`](#hostapi-broadcastmsg) &middot; [`HostApi.broadCastMsgByType`](#hostapi-broadcastmsgbytype) &middot; [`HostApi.changeCommodityList`](#hostapi-changecommoditylist) &middot; [`HostApi.getDBData`](#hostapi-getdbdata) &middot; [`HostApi.log`](#hostapi-log) &middot; [`HostApi.platformOrderDispose`](#hostapi-platformorderdispose) &middot; [`HostApi.resetPos`](#hostapi-resetpos) &middot; [`HostApi.resetPosWithYaw`](#hostapi-resetposwithyaw) &middot; [`HostApi.sendAppExpResult`](#hostapi-sendappexpresult) &middot; [`HostApi.sendEnterOtherGame`](#hostapi-sendenterothergame) &middot; [`HostApi.sendGameStatus`](#hostapi-sendgamestatus) &middot; [`HostApi.sendGameover`](#hostapi-sendgameover) &middot; [`HostApi.sendGotoOtherGame`](#hostapi-sendgotoothergame) &middot; [`HostApi.sendKillMsg`](#hostapi-sendkillmsg) &middot; [`HostApi.sendRankData`](#hostapi-sendrankdata) &middot; [`HostApi.sendStartGame`](#hostapi-sendstartgame) &middot; [`HostApi.setDBData`](#hostapi-setdbdata) &middot; [`HostApi.setDBLogData`](#hostapi-setdblogdata) &middot; [`HostApi.setServerGameType`](#hostapi-setservergametype) &middot; [`HostApi.stopServer`](#hostapi-stopserver) &middot; [`HttpListener.onHttpResponse`](#fn-httplistener-onhttpresponse) &middot; [`HttpListener:init`](#fn-httplistener-init) &middot; `HttpRequest` &middot; [`HttpResponseEvent`](#event-httpresponseevent)

**I** &middot; [`ICommonTask:onTick`](#fn-icommontask-ontick) &middot; [`IMessages:longWaitingHint`](#fn-imessages-longwaitinghint) &middot; [`IMessages:msgCanStartGame`](#fn-imessages-msgcanstartgame) &middot; [`IMessages:msgCloseServerTimeHint`](#fn-imessages-msgcloseservertimehint) &middot; [`IMessages:msgDontBuyGoodsByNoStart`](#fn-imessages-msgdontbuygoodsbynostart) &middot; [`IMessages:msgDoubleKill`](#fn-imessages-msgdoublekill) &middot; [`IMessages:msgFightGameStart`](#fn-imessages-msgfightgamestart) &middot; [`IMessages:msgFightGameStartTimeHint`](#fn-imessages-msgfightgamestarttimehint) &middot; [`IMessages:msgFirstKill`](#fn-imessages-msgfirstkill) &middot; [`IMessages:msgFiveKill`](#fn-imessages-msgfivekill) &middot; [`IMessages:msgGameEndTimeHint`](#fn-imessages-msggameendtimehint) &middot; [`IMessages:msgGameHasStarting`](#fn-imessages-msggamehasstarting) &middot; [`IMessages:msgGameOver`](#fn-imessages-msggameover) &middot; [`IMessages:msgGameOverByDBDataError`](#fn-imessages-msggameoverbydbdataerror) &middot; [`IMessages:msgGameOverByLittle`](#fn-imessages-msggameoverbylittle) &middot; [`IMessages:msgGameOverDeath`](#fn-imessages-msggameoverdeath) &middot; [`IMessages:msgGameOverHint`](#fn-imessages-msggameoverhint) &middot; [`IMessages:msgGameOverWin`](#fn-imessages-msggameoverwin) &middot; [`IMessages:msgGamePlayerIsEnough`](#fn-imessages-msggameplayerisenough) &middot; [`IMessages:msgGamePlayerNum`](#fn-imessages-msggameplayernum) &middot; [`IMessages:msgGameResetHint`](#fn-imessages-msggameresethint) &middot; [`IMessages:msgGameRunning`](#fn-imessages-msggamerunning) &middot; [`IMessages:msgGameScore`](#fn-imessages-msggamescore) &middot; [`IMessages:msgGameStart`](#fn-imessages-msggamestart) &middot; [`IMessages:msgGameStartTimeHint`](#fn-imessages-msggamestarttimehint) &middot; [`IMessages:msgNoWinner`](#fn-imessages-msgnowinner) &middot; [`IMessages:msgPlayerCount`](#fn-imessages-msgplayercount) &middot; [`IMessages:msgPlayerKillPlayer`](#fn-imessages-msgplayerkillplayer) &middot; [`IMessages:msgPlayerLeft`](#fn-imessages-msgplayerleft) &middot; [`IMessages:msgPlayerQuit`](#fn-imessages-msgplayerquit) &middot; [`IMessages:msgPvpEnableAfter`](#fn-imessages-msgpvpenableafter) &middot; [`IMessages:msgReadyHint`](#fn-imessages-msgreadyhint) &middot; [`IMessages:msgRespawn`](#fn-imessages-msgrespawn) &middot; [`IMessages:msgTeleportToDeadPlace`](#fn-imessages-msgteleporttodeadplace) &middot; [`IMessages:msgThirdKill`](#fn-imessages-msgthirdkill) &middot; [`IMessages:msgTimeTick`](#fn-imessages-msgtimetick) &middot; [`IMessages:msgWaitPlayer`](#fn-imessages-msgwaitplayer) &middot; [`IMessages:msgWaitPlayerEnough`](#fn-imessages-msgwaitplayerenough) &middot; [`IMessages:msgWelcomePlayer`](#fn-imessages-msgwelcomeplayer) &middot; [`IMessages:msgWinnerInfo`](#fn-imessages-msgwinnerinfo) &middot; [`INT2HEX`](#fn-int2hex) &middot; [`ITask:ctor`](#fn-itask-ctor) &middot; [`ITask:onCreate`](#fn-itask-oncreate) &middot; [`ITask:onTick`](#fn-itask-ontick) &middot; [`ITask:pureStop`](#fn-itask-purestop) &middot; [`ITask:start`](#fn-itask-start) &middot; [`ITask:stop`](#fn-itask-stop) &middot; [`InventoryUtil:fillItem`](#fn-inventoryutil-fillitem) &middot; [`InventoryUtil:fillItemByType`](#fn-inventoryutil-fillitembytype) &middot; [`InventoryUtil:getPlayerEnderInventoryInfo`](#fn-inventoryutil-getplayerenderinventoryinfo) &middot; [`InventoryUtil:getPlayerInventoryInfo`](#fn-inventoryutil-getplayerinventoryinfo) &middot; [`InventoryUtil:prepareChest`](#fn-inventoryutil-preparechest) &middot; [`InventoryUtil:randomItem`](#fn-inventoryutil-randomitem) &middot; [`InventoryUtil:randomItemNum`](#fn-inventoryutil-randomitemnum) &middot; `Item` &middot; [`isemptyline`](#fn-isemptyline)

**J** &middot; [`json.decode`](#fn-json-decode) &middot; [`json.encode`](#fn-json-encode)

**L** &middot; [`LuaTimer:cancel`](#fn-luatimer-cancel) &middot; [`LuaTimer:onTick`](#fn-luatimer-ontick) &middot; [`LuaTimer:schedule`](#fn-luatimer-schedule) &middot; [`LuaTimer:scheduleTimer`](#fn-luatimer-scheduletimer) &middot; [`ltrim`](#fn-ltrim)

**M** &middot; [`M.new`](#fn-m-new) &middot; [`M.randomseed`](#fn-m-randomseed) &middot; [`M.seed`](#fn-m-seed) &middot; [`MerchantUtil:addCommodity`](#fn-merchantutil-addcommodity) &middot; [`MerchantUtil:changeCommodityList`](#fn-merchantutil-changecommoditylist) &middot; [`MsgSender.sendBottomTips`](#fn-msgsender-sendbottomtips) &middot; [`MsgSender.sendBottomTipsToTarget`](#fn-msgsender-sendbottomtipstotarget) &middot; [`MsgSender.sendCenterTips`](#fn-msgsender-sendcentertips) &middot; [`MsgSender.sendCenterTipsToTarget`](#fn-msgsender-sendcentertipstotarget) &middot; [`MsgSender.sendKillMsg`](#fn-msgsender-sendkillmsg) &middot; [`MsgSender.sendKillMsgToTarget`](#fn-msgsender-sendkillmsgtotarget) &middot; [`MsgSender.sendMsg`](#fn-msgsender-sendmsg) &middot; [`MsgSender.sendMsgToTarget`](#fn-msgsender-sendmsgtotarget) &middot; [`MsgSender.sendOtherTips`](#fn-msgsender-sendothertips) &middot; [`MsgSender.sendOtherTipsToTarget`](#fn-msgsender-sendothertipstotarget) &middot; [`MsgSender.sendTopTips`](#fn-msgsender-sendtoptips) &middot; [`MsgSender.sendTopTipsToTarget`](#fn-msgsender-sendtoptipstotarget)

**N** &middot; [`Null.__tostring`](#fn-null-tostring) &middot; [`Null.isnull`](#fn-null-isnull) &middot; [`NumberUtil.getIntPart`](#fn-numberutil-getintpart) &middot; [`next_char`](#fn-next-char)

**P** &middot; [`PlayerClassInfo:addClasses`](#fn-playerclassinfo-addclasses) &middot; [`PlayerClassInfo:getClasses`](#fn-playerclassinfo-getclasses) &middot; [`PlayerClassInfo:removeClasses`](#fn-playerclassinfo-removeclasses) &middot; [`PlayerDynamicAttrEvent`](#event-playerdynamicattrevent) &middot; [`PlayerFirstSpawnEvent`](#event-playerfirstspawnevent) &middot; [`PlayerLoginEvent`](#event-playerloginevent) &middot; [`PlayerLogoutEvent`](#event-playerlogoutevent) &middot; [`PlayerManager:addPlayer`](#fn-playermanager-addplayer) &middot; [`PlayerManager:copyPlayers`](#fn-playermanager-copyplayers) &middot; [`PlayerManager:getMaxPlayer`](#fn-playermanager-getmaxplayer) &middot; [`PlayerManager:getPlayerByEntityId`](#fn-playermanager-getplayerbyentityid) &middot; [`PlayerManager:getPlayerByPlayerMP`](#fn-playermanager-getplayerbyplayermp) &middot; [`PlayerManager:getPlayerByRakssid`](#fn-playermanager-getplayerbyrakssid) &middot; [`PlayerManager:getPlayerByUserId`](#fn-playermanager-getplayerbyuserid) &middot; [`PlayerManager:getPlayerCount`](#fn-playermanager-getplayercount) &middot; [`PlayerManager:getPlayers`](#fn-playermanager-getplayers) &middot; [`PlayerManager:isPlayerEmpty`](#fn-playermanager-isplayerempty) &middot; [`PlayerManager:isPlayerEnough`](#fn-playermanager-isplayerenough) &middot; [`PlayerManager:isPlayerExists`](#fn-playermanager-isplayerexists) &middot; [`PlayerManager:isPlayerFull`](#fn-playermanager-isplayerfull) &middot; [`PlayerManager:setMaxPlayer`](#fn-playermanager-setmaxplayer) &middot; [`PlayerManager:subPlayer`](#fn-playermanager-subplayer) &middot; [`PlayerReadyEvent`](#event-playerreadyevent) &middot; [`PlayerRespawnEvent`](#event-playerrespawnevent) &middot; [`parse`](#fn-parse) &middot; [`parse_array`](#fn-parse-array) &middot; [`parse_literal`](#fn-parse-literal) &middot; [`parse_number`](#fn-parse-number) &middot; [`parse_object`](#fn-parse-object) &middot; [`parse_string`](#fn-parse-string) &middot; [`parse_unicode_escape`](#fn-parse-unicode-escape) &middot; [`parseblockstylestring`](#fn-parseblockstylestring) &middot; [`parsedocuments`](#fn-parsedocuments) &middot; [`parseflowstyle`](#fn-parseflowstyle) &middot; [`parsemap`](#fn-parsemap) &middot; [`parsescalar`](#fn-parsescalar) &middot; [`parseseq`](#fn-parseseq) &middot; [`parseset`](#fn-parseset) &middot; [`parsestring`](#fn-parsestring) &middot; [`parsetimestamp`](#fn-parsetimestamp)

**R** &middot; [`RankManager:addPlayerDayRank`](#fn-rankmanager-addplayerdayrank) &middot; [`RankManager:addPlayerWeekRank`](#fn-rankmanager-addplayerweekrank) &middot; [`RankManager:buildPlayerRank`](#fn-rankmanager-buildplayerrank) &middot; [`RankManager:clearRanks`](#fn-rankmanager-clearranks) &middot; [`RankManager:removePlayerRank`](#fn-rankmanager-removeplayerrank) &middot; [`RankManager:sendPlayerRank`](#fn-rankmanager-sendplayerrank) &middot; [`ReportManager:closeServer`](#fn-reportmanager-closeserver) &middot; [`ReportManager:getUserReportCache`](#fn-reportmanager-getuserreportcache) &middot; [`ReportManager:onPlayerLogout`](#fn-reportmanager-onplayerlogout) &middot; [`ReportManager:reportUserCount`](#fn-reportmanager-reportusercount) &middot; [`ReportManager:reportUserData`](#fn-reportmanager-reportuserdata) &middot; [`ReportManager:reportUserIntegral`](#fn-reportmanager-reportuserintegral) &middot; [`ReportManager:reportUserKill`](#fn-reportmanager-reportuserkill) &middot; [`ReportManager:reportUserRank`](#fn-reportmanager-reportuserrank) &middot; [`ReportManager:reportUserTime`](#fn-reportmanager-reportusertime) &middot; [`ReportManager:setGameType`](#fn-reportmanager-setgametype) &middot; [`ReportManager:setRankType`](#fn-reportmanager-setranktype) &middot; [`ReportManager:tryCloseServer`](#fn-reportmanager-trycloseserver) &middot; [`ReportManager:tryReportGameData`](#fn-reportmanager-tryreportgamedata) &middot; [`RewardManager:addRewardQueue`](#fn-rewardmanager-addrewardqueue) &middot; [`RewardManager:getQueueReward`](#fn-rewardmanager-getqueuereward) &middot; [`RewardManager:getStartGameTime`](#fn-rewardmanager-getstartgametime) &middot; [`RewardManager:getStartPlayerCount`](#fn-rewardmanager-getstartplayercount) &middot; [`RewardManager:getUserGoldReward`](#fn-rewardmanager-getusergoldreward) &middot; [`RewardManager:getUserReward`](#fn-rewardmanager-getuserreward) &middot; [`RewardManager:isUserRewardFinish`](#fn-rewardmanager-isuserrewardfinish) &middot; [`RewardManager:setGameType`](#fn-rewardmanager-setgametype) &middot; [`RewardManager:startGame`](#fn-rewardmanager-startgame) &middot; [`rtrim`](#fn-rtrim)

**S** &middot; [`SeasonManager:addUserHonor`](#fn-seasonmanager-adduserhonor) &middot; [`SeasonManager:trySaveHonorResult`](#fn-seasonmanager-trysavehonorresult) &middot; `SecTimer` &middot; [`StringUtil.split`](#fn-stringutil-split) &middot; [`StringUtil.v2s`](#fn-stringutil-v2s) &middot; [`select`](#fn-select) &middot; [`startswith`](#fn-startswith) &middot; [`startswithline`](#fn-startswithline)

**T** &middot; [`TextFormat:getTipType`](#fn-textformat-gettiptype) &middot; [`types.timestamp:__init`](#fn-types-timestamp-init) &middot; [`types.timestamp:__tostring`](#fn-types-timestamp-tostring) &middot; [`types.timestamp:gettz`](#fn-types-timestamp-gettz)

**U** &middot; [`UserExpManager:addExpCache`](#fn-userexpmanager-addexpcache) &middot; [`UserExpManager:addUserExp`](#fn-userexpmanager-adduserexp) &middot; [`UserExpManager:getUserExpCache`](#fn-userexpmanager-getuserexpcache) &middot; [`UserExpManager:removeExpCache`](#fn-userexpmanager-removeexpcache) &middot; [`UserExpManager:sendExpResult`](#fn-userexpmanager-sendexpresult) &middot; [`UserExpManager:tryCalculationExp`](#fn-userexpmanager-trycalculationexp) &middot; [`UserExpManager:trySaveExpResult`](#fn-userexpmanager-trysaveexpresult) &middot; [`UserInfoCache:FilterExistedUserId`](#fn-userinfocache-filterexisteduserid) &middot; [`UserInfoCache:GetCache`](#fn-userinfocache-getcache) &middot; [`UserInfoCache:GetCacheByUserIds`](#fn-userinfocache-getcachebyuserids)

**V** &middot; `Vector3` &middot; [`Vector3.new`](#class-vector3) &middot; `Vector3i` &middot; [`Vector3i.new`](#class-vector3i) &middot; [`VectorUtil.equal`](#fn-vectorutil-equal) &middot; [`VectorUtil.hashVector3`](#fn-vectorutil-hashvector3) &middot; [`VectorUtil.newVector3`](#fn-vectorutil-newvector3) &middot; [`VectorUtil.newVector3i`](#fn-vectorutil-newvector3i) &middot; [`VectorUtil.toBlockVector3`](#fn-vectorutil-toblockvector3) &middot; [`VectorUtil.toVector3`](#fn-vectorutil-tovector3) &middot; [`VectorUtil.toVector3i`](#fn-vectorutil-tovector3i) &middot; [`v2s`](#fn-v2s) &middot; [`var_dump`](#fn-var-dump)

**W** &middot; [`WalletUtils:addCoinMappings`](#fn-walletutils-addcoinmappings) &middot; [`WebResponse:formatData`](#fn-webresponse-formatdata) &middot; [`WebResponse:formatResult`](#fn-webresponse-formatresult) &middot; [`WebResponse:formatResultCode`](#fn-webresponse-formatresultcode) &middot; [`WebResponse:onCallFunc`](#fn-webresponse-oncallfunc) &middot; [`WebResponse:onCheckMails`](#fn-webresponse-oncheckmails) &middot; [`WebResponse:onGetBlockymodsExp`](#fn-webresponse-ongetblockymodsexp) &middot; [`WebResponse:onGetBlockymodsExpRole`](#fn-webresponse-ongetblockymodsexprole) &middot; [`WebResponse:onGetBlockymodsGoldReward`](#fn-webresponse-ongetblockymodsgoldreward) &middot; [`WebResponse:onGetBlockymodsReward`](#fn-webresponse-ongetblockymodsreward) &middot; [`WebResponse:onGetBlockymodsRewardList`](#fn-webresponse-ongetblockymodsrewardlist) &middot; [`WebResponse:onGetBlockymodsUserAttr`](#fn-webresponse-ongetblockymodsuserattr) &middot; [`WebResponse:onGetBlockymodsUserInfos`](#fn-webresponse-ongetblockymodsuserinfos) &middot; [`WebResponse:onGetBlockymodsUserSeasonInfo`](#fn-webresponse-ongetblockymodsuserseasoninfo) &middot; [`WebResponse:onGetMcpeonlineReport`](#fn-webresponse-ongetmcpeonlinereport) &middot; [`WebResponse:onGetMcpeonlineReward`](#fn-webresponse-ongetmcpeonlinereward) &middot; [`WebResponse:onGetMcpeonlineUserAttr`](#fn-webresponse-ongetmcpeonlineuserattr) &middot; [`WebResponse:onReportBlockymodsData`](#fn-webresponse-onreportblockymodsdata) &middot; [`WebResponse:onReportBlockymodsList`](#fn-webresponse-onreportblockymodslist) &middot; [`WebResponse:onSaveBlockymodsExp`](#fn-webresponse-onsaveblockymodsexp) &middot; [`WebResponse:onSaveBlockymodsHonor`](#fn-webresponse-onsaveblockymodshonor) &middot; [`WebResponse:onSendMails`](#fn-webresponse-onsendmails) &middot; [`WebResponse:onUpdateBlockymodsUserSeasonReward`](#fn-webresponse-onupdateblockymodsuserseasonreward) &middot; [`WebResponse:onUpdateMails`](#fn-webresponse-onupdatemails) &middot; [`WebResponse:registerCallBack`](#fn-webresponse-registercallback) &middot; [`WebResponse:unregisterCallBack`](#fn-webresponse-unregistercallback) &middot; [`WebService:GetBlockymodsExpRule`](#fn-webservice-getblockymodsexprule) &middot; [`WebService:GetBlockymodsGoldReward`](#fn-webservice-getblockymodsgoldreward) &middot; [`WebService:GetBlockymodsReward`](#fn-webservice-getblockymodsreward) &middot; [`WebService:GetBlockymodsRewardList`](#fn-webservice-getblockymodsrewardlist) &middot; [`WebService:GetBlockymodsUserAttr`](#fn-webservice-getblockymodsuserattr) &middot; [`WebService:GetBlockymodsUserExp`](#fn-webservice-getblockymodsuserexp) &middot; [`WebService:GetBlockymodsUserInfos`](#fn-webservice-getblockymodsuserinfos) &middot; [`WebService:GetBlockymodsUserSeasonInfo`](#fn-webservice-getblockymodsuserseasoninfo) &middot; [`WebService:GetPlayerReward`](#fn-webservice-getplayerreward) &middot; [`WebService:GetUserAttr`](#fn-webservice-getuserattr) &middot; [`WebService:PostGameResult`](#fn-webservice-postgameresult) &middot; [`WebService:ReportBlockymodsData`](#fn-webservice-reportblockymodsdata) &middot; [`WebService:ReportBlockymodsList`](#fn-webservice-reportblockymodslist) &middot; [`WebService:SaveBlockymodsUsersExp`](#fn-webservice-saveblockymodsusersexp) &middot; [`WebService:SaveBlockymodsUsersHonor`](#fn-webservice-saveblockymodsusershonor) &middot; [`WebService:UpdateBlockymodsUserSeasonReward`](#fn-webservice-updateblockymodsuserseasonreward) &middot; [`WebService:UseProps`](#fn-webservice-useprops) &middot; [`WebService:checkMails`](#fn-webservice-checkmails) &middot; [`WebService:init`](#fn-webservice-init) &middot; [`WebService:initGameId`](#fn-webservice-initgameid) &middot; [`WebService:sendMails`](#fn-webservice-sendmails) &middot; [`WebService:sendMsg`](#fn-webservice-sendmsg) &middot; [`WebService:updateMails`](#fn-webservice-updatemails)

**#** &middot; [`_copy`](#fn-copy)

