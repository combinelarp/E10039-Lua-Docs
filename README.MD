# Blockman Lua Reference Manual

## Common Framework

Complete function reference for the shared Lua layer of the Blockman engine, extracted directly from the sources. Game-mode trees are out of scope; this document covers only the reusable framework that every mode builds on.

- **Scope:** `D:/ClooberGO SOURCE!/BlockMango10039/Script/Common`
- **Source root:** `D:/ClooberGO SOURCE!/BlockMango10039/Script`
- **Generated:** 2026-09-26 16:47

| Statistic | Count |
| --- | ---: |
| Lua files in scope | 47 |
| Source lines | 5,890 |
| **Functions documented** | **402** |
| Distinct signatures | 401 |
| Table / namespace definitions | 79 |
| Constants & enum members | 762 |
| Engine host methods used | 40 |
| Engine events used | 9 |
| Engine data classes used | 2 |

---

## Contents

- [1. Overview](#1-overview)
- [2. Engine Host API](#2-engine-host-api-used-by-the-common-layer)
- [3. Framework by Package](#3-framework-by-package)
- [4. Alphabetical Signature Index](#4-alphabetical-signature-index)
- [5. Constants and Enums](#5-constants-and-enums)
- [6. Module Dependencies](#6-module-dependencies)
- [Appendix A: File Inventory](#appendix-a-file-inventory)

---

## 1. Overview

The common layer sits between the engine host and the individual game modes. It owns four concerns:

1. **Event plumbing** - `base/listener` subscribes to the engine events and dispatches them to the active mode, so a mode never touches `registerCallBack` directly.
2. **Client protocol** - `base/messages` builds every packet sent to clients, and `base/web` performs the HTTP calls to the game server.
3. **Persistence and state** - `base/data`, `base/util` and `base/DBManager` wrap the host database and world accessors.
4. **Cross-cutting rules** - `base/rank`, `base/prop`, `base/exp`, `base/season` and `base/task` implement ranking, player attributes, progression, seasonal resets and the tick scheduler.

A mode supplies its own `Match.lua`, listeners and data objects, then delegates all shared behaviour to this layer.

### 1.1 Reading the entries

| Form | Meaning |
| --- | --- |
| `function Owner.name (a, b)` | Dot form: no implicit receiver. |
| `function Owner:name (a, b)` | Colon form: the first parameter is the implicit `self`. |
| `local function fn (a)` | File-local; not reachable from other files. |
| `Owner.fn = function (a)` | Function assigned to an existing table field. |

Every entry gives the file path relative to the source root plus a line number. The **Notes** column carries the nearest preceding `--` comment where the source provides one; the sources carry no formal annotations, so most notes are empty.

---

## 2. Engine Host API

Symbols the host injects into the Lua VM. They are implemented in C++ inside the game binary and are not present in these sources, so they are recovered by finding globals the common layer calls but never assigns. Only symbols reachable from `Common` are listed.

### 2.1 HostApi

The primary engine bridge, exposed as a singleton.

| Method | Calls in common layer | Example location |
| --- | ---: | --- |
| `addCoinMapping` | 1 | `Common/base/util/WalletUtils.lua` |
| `addCommodity` | 1 | `Common/base/util/MerchantUtil.lua` |
| `broadCastMsg` | 1 | `Common/base/util/MsgSender.lua` |
| `broadCastMsgByType` | 1 | `Common/base/util/MsgSender.lua` |
| `changeCommodityList` | 1 | `Common/base/util/MerchantUtil.lua` |
| `getDBData` | 1 | `Common/base/DBManager.lua` |
| `log` | 5 | `Common/base/PlayerManager.lua` |
| `platformOrderDispose` | 1 | `Common/base/util/EngineUtil.lua` |
| `resetPos` | 1 | `Common/base/data/BasePlayer.lua` |
| `resetPosWithYaw` | 1 | `Common/base/data/BasePlayer.lua` |
| `sendAppExpResult` | 1 | `Common/base/exp/UserExpManager.lua` |
| `sendEnterOtherGame` | 1 | `Common/base/util/EngineUtil.lua` |
| `sendGameStatus` | 1 | `Common/base/listener/BaseListener.lua` |
| `sendGameover` | 1 | `Common/base/listener/BaseListener.lua` |
| `sendGotoOtherGame` | 1 | `Common/base/util/EngineUtil.lua` |
| `sendKillMsg` | 1 | `Common/base/util/MsgSender.lua` |
| `sendRankData` | 1 | `Common/base/rank/RankManager.lua` |
| `sendStartGame` | 1 | `Common/base/listener/BaseListener.lua` |
| `setDBData` | 1 | `Common/base/DBManager.lua` |
| `setDBLogData` | 1 | `Common/base/util/GameLog.lua` |
| `setServerGameType` | 1 | `Common/base/BaseMain.lua` |
| `stopServer` | 1 | `Common/base/ReportManager.lua` |

### 2.2 Engine events

Subscribed with `Event.registerCallBack (fn)`. `base/listener/BaseListener.lua` wires the core set at start-up.

| Event | Registered in |
| --- | --- |
| `GameInitEvent` | `Common/base/listener/BaseListener.lua` |
| `GetDataFromDBEvent` | `Common/base/listener/BaseListener.lua` |
| `HttpResponseEvent` | `Common/base/listener/HttpListener.lua` |
| `PlayerDynamicAttrEvent` | `Common/base/listener/BaseListener.lua` |
| `PlayerFirstSpawnEvent` | `Common/base/listener/BaseListener.lua` |
| `PlayerLoginEvent` | `Common/base/listener/BaseListener.lua` |
| `PlayerLogoutEvent` | `Common/base/listener/BaseListener.lua` |
| `PlayerReadyEvent` | `Common/base/listener/BaseListener.lua` |
| `PlayerRespawnEvent` | `Common/base/listener/BaseListener.lua` |

### 2.3 Engine data classes

Constructed from Lua with `Class.new (...)`; fields are populated by the host, so treat them as read-mostly.

| Class | Used in |
| --- | --- |
| `Vector3.new` | `Common/base/data/BasePlayer.lua`, `Common/base/util/VectorUtil.lua` |
| `Vector3i.new` | `Common/base/util/InventoryUtil.lua`, `Common/base/util/VectorUtil.lua` |

### 2.4 Other host objects

| Object | Methods observed |
| --- | --- |
| `CommonTask` | `start` |
| `HttpRequest` | `asyncGet`, `asyncPost` |
| `Item` | `getItemById` |
| `SecTimer` | `createTimer`, `startTimer`, `stopTimer` |
| `Vector3` | `new` |
| `Vector3i` | `new` |

---

## 3. Framework by Package

| Package | Functions | Description |
| --- | ---: | --- |
| [base](#pkg-base) | 45 | Entry points, bootstrapping and shared helpers. |
| [base/cache](#pkg-basecache) | 3 | In-memory caching layer. |
| [base/classes](#pkg-baseclasses) | 3 | Player class / role descriptors. |
| [base/data](#pkg-basedata) | 108 | World, engine and entity state accessors. |
| [base/exp](#pkg-baseexp) | 16 | Experience and progression rules. |
| [base/listener](#pkg-baselistener) | 12 | Event subscription and the base listener contract. |
| [base/messages](#pkg-basemessages) | 40 | Client protocol message construction. |
| [base/prop](#pkg-baseprop) | 5 | Player property and attribute access. |
| [base/rank](#pkg-baserank) | 6 | Ranking, leaderboard and score submission. |
| [base/season](#pkg-baseseason) | 2 | Seasonal content and reset logic. |
| [base/task](#pkg-basetask) | 11 | Tick and scheduled task framework. |
| [base/util](#pkg-baseutil) | 103 | General utilities (db, http, json, uuid, dates, logging). |
| [base/web](#pkg-baseweb) | 48 | HTTP / WebService calls to the game server. |

### base

_Entry points, bootstrapping and shared helpers._

**45 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `BaseMain:setGameType(GameType)` | method | `Common/base/BaseMain.lua:22` |  |
| `BaseMain:getGameType()` | method | `Common/base/BaseMain.lua:29` |  |
| `DBManager:addCache(key, data)` | method | `Common/base/DBManager.lua:9` |  |
| `DBManager:getCache(key)` | method | `Common/base/DBManager.lua:13` |  |
| `DBManager:removeCache(userId)` | method | `Common/base/DBManager.lua:17` |  |
| `DBManager:initPlayerData(userId, subKey, data)` | method | `Common/base/DBManager.lua:26` |  |
| `DBManager:savePlayerData(userId, subKey, data, immediate)` | method | `Common/base/DBManager.lua:31` |  |
| `DBManager:getPlayerData(userId, subKey)` | method | `Common/base/DBManager.lua:41` |  |
| `PlayerManager:setMaxPlayer(maxPlayer)` | method | `Common/base/PlayerManager.lua:15` |  |
| `PlayerManager:getMaxPlayer()` | method | `Common/base/PlayerManager.lua:19` |  |
| `PlayerManager:isPlayerFull()` | method | `Common/base/PlayerManager.lua:23` |  |
| `PlayerManager:isPlayerEmpty()` | method | `Common/base/PlayerManager.lua:27` |  |
| `PlayerManager:isPlayerEnough(players)` | method | `Common/base/PlayerManager.lua:31` |  |
| `PlayerManager:isPlayerExists()` | method | `Common/base/PlayerManager.lua:35` |  |
| `PlayerManager:addPlayer(player)` | method | `Common/base/PlayerManager.lua:39` |  |
| `PlayerManager:subPlayer(player)` | method | `Common/base/PlayerManager.lua:46` |  |
| `PlayerManager:getPlayerCount()` | method | `Common/base/PlayerManager.lua:64` |  |
| `PlayerManager:getPlayerByRakssid(rakssId)` | method | `Common/base/PlayerManager.lua:68` |  |
| `PlayerManager:getPlayerByUserId(userId)` | method | `Common/base/PlayerManager.lua:77` |  |
| `PlayerManager:getPlayerByEntityId(entityId)` | method | `Common/base/PlayerManager.lua:86` |  |
| `PlayerManager:getPlayerByPlayerMP(playerMP)` | method | `Common/base/PlayerManager.lua:95` |  |
| `PlayerManager:getPlayers()` | method | `Common/base/PlayerManager.lua:102` |  |
| `PlayerManager:copyPlayers()` | method | `Common/base/PlayerManager.lua:106` |  |
| `ReportManager:setGameType(GameType)` | method | `Common/base/ReportManager.lua:13` |  |
| `ReportManager:setRankType(RankType)` | method | `Common/base/ReportManager.lua:17` |  |
| `ReportManager:closeServer()` | method | `Common/base/ReportManager.lua:21` |  |
| `ReportManager:onPlayerLogout(player)` | method | `Common/base/ReportManager.lua:32` |  |
| `ReportManager:getUserReportCache(userId)` | method | `Common/base/ReportManager.lua:40` |  |
| `ReportManager:reportUserData(userId, kills, rank, isCount)` | method | `Common/base/ReportManager.lua:60` |  |
| `ReportManager:reportUserKill(userId, kill)` | method | `Common/base/ReportManager.lua:75` |  |
| `ReportManager:reportUserTime(userId, time)` | method | `Common/base/ReportManager.lua:83` |  |
| `ReportManager:reportUserRank(userId, rank)` | method | `Common/base/ReportManager.lua:91` |  |
| `ReportManager:reportUserCount(userId)` | method | `Common/base/ReportManager.lua:99` |  |
| `ReportManager:reportUserIntegral(userId, integral)` | method | `Common/base/ReportManager.lua:107` |  |
| `ReportManager:tryReportGameData()` | method | `Common/base/ReportManager.lua:116` |  |
| `ReportManager:tryCloseServer()` | method | `Common/base/ReportManager.lua:127` |  |
| `RewardManager:setGameType(GameType)` | method | `Common/base/RewardManager.lua:14` |  |
| `RewardManager:startGame()` | method | `Common/base/RewardManager.lua:18` |  |
| `RewardManager:getStartPlayerCount()` | method | `Common/base/RewardManager.lua:25` |  |
| `RewardManager:getStartGameTime()` | method | `Common/base/RewardManager.lua:29` |  |
| `RewardManager:isUserRewardFinish(userId)` | method | `Common/base/RewardManager.lua:33` |  |
| `RewardManager:addRewardQueue(userId, rank)` | method | `Common/base/RewardManager.lua:37` |  |
| `RewardManager:getQueueReward(func, ...)` | method | `Common/base/RewardManager.lua:49` |  |
| `RewardManager:getUserReward(userId, rank, func)` | method | `Common/base/RewardManager.lua:73` |  |
| `RewardManager:getUserGoldReward(userId, gold, func)` | method | `Common/base/RewardManager.lua:91` |  |

### base/cache

_In-memory caching layer._

**3 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `UserInfoCache:FilterExistedUserId(userIds)` | method | `Common/base/cache/UserInfoCache.lua:8` |  |
| `UserInfoCache:GetCacheByUserIds(userIds, func, key, ...)` | method | `Common/base/cache/UserInfoCache.lua:18` |  |
| `UserInfoCache:GetCache(userId)` | method | `Common/base/cache/UserInfoCache.lua:38` |  |

### base/classes

_Player class / role descriptors._

**3 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `PlayerClassInfo:addClasses(platformId, classInfo)` | method | `Common/base/classes/PlayerClassInfo.lua:6` |  |
| `PlayerClassInfo:getClasses(platformId)` | method | `Common/base/classes/PlayerClassInfo.lua:10` |  |
| `PlayerClassInfo:removeClasses(platformId)` | method | `Common/base/classes/PlayerClassInfo.lua:14` |  |

### base/data

_World, engine and entity state accessors._

**108 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `CBasePlayer:initMemberVariables()` | method | `Common/base/data/BasePlayer.lua:13` |  |
| `CBasePlayer:getEntityPlayer()` | method | `Common/base/data/BasePlayer.lua:56` |  |
| `CBasePlayer:ctor(clientPeer)` | method | `Common/base/data/BasePlayer.lua:60` |  |
| `CBasePlayer:respawn(clientPeer)` | method | `Common/base/data/BasePlayer.lua:66` |  |
| `CBasePlayer:initStaticAttr(teamId)` | method | `Common/base/data/BasePlayer.lua:76` |  |
| `CBasePlayer:sendLoginResult(isSuccess, teamId, teamName, playerNum, maxPlayer)` | method | `Common/base/data/BasePlayer.lua:91` |  |
| `CBasePlayer:onPlayerLogout()` | method | `Common/base/data/BasePlayer.lua:97` |  |
| `CBasePlayer:getDisplayName()` | method | `Common/base/data/BasePlayer.lua:103` |  |
| `CBasePlayer:addNightVisionPotionEffect(seconds)` | method | `Common/base/data/BasePlayer.lua:107` |  |
| `CBasePlayer:useAppProps()` | method | `Common/base/data/BasePlayer.lua:113` |  |
| `CBasePlayer:onUseCustomProp(propId, itemId)` | method | `Common/base/data/BasePlayer.lua:117` |  |
| `CBasePlayer:addItem(id, num, damage, isReverse)` | method | `Common/base/data/BasePlayer.lua:121` |  |
| `CBasePlayer:addGunItem(id, num, damage, bullets, isReverse)` | method | `Common/base/data/BasePlayer.lua:127` |  |
| `CBasePlayer:addItemToEnderChest(slot, id, num, damage)` | method | `Common/base/data/BasePlayer.lua:133` |  |
| `CBasePlayer:addGunItemToEnderChest(slot, id, num, damage, bullets)` | method | `Common/base/data/BasePlayer.lua:139` |  |
| `CBasePlayer:addGunDamage(gunId, damage)` | method | `Common/base/data/BasePlayer.lua:145` |  |
| `CBasePlayer:addGunBulletNum(gunId, bulletNum)` | method | `Common/base/data/BasePlayer.lua:151` |  |
| `CBasePlayer:subGunRecoil(gunId, recoil)` | method | `Common/base/data/BasePlayer.lua:157` |  |
| `CBasePlayer:removeItem(id, num)` | method | `Common/base/data/BasePlayer.lua:163` |  |
| `CBasePlayer:equipArmor(id, damage)` | method | `Common/base/data/BasePlayer.lua:170` |  |
| `CBasePlayer:replaceItem(id, num, damage, stackIndex, clipBullet)` | method | `Common/base/data/BasePlayer.lua:176` |  |
| `CBasePlayer:addEchantmentItem(id, num, damage, enchantmentId, enchantmentLevel)` | method | `Common/base/data/BasePlayer.lua:182` |  |
| `CBasePlayer:addEchantmentsItem(id, num, damage, enchantments)` | method | `Common/base/data/BasePlayer.lua:191` |  |
| `CBasePlayer:getHealth()` | method | `Common/base/data/BasePlayer.lua:197` |  |
| `CBasePlayer:setHealth(health)` | method | `Common/base/data/BasePlayer.lua:204` |  |
| `CBasePlayer:addHealth(hp)` | method | `Common/base/data/BasePlayer.lua:213` |  |
| `CBasePlayer:subHealth(hp)` | method | `Common/base/data/BasePlayer.lua:219` |  |
| `CBasePlayer:updateExp(level, exp, maxExp)` | method | `Common/base/data/BasePlayer.lua:225` |  |
| `CBasePlayer:getPosition()` | method | `Common/base/data/BasePlayer.lua:231` |  |
| `CBasePlayer:getYaw()` | method | `Common/base/data/BasePlayer.lua:238` |  |
| `CBasePlayer:setPositionAndRotation(vec, yaw, pitch)` | method | `Common/base/data/BasePlayer.lua:245` |  |
| `CBasePlayer:getSex()` | method | `Common/base/data/BasePlayer.lua:251` |  |
| `CBasePlayer:isWatchMode()` | method | `Common/base/data/BasePlayer.lua:258` |  |
| `CBasePlayer:setAllowFlying(flying)` | method | `Common/base/data/BasePlayer.lua:265` |  |
| `CBasePlayer:setFoodLevel(value)` | method | `Common/base/data/BasePlayer.lua:271` |  |
| `CBasePlayer:getFoodLevel()` | method | `Common/base/data/BasePlayer.lua:280` |  |
| `CBasePlayer:setSpeedAddition(level)` | method | `Common/base/data/BasePlayer.lua:287` |  |
| `CBasePlayer:getItemNumById(id)` | method | `Common/base/data/BasePlayer.lua:293` |  |
| `CBasePlayer:clearInv()` | method | `Common/base/data/BasePlayer.lua:301` |  |
| `CBasePlayer:getInventory()` | method | `Common/base/data/BasePlayer.lua:308` |  |
| `CBasePlayer:changeMaxHealth(health)` | method | `Common/base/data/BasePlayer.lua:314` |  |
| `CBasePlayer:getMaxHealth()` | method | `Common/base/data/BasePlayer.lua:320` |  |
| `CBasePlayer:getHeldItemId()` | method | `Common/base/data/BasePlayer.lua:327` |  |
| `CBasePlayer:setCurrency(currency)` | method | `Common/base/data/BasePlayer.lua:334` |  |
| `CBasePlayer:addCurrency(currency)` | method | `Common/base/data/BasePlayer.lua:340` |  |
| `CBasePlayer:subCurrency(currency)` | method | `Common/base/data/BasePlayer.lua:346` |  |
| `CBasePlayer:setUnlockedCommodities(unlockedCommodities)` | method | `Common/base/data/BasePlayer.lua:352` |  |
| `CBasePlayer:getCurrency()` | method | `Common/base/data/BasePlayer.lua:358` |  |
| `CBasePlayer:addBackpackCapacity(capacity)` | method | `Common/base/data/BasePlayer.lua:365` |  |
| `CBasePlayer:subBackpackCapacity(capacity)` | method | `Common/base/data/BasePlayer.lua:371` |  |
| `CBasePlayer:resetBackpack(capacity, maxCapacity)` | method | `Common/base/data/BasePlayer.lua:377` |  |
| `CBasePlayer:setArmItem(itemId)` | method | `Common/base/data/BasePlayer.lua:383` |  |
| `CBasePlayer:fillInvetory(inventory)` | method | `Common/base/data/BasePlayer.lua:389` |  |
| `CBasePlayer:fillEnderInvetory(enderInv)` | method | `Common/base/data/BasePlayer.lua:412` |  |
| `CBasePlayer:getEnderInventory()` | method | `Common/base/data/BasePlayer.lua:431` |  |
| `CBasePlayer:addEffect(id, time, level)` | method | `Common/base/data/BasePlayer.lua:437` |  |
| `CBasePlayer:removeEffect(id)` | method | `Common/base/data/BasePlayer.lua:443` |  |
| `CBasePlayer:clearEffects()` | method | `Common/base/data/BasePlayer.lua:449` |  |
| `CBasePlayer:setRespawnPos(pos)` | method | `Common/base/data/BasePlayer.lua:455` |  |
| `CBasePlayer:teleportPos(pos)` | method | `Common/base/data/BasePlayer.lua:461` |  |
| `CBasePlayer:teleportPosWithYaw(pos, yaw)` | method | `Common/base/data/BasePlayer.lua:465` |  |
| `CBasePlayer:changeClothes(partName, actorId)` | method | `Common/base/data/BasePlayer.lua:469` |  |
| `CBasePlayer:resetClothes()` | method | `Common/base/data/BasePlayer.lua:475` |  |
| `CBasePlayer:playSkillEffect(name, duration, range, color)` | method | `Common/base/data/BasePlayer.lua:481` |  |
| `CBasePlayer:sendSkillEffect(effectId)` | method | `Common/base/data/BasePlayer.lua:487` |  |
| `CBasePlayer:getDefence()` | method | `Common/base/data/BasePlayer.lua:493` |  |
| `CBasePlayer:getAttack()` | method | `Common/base/data/BasePlayer.lua:497` |  |
| `CBasePlayer:buildShowName()` | method | `Common/base/data/BasePlayer.lua:501` |  |
| `CBasePlayer:initDynamicAttr()` | method | `Common/base/data/BasePlayer.lua:548` |  |
| `CBasePlayer:setShowName(name, targetId)` | method | `Common/base/data/BasePlayer.lua:564` |  |
| `CBasePlayer:setTeam(team)` | method | `Common/base/data/BasePlayer.lua:570` |  |
| `CBasePlayer:changeHeart(hp, maxHp)` | method | `Common/base/data/BasePlayer.lua:576` |  |
| `CBasePlayer:getEntityId()` | method | `Common/base/data/BasePlayer.lua:582` |  |
| `CBasePlayer:addVehicles(vehicles)` | method | `Common/base/data/BasePlayer.lua:589` |  |
| `CBasePlayer:leaveVehicle()` | method | `Common/base/data/BasePlayer.lua:599` |  |
| `CBasePlayer:consumeDiamonds(uniqueId, diamonds, remark, isConsume)` | method | `Common/base/data/BasePlayer.lua:605` |  |
| `CBasePlayer:isVisitor()` | method | `Common/base/data/BasePlayer.lua:615` |  |
| `CBasePlayer:setPersonalShopArea(startPos, endPos)` | method | `Common/base/data/BasePlayer.lua:622` |  |
| `CBasePlayer:getDiamond()` | method | `Common/base/data/BasePlayer.lua:628` |  |
| `CBasePlayer:getGold()` | method | `Common/base/data/BasePlayer.lua:635` |  |
| `CBasePlayer:setOnFrozen(frozenTime)` | method | `Common/base/data/BasePlayer.lua:642` |  |
| `CBasePlayer:setOnFire(fireTime)` | method | `Common/base/data/BasePlayer.lua:648` |  |
| `CBasePlayer:addCustomEffect(name, effectName, duration)` | method | `Common/base/data/BasePlayer.lua:654` |  |
| `CBasePlayer:getRanch()` | method | `Common/base/data/BasePlayer.lua:660` |  |
| `CBasePlayer:getBirdSimulator()` | method | `Common/base/data/BasePlayer.lua:666` |  |
| `CBasePlayer:setBagInfo(curCapacity, maxCapacity)` | method | `Common/base/data/BasePlayer.lua:672` |  |
| `CBasePlayer:setBirdConvert(isConvert)` | method | `Common/base/data/BasePlayer.lua:678` |  |
| `EngineWorld:setWorld(world)` | method | `Common/base/data/EngineWorld.lua:9` |  |
| `EngineWorld:getWorld()` | method | `Common/base/data/EngineWorld.lua:13` |  |
| `EngineWorld:stopWorldTime(time)` | method | `Common/base/data/EngineWorld.lua:17` |  |
| `EngineWorld:resetMap()` | method | `Common/base/data/EngineWorld.lua:22` |  |
| `EngineWorld:setBlockToAir(blockPos)` | method | `Common/base/data/EngineWorld.lua:26` |  |
| `EngineWorld:setBlock(blockPos, id, meta, flag, immediate)` | method | `Common/base/data/EngineWorld.lua:30` |  |
| `EngineWorld:getBlockId(blockPos)` | method | `Common/base/data/EngineWorld.lua:37` |  |
| `EngineWorld:getEntity(entityId)` | method | `Common/base/data/EngineWorld.lua:41` |  |
| `EngineWorld:removeEntity(entityId)` | method | `Common/base/data/EngineWorld.lua:45` |  |
| `EngineWorld:addEntityItem(itemId, count, meta, life, pos, motion, isCombine, isEverLife)` | method | `Common/base/data/EngineWorld.lua:49` |  |
| `EngineWorld:addRankNpc(position, yaw, name)` | method | `Common/base/data/EngineWorld.lua:56` |  |
| `EngineWorld:addMerchantNpc(position, yaw, name)` | method | `Common/base/data/EngineWorld.lua:60` |  |
| `EngineWorld:addVehicleNpc(position, type, yaw)` | method | `Common/base/data/EngineWorld.lua:64` |  |
| `EngineWorld:addCreatureNpc(position, id, yaw, actor, name, userId)` | method | `Common/base/data/EngineWorld.lua:68` |  |
| `EngineWorld:addActorNpc(position, yaw, actor, name, skill, effect, isCanObstruct, isCollided)` | method | `Common/base/data/EngineWorld.lua:80` |  |
| `EngineWorld:addSessionNpc(position, yaw, type, name, actor, body, bodyId, content, isPerson, isCanObstruct)` | method | `Common/base/data/EngineWorld.lua:99` |  |
| `EngineWorld:addSimpleEffect(name, position, yaw, duration, targetId, scale)` | method | `Common/base/data/EngineWorld.lua:110` |  |
| `EngineWorld:changePlayerActor(player, actor, bodyId)` | method | `Common/base/data/EngineWorld.lua:117` |  |
| `EngineWorld:restorePlayerActor(player)` | method | `Common/base/data/EngineWorld.lua:124` |  |
| `EngineWorld:addGoodsGroupToShop(type, icon, name)` | method | `Common/base/data/EngineWorld.lua:131` |  |
| `EngineWorld:addGoodsToShop(type, goods)` | method | `Common/base/data/EngineWorld.lua:135` |  |

### base/exp

_Experience and progression rules._

**16 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `ExpRule:initRole(data)` | method | `Common/base/exp/ExpRule.lua:22` |  |
| `ExpRule:disable()` | method | `Common/base/exp/ExpRule.lua:59` |  |
| `ExpRule:isEnable()` | method | `Common/base/exp/ExpRule.lua:63` |  |
| `ExpRule:isExpMaxToday(hasGet)` | method | `Common/base/exp/ExpRule.lua:67` |  |
| `ExpRule:getAddExp(time, win, camps)` | method | `Common/base/exp/ExpRule.lua:71` |  |
| `ExpRule:correctAddExp(hasGet, addExp)` | method | `Common/base/exp/ExpRule.lua:85` |  |
| `ExpRule:checkTodayExpFull(hasGet, addExp)` | method | `Common/base/exp/ExpRule.lua:109` |  |
| `ExpRule:checkLvUp(curLv, curExp, addExp)` | method | `Common/base/exp/ExpRule.lua:118` |  |
| `ExpRule:getAverageUpExp(curLv, toLv)` | method | `Common/base/exp/ExpRule.lua:139` |  |
| `UserExpManager:getUserExpCache(userId)` | method | `Common/base/exp/UserExpManager.lua:13` |  |
| `UserExpManager:addExpCache(userId, curLv, curExp, hasGet)` | method | `Common/base/exp/UserExpManager.lua:20` |  |
| `UserExpManager:removeExpCache(userId)` | method | `Common/base/exp/UserExpManager.lua:42` |  |
| `UserExpManager:addUserExp(userId, win, camps, global)` | method | `Common/base/exp/UserExpManager.lua:63` |  |
| `UserExpManager:sendExpResult(userId, status)` | method | `Common/base/exp/UserExpManager.lua:101` |  |
| `UserExpManager:tryCalculationExp()` | method | `Common/base/exp/UserExpManager.lua:115` |  |
| `UserExpManager:trySaveExpResult()` | method | `Common/base/exp/UserExpManager.lua:127` |  |

### base/listener

_Event subscription and the base listener contract._

**12 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `BaseListener.registerCallBack(event, func)` | namespaced function | `Common/base/listener/BaseListener.lua:13` |  |
| `BaseListener:init()` | method | `Common/base/listener/BaseListener.lua:17` |  |
| `BaseListener.onGameInit(GamePath, ver, serverworld, config, initPos)` | namespaced function | `Common/base/listener/BaseListener.lua:28` |  |
| `BaseListener.onPlayerDynamicAttr(platformId, class, team, region, pioneer, vip)` | namespaced function | `Common/base/listener/BaseListener.lua:42` |  |
| `BaseListener.onPlayerLogin(clientPeer)` | namespaced function | `Common/base/listener/BaseListener.lua:52` |  |
| `BaseListener.onPlayerLogout(userId)` | namespaced function | `Common/base/listener/BaseListener.lua:84` |  |
| `BaseListener.onPlayerReady(rakssid)` | namespaced function | `Common/base/listener/BaseListener.lua:103` |  |
| `BaseListener.onPlayerFirstSpawn(clientPeer)` | namespaced function | `Common/base/listener/BaseListener.lua:119` |  |
| `BaseListener.onPlayerRespawn(clientPeer)` | namespaced function | `Common/base/listener/BaseListener.lua:131` |  |
| `BaseListener.onGetDataFromDB(userId, subKey, data)` | namespaced function | `Common/base/listener/BaseListener.lua:147` |  |
| `HttpListener:init()` | method | `Common/base/listener/HttpListener.lua:9` |  |
| `HttpListener.onHttpResponse(result, marked)` | namespaced function | `Common/base/listener/HttpListener.lua:13` |  |

### base/messages

_Client protocol message construction._

**40 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `IMessages:msgWelcomePlayer(game)` | method | `Common/base/messages/IMessages.lua:16` |  |
| `IMessages:msgWaitPlayerEnough(time, unit, ifHalf)` | method | `Common/base/messages/IMessages.lua:20` |  |
| `IMessages:msgGameStartTimeHint(time, unit, ifHalf)` | method | `Common/base/messages/IMessages.lua:24` |  |
| `IMessages:msgGameEndTimeHint(time, unit, ifHalf)` | method | `Common/base/messages/IMessages.lua:28` |  |
| `IMessages:msgPvpEnableAfter(time, unit, ifHalf)` | method | `Common/base/messages/IMessages.lua:32` |  |
| `IMessages:msgFightGameStartTimeHint(time, unit, ifHalf)` | method | `Common/base/messages/IMessages.lua:36` |  |
| `IMessages:msgCloseServerTimeHint(time, unit, ifHalf)` | method | `Common/base/messages/IMessages.lua:40` |  |
| `IMessages:msgGameStart()` | method | `Common/base/messages/IMessages.lua:44` |  |
| `IMessages:msgReadyHint(time)` | method | `Common/base/messages/IMessages.lua:48` |  |
| `IMessages:msgGameOverByLittle()` | method | `Common/base/messages/IMessages.lua:52` |  |
| `IMessages:msgPlayerLeft(num)` | method | `Common/base/messages/IMessages.lua:56` |  |
| `IMessages:msgPlayerCount(num)` | method | `Common/base/messages/IMessages.lua:60` |  |
| `IMessages:msgPlayerQuit(name)` | method | `Common/base/messages/IMessages.lua:64` |  |
| `IMessages:msgPlayerKillPlayer(deceased, killer)` | method | `Common/base/messages/IMessages.lua:68` |  |
| `IMessages:msgFirstKill(name)` | method | `Common/base/messages/IMessages.lua:72` |  |
| `IMessages:msgDoubleKill(name)` | method | `Common/base/messages/IMessages.lua:76` |  |
| `IMessages:msgThirdKill(name)` | method | `Common/base/messages/IMessages.lua:80` |  |
| `IMessages:msgFiveKill(name)` | method | `Common/base/messages/IMessages.lua:84` |  |
| `IMessages:msgFightGameStart()` | method | `Common/base/messages/IMessages.lua:88` |  |
| `IMessages:msgWinnerInfo(name)` | method | `Common/base/messages/IMessages.lua:92` |  |
| `IMessages:msgTeleportToDeadPlace(second)` | method | `Common/base/messages/IMessages.lua:96` |  |
| `IMessages:msgGameOverWin()` | method | `Common/base/messages/IMessages.lua:100` |  |
| `IMessages:msgGameOverDeath()` | method | `Common/base/messages/IMessages.lua:104` |  |
| `IMessages:msgGameOver()` | method | `Common/base/messages/IMessages.lua:108` |  |
| `IMessages:msgGameOverByDBDataError()` | method | `Common/base/messages/IMessages.lua:111` |  |
| `IMessages:msgNoWinner(type)` | method | `Common/base/messages/IMessages.lua:115` |  |
| `IMessages:msgGameScore(score)` | method | `Common/base/messages/IMessages.lua:123` |  |
| `IMessages:msgWaitPlayer()` | method | `Common/base/messages/IMessages.lua:127` |  |
| `IMessages:msgCanStartGame()` | method | `Common/base/messages/IMessages.lua:131` |  |
| `IMessages:msgGameRunning()` | method | `Common/base/messages/IMessages.lua:135` |  |
| `IMessages:msgGamePlayerNum(curPlayers, startPlayers)` | method | `Common/base/messages/IMessages.lua:139` |  |
| `IMessages:msgGamePlayerIsEnough()` | method | `Common/base/messages/IMessages.lua:143` |  |
| `IMessages:msgGameHasStarting()` | method | `Common/base/messages/IMessages.lua:147` |  |
| `IMessages:msgDontBuyGoodsByNoStart()` | method | `Common/base/messages/IMessages.lua:151` |  |
| `IMessages:msgRespawn(name)` | method | `Common/base/messages/IMessages.lua:155` |  |
| `IMessages:msgGameOverHint(seconds)` | method | `Common/base/messages/IMessages.lua:159` |  |
| `IMessages:msgGameResetHint()` | method | `Common/base/messages/IMessages.lua:163` |  |
| `IMessages:longWaitingHint()` | method | `Common/base/messages/IMessages.lua:167` |  |
| `IMessages:msgTimeTick(time, unit, ifHalf)` | method | `Common/base/messages/IMessages.lua:172` | unit 0:minute 1:second |
| `TextFormat:getTipType(num)` | method | `Common/base/messages/TextFormat.lua:25` |  |

### base/prop

_Player property and attribute access._

**5 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `AppProps:init(props)` | method | `Common/base/prop/AppProps.lua:15` |  |
| `AppProps:initProps(props)` | method | `Common/base/prop/AppProps.lua:19` |  |
| `AppProps:hasProp(player, propId)` | method | `Common/base/prop/AppProps.lua:57` |  |
| `AppProps:useAppProps(player, isFirst)` | method | `Common/base/prop/AppProps.lua:66` |  |
| `AppProps:useAppProp(player, prop)` | method | `Common/base/prop/AppProps.lua:87` |  |

### base/rank

_Ranking, leaderboard and score submission._

**6 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `RankManager:removePlayerRank(rakssid)` | method | `Common/base/rank/RankManager.lua:9` |  |
| `RankManager:buildPlayerRank(rakssid, key)` | method | `Common/base/rank/RankManager.lua:19` |  |
| `RankManager:addPlayerDayRank(player, key, rank, score, func, ...)` | method | `Common/base/rank/RankManager.lua:43` |  |
| `RankManager:addPlayerWeekRank(player, key, rank, score, func, ...)` | method | `Common/base/rank/RankManager.lua:61` |  |
| `RankManager:sendPlayerRank(rakssid, key, data, entityId)` | method | `Common/base/rank/RankManager.lua:79` |  |
| `RankManager:clearRanks()` | method | `Common/base/rank/RankManager.lua:97` |  |

### base/season

_Seasonal content and reset logic._

**2 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `SeasonManager:addUserHonor(userId, honor, level)` | method | `Common/base/season/SeasonManager.lua:10` |  |
| `SeasonManager:trySaveHonorResult()` | method | `Common/base/season/SeasonManager.lua:18` |  |

### base/task

_Tick and scheduled task framework._

**11 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `ICommonTask:onTick(ticks)` | method | `Common/base/task/CommonTask.lua:12` |  |
| `ITask:ctor(status)` | method | `Common/base/task/ITask.lua:11` |  |
| `ITask:onCreate()` | method | `Common/base/task/ITask.lua:22` |  |
| `ITask:onTick(ticks)` | method | `Common/base/task/ITask.lua:26` |  |
| `ITask:start()` | method | `Common/base/task/ITask.lua:30` |  |
| `ITask:stop()` | method | `Common/base/task/ITask.lua:34` |  |
| `ITask:pureStop()` | method | `Common/base/task/ITask.lua:38` |  |
| `LuaTimer:onTick()` | method | `Common/base/task/LuaTimer.lua:18` |  |
| `LuaTimer:schedule(action, delay, period, ...)` | method | `Common/base/task/LuaTimer.lua:72` |  |
| `LuaTimer:scheduleTimer(action, period, times, ...)` | method | `Common/base/task/LuaTimer.lua:106` |  |
| `LuaTimer:cancel(key)` | method | `Common/base/task/LuaTimer.lua:125` |  |

### base/util

_General utilities (db, http, json, uuid, dates, logging)._

**103 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `CsvUtil.getRowContent(file, filePath)` | namespaced function | `Common/base/util/CsvUtil.lua:9` |  |
| `CsvUtil.loadCsvFile(filePath, tRow)` | namespaced function | `Common/base/util/CsvUtil.lua:58` |  |
| `DateUtil.getYearWeek(time)` | namespaced function | `Common/base/util/DateUtil.lua:9` |  |
| `DateUtil.getWeeksOfYear(time)` | namespaced function | `Common/base/util/DateUtil.lua:13` |  |
| `DateUtil.getYearDay(time)` | namespaced function | `Common/base/util/DateUtil.lua:29` |  |
| `DateUtil.getDate(time)` | namespaced function | `Common/base/util/DateUtil.lua:36` |  |
| `DateUtil.getDateString(time)` | namespaced function | `Common/base/util/DateUtil.lua:43` |  |
| `DateUtil.getWeekLastTime(time)` | namespaced function | `Common/base/util/DateUtil.lua:50` |  |
| `DateUtil.getDayLastTime(time)` | namespaced function | `Common/base/util/DateUtil.lua:61` |  |
| `DateUtil.getWeekSeconds()` | namespaced function | `Common/base/util/DateUtil.lua:69` |  |
| `EngineUtil.sendEnterOtherGame(rakssid, gameType, targetId, mapId)` | namespaced function | `Common/base/util/EngineUtil.lua:8` |  |
| `EngineUtil.sendGotoOtherGame(rakssid, gameType, targetId, mapId)` | namespaced function | `Common/base/util/EngineUtil.lua:13` |  |
| `EngineUtil.disposePlatformOrder(userId, orderId, isConsume)` | namespaced function | `Common/base/util/EngineUtil.lua:18` |  |
| `GameLog.log(userId, type, action, data, immediate)` | namespaced function | `Common/base/util/GameLog.lua:8` |  |
| `InventoryUtil:fillItem(inv, config)` | method | `Common/base/util/InventoryUtil.lua:7` |  |
| `InventoryUtil:fillItemByType(inv, slotBegin, config, rangeLow, rangeUpper)` | method | `Common/base/util/InventoryUtil.lua:22` |  |
| `InventoryUtil:randomItem(subconfig, numMin, numMax)` | method | `Common/base/util/InventoryUtil.lua:37` |  |
| `InventoryUtil:randomItemNum(item)` | method | `Common/base/util/InventoryUtil.lua:97` |  |
| `InventoryUtil:prepareChest(config)` | method | `Common/base/util/InventoryUtil.lua:109` |  |
| `InventoryUtil:getPlayerInventoryInfo(player)` | method | `Common/base/util/InventoryUtil.lua:119` |  |
| `InventoryUtil:getPlayerEnderInventoryInfo(player)` | method | `Common/base/util/InventoryUtil.lua:152` |  |
| `MerchantUtil:addCommodity(index, type, goods)` | method | `Common/base/util/MerchantUtil.lua:7` |  |
| `MerchantUtil:changeCommodityList(entityId, rakssId, index)` | method | `Common/base/util/MerchantUtil.lua:11` |  |
| `MsgSender.sendMsg(msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:7` |  |
| `MsgSender.sendMsgToTarget(rakssid, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:11` |  |
| `MsgSender.sendTopTips(time, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:24` |  |
| `MsgSender.sendTopTipsToTarget(rakssid, time, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:28` |  |
| `MsgSender.sendBottomTips(time, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:41` |  |
| `MsgSender.sendBottomTipsToTarget(rakssid, time, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:45` |  |
| `MsgSender.sendCenterTips(time, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:58` |  |
| `MsgSender.sendCenterTipsToTarget(rakssid, time, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:62` |  |
| `MsgSender.sendOtherTips(time, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:75` |  |
| `MsgSender.sendOtherTipsToTarget(rakssid, time, msg, args)` | namespaced function | `Common/base/util/MsgSender.lua:79` |  |
| `MsgSender.sendKillMsg(killer, dead, armsId, kills, head, color)` | namespaced function | `Common/base/util/MsgSender.lua:92` |  |
| `MsgSender.sendKillMsgToTarget(rakssid, killer, dead, armsId, kills, head, color)` | namespaced function | `Common/base/util/MsgSender.lua:96` |  |
| `NumberUtil.getIntPart(num)` | namespaced function | `Common/base/util/NumberUtil.lua:7` |  |
| `StringUtil.split(str, reps, keep_blank)` | namespaced function | `Common/base/util/StringUtil.lua:7` |  |
| `v2s(value, sp, restLevel)` | file-local function | `Common/base/util/StringUtil.lua:32` |  |
| `StringUtil.v2s(value, maxLevel)` | namespaced function | `Common/base/util/StringUtil.lua:63` |  |
| `VectorUtil.newVector3i(x, y, z)` | namespaced function | `Common/base/util/VectorUtil.lua:8` |  |
| `VectorUtil.newVector3(x, y, z)` | namespaced function | `Common/base/util/VectorUtil.lua:16` |  |
| `VectorUtil.toVector3(vec3i)` | namespaced function | `Common/base/util/VectorUtil.lua:24` |  |
| `VectorUtil.toVector3i(vec3)` | namespaced function | `Common/base/util/VectorUtil.lua:32` |  |
| `VectorUtil.hashVector3(vec3)` | namespaced function | `Common/base/util/VectorUtil.lua:40` |  |
| `VectorUtil.toBlockVector3(x, y, z)` | namespaced function | `Common/base/util/VectorUtil.lua:44` |  |
| `VectorUtil.equal(vec3_1, vec3_2)` | namespaced function | `Common/base/util/VectorUtil.lua:57` |  |
| `WalletUtils:addCoinMappings(coinMapping)` | method | `Common/base/util/WalletUtils.lua:8` |  |
| `clone(object)` | global function | `Common/base/util/class.lua:3` |  |
| `_copy(object)` | file-local function | `Common/base/util/class.lua:5` |  |
| `class(classname, super)` | global function | `Common/base/util/class.lua:21` |  |
| `cls.ctor()` | assigned function | `Common/base/util/class.lua:43` |  |
| `cls.new(...)` | namespaced function | `Common/base/util/class.lua:47` |  |
| `cls.new(...)` | namespaced function | `Common/base/util/class.lua:69` |  |
| `escape_char(c)` | file-local function | `Common/base/util/json.lua:34` |  |
| `encode_nil(val)` | file-local function | `Common/base/util/json.lua:39` |  |
| `encode_table(val, stack)` | file-local function | `Common/base/util/json.lua:44` |  |
| `encode_string(val)` | file-local function | `Common/base/util/json.lua:86` |  |
| `encode_number(val)` | file-local function | `Common/base/util/json.lua:91` |  |
| `encode(val, stack)` | assigned function | `Common/base/util/json.lua:109` |  |
| `json.encode(val)` | namespaced function | `Common/base/util/json.lua:119` |  |
| `create_set(...)` | file-local function | `Common/base/util/json.lua:130` |  |
| `next_char(str, idx, set, negate)` | file-local function | `Common/base/util/json.lua:150` |  |
| `decode_error(str, idx, msg)` | file-local function | `Common/base/util/json.lua:160` |  |
| `codepoint_to_utf8(n)` | file-local function | `Common/base/util/json.lua:174` |  |
| `parse_unicode_escape(s)` | file-local function | `Common/base/util/json.lua:191` |  |
| `parse_string(str, i)` | file-local function | `Common/base/util/json.lua:203` |  |
| `parse_number(str, i)` | file-local function | `Common/base/util/json.lua:256` |  |
| `parse_literal(str, i)` | file-local function | `Common/base/util/json.lua:267` |  |
| `parse_array(str, i)` | file-local function | `Common/base/util/json.lua:277` |  |
| `parse_object(str, i)` | file-local function | `Common/base/util/json.lua:304` |  |
| `parse(str, idx)` | assigned function | `Common/base/util/json.lua:362` |  |
| `json.decode(str)` | namespaced function | `Common/base/util/json.lua:372` |  |
| `select(list, pred)` | file-local function | `Common/base/util/tinyyaml.lua:19` | utils |
| `startswith(haystack, needle)` | file-local function | `Common/base/util/tinyyaml.lua:33` |  |
| `ltrim(str)` | file-local function | `Common/base/util/tinyyaml.lua:37` |  |
| `rtrim(str)` | file-local function | `Common/base/util/tinyyaml.lua:41` |  |
| `class.__meta.__call(cls, ...)` | namespaced function | `Common/base/util/tinyyaml.lua:49` |  |
| `class.def(base, type, cls)` | namespaced function | `Common/base/util/tinyyaml.lua:57` |  |
| `Null.__tostring()` | namespaced function | `Common/base/util/tinyyaml.lua:82` |  |
| `Null.isnull(v)` | namespaced function | `Common/base/util/tinyyaml.lua:85` |  |
| `types.timestamp:__init(y, m, d, h, i, s, f, z)` | method | `Common/base/util/tinyyaml.lua:96` |  |
| `types.timestamp:__tostring()` | method | `Common/base/util/tinyyaml.lua:113` |  |
| `types.timestamp:gettz()` | method | `Common/base/util/tinyyaml.lua:121` |  |
| `countindent(line)` | file-local function | `Common/base/util/tinyyaml.lua:136` |  |
| `parsestring(line, stopper)` | file-local function | `Common/base/util/tinyyaml.lua:144` |  |
| `isemptyline(line)` | file-local function | `Common/base/util/tinyyaml.lua:211` |  |
| `startswithline(line, needle)` | file-local function | `Common/base/util/tinyyaml.lua:215` |  |
| `parseflowstyle(line, lines)` | file-local function | `Common/base/util/tinyyaml.lua:219` |  |
| `parseblockstylestring(line, lines, indent)` | file-local function | `Common/base/util/tinyyaml.lua:290` |  |
| `parsetimestamp(line)` | file-local function | `Common/base/util/tinyyaml.lua:365` |  |
| `parsescalar(line, lines, indent)` | file-local function | `Common/base/util/tinyyaml.lua:404` |  |
| `parseseq(line, lines, indent)` | file-local function | `Common/base/util/tinyyaml.lua:468` |  |
| `parseset(line, lines, indent)` | file-local function | `Common/base/util/tinyyaml.lua:536` |  |
| `parsemap(line, lines, indent)` | global function | `Common/base/util/tinyyaml.lua:599` |  |
| `parsedocuments(lines)` | file-local function | `Common/base/util/tinyyaml.lua:689` | : (list<str>)->dict |
| `parse(source)` | file-local function | `Common/base/util/tinyyaml.lua:751` |  |
| `BITWISE(x, y, matrix)` | file-local function | `Common/base/util/uuid.lua:52` | performs the bitwise operation specified by truth matrix on two numbers. |
| `INT2HEX(x)` | file-local function | `Common/base/util/uuid.lua:64` |  |
| `M.new(hwaddr)` | namespaced function | `Common/base/util/uuid.lua:97` | Creates a new uuid. Either provide a unique hex string, or make sure the random seed is properly set. The module table itself is a shortcut to this function, so 'my_uuid = uuid.new()' equals 'my_uuid = uuid()'.  For proper use there are 3 options;  1. first require 'luasocket', then call 'uuid.seed()', and request a uuid using no parameter, eg. 'my_uuid = uuid()' 2. use 'uuid' without 'luasocket', set a random seed using 'uuid.randomseed(some_good_seed)', and request a uuid using no parameter, eg. 'my_uuid = uuid()' 3. use 'uuid' without 'luasocket', and request a uuid using an unique hex string, eg. 'my_uuid = uuid(my_networkcard_macaddress)'  @return a properly formatted uuid string @param hwaddr (optional) string containing a unique hex value (e.g.: '00:0c:29:69:41:c6'), to be used to compensate for the lesser 'math_random()' function. Use a mac address for solid results. If omitted, a fully randomized uuid will be generated, but then you must ensure that the random seed is set properly! @usage local uuid = require("uuid") print("here's a new uuid: ",uuid()) |
| `M.randomseed(seed)` | namespaced function | `Common/base/util/uuid.lua:170` | Improved randomseed function. Lua 5.1 and 5.2 both truncate the seed given if it exceeds the integer range. If this happens, the seed will be 0 or 1 and all randomness will be gone (each application run will generate the same sequence of random numbers in that case). This improved version drops the most significant bits in those cases to get the seed within the proper range again. @param seed the random seed to set (integer from 0 - 2^32, negative values will be made positive) @return the (potentially modified) seed used @usage local socket = require("socket")  -- gettime() has higher precision than os.time() local uuid = require("uuid") -- see also example at uuid.seed() uuid.randomseed(socket.gettime()*10000) print("here's a new uuid: ",uuid()) |
| `M.seed()` | namespaced function | `Common/base/util/uuid.lua:199` | Seeds the random generator. It does so in 2 possible ways;  1. use 'os.time()': this only offers resolution to one second (used when LuaSocket hasn't been loaded yet 2. use luasocket 'gettime()' function, but it only does so when LuaSocket has been required already. @usage local socket = require("socket")  -- gettime() has higher precision than os.time() -- LuaSocket loaded, so below line does the same as the example from randomseed() uuid.seed() print("here's a new uuid: ",uuid()) |
| `var_dump(_t)` | global function | `Common/base/util/vardump.lua:4` |  |
| `doT2S(_i, _v)` | global function | `Common/base/util/vardump.lua:6` |  |

### base/web

_HTTP / WebService calls to the game server._

**48 function(s)**

| Signature | Kind | Location | Notes |
| --- | --- | --- | --- |
| `WebResponse:registerCallBack(func, marked, ...)` | method | `Common/base/web/WebResponse.lua:11` |  |
| `WebResponse:unregisterCallBack(marked)` | method | `Common/base/web/WebResponse.lua:18` |  |
| `WebResponse:onCallFunc(data, marked)` | method | `Common/base/web/WebResponse.lua:24` |  |
| `WebResponse:onGetMcpeonlineUserAttr(result, marked)` | method | `Common/base/web/WebResponse.lua:32` |  |
| `WebResponse:onGetMcpeonlineReward(result, marked)` | method | `Common/base/web/WebResponse.lua:57` |  |
| `WebResponse:onGetMcpeonlineReport(result, marked)` | method | `Common/base/web/WebResponse.lua:74` |  |
| `WebResponse:onGetBlockymodsUserAttr(result, marked)` | method | `Common/base/web/WebResponse.lua:78` |  |
| `WebResponse:onGetBlockymodsReward(result, marked)` | method | `Common/base/web/WebResponse.lua:95` |  |
| `WebResponse:onGetBlockymodsGoldReward(result, marked)` | method | `Common/base/web/WebResponse.lua:110` |  |
| `WebResponse:onGetBlockymodsExpRole(result, marked)` | method | `Common/base/web/WebResponse.lua:125` |  |
| `WebResponse:onGetBlockymodsExp(result, marked)` | method | `Common/base/web/WebResponse.lua:136` |  |
| `WebResponse:onSaveBlockymodsExp(result, marked)` | method | `Common/base/web/WebResponse.lua:148` |  |
| `WebResponse:onSaveBlockymodsHonor(result, marked)` | method | `Common/base/web/WebResponse.lua:152` |  |
| `WebResponse:onGetBlockymodsUserSeasonInfo(result, marked)` | method | `Common/base/web/WebResponse.lua:156` |  |
| `WebResponse:onUpdateBlockymodsUserSeasonReward(result, marked)` | method | `Common/base/web/WebResponse.lua:162` |  |
| `WebResponse:onReportBlockymodsData(result, marked)` | method | `Common/base/web/WebResponse.lua:176` |  |
| `WebResponse:onGetBlockymodsUserInfos(result, marked)` | method | `Common/base/web/WebResponse.lua:180` |  |
| `WebResponse:onGetBlockymodsRewardList(result, marked)` | method | `Common/base/web/WebResponse.lua:186` |  |
| `WebResponse:onReportBlockymodsList(result, marked)` | method | `Common/base/web/WebResponse.lua:203` |  |
| `WebResponse:onCheckMails(result, marked)` | method | `Common/base/web/WebResponse.lua:207` |  |
| `WebResponse:onSendMails(result, marked)` | method | `Common/base/web/WebResponse.lua:234` |  |
| `WebResponse:onUpdateMails(result, marked)` | method | `Common/base/web/WebResponse.lua:244` |  |
| `WebResponse:formatResult(result)` | method | `Common/base/web/WebResponse.lua:254` |  |
| `WebResponse:formatResultCode(result)` | method | `Common/base/web/WebResponse.lua:266` |  |
| `WebResponse:formatData(result)` | method | `Common/base/web/WebResponse.lua:278` |  |
| `WebService:init(version, config)` | method | `Common/base/web/WebService.lua:35` |  |
| `WebService:initGameId()` | method | `Common/base/web/WebService.lua:44` |  |
| `WebService:GetUserAttr(userId, gameType)` | method | `Common/base/web/WebService.lua:50` |  |
| `WebService:GetPlayerReward(userId, gametype, kills, iswin, gameId)` | method | `Common/base/web/WebService.lua:68` |  |
| `WebService:PostGameResult(gameType, resultJson)` | method | `Common/base/web/WebService.lua:97` |  |
| `WebService:UseProps(userId, PropId)` | method | `Common/base/web/WebService.lua:109` |  |
| `WebService:GetBlockymodsUserAttr(userId, gameType)` | method | `Common/base/web/WebService.lua:113` |  |
| `WebService:GetBlockymodsReward(userId, gameType, data)` | method | `Common/base/web/WebService.lua:134` |  |
| `WebService:GetBlockymodsGoldReward(userId, gameType, golds)` | method | `Common/base/web/WebService.lua:157` |  |
| `WebService:GetBlockymodsUserInfos(userIds, key)` | method | `Common/base/web/WebService.lua:182` |  |
| `WebService:GetBlockymodsExpRule()` | method | `Common/base/web/WebService.lua:210` |  |
| `WebService:GetBlockymodsUserExp(userId)` | method | `Common/base/web/WebService.lua:224` |  |
| `WebService:SaveBlockymodsUsersExp(data)` | method | `Common/base/web/WebService.lua:240` |  |
| `WebService:SaveBlockymodsUsersHonor(data)` | method | `Common/base/web/WebService.lua:248` |  |
| `WebService:GetBlockymodsUserSeasonInfo(userId, isLast)` | method | `Common/base/web/WebService.lua:256` |  |
| `WebService:UpdateBlockymodsUserSeasonReward(userId)` | method | `Common/base/web/WebService.lua:283` |  |
| `WebService:ReportBlockymodsData(userId, gameType, time, kills, rank, isCount, integral, type)` | method | `Common/base/web/WebService.lua:300` |  |
| `WebService:GetBlockymodsRewardList(data)` | method | `Common/base/web/WebService.lua:327` |  |
| `WebService:ReportBlockymodsList(data)` | method | `Common/base/web/WebService.lua:339` |  |
| `WebService:checkMails(userId, gameType)` | method | `Common/base/web/WebService.lua:351` |  |
| `WebService:sendMails(fromUser, toUser, mailType, title, content, gameType)` | method | `Common/base/web/WebService.lua:371` |  |
| `WebService:updateMails(userId, id, status)` | method | `Common/base/web/WebService.lua:392` |  |
| `WebService:sendMsg(userId, targetIds, content, msgType, scope, gameType)` | method | `Common/base/web/WebService.lua:410` |  |

---

## 4. Alphabetical Signature Index

Every distinct signature in the common layer, sorted by name. Use this to answer "does this exist?" without scanning the package listings.

**401 unique signatures** across 402 definitions.

| Signature | Kind | Count | Example location |
| --- | --- | ---: | --- |
| `AppProps:hasProp(player, propId)` | method | 1 | `Common/base/prop/AppProps.lua:57` |
| `AppProps:init(props)` | method | 1 | `Common/base/prop/AppProps.lua:15` |
| `AppProps:initProps(props)` | method | 1 | `Common/base/prop/AppProps.lua:19` |
| `AppProps:useAppProp(player, prop)` | method | 1 | `Common/base/prop/AppProps.lua:87` |
| `AppProps:useAppProps(player, isFirst)` | method | 1 | `Common/base/prop/AppProps.lua:66` |
| `BITWISE(x, y, matrix)` | file-local function | 1 | `Common/base/util/uuid.lua:52` |
| `BaseListener.onGameInit(GamePath, ver, serverworld, config, initPos)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:28` |
| `BaseListener.onGetDataFromDB(userId, subKey, data)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:147` |
| `BaseListener.onPlayerDynamicAttr(platformId, class, team, region, pioneer, vip)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:42` |
| `BaseListener.onPlayerFirstSpawn(clientPeer)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:119` |
| `BaseListener.onPlayerLogin(clientPeer)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:52` |
| `BaseListener.onPlayerLogout(userId)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:84` |
| `BaseListener.onPlayerReady(rakssid)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:103` |
| `BaseListener.onPlayerRespawn(clientPeer)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:131` |
| `BaseListener.registerCallBack(event, func)` | namespaced function | 1 | `Common/base/listener/BaseListener.lua:13` |
| `BaseListener:init()` | method | 1 | `Common/base/listener/BaseListener.lua:17` |
| `BaseMain:getGameType()` | method | 1 | `Common/base/BaseMain.lua:29` |
| `BaseMain:setGameType(GameType)` | method | 1 | `Common/base/BaseMain.lua:22` |
| `CBasePlayer:addBackpackCapacity(capacity)` | method | 1 | `Common/base/data/BasePlayer.lua:365` |
| `CBasePlayer:addCurrency(currency)` | method | 1 | `Common/base/data/BasePlayer.lua:340` |
| `CBasePlayer:addCustomEffect(name, effectName, duration)` | method | 1 | `Common/base/data/BasePlayer.lua:654` |
| `CBasePlayer:addEchantmentItem(id, num, damage, enchantmentId, enchantmentLevel)` | method | 1 | `Common/base/data/BasePlayer.lua:182` |
| `CBasePlayer:addEchantmentsItem(id, num, damage, enchantments)` | method | 1 | `Common/base/data/BasePlayer.lua:191` |
| `CBasePlayer:addEffect(id, time, level)` | method | 1 | `Common/base/data/BasePlayer.lua:437` |
| `CBasePlayer:addGunBulletNum(gunId, bulletNum)` | method | 1 | `Common/base/data/BasePlayer.lua:151` |
| `CBasePlayer:addGunDamage(gunId, damage)` | method | 1 | `Common/base/data/BasePlayer.lua:145` |
| `CBasePlayer:addGunItem(id, num, damage, bullets, isReverse)` | method | 1 | `Common/base/data/BasePlayer.lua:127` |
| `CBasePlayer:addGunItemToEnderChest(slot, id, num, damage, bullets)` | method | 1 | `Common/base/data/BasePlayer.lua:139` |
| `CBasePlayer:addHealth(hp)` | method | 1 | `Common/base/data/BasePlayer.lua:213` |
| `CBasePlayer:addItem(id, num, damage, isReverse)` | method | 1 | `Common/base/data/BasePlayer.lua:121` |
| `CBasePlayer:addItemToEnderChest(slot, id, num, damage)` | method | 1 | `Common/base/data/BasePlayer.lua:133` |
| `CBasePlayer:addNightVisionPotionEffect(seconds)` | method | 1 | `Common/base/data/BasePlayer.lua:107` |
| `CBasePlayer:addVehicles(vehicles)` | method | 1 | `Common/base/data/BasePlayer.lua:589` |
| `CBasePlayer:buildShowName()` | method | 1 | `Common/base/data/BasePlayer.lua:501` |
| `CBasePlayer:changeClothes(partName, actorId)` | method | 1 | `Common/base/data/BasePlayer.lua:469` |
| `CBasePlayer:changeHeart(hp, maxHp)` | method | 1 | `Common/base/data/BasePlayer.lua:576` |
| `CBasePlayer:changeMaxHealth(health)` | method | 1 | `Common/base/data/BasePlayer.lua:314` |
| `CBasePlayer:clearEffects()` | method | 1 | `Common/base/data/BasePlayer.lua:449` |
| `CBasePlayer:clearInv()` | method | 1 | `Common/base/data/BasePlayer.lua:301` |
| `CBasePlayer:consumeDiamonds(uniqueId, diamonds, remark, isConsume)` | method | 1 | `Common/base/data/BasePlayer.lua:605` |
| `CBasePlayer:ctor(clientPeer)` | method | 1 | `Common/base/data/BasePlayer.lua:60` |
| `CBasePlayer:equipArmor(id, damage)` | method | 1 | `Common/base/data/BasePlayer.lua:170` |
| `CBasePlayer:fillEnderInvetory(enderInv)` | method | 1 | `Common/base/data/BasePlayer.lua:412` |
| `CBasePlayer:fillInvetory(inventory)` | method | 1 | `Common/base/data/BasePlayer.lua:389` |
| `CBasePlayer:getAttack()` | method | 1 | `Common/base/data/BasePlayer.lua:497` |
| `CBasePlayer:getBirdSimulator()` | method | 1 | `Common/base/data/BasePlayer.lua:666` |
| `CBasePlayer:getCurrency()` | method | 1 | `Common/base/data/BasePlayer.lua:358` |
| `CBasePlayer:getDefence()` | method | 1 | `Common/base/data/BasePlayer.lua:493` |
| `CBasePlayer:getDiamond()` | method | 1 | `Common/base/data/BasePlayer.lua:628` |
| `CBasePlayer:getDisplayName()` | method | 1 | `Common/base/data/BasePlayer.lua:103` |
| `CBasePlayer:getEnderInventory()` | method | 1 | `Common/base/data/BasePlayer.lua:431` |
| `CBasePlayer:getEntityId()` | method | 1 | `Common/base/data/BasePlayer.lua:582` |
| `CBasePlayer:getEntityPlayer()` | method | 1 | `Common/base/data/BasePlayer.lua:56` |
| `CBasePlayer:getFoodLevel()` | method | 1 | `Common/base/data/BasePlayer.lua:280` |
| `CBasePlayer:getGold()` | method | 1 | `Common/base/data/BasePlayer.lua:635` |
| `CBasePlayer:getHealth()` | method | 1 | `Common/base/data/BasePlayer.lua:197` |
| `CBasePlayer:getHeldItemId()` | method | 1 | `Common/base/data/BasePlayer.lua:327` |
| `CBasePlayer:getInventory()` | method | 1 | `Common/base/data/BasePlayer.lua:308` |
| `CBasePlayer:getItemNumById(id)` | method | 1 | `Common/base/data/BasePlayer.lua:293` |
| `CBasePlayer:getMaxHealth()` | method | 1 | `Common/base/data/BasePlayer.lua:320` |
| `CBasePlayer:getPosition()` | method | 1 | `Common/base/data/BasePlayer.lua:231` |
| `CBasePlayer:getRanch()` | method | 1 | `Common/base/data/BasePlayer.lua:660` |
| `CBasePlayer:getSex()` | method | 1 | `Common/base/data/BasePlayer.lua:251` |
| `CBasePlayer:getYaw()` | method | 1 | `Common/base/data/BasePlayer.lua:238` |
| `CBasePlayer:initDynamicAttr()` | method | 1 | `Common/base/data/BasePlayer.lua:548` |
| `CBasePlayer:initMemberVariables()` | method | 1 | `Common/base/data/BasePlayer.lua:13` |
| `CBasePlayer:initStaticAttr(teamId)` | method | 1 | `Common/base/data/BasePlayer.lua:76` |
| `CBasePlayer:isVisitor()` | method | 1 | `Common/base/data/BasePlayer.lua:615` |
| `CBasePlayer:isWatchMode()` | method | 1 | `Common/base/data/BasePlayer.lua:258` |
| `CBasePlayer:leaveVehicle()` | method | 1 | `Common/base/data/BasePlayer.lua:599` |
| `CBasePlayer:onPlayerLogout()` | method | 1 | `Common/base/data/BasePlayer.lua:97` |
| `CBasePlayer:onUseCustomProp(propId, itemId)` | method | 1 | `Common/base/data/BasePlayer.lua:117` |
| `CBasePlayer:playSkillEffect(name, duration, range, color)` | method | 1 | `Common/base/data/BasePlayer.lua:481` |
| `CBasePlayer:removeEffect(id)` | method | 1 | `Common/base/data/BasePlayer.lua:443` |
| `CBasePlayer:removeItem(id, num)` | method | 1 | `Common/base/data/BasePlayer.lua:163` |
| `CBasePlayer:replaceItem(id, num, damage, stackIndex, clipBullet)` | method | 1 | `Common/base/data/BasePlayer.lua:176` |
| `CBasePlayer:resetBackpack(capacity, maxCapacity)` | method | 1 | `Common/base/data/BasePlayer.lua:377` |
| `CBasePlayer:resetClothes()` | method | 1 | `Common/base/data/BasePlayer.lua:475` |
| `CBasePlayer:respawn(clientPeer)` | method | 1 | `Common/base/data/BasePlayer.lua:66` |
| `CBasePlayer:sendLoginResult(isSuccess, teamId, teamName, playerNum, maxPlayer)` | method | 1 | `Common/base/data/BasePlayer.lua:91` |
| `CBasePlayer:sendSkillEffect(effectId)` | method | 1 | `Common/base/data/BasePlayer.lua:487` |
| `CBasePlayer:setAllowFlying(flying)` | method | 1 | `Common/base/data/BasePlayer.lua:265` |
| `CBasePlayer:setArmItem(itemId)` | method | 1 | `Common/base/data/BasePlayer.lua:383` |
| `CBasePlayer:setBagInfo(curCapacity, maxCapacity)` | method | 1 | `Common/base/data/BasePlayer.lua:672` |
| `CBasePlayer:setBirdConvert(isConvert)` | method | 1 | `Common/base/data/BasePlayer.lua:678` |
| `CBasePlayer:setCurrency(currency)` | method | 1 | `Common/base/data/BasePlayer.lua:334` |
| `CBasePlayer:setFoodLevel(value)` | method | 1 | `Common/base/data/BasePlayer.lua:271` |
| `CBasePlayer:setHealth(health)` | method | 1 | `Common/base/data/BasePlayer.lua:204` |
| `CBasePlayer:setOnFire(fireTime)` | method | 1 | `Common/base/data/BasePlayer.lua:648` |
| `CBasePlayer:setOnFrozen(frozenTime)` | method | 1 | `Common/base/data/BasePlayer.lua:642` |
| `CBasePlayer:setPersonalShopArea(startPos, endPos)` | method | 1 | `Common/base/data/BasePlayer.lua:622` |
| `CBasePlayer:setPositionAndRotation(vec, yaw, pitch)` | method | 1 | `Common/base/data/BasePlayer.lua:245` |
| `CBasePlayer:setRespawnPos(pos)` | method | 1 | `Common/base/data/BasePlayer.lua:455` |
| `CBasePlayer:setShowName(name, targetId)` | method | 1 | `Common/base/data/BasePlayer.lua:564` |
| `CBasePlayer:setSpeedAddition(level)` | method | 1 | `Common/base/data/BasePlayer.lua:287` |
| `CBasePlayer:setTeam(team)` | method | 1 | `Common/base/data/BasePlayer.lua:570` |
| `CBasePlayer:setUnlockedCommodities(unlockedCommodities)` | method | 1 | `Common/base/data/BasePlayer.lua:352` |
| `CBasePlayer:subBackpackCapacity(capacity)` | method | 1 | `Common/base/data/BasePlayer.lua:371` |
| `CBasePlayer:subCurrency(currency)` | method | 1 | `Common/base/data/BasePlayer.lua:346` |
| `CBasePlayer:subGunRecoil(gunId, recoil)` | method | 1 | `Common/base/data/BasePlayer.lua:157` |
| `CBasePlayer:subHealth(hp)` | method | 1 | `Common/base/data/BasePlayer.lua:219` |
| `CBasePlayer:teleportPos(pos)` | method | 1 | `Common/base/data/BasePlayer.lua:461` |
| `CBasePlayer:teleportPosWithYaw(pos, yaw)` | method | 1 | `Common/base/data/BasePlayer.lua:465` |
| `CBasePlayer:updateExp(level, exp, maxExp)` | method | 1 | `Common/base/data/BasePlayer.lua:225` |
| `CBasePlayer:useAppProps()` | method | 1 | `Common/base/data/BasePlayer.lua:113` |
| `CsvUtil.getRowContent(file, filePath)` | namespaced function | 1 | `Common/base/util/CsvUtil.lua:9` |
| `CsvUtil.loadCsvFile(filePath, tRow)` | namespaced function | 1 | `Common/base/util/CsvUtil.lua:58` |
| `DBManager:addCache(key, data)` | method | 1 | `Common/base/DBManager.lua:9` |
| `DBManager:getCache(key)` | method | 1 | `Common/base/DBManager.lua:13` |
| `DBManager:getPlayerData(userId, subKey)` | method | 1 | `Common/base/DBManager.lua:41` |
| `DBManager:initPlayerData(userId, subKey, data)` | method | 1 | `Common/base/DBManager.lua:26` |
| `DBManager:removeCache(userId)` | method | 1 | `Common/base/DBManager.lua:17` |
| `DBManager:savePlayerData(userId, subKey, data, immediate)` | method | 1 | `Common/base/DBManager.lua:31` |
| `DateUtil.getDate(time)` | namespaced function | 1 | `Common/base/util/DateUtil.lua:36` |
| `DateUtil.getDateString(time)` | namespaced function | 1 | `Common/base/util/DateUtil.lua:43` |
| `DateUtil.getDayLastTime(time)` | namespaced function | 1 | `Common/base/util/DateUtil.lua:61` |
| `DateUtil.getWeekLastTime(time)` | namespaced function | 1 | `Common/base/util/DateUtil.lua:50` |
| `DateUtil.getWeekSeconds()` | namespaced function | 1 | `Common/base/util/DateUtil.lua:69` |
| `DateUtil.getWeeksOfYear(time)` | namespaced function | 1 | `Common/base/util/DateUtil.lua:13` |
| `DateUtil.getYearDay(time)` | namespaced function | 1 | `Common/base/util/DateUtil.lua:29` |
| `DateUtil.getYearWeek(time)` | namespaced function | 1 | `Common/base/util/DateUtil.lua:9` |
| `EngineUtil.disposePlatformOrder(userId, orderId, isConsume)` | namespaced function | 1 | `Common/base/util/EngineUtil.lua:18` |
| `EngineUtil.sendEnterOtherGame(rakssid, gameType, targetId, mapId)` | namespaced function | 1 | `Common/base/util/EngineUtil.lua:8` |
| `EngineUtil.sendGotoOtherGame(rakssid, gameType, targetId, mapId)` | namespaced function | 1 | `Common/base/util/EngineUtil.lua:13` |
| `EngineWorld:addActorNpc(position, yaw, actor, name, skill, effect, isCanObstruct, isCollided)` | method | 1 | `Common/base/data/EngineWorld.lua:80` |
| `EngineWorld:addCreatureNpc(position, id, yaw, actor, name, userId)` | method | 1 | `Common/base/data/EngineWorld.lua:68` |
| `EngineWorld:addEntityItem(itemId, count, meta, life, pos, motion, isCombine, isEverLife)` | method | 1 | `Common/base/data/EngineWorld.lua:49` |
| `EngineWorld:addGoodsGroupToShop(type, icon, name)` | method | 1 | `Common/base/data/EngineWorld.lua:131` |
| `EngineWorld:addGoodsToShop(type, goods)` | method | 1 | `Common/base/data/EngineWorld.lua:135` |
| `EngineWorld:addMerchantNpc(position, yaw, name)` | method | 1 | `Common/base/data/EngineWorld.lua:60` |
| `EngineWorld:addRankNpc(position, yaw, name)` | method | 1 | `Common/base/data/EngineWorld.lua:56` |
| `EngineWorld:addSessionNpc(position, yaw, type, name, actor, body, bodyId, content, isPerson, isCanObstruct)` | method | 1 | `Common/base/data/EngineWorld.lua:99` |
| `EngineWorld:addSimpleEffect(name, position, yaw, duration, targetId, scale)` | method | 1 | `Common/base/data/EngineWorld.lua:110` |
| `EngineWorld:addVehicleNpc(position, type, yaw)` | method | 1 | `Common/base/data/EngineWorld.lua:64` |
| `EngineWorld:changePlayerActor(player, actor, bodyId)` | method | 1 | `Common/base/data/EngineWorld.lua:117` |
| `EngineWorld:getBlockId(blockPos)` | method | 1 | `Common/base/data/EngineWorld.lua:37` |
| `EngineWorld:getEntity(entityId)` | method | 1 | `Common/base/data/EngineWorld.lua:41` |
| `EngineWorld:getWorld()` | method | 1 | `Common/base/data/EngineWorld.lua:13` |
| `EngineWorld:removeEntity(entityId)` | method | 1 | `Common/base/data/EngineWorld.lua:45` |
| `EngineWorld:resetMap()` | method | 1 | `Common/base/data/EngineWorld.lua:22` |
| `EngineWorld:restorePlayerActor(player)` | method | 1 | `Common/base/data/EngineWorld.lua:124` |
| `EngineWorld:setBlock(blockPos, id, meta, flag, immediate)` | method | 1 | `Common/base/data/EngineWorld.lua:30` |
| `EngineWorld:setBlockToAir(blockPos)` | method | 1 | `Common/base/data/EngineWorld.lua:26` |
| `EngineWorld:setWorld(world)` | method | 1 | `Common/base/data/EngineWorld.lua:9` |
| `EngineWorld:stopWorldTime(time)` | method | 1 | `Common/base/data/EngineWorld.lua:17` |
| `ExpRule:checkLvUp(curLv, curExp, addExp)` | method | 1 | `Common/base/exp/ExpRule.lua:118` |
| `ExpRule:checkTodayExpFull(hasGet, addExp)` | method | 1 | `Common/base/exp/ExpRule.lua:109` |
| `ExpRule:correctAddExp(hasGet, addExp)` | method | 1 | `Common/base/exp/ExpRule.lua:85` |
| `ExpRule:disable()` | method | 1 | `Common/base/exp/ExpRule.lua:59` |
| `ExpRule:getAddExp(time, win, camps)` | method | 1 | `Common/base/exp/ExpRule.lua:71` |
| `ExpRule:getAverageUpExp(curLv, toLv)` | method | 1 | `Common/base/exp/ExpRule.lua:139` |
| `ExpRule:initRole(data)` | method | 1 | `Common/base/exp/ExpRule.lua:22` |
| `ExpRule:isEnable()` | method | 1 | `Common/base/exp/ExpRule.lua:63` |
| `ExpRule:isExpMaxToday(hasGet)` | method | 1 | `Common/base/exp/ExpRule.lua:67` |
| `GameLog.log(userId, type, action, data, immediate)` | namespaced function | 1 | `Common/base/util/GameLog.lua:8` |
| `HttpListener.onHttpResponse(result, marked)` | namespaced function | 1 | `Common/base/listener/HttpListener.lua:13` |
| `HttpListener:init()` | method | 1 | `Common/base/listener/HttpListener.lua:9` |
| `ICommonTask:onTick(ticks)` | method | 1 | `Common/base/task/CommonTask.lua:12` |
| `IMessages:longWaitingHint()` | method | 1 | `Common/base/messages/IMessages.lua:167` |
| `IMessages:msgCanStartGame()` | method | 1 | `Common/base/messages/IMessages.lua:131` |
| `IMessages:msgCloseServerTimeHint(time, unit, ifHalf)` | method | 1 | `Common/base/messages/IMessages.lua:40` |
| `IMessages:msgDontBuyGoodsByNoStart()` | method | 1 | `Common/base/messages/IMessages.lua:151` |
| `IMessages:msgDoubleKill(name)` | method | 1 | `Common/base/messages/IMessages.lua:76` |
| `IMessages:msgFightGameStart()` | method | 1 | `Common/base/messages/IMessages.lua:88` |
| `IMessages:msgFightGameStartTimeHint(time, unit, ifHalf)` | method | 1 | `Common/base/messages/IMessages.lua:36` |
| `IMessages:msgFirstKill(name)` | method | 1 | `Common/base/messages/IMessages.lua:72` |
| `IMessages:msgFiveKill(name)` | method | 1 | `Common/base/messages/IMessages.lua:84` |
| `IMessages:msgGameEndTimeHint(time, unit, ifHalf)` | method | 1 | `Common/base/messages/IMessages.lua:28` |
| `IMessages:msgGameHasStarting()` | method | 1 | `Common/base/messages/IMessages.lua:147` |
| `IMessages:msgGameOver()` | method | 1 | `Common/base/messages/IMessages.lua:108` |
| `IMessages:msgGameOverByDBDataError()` | method | 1 | `Common/base/messages/IMessages.lua:111` |
| `IMessages:msgGameOverByLittle()` | method | 1 | `Common/base/messages/IMessages.lua:52` |
| `IMessages:msgGameOverDeath()` | method | 1 | `Common/base/messages/IMessages.lua:104` |
| `IMessages:msgGameOverHint(seconds)` | method | 1 | `Common/base/messages/IMessages.lua:159` |
| `IMessages:msgGameOverWin()` | method | 1 | `Common/base/messages/IMessages.lua:100` |
| `IMessages:msgGamePlayerIsEnough()` | method | 1 | `Common/base/messages/IMessages.lua:143` |
| `IMessages:msgGamePlayerNum(curPlayers, startPlayers)` | method | 1 | `Common/base/messages/IMessages.lua:139` |
| `IMessages:msgGameResetHint()` | method | 1 | `Common/base/messages/IMessages.lua:163` |
| `IMessages:msgGameRunning()` | method | 1 | `Common/base/messages/IMessages.lua:135` |
| `IMessages:msgGameScore(score)` | method | 1 | `Common/base/messages/IMessages.lua:123` |
| `IMessages:msgGameStart()` | method | 1 | `Common/base/messages/IMessages.lua:44` |
| `IMessages:msgGameStartTimeHint(time, unit, ifHalf)` | method | 1 | `Common/base/messages/IMessages.lua:24` |
| `IMessages:msgNoWinner(type)` | method | 1 | `Common/base/messages/IMessages.lua:115` |
| `IMessages:msgPlayerCount(num)` | method | 1 | `Common/base/messages/IMessages.lua:60` |
| `IMessages:msgPlayerKillPlayer(deceased, killer)` | method | 1 | `Common/base/messages/IMessages.lua:68` |
| `IMessages:msgPlayerLeft(num)` | method | 1 | `Common/base/messages/IMessages.lua:56` |
| `IMessages:msgPlayerQuit(name)` | method | 1 | `Common/base/messages/IMessages.lua:64` |
| `IMessages:msgPvpEnableAfter(time, unit, ifHalf)` | method | 1 | `Common/base/messages/IMessages.lua:32` |
| `IMessages:msgReadyHint(time)` | method | 1 | `Common/base/messages/IMessages.lua:48` |
| `IMessages:msgRespawn(name)` | method | 1 | `Common/base/messages/IMessages.lua:155` |
| `IMessages:msgTeleportToDeadPlace(second)` | method | 1 | `Common/base/messages/IMessages.lua:96` |
| `IMessages:msgThirdKill(name)` | method | 1 | `Common/base/messages/IMessages.lua:80` |
| `IMessages:msgTimeTick(time, unit, ifHalf)` | method | 1 | `Common/base/messages/IMessages.lua:172` |
| `IMessages:msgWaitPlayer()` | method | 1 | `Common/base/messages/IMessages.lua:127` |
| `IMessages:msgWaitPlayerEnough(time, unit, ifHalf)` | method | 1 | `Common/base/messages/IMessages.lua:20` |
| `IMessages:msgWelcomePlayer(game)` | method | 1 | `Common/base/messages/IMessages.lua:16` |
| `IMessages:msgWinnerInfo(name)` | method | 1 | `Common/base/messages/IMessages.lua:92` |
| `INT2HEX(x)` | file-local function | 1 | `Common/base/util/uuid.lua:64` |
| `ITask:ctor(status)` | method | 1 | `Common/base/task/ITask.lua:11` |
| `ITask:onCreate()` | method | 1 | `Common/base/task/ITask.lua:22` |
| `ITask:onTick(ticks)` | method | 1 | `Common/base/task/ITask.lua:26` |
| `ITask:pureStop()` | method | 1 | `Common/base/task/ITask.lua:38` |
| `ITask:start()` | method | 1 | `Common/base/task/ITask.lua:30` |
| `ITask:stop()` | method | 1 | `Common/base/task/ITask.lua:34` |
| `InventoryUtil:fillItem(inv, config)` | method | 1 | `Common/base/util/InventoryUtil.lua:7` |
| `InventoryUtil:fillItemByType(inv, slotBegin, config, rangeLow, rangeUpper)` | method | 1 | `Common/base/util/InventoryUtil.lua:22` |
| `InventoryUtil:getPlayerEnderInventoryInfo(player)` | method | 1 | `Common/base/util/InventoryUtil.lua:152` |
| `InventoryUtil:getPlayerInventoryInfo(player)` | method | 1 | `Common/base/util/InventoryUtil.lua:119` |
| `InventoryUtil:prepareChest(config)` | method | 1 | `Common/base/util/InventoryUtil.lua:109` |
| `InventoryUtil:randomItem(subconfig, numMin, numMax)` | method | 1 | `Common/base/util/InventoryUtil.lua:37` |
| `InventoryUtil:randomItemNum(item)` | method | 1 | `Common/base/util/InventoryUtil.lua:97` |
| `LuaTimer:cancel(key)` | method | 1 | `Common/base/task/LuaTimer.lua:125` |
| `LuaTimer:onTick()` | method | 1 | `Common/base/task/LuaTimer.lua:18` |
| `LuaTimer:schedule(action, delay, period, ...)` | method | 1 | `Common/base/task/LuaTimer.lua:72` |
| `LuaTimer:scheduleTimer(action, period, times, ...)` | method | 1 | `Common/base/task/LuaTimer.lua:106` |
| `M.new(hwaddr)` | namespaced function | 1 | `Common/base/util/uuid.lua:97` |
| `M.randomseed(seed)` | namespaced function | 1 | `Common/base/util/uuid.lua:170` |
| `M.seed()` | namespaced function | 1 | `Common/base/util/uuid.lua:199` |
| `MerchantUtil:addCommodity(index, type, goods)` | method | 1 | `Common/base/util/MerchantUtil.lua:7` |
| `MerchantUtil:changeCommodityList(entityId, rakssId, index)` | method | 1 | `Common/base/util/MerchantUtil.lua:11` |
| `MsgSender.sendBottomTips(time, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:41` |
| `MsgSender.sendBottomTipsToTarget(rakssid, time, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:45` |
| `MsgSender.sendCenterTips(time, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:58` |
| `MsgSender.sendCenterTipsToTarget(rakssid, time, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:62` |
| `MsgSender.sendKillMsg(killer, dead, armsId, kills, head, color)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:92` |
| `MsgSender.sendKillMsgToTarget(rakssid, killer, dead, armsId, kills, head, color)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:96` |
| `MsgSender.sendMsg(msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:7` |
| `MsgSender.sendMsgToTarget(rakssid, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:11` |
| `MsgSender.sendOtherTips(time, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:75` |
| `MsgSender.sendOtherTipsToTarget(rakssid, time, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:79` |
| `MsgSender.sendTopTips(time, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:24` |
| `MsgSender.sendTopTipsToTarget(rakssid, time, msg, args)` | namespaced function | 1 | `Common/base/util/MsgSender.lua:28` |
| `Null.__tostring()` | namespaced function | 1 | `Common/base/util/tinyyaml.lua:82` |
| `Null.isnull(v)` | namespaced function | 1 | `Common/base/util/tinyyaml.lua:85` |
| `NumberUtil.getIntPart(num)` | namespaced function | 1 | `Common/base/util/NumberUtil.lua:7` |
| `PlayerClassInfo:addClasses(platformId, classInfo)` | method | 1 | `Common/base/classes/PlayerClassInfo.lua:6` |
| `PlayerClassInfo:getClasses(platformId)` | method | 1 | `Common/base/classes/PlayerClassInfo.lua:10` |
| `PlayerClassInfo:removeClasses(platformId)` | method | 1 | `Common/base/classes/PlayerClassInfo.lua:14` |
| `PlayerManager:addPlayer(player)` | method | 1 | `Common/base/PlayerManager.lua:39` |
| `PlayerManager:copyPlayers()` | method | 1 | `Common/base/PlayerManager.lua:106` |
| `PlayerManager:getMaxPlayer()` | method | 1 | `Common/base/PlayerManager.lua:19` |
| `PlayerManager:getPlayerByEntityId(entityId)` | method | 1 | `Common/base/PlayerManager.lua:86` |
| `PlayerManager:getPlayerByPlayerMP(playerMP)` | method | 1 | `Common/base/PlayerManager.lua:95` |
| `PlayerManager:getPlayerByRakssid(rakssId)` | method | 1 | `Common/base/PlayerManager.lua:68` |
| `PlayerManager:getPlayerByUserId(userId)` | method | 1 | `Common/base/PlayerManager.lua:77` |
| `PlayerManager:getPlayerCount()` | method | 1 | `Common/base/PlayerManager.lua:64` |
| `PlayerManager:getPlayers()` | method | 1 | `Common/base/PlayerManager.lua:102` |
| `PlayerManager:isPlayerEmpty()` | method | 1 | `Common/base/PlayerManager.lua:27` |
| `PlayerManager:isPlayerEnough(players)` | method | 1 | `Common/base/PlayerManager.lua:31` |
| `PlayerManager:isPlayerExists()` | method | 1 | `Common/base/PlayerManager.lua:35` |
| `PlayerManager:isPlayerFull()` | method | 1 | `Common/base/PlayerManager.lua:23` |
| `PlayerManager:setMaxPlayer(maxPlayer)` | method | 1 | `Common/base/PlayerManager.lua:15` |
| `PlayerManager:subPlayer(player)` | method | 1 | `Common/base/PlayerManager.lua:46` |
| `RankManager:addPlayerDayRank(player, key, rank, score, func, ...)` | method | 1 | `Common/base/rank/RankManager.lua:43` |
| `RankManager:addPlayerWeekRank(player, key, rank, score, func, ...)` | method | 1 | `Common/base/rank/RankManager.lua:61` |
| `RankManager:buildPlayerRank(rakssid, key)` | method | 1 | `Common/base/rank/RankManager.lua:19` |
| `RankManager:clearRanks()` | method | 1 | `Common/base/rank/RankManager.lua:97` |
| `RankManager:removePlayerRank(rakssid)` | method | 1 | `Common/base/rank/RankManager.lua:9` |
| `RankManager:sendPlayerRank(rakssid, key, data, entityId)` | method | 1 | `Common/base/rank/RankManager.lua:79` |
| `ReportManager:closeServer()` | method | 1 | `Common/base/ReportManager.lua:21` |
| `ReportManager:getUserReportCache(userId)` | method | 1 | `Common/base/ReportManager.lua:40` |
| `ReportManager:onPlayerLogout(player)` | method | 1 | `Common/base/ReportManager.lua:32` |
| `ReportManager:reportUserCount(userId)` | method | 1 | `Common/base/ReportManager.lua:99` |
| `ReportManager:reportUserData(userId, kills, rank, isCount)` | method | 1 | `Common/base/ReportManager.lua:60` |
| `ReportManager:reportUserIntegral(userId, integral)` | method | 1 | `Common/base/ReportManager.lua:107` |
| `ReportManager:reportUserKill(userId, kill)` | method | 1 | `Common/base/ReportManager.lua:75` |
| `ReportManager:reportUserRank(userId, rank)` | method | 1 | `Common/base/ReportManager.lua:91` |
| `ReportManager:reportUserTime(userId, time)` | method | 1 | `Common/base/ReportManager.lua:83` |
| `ReportManager:setGameType(GameType)` | method | 1 | `Common/base/ReportManager.lua:13` |
| `ReportManager:setRankType(RankType)` | method | 1 | `Common/base/ReportManager.lua:17` |
| `ReportManager:tryCloseServer()` | method | 1 | `Common/base/ReportManager.lua:127` |
| `ReportManager:tryReportGameData()` | method | 1 | `Common/base/ReportManager.lua:116` |
| `RewardManager:addRewardQueue(userId, rank)` | method | 1 | `Common/base/RewardManager.lua:37` |
| `RewardManager:getQueueReward(func, ...)` | method | 1 | `Common/base/RewardManager.lua:49` |
| `RewardManager:getStartGameTime()` | method | 1 | `Common/base/RewardManager.lua:29` |
| `RewardManager:getStartPlayerCount()` | method | 1 | `Common/base/RewardManager.lua:25` |
| `RewardManager:getUserGoldReward(userId, gold, func)` | method | 1 | `Common/base/RewardManager.lua:91` |
| `RewardManager:getUserReward(userId, rank, func)` | method | 1 | `Common/base/RewardManager.lua:73` |
| `RewardManager:isUserRewardFinish(userId)` | method | 1 | `Common/base/RewardManager.lua:33` |
| `RewardManager:setGameType(GameType)` | method | 1 | `Common/base/RewardManager.lua:14` |
| `RewardManager:startGame()` | method | 1 | `Common/base/RewardManager.lua:18` |
| `SeasonManager:addUserHonor(userId, honor, level)` | method | 1 | `Common/base/season/SeasonManager.lua:10` |
| `SeasonManager:trySaveHonorResult()` | method | 1 | `Common/base/season/SeasonManager.lua:18` |
| `StringUtil.split(str, reps, keep_blank)` | namespaced function | 1 | `Common/base/util/StringUtil.lua:7` |
| `StringUtil.v2s(value, maxLevel)` | namespaced function | 1 | `Common/base/util/StringUtil.lua:63` |
| `TextFormat:getTipType(num)` | method | 1 | `Common/base/messages/TextFormat.lua:25` |
| `UserExpManager:addExpCache(userId, curLv, curExp, hasGet)` | method | 1 | `Common/base/exp/UserExpManager.lua:20` |
| `UserExpManager:addUserExp(userId, win, camps, global)` | method | 1 | `Common/base/exp/UserExpManager.lua:63` |
| `UserExpManager:getUserExpCache(userId)` | method | 1 | `Common/base/exp/UserExpManager.lua:13` |
| `UserExpManager:removeExpCache(userId)` | method | 1 | `Common/base/exp/UserExpManager.lua:42` |
| `UserExpManager:sendExpResult(userId, status)` | method | 1 | `Common/base/exp/UserExpManager.lua:101` |
| `UserExpManager:tryCalculationExp()` | method | 1 | `Common/base/exp/UserExpManager.lua:115` |
| `UserExpManager:trySaveExpResult()` | method | 1 | `Common/base/exp/UserExpManager.lua:127` |
| `UserInfoCache:FilterExistedUserId(userIds)` | method | 1 | `Common/base/cache/UserInfoCache.lua:8` |
| `UserInfoCache:GetCache(userId)` | method | 1 | `Common/base/cache/UserInfoCache.lua:38` |
| `UserInfoCache:GetCacheByUserIds(userIds, func, key, ...)` | method | 1 | `Common/base/cache/UserInfoCache.lua:18` |
| `VectorUtil.equal(vec3_1, vec3_2)` | namespaced function | 1 | `Common/base/util/VectorUtil.lua:57` |
| `VectorUtil.hashVector3(vec3)` | namespaced function | 1 | `Common/base/util/VectorUtil.lua:40` |
| `VectorUtil.newVector3(x, y, z)` | namespaced function | 1 | `Common/base/util/VectorUtil.lua:16` |
| `VectorUtil.newVector3i(x, y, z)` | namespaced function | 1 | `Common/base/util/VectorUtil.lua:8` |
| `VectorUtil.toBlockVector3(x, y, z)` | namespaced function | 1 | `Common/base/util/VectorUtil.lua:44` |
| `VectorUtil.toVector3(vec3i)` | namespaced function | 1 | `Common/base/util/VectorUtil.lua:24` |
| `VectorUtil.toVector3i(vec3)` | namespaced function | 1 | `Common/base/util/VectorUtil.lua:32` |
| `WalletUtils:addCoinMappings(coinMapping)` | method | 1 | `Common/base/util/WalletUtils.lua:8` |
| `WebResponse:formatData(result)` | method | 1 | `Common/base/web/WebResponse.lua:278` |
| `WebResponse:formatResult(result)` | method | 1 | `Common/base/web/WebResponse.lua:254` |
| `WebResponse:formatResultCode(result)` | method | 1 | `Common/base/web/WebResponse.lua:266` |
| `WebResponse:onCallFunc(data, marked)` | method | 1 | `Common/base/web/WebResponse.lua:24` |
| `WebResponse:onCheckMails(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:207` |
| `WebResponse:onGetBlockymodsExp(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:136` |
| `WebResponse:onGetBlockymodsExpRole(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:125` |
| `WebResponse:onGetBlockymodsGoldReward(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:110` |
| `WebResponse:onGetBlockymodsReward(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:95` |
| `WebResponse:onGetBlockymodsRewardList(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:186` |
| `WebResponse:onGetBlockymodsUserAttr(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:78` |
| `WebResponse:onGetBlockymodsUserInfos(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:180` |
| `WebResponse:onGetBlockymodsUserSeasonInfo(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:156` |
| `WebResponse:onGetMcpeonlineReport(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:74` |
| `WebResponse:onGetMcpeonlineReward(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:57` |
| `WebResponse:onGetMcpeonlineUserAttr(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:32` |
| `WebResponse:onReportBlockymodsData(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:176` |
| `WebResponse:onReportBlockymodsList(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:203` |
| `WebResponse:onSaveBlockymodsExp(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:148` |
| `WebResponse:onSaveBlockymodsHonor(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:152` |
| `WebResponse:onSendMails(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:234` |
| `WebResponse:onUpdateBlockymodsUserSeasonReward(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:162` |
| `WebResponse:onUpdateMails(result, marked)` | method | 1 | `Common/base/web/WebResponse.lua:244` |
| `WebResponse:registerCallBack(func, marked, ...)` | method | 1 | `Common/base/web/WebResponse.lua:11` |
| `WebResponse:unregisterCallBack(marked)` | method | 1 | `Common/base/web/WebResponse.lua:18` |
| `WebService:GetBlockymodsExpRule()` | method | 1 | `Common/base/web/WebService.lua:210` |
| `WebService:GetBlockymodsGoldReward(userId, gameType, golds)` | method | 1 | `Common/base/web/WebService.lua:157` |
| `WebService:GetBlockymodsReward(userId, gameType, data)` | method | 1 | `Common/base/web/WebService.lua:134` |
| `WebService:GetBlockymodsRewardList(data)` | method | 1 | `Common/base/web/WebService.lua:327` |
| `WebService:GetBlockymodsUserAttr(userId, gameType)` | method | 1 | `Common/base/web/WebService.lua:113` |
| `WebService:GetBlockymodsUserExp(userId)` | method | 1 | `Common/base/web/WebService.lua:224` |
| `WebService:GetBlockymodsUserInfos(userIds, key)` | method | 1 | `Common/base/web/WebService.lua:182` |
| `WebService:GetBlockymodsUserSeasonInfo(userId, isLast)` | method | 1 | `Common/base/web/WebService.lua:256` |
| `WebService:GetPlayerReward(userId, gametype, kills, iswin, gameId)` | method | 1 | `Common/base/web/WebService.lua:68` |
| `WebService:GetUserAttr(userId, gameType)` | method | 1 | `Common/base/web/WebService.lua:50` |
| `WebService:PostGameResult(gameType, resultJson)` | method | 1 | `Common/base/web/WebService.lua:97` |
| `WebService:ReportBlockymodsData(userId, gameType, time, kills, rank, isCount, integral, type)` | method | 1 | `Common/base/web/WebService.lua:300` |
| `WebService:ReportBlockymodsList(data)` | method | 1 | `Common/base/web/WebService.lua:339` |
| `WebService:SaveBlockymodsUsersExp(data)` | method | 1 | `Common/base/web/WebService.lua:240` |
| `WebService:SaveBlockymodsUsersHonor(data)` | method | 1 | `Common/base/web/WebService.lua:248` |
| `WebService:UpdateBlockymodsUserSeasonReward(userId)` | method | 1 | `Common/base/web/WebService.lua:283` |
| `WebService:UseProps(userId, PropId)` | method | 1 | `Common/base/web/WebService.lua:109` |
| `WebService:checkMails(userId, gameType)` | method | 1 | `Common/base/web/WebService.lua:351` |
| `WebService:init(version, config)` | method | 1 | `Common/base/web/WebService.lua:35` |
| `WebService:initGameId()` | method | 1 | `Common/base/web/WebService.lua:44` |
| `WebService:sendMails(fromUser, toUser, mailType, title, content, gameType)` | method | 1 | `Common/base/web/WebService.lua:371` |
| `WebService:sendMsg(userId, targetIds, content, msgType, scope, gameType)` | method | 1 | `Common/base/web/WebService.lua:410` |
| `WebService:updateMails(userId, id, status)` | method | 1 | `Common/base/web/WebService.lua:392` |
| `_copy(object)` | file-local function | 1 | `Common/base/util/class.lua:5` |
| `class(classname, super)` | global function | 1 | `Common/base/util/class.lua:21` |
| `class.__meta.__call(cls, ...)` | namespaced function | 1 | `Common/base/util/tinyyaml.lua:49` |
| `class.def(base, type, cls)` | namespaced function | 1 | `Common/base/util/tinyyaml.lua:57` |
| `clone(object)` | global function | 1 | `Common/base/util/class.lua:3` |
| `cls.ctor()` | assigned function | 1 | `Common/base/util/class.lua:43` |
| `cls.new(...)` | namespaced function | 2 | `Common/base/util/class.lua:47` |
| `codepoint_to_utf8(n)` | file-local function | 1 | `Common/base/util/json.lua:174` |
| `countindent(line)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:136` |
| `create_set(...)` | file-local function | 1 | `Common/base/util/json.lua:130` |
| `decode_error(str, idx, msg)` | file-local function | 1 | `Common/base/util/json.lua:160` |
| `doT2S(_i, _v)` | global function | 1 | `Common/base/util/vardump.lua:6` |
| `encode(val, stack)` | assigned function | 1 | `Common/base/util/json.lua:109` |
| `encode_nil(val)` | file-local function | 1 | `Common/base/util/json.lua:39` |
| `encode_number(val)` | file-local function | 1 | `Common/base/util/json.lua:91` |
| `encode_string(val)` | file-local function | 1 | `Common/base/util/json.lua:86` |
| `encode_table(val, stack)` | file-local function | 1 | `Common/base/util/json.lua:44` |
| `escape_char(c)` | file-local function | 1 | `Common/base/util/json.lua:34` |
| `isemptyline(line)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:211` |
| `json.decode(str)` | namespaced function | 1 | `Common/base/util/json.lua:372` |
| `json.encode(val)` | namespaced function | 1 | `Common/base/util/json.lua:119` |
| `ltrim(str)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:37` |
| `next_char(str, idx, set, negate)` | file-local function | 1 | `Common/base/util/json.lua:150` |
| `parse(source)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:751` |
| `parse(str, idx)` | assigned function | 1 | `Common/base/util/json.lua:362` |
| `parse_array(str, i)` | file-local function | 1 | `Common/base/util/json.lua:277` |
| `parse_literal(str, i)` | file-local function | 1 | `Common/base/util/json.lua:267` |
| `parse_number(str, i)` | file-local function | 1 | `Common/base/util/json.lua:256` |
| `parse_object(str, i)` | file-local function | 1 | `Common/base/util/json.lua:304` |
| `parse_string(str, i)` | file-local function | 1 | `Common/base/util/json.lua:203` |
| `parse_unicode_escape(s)` | file-local function | 1 | `Common/base/util/json.lua:191` |
| `parseblockstylestring(line, lines, indent)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:290` |
| `parsedocuments(lines)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:689` |
| `parseflowstyle(line, lines)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:219` |
| `parsemap(line, lines, indent)` | global function | 1 | `Common/base/util/tinyyaml.lua:599` |
| `parsescalar(line, lines, indent)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:404` |
| `parseseq(line, lines, indent)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:468` |
| `parseset(line, lines, indent)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:536` |
| `parsestring(line, stopper)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:144` |
| `parsetimestamp(line)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:365` |
| `rtrim(str)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:41` |
| `select(list, pred)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:19` |
| `startswith(haystack, needle)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:33` |
| `startswithline(line, needle)` | file-local function | 1 | `Common/base/util/tinyyaml.lua:215` |
| `types.timestamp:__init(y, m, d, h, i, s, f, z)` | method | 1 | `Common/base/util/tinyyaml.lua:96` |
| `types.timestamp:__tostring()` | method | 1 | `Common/base/util/tinyyaml.lua:113` |
| `types.timestamp:gettz()` | method | 1 | `Common/base/util/tinyyaml.lua:121` |
| `v2s(value, sp, restLevel)` | file-local function | 1 | `Common/base/util/StringUtil.lua:32` |
| `var_dump(_t)` | global function | 1 | `Common/base/util/vardump.lua:4` |

---

## 5. Constants and Enums

Field assignments owned by each table. Values longer than 60 characters and function assignments are omitted.

**31 constant group(s)** retained; 12 low-value local/scratch table(s) omitted.

| Group | Members |
| --- | ---: |
| [AppProps](#const-appprops) | 3 |
| [BlockID](#const-blockid) | 167 |
| [ExpRule](#const-exprule) | 4 |
| [GameOverCode](#const-gameovercode) | 7 |
| [IMessages](#const-imessages) | 4 |
| [ITask](#const-itask) | 3 |
| [ItemID](#const-itemid) | 173 |
| [LuaTimer](#const-luatimer) | 3 |
| [MailType](#const-mailtype) | 7 |
| [PotionID](#const-potionid) | 23 |
| [PotionItemID](#const-potionitemid) | 22 |
| [ReportManager](#const-reportmanager) | 4 |
| [RewardManager](#const-rewardmanager) | 4 |
| [ScoreID](#const-scoreid) | 8 |
| [TextFormat](#const-textformat) | 14 |
| [WebRequestType](#const-webrequesttype) | 24 |
| [WebService](#const-webservice) | 25 |
| [cache](#const-cache) | 6 |
| [cls](#const-cls) | 13 |
| [data](#const-data) | 38 |
| [expInfo](#const-expinfo) | 5 |
| [item](#const-item) | 38 |
| [player](#const-player) | 13 |
| [player.staticAttr](#const-playerstaticattr) | 15 |
| [self](#const-self) | 57 |
| [self.dynamicAttr](#const-selfdynamicattr) | 12 |
| [self.staticAttr](#const-selfstaticattr) | 13 |
| [settlement](#const-settlement) | 7 |
| [task](#const-task) | 14 |
| [v](#const-v) | 5 |
| [vec3](#const-vec3) | 6 |

### AppProps

| Member | Value | Location |
| --- | --- | --- |
| `TYPE_BUFF` | `2` | `Common/base/prop/AppProps.lua:10` |
| `TYPE_CUSTOM` | `3` | `Common/base/prop/AppProps.lua:11` |
| `TYPE_ITEM` | `1` | `Common/base/prop/AppProps.lua:9` |

### BlockID

| Member | Value | Location |
| --- | --- | --- |
| `AIR` | `0` | `Common/base/define/BlockID.lua:4` |
| `ANVIL` | `145` | `Common/base/define/BlockID.lua:149` |
| `BEACON` | `138` | `Common/base/define/BlockID.lua:142` |
| `BED` | `26` | `Common/base/define/BlockID.lua:30` |
| `BEDROCK` | `7` | `Common/base/define/BlockID.lua:11` |
| `BLOCK_CLAY` | `82` | `Common/base/define/BlockID.lua:86` |
| `BLOCK_COAL` | `173` | `Common/base/define/BlockID.lua:170` |
| `BLOCK_DIAMOND` | `57` | `Common/base/define/BlockID.lua:61` |
| `BLOCK_EMERALD` | `133` | `Common/base/define/BlockID.lua:137` |
| `BLOCK_GOLD` | `41` | `Common/base/define/BlockID.lua:45` |
| `BLOCK_IRON` | `42` | `Common/base/define/BlockID.lua:46` |
| `BLOCK_LAPIS` | `22` | `Common/base/define/BlockID.lua:26` |
| `BLOCK_NETHER_QUARTZ` | `155` | `Common/base/define/BlockID.lua:159` |
| `BLOCK_REDSTONE` | `152` | `Common/base/define/BlockID.lua:156` |
| `BLOCK_SNOW` | `80` | `Common/base/define/BlockID.lua:84` |
| `BOOK_SHELF` | `47` | `Common/base/define/BlockID.lua:51` |
| `BREWING_STAND` | `117` | `Common/base/define/BlockID.lua:121` |
| `BRICK` | `45` | `Common/base/define/BlockID.lua:49` |
| `CACTUS` | `81` | `Common/base/define/BlockID.lua:85` |
| `CAKE` | `92` | `Common/base/define/BlockID.lua:96` |
| `CARROT` | `141` | `Common/base/define/BlockID.lua:145` |
| `CAULDRON` | `118` | `Common/base/define/BlockID.lua:122` |
| `CHEST` | `54` | `Common/base/define/BlockID.lua:58` |
| `CHEST_TRAPPED` | `146` | `Common/base/define/BlockID.lua:150` |
| `CLAY_HARDENED` | `172` | `Common/base/define/BlockID.lua:169` |
| `CLAY_HARDENED_STAINED` | `159` | `Common/base/define/BlockID.lua:163` |
| `CLOTH` | `35` | `Common/base/define/BlockID.lua:39` |
| `COBBLESTONE` | `4` | `Common/base/define/BlockID.lua:8` |
| `COBBLESTONE_MOSSY` | `48` | `Common/base/define/BlockID.lua:52` |
| `COBBLESTONE_WALL` | `139` | `Common/base/define/BlockID.lua:143` |
| `COCOA_PLANT` | `127` | `Common/base/define/BlockID.lua:131` |
| `COMMAND_BLOCK` | `137` | `Common/base/define/BlockID.lua:141` |
| `CROPS` | `59` | `Common/base/define/BlockID.lua:63` |
| `DAYLIGHT_SENSOR` | `151` | `Common/base/define/BlockID.lua:155` |
| `DEAD_BUSH` | `32` | `Common/base/define/BlockID.lua:36` |
| `DIRT` | `3` | `Common/base/define/BlockID.lua:7` |
| `DISPENSER` | `23` | `Common/base/define/BlockID.lua:27` |
| `DOOR_IRON` | `71` | `Common/base/define/BlockID.lua:75` |
| `DOOR_WOOD` | `64` | `Common/base/define/BlockID.lua:68` |
| `DRAGON_EGG` | `122` | `Common/base/define/BlockID.lua:126` |
| `DROPPER` | `158` | `Common/base/define/BlockID.lua:162` |
| `ENCHANTMENT_TABLE` | `116` | `Common/base/define/BlockID.lua:120` |
| `ENDER_CHEST` | `130` | `Common/base/define/BlockID.lua:134` |
| `END_PORTAL` | `119` | `Common/base/define/BlockID.lua:123` |
| `END_PORTAL_FRAME` | `120` | `Common/base/define/BlockID.lua:124` |
| `FENCE` | `85` | `Common/base/define/BlockID.lua:89` |
| `FENCE_GATE` | `107` | `Common/base/define/BlockID.lua:111` |
| `FENCE_IRON` | `101` | `Common/base/define/BlockID.lua:105` |
| `FIRE` | `51` | `Common/base/define/BlockID.lua:55` |
| `FLOWER_POT` | `140` | `Common/base/define/BlockID.lua:144` |
| `FURNACE_BURNING` | `62` | `Common/base/define/BlockID.lua:66` |
| `FURNACE_IDLE` | `61` | `Common/base/define/BlockID.lua:65` |
| `GLASS` | `20` | `Common/base/define/BlockID.lua:24` |
| `GLOW_STONE` | `89` | `Common/base/define/BlockID.lua:93` |
| `GRASS` | `2` | `Common/base/define/BlockID.lua:6` |
| `GRAVEL` | `13` | `Common/base/define/BlockID.lua:17` |
| `HAY_BLOCK` | `170` | `Common/base/define/BlockID.lua:167` |
| `HOPPER_BLOCK` | `154` | `Common/base/define/BlockID.lua:158` |
| `ICE` | `79` | `Common/base/define/BlockID.lua:83` |
| `JUKEBOX` | `84` | `Common/base/define/BlockID.lua:88` |
| `LADDER` | `65` | `Common/base/define/BlockID.lua:69` |
| `LAVA_MOVING` | `10` | `Common/base/define/BlockID.lua:14` |
| `LAVA_STILL` | `11` | `Common/base/define/BlockID.lua:15` |
| `LEAVES` | `18` | `Common/base/define/BlockID.lua:22` |
| `LEVER` | `69` | `Common/base/define/BlockID.lua:73` |
| `LOCKED_CHEST` | `95` | `Common/base/define/BlockID.lua:99` |
| `MELON` | `103` | `Common/base/define/BlockID.lua:107` |
| `MELON_STEM` | `105` | `Common/base/define/BlockID.lua:109` |
| `MOB_SPAWNER` | `52` | `Common/base/define/BlockID.lua:56` |
| `MUSHROOM_BROWN` | `39` | `Common/base/define/BlockID.lua:43` |
| `MUSHROOM_CAPBROWN` | `99` | `Common/base/define/BlockID.lua:103` |
| `MUSHROOM_CAP_RED` | `100` | `Common/base/define/BlockID.lua:104` |
| `MUSHROOM_RED` | `40` | `Common/base/define/BlockID.lua:44` |
| `MUSIC` | `25` | `Common/base/define/BlockID.lua:29` |
| `MYCELIUM` | `110` | `Common/base/define/BlockID.lua:114` |
| `NETHERRACK` | `87` | `Common/base/define/BlockID.lua:91` |
| `NETHER_BRICK` | `112` | `Common/base/define/BlockID.lua:116` |
| `NETHER_FENCE` | `113` | `Common/base/define/BlockID.lua:117` |
| `NETHER_STALK` | `115` | `Common/base/define/BlockID.lua:119` |
| `OBSIDIAN` | `49` | `Common/base/define/BlockID.lua:53` |
| `ORE_COAL` | `16` | `Common/base/define/BlockID.lua:20` |
| `ORE_DIAMOND` | `56` | `Common/base/define/BlockID.lua:60` |
| `ORE_EMERALD` | `129` | `Common/base/define/BlockID.lua:133` |
| `ORE_GOLD` | `14` | `Common/base/define/BlockID.lua:18` |
| `ORE_IRON` | `15` | `Common/base/define/BlockID.lua:19` |
| `ORE_LAPIS` | `21` | `Common/base/define/BlockID.lua:25` |
| `ORE_NETHER_QUARTZ` | `153` | `Common/base/define/BlockID.lua:157` |
| `ORE_READSTONE_GLOWING` | `74` | `Common/base/define/BlockID.lua:78` |
| `ORE_REDSTONE` | `73` | `Common/base/define/BlockID.lua:77` |
| `PACKED_ICE` | `174` | `Common/base/define/BlockID.lua:171` |
| `PISTON_BASE` | `33` | `Common/base/define/BlockID.lua:37` |
| `PISTON_EXTENSION` | `34` | `Common/base/define/BlockID.lua:38` |
| `PISTON_MOVING` | `36` | `Common/base/define/BlockID.lua:40` |
| `PISTON_STICKY_BASE` | `29` | `Common/base/define/BlockID.lua:33` |
| `PLANKS` | `5` | `Common/base/define/BlockID.lua:9` |
| `PLANT_RED` | `38` | `Common/base/define/BlockID.lua:42` |
| `PLANT_YELLOW` | `37` | `Common/base/define/BlockID.lua:41` |
| `PORTAL` | `90` | `Common/base/define/BlockID.lua:94` |
| `POTATO` | `142` | `Common/base/define/BlockID.lua:146` |
| `PRESSURE_PLATE_GOLD` | `147` | `Common/base/define/BlockID.lua:151` |
| `PRESSURE_PLATE_IRON` | `148` | `Common/base/define/BlockID.lua:152` |
| `PRESSURE_PLATE_PLANKS` | `72` | `Common/base/define/BlockID.lua:76` |
| `PRESSURE_PLATE_STONE` | `70` | `Common/base/define/BlockID.lua:74` |
| `PUMPKIN` | `86` | `Common/base/define/BlockID.lua:90` |
| `PUMPKIN_LANTERN` | `91` | `Common/base/define/BlockID.lua:95` |
| `PUMPKIN_STEM` | `104` | `Common/base/define/BlockID.lua:108` |
| `RAIL` | `66` | `Common/base/define/BlockID.lua:70` |
| `RAIL_ACTIVATOR` | `157` | `Common/base/define/BlockID.lua:161` |
| `RAIL_DETECTOR` | `28` | `Common/base/define/BlockID.lua:32` |
| `RAIL_POWERED` | `27` | `Common/base/define/BlockID.lua:31` |
| `RAIL_RECEIVE` | `175` | `Common/base/define/BlockID.lua:172` |
| `REDSTONE_COMPARATOR_ACTIVE` | `150` | `Common/base/define/BlockID.lua:154` |
| `REDSTONE_COMPARATOR_IDLE` | `149` | `Common/base/define/BlockID.lua:153` |
| `REDSTONE_LAMP_ACTIVE` | `124` | `Common/base/define/BlockID.lua:128` |
| `REDSTONE_LAMP_IDLE` | `123` | `Common/base/define/BlockID.lua:127` |
| `REDSTONE_REPEATER_ACTIVE` | `94` | `Common/base/define/BlockID.lua:98` |
| `REDSTONE_REPEATER_IDLE` | `93` | `Common/base/define/BlockID.lua:97` |
| `REDSTONE_WIRE` | `55` | `Common/base/define/BlockID.lua:59` |
| `REED` | `83` | `Common/base/define/BlockID.lua:87` |
| `SAND` | `12` | `Common/base/define/BlockID.lua:16` |
| `SAND_STONE` | `24` | `Common/base/define/BlockID.lua:28` |
| `SAPLING` | `6` | `Common/base/define/BlockID.lua:10` |
| `SIGN_POST` | `63` | `Common/base/define/BlockID.lua:67` |
| `SIGN_WALL` | `68` | `Common/base/define/BlockID.lua:72` |
| `SILVERFISH` | `97` | `Common/base/define/BlockID.lua:101` |
| `SKULL` | `144` | `Common/base/define/BlockID.lua:148` |
| `SLOW_SAND` | `88` | `Common/base/define/BlockID.lua:92` |
| `SNOW` | `78` | `Common/base/define/BlockID.lua:82` |
| `SPONGE` | `19` | `Common/base/define/BlockID.lua:23` |
| `STAIRS_BRICK` | `108` | `Common/base/define/BlockID.lua:112` |
| `STAIRS_COBBLESTONE` | `67` | `Common/base/define/BlockID.lua:71` |
| `STAIRS_NETHER_BRICK` | `114` | `Common/base/define/BlockID.lua:118` |
| `STAIRS_NETHER_QUARTZ` | `156` | `Common/base/define/BlockID.lua:160` |
| `STAIRS_SAND_STONE` | `128` | `Common/base/define/BlockID.lua:132` |
| `STAIRS_STONE_BRICK` | `109` | `Common/base/define/BlockID.lua:113` |
| `STAIRS_WOOD_BIRCH` | `135` | `Common/base/define/BlockID.lua:139` |
| `STAIRS_WOOD_JUNGLE` | `136` | `Common/base/define/BlockID.lua:140` |
| `STAIRS_WOOD_OAK` | `53` | `Common/base/define/BlockID.lua:57` |
| `STAIRS_WOOD_SPRUCE` | `134` | `Common/base/define/BlockID.lua:138` |
| `STONE` | `1` | `Common/base/define/BlockID.lua:5` |
| `STONE_BRICK` | `98` | `Common/base/define/BlockID.lua:102` |
| `STONE_BUTTON` | `77` | `Common/base/define/BlockID.lua:81` |
| `STONE_DOUBLE_SLAB` | `43` | `Common/base/define/BlockID.lua:47` |
| `STONE_SINGLE_SLAB` | `44` | `Common/base/define/BlockID.lua:48` |
| `TALL_GRASS` | `31` | `Common/base/define/BlockID.lua:35` |
| `THIN_GLASS` | `102` | `Common/base/define/BlockID.lua:106` |
| `TILLED_FIELD` | `60` | `Common/base/define/BlockID.lua:64` |
| `TNT` | `46` | `Common/base/define/BlockID.lua:50` |
| `TORCH_REDSTONE_ACTIVE` | `76` | `Common/base/define/BlockID.lua:80` |
| `TORCH_REDSTONE_IDLE` | `75` | `Common/base/define/BlockID.lua:79` |
| `TORCH_WOOD` | `50` | `Common/base/define/BlockID.lua:54` |
| `TRAPDOOR` | `96` | `Common/base/define/BlockID.lua:100` |
| `TRIP_WIRE` | `132` | `Common/base/define/BlockID.lua:136` |
| `TRIP_WIRE_SOURCE` | `131` | `Common/base/define/BlockID.lua:135` |
| `VINE` | `106` | `Common/base/define/BlockID.lua:110` |
| `WATERLILY` | `111` | `Common/base/define/BlockID.lua:115` |
| `WATER_MOVING` | `8` | `Common/base/define/BlockID.lua:12` |
| `WATER_STILL` | `9` | `Common/base/define/BlockID.lua:13` |
| `WEB` | `30` | `Common/base/define/BlockID.lua:34` |
| `WHITE_STONE` | `121` | `Common/base/define/BlockID.lua:125` |
| `WOOD` | `17` | `Common/base/define/BlockID.lua:21` |
| `WOOD2` | `162` | `Common/base/define/BlockID.lua:165` |
| `WOODEN_BUTTON` | `143` | `Common/base/define/BlockID.lua:147` |
| `WOOD_DOUBLE_SLAB` | `125` | `Common/base/define/BlockID.lua:129` |
| `WOOD_SINGLE_SLAB` | `126` | `Common/base/define/BlockID.lua:130` |
| `WOOL_CARPET` | `171` | `Common/base/define/BlockID.lua:168` |
| `WORK_BENCH` | `58` | `Common/base/define/BlockID.lua:62` |

### ExpRule

| Member | Value | Location |
| --- | --- | --- |
| `DayMaxExp` | `1000` | `Common/base/exp/ExpRule.lua:9` |
| `Enable` | `true` | `Common/base/exp/ExpRule.lua:7` |
| `ExpCoefficient` | `1` | `Common/base/exp/ExpRule.lua:10` |
| `MaxLv` | `0` | `Common/base/exp/ExpRule.lua:8` |

### GameOverCode

| Member | Value | Location |
| --- | --- | --- |
| `GameOver` | `1` | `Common/base/code/GameOverCode.lua:8` |
| `GameStarted` | `2` | `Common/base/code/GameOverCode.lua:9` |
| `LoadDBDataError` | `6` | `Common/base/code/GameOverCode.lua:13` |
| `NoThisTeam` | `5` | `Common/base/code/GameOverCode.lua:12` |
| `PlayerIsEnough` | `3` | `Common/base/code/GameOverCode.lua:10` |
| `ReadyNoPlayer` | `4` | `Common/base/code/GameOverCode.lua:11` |
| `Success` | `0` | `Common/base/code/GameOverCode.lua:7` |

### IMessages

| Member | Value | Location |
| --- | --- | --- |
| `TYPE_PLAYER` | `0` | `Common/base/messages/IMessages.lua:13` |
| `TYPE_TEAM` | `1` | `Common/base/messages/IMessages.lua:14` |
| `UNIT_MINUTE` | `0` | `Common/base/messages/IMessages.lua:10` |
| `UNIT_SECOND` | `1` | `Common/base/messages/IMessages.lua:11` |

### ITask

| Member | Value | Location |
| --- | --- | --- |
| `created` | `false` | `Common/base/task/ITask.lua:9` |
| `status` | `0` | `Common/base/task/ITask.lua:8` |
| `timer` | `0` | `Common/base/task/ITask.lua:7` |

### ItemID

| Member | Value | Location |
| --- | --- | --- |
| `APPLE_GOLD` | `322` | `Common/base/define/ItemID.lua:73` |
| `APPLE_RED` | `260` | `Common/base/define/ItemID.lua:11` |
| `ARROW` | `262` | `Common/base/define/ItemID.lua:13` |
| `AXE_DIAMOND` | `279` | `Common/base/define/ItemID.lua:30` |
| `AXE_GOLD` | `286` | `Common/base/define/ItemID.lua:37` |
| `AXE_IRON` | `258` | `Common/base/define/ItemID.lua:9` |
| `AXE_STONE` | `275` | `Common/base/define/ItemID.lua:26` |
| `AXE_WOOD` | `271` | `Common/base/define/ItemID.lua:22` |
| `BAKED_POTATO` | `393` | `Common/base/define/ItemID.lua:144` |
| `BED` | `355` | `Common/base/define/ItemID.lua:106` |
| `BEEF_COOKED` | `364` | `Common/base/define/ItemID.lua:115` |
| `BEEF_RAW` | `363` | `Common/base/define/ItemID.lua:114` |
| `BLAZE_POWDER` | `377` | `Common/base/define/ItemID.lua:128` |
| `BLAZE_ROD` | `369` | `Common/base/define/ItemID.lua:120` |
| `BLOCKMAN_CRATE` | `342` | `Common/base/define/ItemID.lua:93` |
| `BLOCKMAN_EMPTY` | `328` | `Common/base/define/ItemID.lua:79` |
| `BLOCKMAN_HOPPER` | `408` | `Common/base/define/ItemID.lua:159` |
| `BLOCKMAN_POWERED` | `343` | `Common/base/define/ItemID.lua:94` |
| `BLOCKMAN_TNT` | `407` | `Common/base/define/ItemID.lua:158` |
| `BOAT` | `333` | `Common/base/define/ItemID.lua:84` |
| `BONE` | `352` | `Common/base/define/ItemID.lua:103` |
| `BOOK` | `340` | `Common/base/define/ItemID.lua:91` |
| `BOOTS_CHAIN` | `305` | `Common/base/define/ItemID.lua:56` |
| `BOOTS_DIAMOND` | `313` | `Common/base/define/ItemID.lua:64` |
| `BOOTS_GOLD` | `317` | `Common/base/define/ItemID.lua:68` |
| `BOOTS_IRON` | `309` | `Common/base/define/ItemID.lua:60` |
| `BOOTS_LEATHER` | `301` | `Common/base/define/ItemID.lua:52` |
| `BOW` | `261` | `Common/base/define/ItemID.lua:12` |
| `BOWL_EMPTY` | `281` | `Common/base/define/ItemID.lua:32` |
| `BOWL_SOUP` | `282` | `Common/base/define/ItemID.lua:33` |
| `BREAD` | `297` | `Common/base/define/ItemID.lua:48` |
| `BREWING_STAND` | `379` | `Common/base/define/ItemID.lua:130` |
| `BRICK` | `336` | `Common/base/define/ItemID.lua:87` |
| `BUCKET_EMPTY` | `325` | `Common/base/define/ItemID.lua:76` |
| `BUCKET_LAVA` | `327` | `Common/base/define/ItemID.lua:78` |
| `BUCKET_MILK` | `335` | `Common/base/define/ItemID.lua:86` |
| `BUCKET_WATER` | `326` | `Common/base/define/ItemID.lua:77` |
| `CAKE` | `354` | `Common/base/define/ItemID.lua:105` |
| `CARROT` | `391` | `Common/base/define/ItemID.lua:142` |
| `CARROT_ON_STICK` | `398` | `Common/base/define/ItemID.lua:149` |
| `CAULDRON` | `380` | `Common/base/define/ItemID.lua:131` |
| `CHICKEN_COOKED` | `366` | `Common/base/define/ItemID.lua:117` |
| `CHICKEN_RAW` | `365` | `Common/base/define/ItemID.lua:116` |
| `CLAY` | `337` | `Common/base/define/ItemID.lua:88` |
| `COAL` | `263` | `Common/base/define/ItemID.lua:14` |
| `COMPARATOR` | `404` | `Common/base/define/ItemID.lua:155` |
| `COMPASS` | `345` | `Common/base/define/ItemID.lua:96` |
| `COOKIE` | `357` | `Common/base/define/ItemID.lua:108` |
| `DIAMOND` | `264` | `Common/base/define/ItemID.lua:15` |
| `DIAMOND_HORSE_ARMOR` | `419` | `Common/base/define/ItemID.lua:165` |
| `DOOR_IRON` | `330` | `Common/base/define/ItemID.lua:81` |
| `DOOR_WOOD` | `324` | `Common/base/define/ItemID.lua:75` |
| `DYE_POWDER` | `351` | `Common/base/define/ItemID.lua:102` |
| `EGG` | `344` | `Common/base/define/ItemID.lua:95` |
| `EMERALD` | `388` | `Common/base/define/ItemID.lua:139` |
| `EMPTY_MAP` | `395` | `Common/base/define/ItemID.lua:146` |
| `ENCHANTED_BOOK` | `403` | `Common/base/define/ItemID.lua:154` |
| `ENDER_PEARL` | `368` | `Common/base/define/ItemID.lua:119` |
| `EXP_BOTTLE` | `384` | `Common/base/define/ItemID.lua:135` |
| `EYE_OF_ENDER` | `381` | `Common/base/define/ItemID.lua:132` |
| `FEATHER` | `288` | `Common/base/define/ItemID.lua:39` |
| `FERMENTED_SPIDER_EYE` | `376` | `Common/base/define/ItemID.lua:127` |
| `FIREBALL_CHARGE` | `385` | `Common/base/define/ItemID.lua:136` |
| `FIREWORK` | `401` | `Common/base/define/ItemID.lua:152` |
| `FIREWORK_CHARGE` | `402` | `Common/base/define/ItemID.lua:153` |
| `FISHING_ROD` | `346` | `Common/base/define/ItemID.lua:97` |
| `FISH_COOKED` | `350` | `Common/base/define/ItemID.lua:101` |
| `FISH_RAW` | `349` | `Common/base/define/ItemID.lua:100` |
| `FLINT` | `318` | `Common/base/define/ItemID.lua:69` |
| `FLINT_AND_STEEL` | `259` | `Common/base/define/ItemID.lua:10` |
| `FLOWER_POT` | `390` | `Common/base/define/ItemID.lua:141` |
| `FROZEN_BALL` | `770` | `Common/base/define/ItemID.lua:184` |
| `GHAST_TEAR` | `370` | `Common/base/define/ItemID.lua:121` |
| `GLASS_BOTTLE` | `374` | `Common/base/define/ItemID.lua:125` |
| `GLOW_STONE` | `348` | `Common/base/define/ItemID.lua:99` |
| `GOLDEN_CARROT` | `396` | `Common/base/define/ItemID.lua:147` |
| `GOLDEN_HORSE_ARMOR` | `418` | `Common/base/define/ItemID.lua:164` |
| `GOLD_ARROW` | `500` | `Common/base/define/ItemID.lua:175` |
| `GOLD_HEART` | `498` | `Common/base/define/ItemID.lua:173` |
| `GOLD_NUGGET` | `371` | `Common/base/define/ItemID.lua:122` |
| `GOLD_SHOES` | `499` | `Common/base/define/ItemID.lua:174` |
| `GRENADE` | `425` | `Common/base/define/ItemID.lua:168` |
| `GRENADE_TREMOR` | `769` | `Common/base/define/ItemID.lua:183` |
| `GUN_POWDER` | `289` | `Common/base/define/ItemID.lua:40` |
| `HAND_CUFFS` | `451` | `Common/base/define/ItemID.lua:170` |
| `HELMET_CHAIN` | `302` | `Common/base/define/ItemID.lua:53` |
| `HELMET_DIAMOND` | `310` | `Common/base/define/ItemID.lua:61` |
| `HELMET_GOLD` | `314` | `Common/base/define/ItemID.lua:65` |
| `HELMET_IRON` | `306` | `Common/base/define/ItemID.lua:57` |
| `HELMET_LEATHER` | `298` | `Common/base/define/ItemID.lua:49` |
| `HOE_DIAMOND` | `293` | `Common/base/define/ItemID.lua:44` |
| `HOE_GOLD` | `294` | `Common/base/define/ItemID.lua:45` |
| `HOE_IRON` | `292` | `Common/base/define/ItemID.lua:43` |
| `HOE_STONE` | `291` | `Common/base/define/ItemID.lua:42` |
| `HOE_WOOD` | `290` | `Common/base/define/ItemID.lua:41` |
| `IMAP` | `358` | `Common/base/define/ItemID.lua:109` |
| `INGOT_GOLD` | `266` | `Common/base/define/ItemID.lua:17` |
| `INGOT_IRON` | `265` | `Common/base/define/ItemID.lua:16` |
| `IRON_HORSE_ARMOR` | `417` | `Common/base/define/ItemID.lua:163` |
| `ITEM_FRAME` | `389` | `Common/base/define/ItemID.lua:140` |
| `LEAD` | `420` | `Common/base/define/ItemID.lua:166` |
| `LEATHER` | `334` | `Common/base/define/ItemID.lua:85` |
| `LEGS_CHAIN` | `304` | `Common/base/define/ItemID.lua:55` |
| `LEGS_DIAMOND` | `312` | `Common/base/define/ItemID.lua:63` |
| `LEGS_GOLD` | `316` | `Common/base/define/ItemID.lua:67` |
| `LEGS_IRON` | `308` | `Common/base/define/ItemID.lua:59` |
| `LEGS_LEATHER` | `300` | `Common/base/define/ItemID.lua:51` |
| `MAGMA_CREAM` | `378` | `Common/base/define/ItemID.lua:129` |
| `MELON` | `360` | `Common/base/define/ItemID.lua:111` |
| `MELON_SEEDS` | `362` | `Common/base/define/ItemID.lua:113` |
| `MONSTER_PLACER` | `383` | `Common/base/define/ItemID.lua:134` |
| `NAME_TAG` | `421` | `Common/base/define/ItemID.lua:167` |
| `NETHERRACK_BRICK` | `405` | `Common/base/define/ItemID.lua:156` |
| `NETHER_QUARTZ` | `406` | `Common/base/define/ItemID.lua:157` |
| `NETHER_STALK_SEEDS` | `372` | `Common/base/define/ItemID.lua:123` |
| `NETHER_STAR` | `399` | `Common/base/define/ItemID.lua:150` |
| `NULL` | `0` | `Common/base/define/ItemID.lua:6` |
| `PAINTING` | `321` | `Common/base/define/ItemID.lua:72` |
| `PAPER` | `339` | `Common/base/define/ItemID.lua:90` |
| `PICKAXE_DIAMOND` | `278` | `Common/base/define/ItemID.lua:29` |
| `PICKAXE_IRON` | `257` | `Common/base/define/ItemID.lua:8` |
| `PICKAXE_STONE` | `274` | `Common/base/define/ItemID.lua:25` |
| `PICKAXE_WOOD` | `270` | `Common/base/define/ItemID.lua:21` |
| `PICK_AXE_GOLD` | `285` | `Common/base/define/ItemID.lua:36` |
| `PLATE_CHAIN` | `303` | `Common/base/define/ItemID.lua:54` |
| `PLATE_DIAMOND` | `311` | `Common/base/define/ItemID.lua:62` |
| `PLATE_GOLD` | `315` | `Common/base/define/ItemID.lua:66` |
| `PLATE_IRON` | `307` | `Common/base/define/ItemID.lua:58` |
| `PLATE_LEATHER` | `299` | `Common/base/define/ItemID.lua:50` |
| `POCKET_SUNDIAL` | `347` | `Common/base/define/ItemID.lua:98` |
| `POISONOUS_POTATO` | `394` | `Common/base/define/ItemID.lua:145` |
| `PORK_COOKED` | `320` | `Common/base/define/ItemID.lua:71` |
| `PORK_RAW` | `319` | `Common/base/define/ItemID.lua:70` |
| `POTATO` | `392` | `Common/base/define/ItemID.lua:143` |
| `POTION` | `373` | `Common/base/define/ItemID.lua:124` |
| `PROP_EXP` | `503` | `Common/base/define/ItemID.lua:178` |
| `PROP_GOLD` | `502` | `Common/base/define/ItemID.lua:177` |
| `PROP_IGNORE` | `505` | `Common/base/define/ItemID.lua:181` |
| `PROP_RSTEP` | `504` | `Common/base/define/ItemID.lua:180` |
| `PUMPKIN_PIE` | `400` | `Common/base/define/ItemID.lua:151` |
| `PUMPKIN_SEEDS` | `361` | `Common/base/define/ItemID.lua:112` |
| `QUESTION_MARK` | `501` | `Common/base/define/ItemID.lua:176` |
| `RABBIT_COOKED` | `412` | `Common/base/define/ItemID.lua:161` |
| `RED_STONE` | `331` | `Common/base/define/ItemID.lua:82` |
| `RED_STONE_REPEATER` | `356` | `Common/base/define/ItemID.lua:107` |
| `REED` | `338` | `Common/base/define/ItemID.lua:89` |
| `ROTTEN_FLESH` | `367` | `Common/base/define/ItemID.lua:118` |
| `SADDLE` | `329` | `Common/base/define/ItemID.lua:80` |
| `SEEDS` | `295` | `Common/base/define/ItemID.lua:46` |
| `SHEARS` | `359` | `Common/base/define/ItemID.lua:110` |
| `SHOVEL_DIAMOND` | `277` | `Common/base/define/ItemID.lua:28` |
| `SHOVEL_GOLD` | `284` | `Common/base/define/ItemID.lua:35` |
| `SHOVEL_IRON` | `256` | `Common/base/define/ItemID.lua:7` |
| `SHOVEL_STONE` | `273` | `Common/base/define/ItemID.lua:24` |
| `SHOVEL_WOOD` | `269` | `Common/base/define/ItemID.lua:20` |
| `SIGN` | `323` | `Common/base/define/ItemID.lua:74` |
| `SILK` | `287` | `Common/base/define/ItemID.lua:38` |
| `SKULL` | `397` | `Common/base/define/ItemID.lua:148` |
| `SLIME_BALL` | `341` | `Common/base/define/ItemID.lua:92` |
| `SNOWBALL` | `332` | `Common/base/define/ItemID.lua:83` |
| `SPECKLED_MELON` | `382` | `Common/base/define/ItemID.lua:133` |
| `SPIDER_EYE` | `375` | `Common/base/define/ItemID.lua:126` |
| `STICK` | `280` | `Common/base/define/ItemID.lua:31` |
| `SUGAR` | `353` | `Common/base/define/ItemID.lua:104` |
| `SWORD_DIAMOND` | `276` | `Common/base/define/ItemID.lua:27` |
| `SWORD_GOLD` | `283` | `Common/base/define/ItemID.lua:34` |
| `SWORD_IRON` | `267` | `Common/base/define/ItemID.lua:18` |
| `SWORD_STONE` | `272` | `Common/base/define/ItemID.lua:23` |
| `SWORD_WOOD` | `268` | `Common/base/define/ItemID.lua:19` |
| `TELEPORTHOME` | `462` | `Common/base/define/ItemID.lua:171` |
| `WHEAT` | `296` | `Common/base/define/ItemID.lua:47` |
| `WRITABLE_BOOK` | `386` | `Common/base/define/ItemID.lua:137` |
| `WRITTEN_BOOK` | `387` | `Common/base/define/ItemID.lua:138` |

### LuaTimer

| Member | Value | Location |
| --- | --- | --- |
| `CurTaskKey` | `0` | `Common/base/task/LuaTimer.lua:15` |
| `CurTaskKey` | `LuaTimer.CurTaskKey + 1` | `Common/base/task/LuaTimer.lua:100` |
| `CurTaskKey` | `LuaTimer.CurTaskKey + 1` | `Common/base/task/LuaTimer.lua:119` |

### MailType

| Member | Value | Location |
| --- | --- | --- |
| `BeInvited` | `3     -- 被邀请人(接受邀请的人)奖励邮件` | `Common/base/code/MailType.lua:5` |
| `EveryDayBeSend` | `5     -- 每日互赠 被送者(接受的人)奖励邮件` | `Common/base/code/MailType.lua:7` |
| `EveryDaySend` | `4     -- 每日互赠 发送者(发送的人)奖励邮件` | `Common/base/code/MailType.lua:6` |
| `Explore` | `1     -- 探险奖励邮件` | `Common/base/code/MailType.lua:3` |
| `Invite` | `2     -- 邀请人(发送邀请的人)奖励邮件` | `Common/base/code/MailType.lua:4` |
| `LevelUp` | `7     -- 玩家等级提升奖励` | `Common/base/code/MailType.lua:9` |
| `Task` | `6     -- 任务奖励邮件` | `Common/base/code/MailType.lua:8` |

### PotionID

| Member | Value | Location |
| --- | --- | --- |
| `ABSORPTION` | `22` | `Common/base/define/PotionID.lua:28` |
| `BLINDNESS` | `15` | `Common/base/define/PotionID.lua:21` |
| `CONFUSION` | `9` | `Common/base/define/PotionID.lua:15` |
| `DAMAGE_BOOST` | `5` | `Common/base/define/PotionID.lua:11` |
| `DIG_SLOWDOWN` | `4` | `Common/base/define/PotionID.lua:10` |
| `DIG_SPEED` | `3` | `Common/base/define/PotionID.lua:9` |
| `FIRE_REGISTANCE` | `12` | `Common/base/define/PotionID.lua:18` |
| `HARM` | `7` | `Common/base/define/PotionID.lua:13` |
| `HEAL` | `6` | `Common/base/define/PotionID.lua:12` |
| `HEALTH_BOOST` | `21` | `Common/base/define/PotionID.lua:27` |
| `HUNGER` | `17` | `Common/base/define/PotionID.lua:23` |
| `INVISIBILITY` | `14` | `Common/base/define/PotionID.lua:20` |
| `JUMP` | `8` | `Common/base/define/PotionID.lua:14` |
| `MOVE_SLOWDOWN` | `2` | `Common/base/define/PotionID.lua:8` |
| `MOVE_SPEED` | `1` | `Common/base/define/PotionID.lua:7` |
| `NIGHT_VISION` | `16` | `Common/base/define/PotionID.lua:22` |
| `POISON` | `19` | `Common/base/define/PotionID.lua:25` |
| `REGENERATION` | `10` | `Common/base/define/PotionID.lua:16` |
| `RESISTANCE` | `11` | `Common/base/define/PotionID.lua:17` |
| `SATURATION` | `23` | `Common/base/define/PotionID.lua:29` |
| `WATER_BREATHING` | `13` | `Common/base/define/PotionID.lua:19` |
| `WEAKNESS` | `18` | `Common/base/define/PotionID.lua:24` |
| `WITHER` | `20` | `Common/base/define/PotionID.lua:26` |

### PotionItemID

| Member | Value | Location |
| --- | --- | --- |
| `POTION_HEAL` | `517` | `Common/base/define/PotionItemID.lua:15` |
| `POTION_INVISIBILITY` | `512` | `Common/base/define/PotionItemID.lua:10` |
| `POTION_JUMP` | `514` | `Common/base/define/PotionItemID.lua:12` |
| `POTION_LONG_INVISIBILITY` | `513` | `Common/base/define/PotionItemID.lua:11` |
| `POTION_LONG_JUMP` | `515` | `Common/base/define/PotionItemID.lua:13` |
| `POTION_LONG_MOVE_SPEED` | `520` | `Common/base/define/PotionItemID.lua:18` |
| `POTION_LONG_NIGHT_VISION` | `526` | `Common/base/define/PotionItemID.lua:24` |
| `POTION_LONG_POISON` | `523` | `Common/base/define/PotionItemID.lua:21` |
| `POTION_MEDICINE_PACK` | `483` | `Common/base/define/PotionItemID.lua:8` |
| `POTION_MEDICINE_POTION` | `484` | `Common/base/define/PotionItemID.lua:9` |
| `POTION_MOVE_SPEED` | `519` | `Common/base/define/PotionItemID.lua:17` |
| `POTION_NIGHT_VISION` | `525` | `Common/base/define/PotionItemID.lua:23` |
| `POTION_POISON` | `522` | `Common/base/define/PotionItemID.lua:20` |
| `POTION_SPLASH_HEAL` | `527` | `Common/base/define/PotionItemID.lua:25` |
| `POTION_SPLASH_LONG_POISON` | `530` | `Common/base/define/PotionItemID.lua:28` |
| `POTION_SPLASH_POISON` | `529` | `Common/base/define/PotionItemID.lua:27` |
| `POTION_SPLASH_STRONG_HEAL` | `528` | `Common/base/define/PotionItemID.lua:26` |
| `POTION_SPLASH_STRONG_POISON` | `531` | `Common/base/define/PotionItemID.lua:29` |
| `POTION_STRONG_HEAL` | `518` | `Common/base/define/PotionItemID.lua:16` |
| `POTION_STRONG_JUMP` | `516` | `Common/base/define/PotionItemID.lua:14` |
| `POTION_STRONG_MOVE_SPEED` | `521` | `Common/base/define/PotionItemID.lua:19` |
| `POTION_STRONG_POISON` | `524` | `Common/base/define/PotionItemID.lua:22` |

### ReportManager

| Member | Value | Location |
| --- | --- | --- |
| `GameRankType` | `ReportManager.RankType.Inc` | `Common/base/ReportManager.lua:8` |
| `GameType` | `"g1001"` | `Common/base/ReportManager.lua:9` |
| `IsCloseServer` | `false` | `Common/base/ReportManager.lua:11` |
| `RankType` | `{ Inc = 0, Max = 1 }` | `Common/base/ReportManager.lua:7` |

### RewardManager

| Member | Value | Location |
| --- | --- | --- |
| `GameType` | `"g1001"` | `Common/base/RewardManager.lua:7` |
| `StandardNum` | `10` | `Common/base/RewardManager.lua:8` |
| `StartPlayerCount` | `nil` | `Common/base/RewardManager.lua:9` |
| `StartPlayerTime` | `nil` | `Common/base/RewardManager.lua:10` |

### ScoreID

| Member | Value | Location |
| --- | --- | --- |
| `DESTROY_BED` | `"5" -- 破坏床` | `Common/base/define/ScoreID.lua:11` |
| `DIE` | `"2" -- 死亡` | `Common/base/define/ScoreID.lua:8` |
| `FLAG` | `"6" -- 夺旗` | `Common/base/define/ScoreID.lua:12` |
| `KILL` | `"1"  -- 击杀` | `Common/base/define/ScoreID.lua:7` |
| `LIVE` | `"3" -- 生存` | `Common/base/define/ScoreID.lua:9` |
| `POINT` | `"7" -- 积分` | `Common/base/define/ScoreID.lua:13` |
| `SERIAL_KILL` | `"4" -- 连续击杀` | `Common/base/define/ScoreID.lua:10` |
| `WIN` | `"8" -- 胜局数` | `Common/base/define/ScoreID.lua:14` |

### TextFormat

| Member | Value | Location |
| --- | --- | --- |
| `argsSplit` | `"/t"` | `Common/base/messages/TextFormat.lua:8` |
| `argsTipType` | `"#"` | `Common/base/messages/TextFormat.lua:9` |
| `colorAqua` | `TextFormat.colorStart .. "FF00FFFF"` | `Common/base/messages/TextFormat.lua:23` |
| `colorBlack` | `TextFormat.colorStart .. "FF000000"` | `Common/base/messages/TextFormat.lua:16` |
| `colorBlue` | `TextFormat.colorStart .. "FF0000FF"` | `Common/base/messages/TextFormat.lua:15` |
| `colorEnd` | `"▥"` | `Common/base/messages/TextFormat.lua:12` |
| `colorGold` | `TextFormat.colorStart .. "FFFFD700"` | `Common/base/messages/TextFormat.lua:22` |
| `colorGreen` | `TextFormat.colorStart .. "FF00FF00"` | `Common/base/messages/TextFormat.lua:18` |
| `colorOrange` | `TextFormat.colorStart .. "FFFF8000"` | `Common/base/messages/TextFormat.lua:21` |
| `colorPurple` | `TextFormat.colorStart .. "FF884898"` | `Common/base/messages/TextFormat.lua:20` |
| `colorRed` | `TextFormat.colorStart .. "FFFF0000"` | `Common/base/messages/TextFormat.lua:14` |
| `colorStart` | `"▢"` | `Common/base/messages/TextFormat.lua:11` |
| `colorWrite` | `TextFormat.colorStart .. "FFFFFFFF"` | `Common/base/messages/TextFormat.lua:17` |
| `colorYellow` | `TextFormat.colorStart .. "FFFFFF00"` | `Common/base/messages/TextFormat.lua:19` |

### WebRequestType

| Member | Value | Location |
| --- | --- | --- |
| `BLOCKYMODS_EXP_RULE` | `"blockymods.exp.rule"` | `Common/base/web/WebRequestType.lua:18` |
| `BLOCKYMODS_GOLD_REWARD` | `"blockymods.user.gold.reward:"` | `Common/base/web/WebRequestType.lua:13` |
| `BLOCKYMODS_REPORT` | `"blockymods.user.report:"` | `Common/base/web/WebRequestType.lua:14` |
| `BLOCKYMODS_REPORT_LIST` | `"blockymods.report.list"` | `Common/base/web/WebRequestType.lua:17` |
| `BLOCKYMODS_REWARD` | `"blockymods.user.reward:"` | `Common/base/web/WebRequestType.lua:12` |
| `BLOCKYMODS_REWARD_LIST` | `"blockymods.reward.list"` | `Common/base/web/WebRequestType.lua:16` |
| `BLOCKYMODS_SAVE_EXP` | `"blockymods.save.exp"` | `Common/base/web/WebRequestType.lua:20` |
| `BLOCKYMODS_SAVE_HONOR` | `"blockymods.save.honor"` | `Common/base/web/WebRequestType.lua:21` |
| `BLOCKYMODS_USER_ATTR` | `"blockymods.user.attr:"` | `Common/base/web/WebRequestType.lua:11` |
| `BLOCKYMODS_USER_EXP` | `"blockymods.user.exp:"` | `Common/base/web/WebRequestType.lua:19` |
| `BLOCKYMODS_USER_INFOS` | `"blockymods.user.infos:"` | `Common/base/web/WebRequestType.lua:15` |
| `BLOCKYMODS_USER_SEASON_INFO` | `"blockymods.user.season.info:"` | `Common/base/web/WebRequestType.lua:22` |
| `BLOCKYMODS_USER_SEASON_REWARD` | `"blockymods.user.season.reward:"` | `Common/base/web/WebRequestType.lua:23` |
| `CHECK_MAILS` | `"blockymods.checkMails:"` | `Common/base/web/WebRequestType.lua:26` |
| `EMAIL_STATUS_DELETED` | `"DELETED"` | `Common/base/web/WebRequestType.lua:35` |
| `EMAIL_STATUS_READ` | `"READ"` | `Common/base/web/WebRequestType.lua:33` |
| `EMAIL_STATUS_RECEIVED` | `"RECEIVED"` | `Common/base/web/WebRequestType.lua:34` |
| `EMAIL_STATUS_UNREAD` | `"UNREAD"` | `Common/base/web/WebRequestType.lua:32` |
| `MCPEONLINE_REPORT` | `"mcpeonline.report"` | `Common/base/web/WebRequestType.lua:9` |
| `MCPEONLINE_REWARD` | `"mcpeonline.user.reward:"` | `Common/base/web/WebRequestType.lua:8` |
| `MCPEONLINE_USER_ATTR` | `"mcpeonline.user.attr:"` | `Common/base/web/WebRequestType.lua:7` |
| `SEND_MAILS` | `"blockymods.sendMails:"` | `Common/base/web/WebRequestType.lua:27` |
| `SEND_MSG` | `"blockymods.sendMsg:"` | `Common/base/web/WebRequestType.lua:29` |
| `UPDATE_MAILS` | `"blockymods.updateMails:"` | `Common/base/web/WebRequestType.lua:28` |

### WebService

| Member | Value | Location |
| --- | --- | --- |
| `BlockymodBaseUrl` | `nil` | `Common/base/web/WebService.lua:10` |
| `BlockymodsExpRule` | `"/activity/api/v1/inner/activity/games/settlement/rule"` | `Common/base/web/WebService.lua:24` |
| `BlockymodsReportData` | `"/game/api/v1/inner/users/games/reporting"` | `Common/base/web/WebService.lua:22` |
| `BlockymodsReportList` | `"/game/api/v1/inner/users/games/reporting/list"` | `Common/base/web/WebService.lua:23` |
| `BlockymodsRewardAPI` | `"/game/api/v1/users/games/settlement"` | `Common/base/web/WebService.lua:19` |
| `BlockymodsRewardGoldAPI` | `"/game/api/v1/users/games/settlement/golds"` | `Common/base/web/WebService.lua:20` |
| `BlockymodsRewardList` | `"/game/api/v1/users/inner/games/settlement/list"` | `Common/base/web/WebService.lua:21` |
| `BlockymodsSaveExp` | `"/activity/api/v1/inner/activity/games/records"` | `Common/base/web/WebService.lua:26` |
| `BlockymodsSaveHonor` | `"/gameaide/api/v1/inner/user/segment/integral"` | `Common/base/web/WebService.lua:27` |
| `BlockymodsUserAttr` | `"/user/api/v1/inner/user/details"` | `Common/base/web/WebService.lua:18` |
| `BlockymodsUserExp` | `"/activity/api/v1/inner/activity/games/user/exp"` | `Common/base/web/WebService.lua:25` |
| `BlockymodsUserInfos` | `"/user/api/v1/inner/simple-info"` | `Common/base/web/WebService.lua:17` |
| `BlockymodsUserSeasonInfo` | `"/gameaide/api/v1/inner/user/segment/info"` | `Common/base/web/WebService.lua:28` |
| `BlockymodsUserSeasonReward` | `"/gameaide/api/v1/inner/segment/integral/reward"` | `Common/base/web/WebService.lua:29` |
| `ConsumeAPI` | `"/props/api/{version}/consume/{userid}"` | `Common/base/web/WebService.lua:15` |
| `MailsApi` | `"/gameaide/api/v1/inner/game/mails"` | `Common/base/web/WebService.lua:31` |
| `MailsStatusApi` | `"/gameaide/api/v1/inner/game/mails/{id}/status"` | `Common/base/web/WebService.lua:32` |
| `PropAddr` | `nil` | `Common/base/web/WebService.lua:9` |
| `RankAPI` | `"/minigame/api/{version}/ranks/{gametype}"` | `Common/base/web/WebService.lua:14` |
| `RankAddr` | `nil` | `Common/base/web/WebService.lua:7` |
| `RewardAPI` | `"/minigame/api/{version}/rewards/{gametype}/{userId}"` | `Common/base/web/WebService.lua:13` |
| `RewardAddr` | `nil` | `Common/base/web/WebService.lua:8` |
| `SendMsgApi` | `"/gameaide/api/v1/inner/msg/send"` | `Common/base/web/WebService.lua:33` |
| `UserAttrPath` | `"/minigame/i/api/{version}/user-details/{userId}"` | `Common/base/web/WebService.lua:12` |
| `ver` | `nil` | `Common/base/web/WebService.lua:6` |

### cache

| Member | Value | Location |
| --- | --- | --- |
| `integral` | `cache.integral + (integral or 0)` | `Common/base/ReportManager.lua:112` |
| `isCount` | `1` | `Common/base/ReportManager.lua:104` |
| `kill` | `cache.kill + (kill or 0)` | `Common/base/ReportManager.lua:80` |
| `meanTime` | `cache.meanTime + (time or 0)` | `Common/base/ReportManager.lua:88` |
| `rank` | `rank or 0` | `Common/base/ReportManager.lua:96` |
| `type` | `self.GameRankType` | `Common/base/ReportManager.lua:113` |

### cls

| Member | Value | Location |
| --- | --- | --- |
| `__cname` | `classname` | `Common/base/util/class.lua:44` |
| `__cname` | `classname` | `Common/base/util/class.lua:65` |
| `__create` | `super.__create` | `Common/base/util/class.lua:37` |
| `__create` | `super` | `Common/base/util/class.lua:40` |
| `__ctype` | `1` | `Common/base/util/class.lua:45` |
| `__ctype` | `2 -- lua` | `Common/base/util/class.lua:66` |
| `__index` | `cls` | `Common/base/util/class.lua:67` |
| `__index` | `cls` | `Common/base/util/tinyyaml.lua:64` |
| `__meta` | `mt` | `Common/base/util/tinyyaml.lua:67` |
| `__metatable` | `cls` | `Common/base/util/tinyyaml.lua:65` |
| `__type` | `type` | `Common/base/util/tinyyaml.lua:66` |
| `super` | `super` | `Common/base/util/class.lua:38` |
| `super` | `super` | `Common/base/util/class.lua:60` |

### data

| Member | Value | Location |
| --- | --- | --- |
| `ActorName` | `actor or ""` | `Common/base/data/EngineWorld.lua:82` |
| `EffectName` | `effect or ""` | `Common/base/data/EngineWorld.lua:85` |
| `HeadName` | `name or ""` | `Common/base/data/EngineWorld.lua:83` |
| `IsCanObstruct` | `true` | `Common/base/data/EngineWorld.lua:87` |
| `IsCanObstruct` | `isCanObstruct` | `Common/base/data/EngineWorld.lua:89` |
| `IsCollided` | `true` | `Common/base/data/EngineWorld.lua:92` |
| `IsCollided` | `isCollided` | `Common/base/data/EngineWorld.lua:94` |
| `SkillName` | `skill or ""` | `Common/base/data/EngineWorld.lua:84` |
| `armsId` | `armsId` | `Common/base/util/MsgSender.lua:105` |
| `color` | `color` | `Common/base/util/MsgSender.lua:108` |
| `dead` | `dead` | `Common/base/util/MsgSender.lua:104` |
| `duration` | `os.time() - startGameTime` | `Common/base/RewardManager.lua:81` |
| `gameId` | `gameType` | `Common/base/web/WebService.lua:304` |
| `head` | `head` | `Common/base/util/MsgSender.lua:107` |
| `integral` | `integral` | `Common/base/web/WebService.lua:309` |
| `isCount` | `isCount` | `Common/base/web/WebService.lua:308` |
| `kill` | `kills` | `Common/base/web/WebService.lua:305` |
| `killer` | `killer` | `Common/base/util/MsgSender.lua:103` |
| `kills` | `kills` | `Common/base/util/MsgSender.lua:106` |
| `meanTime` | `time` | `Common/base/web/WebService.lua:306` |
| `name` | `player.name` | `Common/base/rank/RankManager.lua:49` |
| `name` | `player.name` | `Common/base/rank/RankManager.lua:67` |
| `own` | `own` | `Common/base/rank/RankManager.lua:86` |
| `rank` | `rank or 10` | `Common/base/RewardManager.lua:82` |
| `rank` | `rank` | `Common/base/rank/RankManager.lua:46` |
| `rank` | `rank` | `Common/base/rank/RankManager.lua:64` |
| `rank` | `rank` | `Common/base/web/WebService.lua:307` |
| `recordId` | `WebService.gameId` | `Common/base/RewardManager.lua:83` |
| `score` | `tonumber(score)` | `Common/base/rank/RankManager.lua:48` |
| `score` | `tonumber(score)` | `Common/base/rank/RankManager.lua:66` |
| `standardNum` | `self.StandardNum` | `Common/base/RewardManager.lua:84` |
| `startTime` | `os.date("%Y-%m-%d %H:%M:%S", startGameTime)` | `Common/base/RewardManager.lua:86` |
| `totalNum` | `startPlayerCount` | `Common/base/RewardManager.lua:85` |
| `type` | `type` | `Common/base/web/WebService.lua:310` |
| `userId` | `tonumber(tostring(player.userId))` | `Common/base/rank/RankManager.lua:47` |
| `userId` | `tonumber(tostring(player.userId))` | `Common/base/rank/RankManager.lua:65` |
| `vip` | `player.vip` | `Common/base/rank/RankManager.lua:50` |
| `vip` | `player.vip` | `Common/base/rank/RankManager.lua:68` |

### expInfo

| Member | Value | Location |
| --- | --- | --- |
| `addExp` | `expInfo.addExp + addExp` | `Common/base/exp/UserExpManager.lua:83` |
| `addExp` | `addExp` | `Common/base/exp/UserExpManager.lua:91` |
| `hasGet` | `expInfo.hasGet + addExp` | `Common/base/exp/UserExpManager.lua:82` |
| `isAdd` | `true` | `Common/base/exp/UserExpManager.lua:85` |
| `time` | `os.time()` | `Common/base/exp/UserExpManager.lua:84` |

### item

| Member | Value | Location |
| --- | --- | --- |
| `bullets` | `info.bullets` | `Common/base/util/InventoryUtil.lua:137` |
| `bullets` | `info.bullets` | `Common/base/util/InventoryUtil.lua:167` |
| `content` | `v.content` | `Common/base/web/WebResponse.lua:218` |
| `duration` | `os.time() - startGameTime` | `Common/base/RewardManager.lua:63` |
| `effects` | `effects` | `Common/base/prop/AppProps.lua:52` |
| `enchants` | `enchants` | `Common/base/prop/AppProps.lua:39` |
| `fromUser` | `v.fromUser` | `Common/base/web/WebResponse.lua:219` |
| `gameId` | `self.GameType` | `Common/base/RewardManager.lua:60` |
| `id` | `prop.id` | `Common/base/prop/AppProps.lua:22` |
| `id` | `info.id` | `Common/base/util/InventoryUtil.lua:129` |
| `id` | `info.id` | `Common/base/util/InventoryUtil.lua:161` |
| `id` | `v.id` | `Common/base/web/WebResponse.lua:216` |
| `isRepeat` | `tonumber(prop.isRepeat or "1") == 1` | `Common/base/prop/AppProps.lua:24` |
| `itemId` | `tonumber(prop.itemId)` | `Common/base/prop/AppProps.lua:25` |
| `itemMeta` | `tonumber(prop.itemMeta)` | `Common/base/prop/AppProps.lua:26` |
| `itemNum` | `tonumber(prop.itemNum)` | `Common/base/prop/AppProps.lua:27` |
| `mailType` | `v.mailType` | `Common/base/web/WebResponse.lua:222` |
| `meta` | `info.meta` | `Common/base/util/InventoryUtil.lua:130` |
| `meta` | `0` | `Common/base/util/InventoryUtil.lua:136` |
| `meta` | `0` | `Common/base/util/InventoryUtil.lua:144` |
| `meta` | `info.meta` | `Common/base/util/InventoryUtil.lua:162` |
| `meta` | `0` | `Common/base/util/InventoryUtil.lua:166` |
| `num` | `info.num` | `Common/base/util/InventoryUtil.lua:131` |
| `num` | `1` | `Common/base/util/InventoryUtil.lua:135` |
| `num` | `1` | `Common/base/util/InventoryUtil.lua:143` |
| `num` | `info.num` | `Common/base/util/InventoryUtil.lua:163` |
| `num` | `1` | `Common/base/util/InventoryUtil.lua:165` |
| `rank` | `cache.rank or 10` | `Common/base/RewardManager.lua:62` |
| `recordId` | `WebService.gameId` | `Common/base/RewardManager.lua:61` |
| `stackIndex` | `i - 1` | `Common/base/util/InventoryUtil.lua:132` |
| `standardNum` | `self.StandardNum` | `Common/base/RewardManager.lua:64` |
| `startTime` | `os.date("%Y-%m-%d %H:%M:%S", startGameTime)` | `Common/base/RewardManager.lua:66` |
| `status` | `v.status` | `Common/base/web/WebResponse.lua:221` |
| `title` | `v.title` | `Common/base/web/WebResponse.lua:217` |
| `toUser` | `v.toUser` | `Common/base/web/WebResponse.lua:220` |
| `totalNum` | `startPlayerCount` | `Common/base/RewardManager.lua:65` |
| `type` | `tonumber(prop.type)` | `Common/base/prop/AppProps.lua:23` |
| `userId` | `cache.userId` | `Common/base/RewardManager.lua:59` |

### player

| Member | Value | Location |
| --- | --- | --- |
| `adSwitch` | `reward.adSwitch or 0` | `Common/base/web/WebResponse.lua:104` |
| `adSwitch` | `reward.adSwitch or 0` | `Common/base/web/WebResponse.lua:196` |
| `appIntegral` | `0` | `Common/base/ReportManager.lua:37` |
| `available` | `reward.available or 0` | `Common/base/web/WebResponse.lua:102` |
| `available` | `reward.available or 0` | `Common/base/web/WebResponse.lua:117` |
| `available` | `reward.available or 0` | `Common/base/web/WebResponse.lua:194` |
| `gold` | `reward.golds or 0` | `Common/base/web/WebResponse.lua:101` |
| `gold` | `reward.golds or 0` | `Common/base/web/WebResponse.lua:116` |
| `gold` | `reward.golds or 0` | `Common/base/web/WebResponse.lua:193` |
| `hasGet` | `reward.hasGet or 0` | `Common/base/web/WebResponse.lua:103` |
| `hasGet` | `reward.hasGet or 0` | `Common/base/web/WebResponse.lua:118` |
| `hasGet` | `reward.hasGet or 0` | `Common/base/web/WebResponse.lua:195` |
| `vip` | `player.staticAttr.vip` | `Common/base/web/WebResponse.lua:49` |

### player.staticAttr

| Member | Value | Location |
| --- | --- | --- |
| `attack` | `data.att` | `Common/base/web/WebResponse.lua:41` |
| `clanId` | `data.clanId` | `Common/base/web/WebResponse.lua:45` |
| `clanName` | `string.gsub(data.clanName or "", "/n", "")` | `Common/base/web/WebResponse.lua:46` |
| `clanName` | `data.clanName` | `Common/base/web/WebResponse.lua:84` |
| `classes` | `data.classes` | `Common/base/web/WebResponse.lua:47` |
| `defense` | `data.def` | `Common/base/web/WebResponse.lua:42` |
| `hasInit` | `true` | `Common/base/web/WebResponse.lua:48` |
| `hasInit` | `true` | `Common/base/web/WebResponse.lua:87` |
| `health` | `data.heal` | `Common/base/web/WebResponse.lua:43` |
| `lv` | `data.lv` | `Common/base/web/WebResponse.lua:40` |
| `props` | `data.propsId or {}` | `Common/base/web/WebResponse.lua:86` |
| `role` | `data.role` | `Common/base/web/WebResponse.lua:85` |
| `supperPlayer` | `data.spp` | `Common/base/web/WebResponse.lua:39` |
| `title` | `data.title` | `Common/base/web/WebResponse.lua:44` |
| `vip` | `data.vip` | `Common/base/web/WebResponse.lua:38` |

### self

| Member | Value | Location |
| --- | --- | --- |
| `BlockymodBaseUrl` | `config.blockymodsRewardAddr` | `Common/base/web/WebService.lua:40` |
| `DayMaxExp` | `data.maxExpPerDay` | `Common/base/exp/ExpRule.lua:25` |
| `Enable` | `data.needSettle` | `Common/base/exp/ExpRule.lua:23` |
| `Enable` | `false` | `Common/base/exp/ExpRule.lua:60` |
| `ExpCoefficient` | `data.gameCoefficient / 100` | `Common/base/exp/ExpRule.lua:27` |
| `GameRankType` | `RankType` | `Common/base/ReportManager.lua:18` |
| `GameType` | `GameType` | `Common/base/BaseMain.lua:23` |
| `GameType` | `GameType` | `Common/base/ReportManager.lua:14` |
| `GameType` | `GameType` | `Common/base/RewardManager.lua:15` |
| `IsCloseServer` | `true` | `Common/base/ReportManager.lua:29` |
| `MaxLv` | `self.MaxLv + 1` | `Common/base/exp/ExpRule.lua:29` |
| `MaxPlayer` | `maxPlayer` | `Common/base/PlayerManager.lua:16` |
| `PropAddr` | `config.propAddr` | `Common/base/web/WebService.lua:39` |
| `RankAddr` | `config.rankAddr` | `Common/base/web/WebService.lua:37` |
| `RewardAddr` | `config.rewardAddr` | `Common/base/web/WebService.lua:38` |
| `StartGameTime` | `os.time()` | `Common/base/RewardManager.lua:20` |
| `StartPlayerCount` | `PlayerManager:getPlayerCount()` | `Common/base/RewardManager.lua:19` |
| `UpgradeExpTable` | `data.upgradeRule` | `Common/base/exp/ExpRule.lua:26` |
| `appIntegral` | `0` | `Common/base/data/BasePlayer.lua:27` |
| `available` | `0` | `Common/base/data/BasePlayer.lua:20` |
| `clientPeer` | `nil` | `Common/base/data/BasePlayer.lua:14` |
| `clientPeer` | `clientPeer` | `Common/base/data/BasePlayer.lua:67` |
| `clientPeer` | `nil` | `Common/base/data/BasePlayer.lua:98` |
| `created` | `true` | `Common/base/task/ITask.lua:14` |
| `day` | `tonumber(d)` | `Common/base/util/tinyyaml.lua:99` |
| `defaultRank` | `10` | `Common/base/data/BasePlayer.lua:23` |
| `entityPlayerMP` | `nil` | `Common/base/data/BasePlayer.lua:15` |
| `entityPlayerMP` | `clientPeer:getEntityPlayer()` | `Common/base/data/BasePlayer.lua:68` |
| `entityPlayerMP` | `nil` | `Common/base/data/BasePlayer.lua:99` |
| `fraction` | `tonumber(f) * math.pow(10, 3 - #f)` | `Common/base/util/tinyyaml.lua:104` |
| `fraction` | `f` | `Common/base/util/tinyyaml.lua:106` |
| `fraction` | `0` | `Common/base/util/tinyyaml.lua:108` |
| `gameId` | `uuid()` | `Common/base/web/WebService.lua:47` |
| `gold` | `0` | `Common/base/data/BasePlayer.lua:19` |
| `hasGet` | `0` | `Common/base/data/BasePlayer.lua:21` |
| `hour` | `tonumber(h or 0)` | `Common/base/util/tinyyaml.lua:100` |
| `inGameTime` | `os.time()` | `Common/base/data/BasePlayer.lua:26` |
| `isReady` | `false` | `Common/base/data/BasePlayer.lua:25` |
| `minute` | `tonumber(i or 0)` | `Common/base/util/tinyyaml.lua:101` |
| `month` | `tonumber(m)` | `Common/base/util/tinyyaml.lua:98` |
| `name` | `nil` | `Common/base/data/BasePlayer.lua:18` |
| `name` | `clientPeer:getName()` | `Common/base/data/BasePlayer.lua:71` |
| `rakssid` | `0` | `Common/base/data/BasePlayer.lua:17` |
| `rakssid` | `clientPeer:getRakssid()` | `Common/base/data/BasePlayer.lua:70` |
| `respawnTimes` | `-1` | `Common/base/data/BasePlayer.lua:24` |
| `respawnTimes` | `self.respawnTimes + 1` | `Common/base/data/BasePlayer.lua:72` |
| `second` | `tonumber(s or 0)` | `Common/base/util/tinyyaml.lua:102` |
| `status` | `status` | `Common/base/task/ITask.lua:19` |
| `timer` | `SecTimer.createTimer(function(ticks)` | `Common/base/task/ITask.lua:12` |
| `timezone` | `z` | `Common/base/util/tinyyaml.lua:110` |
| `userId` | `0` | `Common/base/data/BasePlayer.lua:16` |
| `userId` | `clientPeer:getPlatformUserId()` | `Common/base/data/BasePlayer.lua:69` |
| `ver` | `version` | `Common/base/web/WebService.lua:36` |
| `vip` | `0` | `Common/base/data/BasePlayer.lua:53` |
| `vip` | `self.dynamicAttr.vip` | `Common/base/data/BasePlayer.lua:560` |
| `world` | `world` | `Common/base/data/EngineWorld.lua:10` |
| `year` | `tonumber(y)` | `Common/base/util/tinyyaml.lua:97` |

### self.dynamicAttr

| Member | Value | Location |
| --- | --- | --- |
| `clz` | `0` | `Common/base/data/BasePlayer.lua:47` |
| `clz` | `attr.class` | `Common/base/data/BasePlayer.lua:554` |
| `hasInit` | `false` | `Common/base/data/BasePlayer.lua:46` |
| `hasInit` | `true` | `Common/base/data/BasePlayer.lua:553` |
| `pioneer` | `true` | `Common/base/data/BasePlayer.lua:50` |
| `pioneer` | `attr.pioneer` | `Common/base/data/BasePlayer.lua:557` |
| `region` | `0` | `Common/base/data/BasePlayer.lua:48` |
| `region` | `attr.region` | `Common/base/data/BasePlayer.lua:555` |
| `team` | `0` | `Common/base/data/BasePlayer.lua:49` |
| `team` | `attr.team` | `Common/base/data/BasePlayer.lua:556` |
| `vip` | `0` | `Common/base/data/BasePlayer.lua:51` |
| `vip` | `attr.vip` | `Common/base/data/BasePlayer.lua:558` |

### self.staticAttr

| Member | Value | Location |
| --- | --- | --- |
| `attack` | `0 -- att` | `Common/base/data/BasePlayer.lua:35` |
| `clanId` | `0 -- clanId` | `Common/base/data/BasePlayer.lua:39` |
| `clanName` | `"" -- clanName` | `Common/base/data/BasePlayer.lua:40` |
| `classes` | `0 --classes` | `Common/base/data/BasePlayer.lua:42` |
| `defense` | `0 -- def` | `Common/base/data/BasePlayer.lua:36` |
| `hasInit` | `false` | `Common/base/data/BasePlayer.lua:30` |
| `health` | `0 --heal` | `Common/base/data/BasePlayer.lua:37` |
| `lv` | `0 -- lv` | `Common/base/data/BasePlayer.lua:34` |
| `props` | `{} --props` | `Common/base/data/BasePlayer.lua:43` |
| `role` | `-1 -- role` | `Common/base/data/BasePlayer.lua:41` |
| `supperPlayer` | `0 -- spp` | `Common/base/data/BasePlayer.lua:33` |
| `title` | `"" -- title` | `Common/base/data/BasePlayer.lua:38` |
| `vip` | `0 -- vip` | `Common/base/data/BasePlayer.lua:32` |

### settlement

| Member | Value | Location |
| --- | --- | --- |
| `activeValues` | `data.use_energy` | `Common/base/web/WebResponse.lua:65` |
| `activeValues` | `0` | `Common/base/web/WebResponse.lua:69` |
| `exp` | `data.gain_exp` | `Common/base/web/WebResponse.lua:63` |
| `exp` | `0` | `Common/base/web/WebResponse.lua:67` |
| `goldCoins` | `data.gain_gold` | `Common/base/web/WebResponse.lua:64` |
| `goldCoins` | `0` | `Common/base/web/WebResponse.lua:68` |
| `kills` | `0` | `Common/base/web/WebResponse.lua:61` |

### task

| Member | Value | Location |
| --- | --- | --- |
| `delay` | `-1` | `Common/base/task/LuaTimer.lua:39` |
| `delay` | `delay` | `Common/base/task/LuaTimer.lua:87` |
| `delay` | `delay` | `Common/base/task/LuaTimer.lua:97` |
| `period` | `period` | `Common/base/task/LuaTimer.lua:92` |
| `period` | `period` | `Common/base/task/LuaTimer.lua:98` |
| `time` | `time` | `Common/base/task/LuaTimer.lua:31` |
| `time` | `time` | `Common/base/task/LuaTimer.lua:38` |
| `time` | `time` | `Common/base/task/LuaTimer.lua:44` |
| `time` | `time` | `Common/base/task/LuaTimer.lua:55` |
| `time` | `time` | `Common/base/task/LuaTimer.lua:62` |
| `times` | `task.times - 1` | `Common/base/task/LuaTimer.lua:56` |
| `type` | `TaskType.Delay` | `Common/base/task/LuaTimer.lua:86` |
| `type` | `TaskType.Period` | `Common/base/task/LuaTimer.lua:91` |
| `type` | `TaskType.DelayAndPeriod` | `Common/base/task/LuaTimer.lua:96` |

### v

| Member | Value | Location |
| --- | --- | --- |
| `choose` | `false` | `Common/base/util/InventoryUtil.lua:47` |
| `choose` | `true` | `Common/base/util/InventoryUtil.lua:87` |
| `num` | `64` | `Common/base/util/InventoryUtil.lua:28` |
| `probUpper` | `total` | `Common/base/util/InventoryUtil.lua:51` |
| `probfloor` | `total` | `Common/base/util/InventoryUtil.lua:48` |

### vec3

| Member | Value | Location |
| --- | --- | --- |
| `x` | `x` | `Common/base/util/VectorUtil.lua:18` |
| `x` | `vec3i.x` | `Common/base/util/VectorUtil.lua:26` |
| `y` | `y` | `Common/base/util/VectorUtil.lua:19` |
| `y` | `vec3i.y` | `Common/base/util/VectorUtil.lua:27` |
| `z` | `z` | `Common/base/util/VectorUtil.lua:20` |
| `z` | `vec3i.z` | `Common/base/util/VectorUtil.lua:28` |

---

## 6. Module Dependencies

`require` calls per file, showing what each part of the common layer pulls in.

### `Common/base/BaseMain.lua`

```lua
require "base.ReportManager"
require "base.RewardManager"
require "base.define.BlockID"
require "base.define.ItemID"
require "base.define.PotionID"
require "base.define.PotionItemID"
require "base.define.ScoreID"
require "base.listener.BaseListener"
require "base.listener.HttpListener"
require "base.util.EngineUtil"
```

### `Common/base/PlayerManager.lua`

```lua
require "base.exp.UserExpManager"
require "base.messages.IMessages"
require "base.rank.RankManager"
require "base.season.SeasonManager"
require "base.util.MsgSender"
```

### `Common/base/data/BasePlayer.lua`

```lua
require "base.classes.PlayerClassInfo"
require "base.messages.TextFormat"
require "base.prop.AppProps"
require "base.util.json"
require "base.web.WebResponse"
require "base.web.WebService"
```

### `Common/base/exp/UserExpManager.lua`

```lua
require "base.exp.ExpRule"
```

### `Common/base/listener/BaseListener.lua`

```lua
require "base.DBManager"
require "base.data.EngineWorld"
require "base.task.CommonTask"
require "base.web.WebService"
```

### `Common/base/listener/HttpListener.lua`

```lua
require "base.web.WebResponse"
```

### `Common/base/messages/IMessages.lua`

```lua
require "base.messages.TextFormat"
require "base.util.NumberUtil"
```

### `Common/base/prop/AppProps.lua`

```lua
require "base.util.StringUtil"
```

### `Common/base/task/CommonTask.lua`

```lua
require "base.task.ITask"
require "base.task.LuaTimer"
require "base.util.class"
```

### `Common/base/task/ITask.lua`

```lua
require "base.util.class"
```

### `Common/base/util/CsvUtil.lua`

```lua
require "base.util.StringUtil"
```

### `Common/base/util/DateUtil.lua`

```lua
require "base.util.NumberUtil"
```

### `Common/base/web/WebResponse.lua`

```lua
require "base.PlayerManager"
require "base.util.json"
```

### `Common/base/web/WebService.lua`

```lua
require "base.util.json"
require "base.web.WebRequestType"
```

---

## Appendix A: File Inventory

47 file(s) in scope, 39 contribute function definitions.

| File | Functions |
| --- | ---: |
| `Common/base/BaseMain.lua` | 2 |
| `Common/base/DBManager.lua` | 6 |
| `Common/base/PlayerManager.lua` | 15 |
| `Common/base/ReportManager.lua` | 13 |
| `Common/base/RewardManager.lua` | 9 |
| `Common/base/cache/UserInfoCache.lua` | 3 |
| `Common/base/classes/PlayerClassInfo.lua` | 3 |
| `Common/base/data/BasePlayer.lua` | 87 |
| `Common/base/data/EngineWorld.lua` | 21 |
| `Common/base/exp/ExpRule.lua` | 9 |
| `Common/base/exp/UserExpManager.lua` | 7 |
| `Common/base/listener/BaseListener.lua` | 10 |
| `Common/base/listener/HttpListener.lua` | 2 |
| `Common/base/messages/IMessages.lua` | 39 |
| `Common/base/messages/TextFormat.lua` | 1 |
| `Common/base/prop/AppProps.lua` | 5 |
| `Common/base/rank/RankManager.lua` | 6 |
| `Common/base/season/SeasonManager.lua` | 2 |
| `Common/base/task/CommonTask.lua` | 1 |
| `Common/base/task/ITask.lua` | 6 |
| `Common/base/task/LuaTimer.lua` | 4 |
| `Common/base/util/CsvUtil.lua` | 2 |
| `Common/base/util/DateUtil.lua` | 8 |
| `Common/base/util/EngineUtil.lua` | 3 |
| `Common/base/util/GameLog.lua` | 1 |
| `Common/base/util/InventoryUtil.lua` | 7 |
| `Common/base/util/MerchantUtil.lua` | 2 |
| `Common/base/util/MsgSender.lua` | 12 |
| `Common/base/util/NumberUtil.lua` | 1 |
| `Common/base/util/StringUtil.lua` | 3 |
| `Common/base/util/VectorUtil.lua` | 7 |
| `Common/base/util/WalletUtils.lua` | 1 |
| `Common/base/util/class.lua` | 6 |
| `Common/base/util/json.lua` | 19 |
| `Common/base/util/tinyyaml.lua` | 24 |
| `Common/base/util/uuid.lua` | 5 |
| `Common/base/util/vardump.lua` | 2 |
| `Common/base/web/WebResponse.lua` | 25 |
| `Common/base/web/WebService.lua` | 23 |

