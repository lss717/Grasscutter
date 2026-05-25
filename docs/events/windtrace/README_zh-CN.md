# 捉迷藏！
关于**捉迷藏**游戏如何工作的文档。\
外部名称：`Windtrace`（风迹）。

# 地图 ID
TODO: 记录 Windtrace 的地图 ID。

TODO: 调查 `ServerGlobalValueChangeNotify`

# 在合作游戏中邀请玩家
1. 客户端发送 `DraftOwnerStartInviteReq`
2. 服务器向所有客户端发送 `DraftOwnerInviteNotify`
3. 服务器发送 `DraftOwnerStartInviteRsp`

# 合作游戏中的匹配
1. 世界主人（房主）与 Gygax 对话并开始 Windtrace 游戏。
2. 数据包 `DraftOwnerInviteNotify` 被发送给各客户端。
3. 客户端以 `DraftGuestReplyInviteReq` 响应（客户端侧）
4. 服务器以 `DraftGuestReplyInviteRsp` 响应
5. 服务器以 `DraftInviteResultNotify` 响应

# 启动 Windtrace
1. 如果 `DraftInviteResultNotify` 成功，服务器将发送一系列数据包。
   1. 一系列 `SceneEntityAppearNotify` 数据包。
   2. `NpcTalkStateNotify`
   3. `PlayerEnterSceneNotify`
   4. `MultistagePlayInfoNotify`
2. 随后玩家被传送至其所在位置的 Windtrace 地图。
3. 服务器向客户端发送数据包。（这是服务端样板代码）
4. 服务器再次向客户端发送 `MultistagePlayInfoNotify`。

# 切换角色 - 他人
1. 服务器向客户端发送 `AvatarEquipChangeNotify` 数据包。
2. 服务器向客户端发送 `SceneTeamUpdateNotify` 数据包。
3. 服务器向客户端发送 `HideAndSeekPlayerSetAvatarNotify` 数据包。

# 准备就绪
1. 客户端向服务器发送 `HideAndSeekSetReadyReq`。
2. 服务器向客户端回复 `HideAndSeekPlayerReadyNotify`。
3. 服务器向客户端发送 `MultistagePlayInfoNotify`。
4. 服务器向客户端回复 `HideAndSeekSetReadyRsp`。
5. 如果所有玩家都已准备就绪，服务器将继续启动 Windtrace。

# 开始 Windtrace
1. 当所有玩家都准备好后，服务器将向玩家发送一系列数据包。
   1. `GalleryStartNotify`
   2. `SceneGalleryInfoNotify`
   3. `MultistagePlayInfoNotify`
   4. `MultistagePlayStageEndNotify`
   5. 这仅在倒计时 `1.` 时发送。

### 注意：
- `GuestReplyInviteRsp` 在 `DraftInviteResultNotify` **之后**发送。

## `DraftOwnerInviteNotify`
- `invite_deadline_time` - 邀请过期时间。
- `draft_id` - Windtrace 的值始终为 `3001`。

## `DraftOwnerStartInviteReq`
- `draft_id` - Windtrace 的值始终为 `3001`。

## `DraftOwnerStartInviteRsp`
- `draft_id` - Windtrace 的值始终为 `3001`。
- `invite_fail_info_list` - 未被邀请的玩家列表。
- `retcode` - 响应码。
- `wrong_uid` - 始终为 `0`。（未文档化）

## `DraftGuestReplyInviteReq`
- `draft_id` - Windtrace 的值始终为 `3001`。
- `is_agree` - 布尔值，表示客户端是否接受邀请。

## `DraftGuestReplyInviteRsp`
- `draft_id` - Windtrace 的值始终为 `3001`。
- `retcode` - 请求的响应码。
- `is_agree` - 布尔值，表示服务器是否确认客户端接受了邀请。

## `DraftInviteResultNotify`
- `draft_id` - Windtrace 的值始终为 `3001`。
- `is_all_agree` - 布尔值，表示所有客户端是否都接受了邀请。

## `NpcTalkStateNotify`
- `is_ban` - 进入 Windtrace 时此值始终为 true。

## `PlayerEnterSceneNotify`
- `pos` - 玩家将被传送到的位置。
  - 该值取决于玩家是猎人还是躲藏者。
  - 该值由服务器设置，必须硬编码或从 JSON 文件中读取。

## `MultistagePlayStageEndNotify`
- `play_index` - 由服务器选择的值。（使用 1）
- `group_id` - Windtrace 的值始终为 `133002121`。

## `MultistagePlayInfoNotify` - 初始 + PostEnterSceneReq
- 图片参考：![img.png](images/multistageplayinfo.png)
- `info` - MultistagePlayInfo 数据。
  - `group_id` - Windtrace 的值始终为 `133002121`。
  - `play_index` - 由服务器选择的值。（使用 1）
  - `hide_and_seek_info` - Windtrace 相关信息。
    - `hider_uid_list` - 躲藏者的 UID（整型）列表。
    - `hunter_uid` - 猎人的 UID（整型）。
    - `map_id` - Windtrace 地图的 ID。
    - `stage_type` - Windtrace 状态。
      - 此值将为 `HIDE_AND_SEEK_STAGE_TYPE_PREPARE`。
    - `battle_info_map` - 包含 UID 到 `HideAndSeekPlayerBattleInfo` 对象的字典。
      - `skill_list` - 由玩家选择的 3 个技能 ID 数组。
      - `avatar_id` - 玩家想要使用的角色 ID。
      - `is_ready` - 玩家的游戏内准备状态。
      - `costume_id` - 玩家角色穿着的服饰 ID。

## `MultistagePlayInfoNotify` - 选择角色
- 图片参考：![img.png](images/pickavatar.png)
- **注意：** 此数据包与初始结构和数据一致。
- `info.hide_and_seek_info.stage_type` - 此值将为 `HIDE_AND_SEEK_STAGE_TYPE_PICK`。

## `MultistagePlayInfoNotify` - 开始 Windtrace
- 图片参考：![img.png](images/startwindtrace.png)
- **注意：** 此数据包与初始结构和数据一致。
- `info.hide_and_seek_info.stage_type` - 此值将为 `HIDE_AND_SEEK_STAGE_TYPE_HIDE`。

## `MultistagePlayInfoNotify` - 搜寻阶段
- 图片参考：![img.png](images/seektime.png)
- **注意：** 此数据包与初始结构和数据一致。
  - `info.hide_and_seek_info.stage_type` - 此值将为 `HIDE_AND_SEEK_STAGE_TYPE_SEEK`。

## `MultistagePlayInfoNotify` - 结束 Windtrace
- 图片参考：![img.png](images/seektime.png)
- **注意：** 此数据包与初始结构和数据一致。
    - `info.hide_and_seek_info.stage_type` - 此值将为 `HIDE_AND_SEEK_STAGE_TYPE_SETTLE`。

## `HideAndSeekPlayerSetAvatarNotify`
- `avatar_id` - 玩家想要使用的新角色 ID。
- `uid` - 更换角色的玩家的 UID。
- `costume_id` - 玩家角色穿着的服饰 ID。

## `HideAndSeekSetReadyRsp`
- `retcode` - 请求的响应码。

## `HideAndSeekPlayerReadyNotify`
- `uid_list` - 已准备就绪的玩家的 UID（整型）列表。

## `GalleryStartNotify`
- `gallery_id` - TODO: 确认此值在 Windtrace 中是否始终为 `7056`。
- `start_time` - Windtrace 的值始终为 `2444`。
  - 显示游戏结束统计信息时，此值为 `200`。
- `owner_uid` - 发起 Windtrace 游戏的玩家的 UID。
- `player_count` - Windtrace 游戏中的玩家数量。
- `end_time` - 此值始终与 `start_time` 相同。

## `SceneGalleryInfoNotify` - 开始 Windtrace
- `gallery_info` - SceneGalleryInfo 数据。
  - `end_time` - 此值始终与 `start_time` 相同。
  - `start_time` - Windtrace 的值始终为 `2444`。
    - 显示游戏结束统计信息时，此值为 `200`。
  - `gallery_id` - 此值始终与 `GalleryStartNotify` 中的 `gallery_id` 相同。
  - `stage` - 场景的当前阶段。
    - 此值将为 `GALLERY_STAGE_TYPE_START`。
  - `owner_uid` - 发起 Windtrace 游戏的玩家的 UID。
  - `hide_and_seek_info` - SceneGalleryHideAndSeekInfo
    - `visible_uid_list` - 仍然存活的玩家 UID（整型）列表。
    - `caught_uid_list` - 已被抓到的玩家 UID（整型）列表。
  - `player_count` - Windtrace 游戏中的玩家数量。
  - `pre_start_end_time` - Windtrace 的值始终为 `0`。

## `HideAndSeekSettleNotify`
- `reason` - 游戏结束的原因。
- `winner_list` - 获胜玩家的 UID（整型）列表。
- `settle_info_list` - HideAndSeekSettleInfo 数据。
  - 这是一个参与游戏的玩家列表。

## `HideAndSeekSettleInfo`
- `card_list` - `ExhibitionDisplayInfo` 的集合
  - 如果未知：硬编码指定值。![img.png](images/defaultexhibitioninfo.png)
  - 测试中这些值是重复出现的。
- `uid` - 参与游戏的玩家的 UID。
- `nickname` - 玩家的昵称。
- `head_image` - 此值始终为 `0`。
- `online_id` - 此值始终为空。
- `profile_picture` - `ProfilePicture` 对象。
- `play_index` - 由服务器选择的值。（使用 1）
- `stage_type` - 阶段类型。（未确定；TODO）
- `cost_time` - 玩家完成游戏所花费的时间。
- `score_list` - 玩家得分列表。

## `ExhibitionDisplayInfo`
- `id` - 奖励的 ID。
- `param` - 给予的奖励数量。
- `detail_param` - 此值**大多数情况下**为 0。
  - 当奖励为游戏时长时，此值**与** param **相同**。（参与奖励）
