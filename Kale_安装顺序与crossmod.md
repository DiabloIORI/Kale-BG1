# Kale crossmod 依赖与安装顺序

记录时间：2026-11-11。数据来源：`Kale.tp2`、`library\Kale_bg1.tpa`、`Dialogue\CrossMod\**`、各对方模组源码，以及本机 `override` / `WeiDU.log` 实测。

## 一、一句话结论

Kale 的 crossmod **不是独立组件**，写死在主组件里，靠"安装那一刻对方的 banter 文件在不在 override 里"决定这段编不编。对方没装＝这段对话静默消失，不报错、也不留痕。

所以：**想让 Kale 主动说出这些闲聊，对方模组必须先装在 Kale 之前。**

但 8 家里有 6 家自己在自己模组里也带了一份内容相同的 "Kale.d"，条件正好相反（探测 `X3KaleB.dlg`，即要求 Kale 先装）。结果就是：同一批对话，只会有一家的那份被编译——**内容不丢，只是挂到谁的对话文件、走谁的 banter 槽**。

唯一没有回头路的是 **Drake**：Drake 侧 `dialogue\crossmod\kale.d` 是死文件（tp2/tpa 里没有任何一处编译它），所以那 2 段只有 Kale 侧有。

## 二、安装期代码（`library\Kale_bg1.tpa` L248–294，原样）

```
ACTION_IF FILE_EXISTS_IN_GAME ~x3milyB.dlg~ THEN BEGIN
  PRINT @47 COMPILE ~Kale/dialogue/CrossMod/Emily.d~   END
ACTION_IF FILE_EXISTS_IN_GAME ~x3VienB.dlg~ THEN BEGIN
  PRINT @48 COMPILE ~Kale/dialogue/CrossMod/Vienxay.d~ END
ACTION_IF FILE_EXISTS_IN_GAME ~x3HelgaB.dlg~ THEN BEGIN
  PRINT @490 COMPILE ~Kale/dialogue/CrossMod/Helga.d~  END
ACTION_IF FILE_EXISTS_IN_GAME ~x3RecB.dlg~ THEN BEGIN
  PRINT @500 COMPILE ~Kale/dialogue/CrossMod/Recorder.d~ END
ACTION_IF FILE_EXISTS_IN_GAME ~BC0Aura.dlg~ THEN BEGIN
  PRINT @501 COMPILE ~Kale/dialogue/CrossMod/Aura.d~   END
ACTION_IF FILE_EXISTS_IN_GAME ~BC0Drake.dlg~ THEN BEGIN
  PRINT @502 COMPILE ~Kale/dialogue/CrossMod/Drake.d~  END
ACTION_IF FILE_EXISTS_IN_GAME ~BC0Siren.dlg~ THEN BEGIN
  PRINT @503 COMPILE ~Kale/dialogue/CrossMod/Sirene.d~ END
ACTION_IF FILE_EXISTS_IN_GAME ~bsufinch.dlg~ THEN BEGIN
  PRINT @504 COMPILE ~Kale/dialogue/CrossMod/Finch.d~  END
```

运行期结构（以 Aura 为例）：

```
CHAIN IF ~InParty("C0Aura") See("C0Aura") ... Global("CrossAuraKale","LOCALS",0)~
THEN X3KaleB KaleAuraBanter1
~…~
DO ~SetGlobal("CrossAuraKale","LOCALS",1)~
== BC0Aura ~…~
== X3KaleB ~…~
EXIT
```

`== BC0Aura` 是对对方 .dlg 的硬引用——文件不存在时这段根本编不过，所以作者加了 `FILE_EXISTS_IN_GAME` 当守卫。`X3KaleB` 在 L163 被 APPEND 进 `interdia.2da`，是 Kale 的 banter 文件，所以放在 Kale 侧的段落在 Kale 的 banter 槽触发；镜射到对方文件里的那段则在对方的 banter 槽触发。

## 三、8 家清单

| 对方 | 探测文件 | Kale 侧段数 | 对方侧也有 Kale.d？ | 本机 override |
|---|---|---|---|---|
| Emily | `x3milyB.dlg` | 8（4 Emily + 4 Isaac） | 有（内容相同） | **已装**（1.65） |
| Vienxay | `x3VienB.dlg` | 4 | 有（内容相同） | 未装 |
| Helga | `x3HelgaB.dlg` | 4 | 有（内容相同） | 未装 |
| Recorder | `x3RecB.dlg` | 4 | 有（内容相同） | 未装 |
| Aura | `BC0Aura.dlg` | 2 | 有（内容相同） | 未装 |
| Drake | `BC0Drake.dlg` | 2 | **有文件但从不编译（死文件）** | 未装 |
| Sirene | `BC0Siren.dlg` | 1 | 有（内容相同） | 未装 |
| Finch | `bsufinch.dlg` | 2 | 无 | **已装**（v6.0） |

合计 27 段。Emily 那 8 段里有 4 段其实是 Kale 与 **Isaac**（Emily 自带的第二个可加入 NPC）的对话，所以 `Emily.d` 会引用 `X3ISAACB.dlg`。

"内容相同"的判定：Emily、Helga、Vienxay、Recorder、Aura、Sirene 六家的 Kale.d 与 Kale 侧对应文件 **段数、状态标签、Global 变量名逐一对应**（Emily 两份逐字比对过，连 `@15` 这类 tra 序号都一样）。因此两组顺序最终拿到的对话是同一批。

## 四、硬前置（不属于 crossmod，但会直接导致装不上）

1. **`OFPATHSANDWAYS.TP2` 组件 12**（Lava 的 Of Paths and Ways，"The 3-Foot-Tall-Fury Kit"）——只有选 tp2 第 3 个组件（`BEGIN @57`，`REQUIRE_COMPONENT`，L294–296）才需要，而且**必须在 Kale 之前装**，否则 REQUIRE_COMPONENT 直接拦下。第 2 个组件用的是 Kale 自带 kit `X3KFK`，无外部依赖。
2. **EET 核心**——EET 分支要 `INCLUDE ~EET/other/cpmvars/eet_cpmvars.tpa~`（L28–31）。本机 WeiDU.log 有 `EET v15.0 #0` ✓。
3. **EET_End 还没装**（本机日志里没有）→ 现在往 EET 里装 BG1 侧 NPC 仍属合法窗口，不会被"必须在 EET_End 之前"卡住。
4. **SoD 部分**由 `FILE_EXISTS_IN_GAME ~bdcaelar.cre~` 触发（L35、L187、L354）；EET 里 SoD 已合并，这段会装。
5. **主组件必须先装**：crossmod 只写在主组件（`BEGIN @0`）里，kit 组件（@53 / @57）只换 .cre，不含对话。

## 五、四个已知 bug（1.69 原版就有，作者在 1.72/1.73 修了一部分；本地源码已修第 1、2、4 条）

1. **Vienxay 侧会卡循环**：`Kale\Dialogue\CrossMod\Vienxay.d` 第 2、3 段都写成 `THEN X3KaleB KaVieBanter2`（L23、L43 同一个标签）。后果是 "Stop ogling me" 那段反复触发、变量不推进。作者给玩家的临时解法：控制台 `C:SetGlobal("CrossVienKale","GLOBAL",2)`；正式修法是把第 3 段改名（如 `KaVieBanter3`）——**本地源码已按此改（见第七节）**。
2. **Aura 与 Finch 第 2 段撞名**：`Finch.d` L21 复制粘贴没改名，仍是 `THEN X3KaleB KaleAuraBanter2`，与 `Aura.d` L26 同标签。两家都装时这两段会互相覆盖（同样表现为重复同一段）——**本地源码已改名为 `KaleFinchBanter2`（见第七节）**。
3. **Emily 的 Isaac 依赖**：`Emily.d` 里有 4 段 `== X3IsaacB`，需要 `X3ISAACB.dlg`。Emily 某些发行版不带 Isaac 时，Kale 会直接 `ERROR locating resource for 'CHAIN3'` 并整体回滚；作者的官方对策是 **Kale 装在 Emily 之前**（这样 Kale 侧跳过，改由 Emily 侧编译）。本机 `X3ISAACB.dlg` 在 override 里 ✓，不受影响。

4. **Isaac 第 1 段抢变量**：`Emily.d` L85 原本查的是 `CrossEmilKale=0`，与 Emily 第 1 段同一个变量，谁先播就把另一段永久锁死（后播的那段永远不满足条件）。**本地源码已改为 `CrossIsaacKale`（见第七节）**。

## 六、针对这台机器的顺序

现状：Emily 1.65 ✓、Finch v6.0 ✓；Aura / Drake / Helga / Recorder / Sirene / Vienxay 全未装；EET v15.0 已装、EET_End 未装；Of Paths and Ways 未装。

**推荐（稳、可增量）**

1. 先装 **Drake**（唯一不可逆的那一步）。
2. 装 **Kale 主组件 + 其中一个 kit 组件** → 拿到 Emily 4 + Isaac 4 + Finch 2 + Drake 2 = **12 段**。
3. 之后随时装 Aura / Helga / Recorder / Vienxay / Sirene —— 它们各自编译自己的 kale.d，再补 Aura 2 + Helga 4 + Recorder 4 + Vienxay 4 + Sirene 1 = **15 段**，不需要回头重装 Kale。
   撞名标签已在源码改好（见第七节），这一步不需要再动。

**想一次拿全（先装齐 8 家再装 Kale）**：也能拿 27 段，但同样得先修 Finch 的标签；Vienxay 的重复标签无论哪种顺序都在，建议一起改。

**顺序装反了怎么办**：不报错、不崩，只是那几段缺席。补装上游后重装 Kale 即可补回（Drake 那段尤其如此）。

## 七、已改的三处（本地源码，2026-11-11）

- `Kale\Dialogue\CrossMod\Finch.d` L21：`THEN X3KaleB KaleAuraBanter2` → `THEN X3KaleB KaleFinchBanter2`（解掉与 Aura 第 2 段撞名）
- `Kale\Dialogue\CrossMod\Vienxay.d` L43：`THEN X3KaleB KaVieBanter2` → `THEN X3KaleB KaVieBanter3`（解掉第 2、3 段同标签导致的循环）
- `Kale\Dialogue\CrossMod\Emily.d` L85/L87：Isaac 第 1 段的变量由 `CrossEmilKale` 改成 `CrossIsaacKale`（原来与 Emily 第 1 段共用同一个 0→1 变量，谁先播谁把另一段锁死；改后与 Isaac 第 2–4 段的 1→2→3→4 衔接一致）

三处都只动拉丁标识符，文本全是 `@` 引用，不影响汉化。改完按 `THEN <dlg> <state>` 逐条核对，`X3KaleB / X3milyB / X3VienB / X3HelgaB / X3RecB / BSUFINCH / X3IsaacB` 七个目标文件内部已无重名标签。
