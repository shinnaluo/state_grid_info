# 本 Fork 的自定义改动：月纯峰谷计费

> 上游：[xiaoshi930/state_grid_info](https://github.com/xiaoshi930/state_grid_info) · 基线版本 **5.1**（2026-11-09）
> 本 Fork：https://github.com/shinnaluo/state_grid_info

## 一、这个 Fork 是什么

把此前直接改在 Home Assistant `custom_components/state_grid_info/` 目录里的本地定制，固化成一份版本化的代码，避免以后每次 HACS 更新都要手动把文件恢复回去。

- **基线**：上游 tag `5.1` 的 main，除下述 3 个文件外与官方 5.1 逐字节一致。
- **版本**：`5.1`（`manifest.json` 与 Release tag 一致，HACS 可直接识别、不会反复提示更新）。
- **验证环境**：Home Assistant Core 2026.11.0，4 个户号同时在线。
- **附带说明**：上游 README 仅在开头加了「本 Fork 说明」一节，正文保持原样。

## 二、改动清单（相对官方 5.1）

| 文件 | 规模 | 内容 |
|---|---|---|
| `const.py` | +3 行 | 新增 `BILLING_STANDARD_MONTH_纯峰谷 = "month_peak_valley"`，加入计费标准列表，中文名「月纯峰谷计费」。 |
| `config_flow.py` | +25 / -4 | 在**新建配置流程**和**选项流程**两处，新增「月纯峰谷」分支的四段单价表单：`CONF_PRICE_TIP / CONF_PRICE_PEAK / CONF_PRICE_FLAT / CONF_PRICE_VALLEY`，默认值 `0 / 0.5683 / 0 / 0.2883`。 |
| `coordinator.py` | +173 / -6 | 新增 `_calculate_flat_tou_cost(day_data)`：不设阶梯，直接按尖/峰/平/谷四段电量 × 对应单价结算；主实体属性输出「尖电价 / 峰电价 / 平电价 / 谷电价」；补齐月度峰谷电量字段 `month_t/p/n/v_ele_num` 与日明细 `thisTPq / thisPPq / thisNPq / thisVPq`（供 hassbox / 青龙 MQTT 数据源使用）。 |
| `manifest.json` | **未改动** | 此前本地为消除更新提示手动把版本写成 `5.1`，官方 5.1 已经是 `5.1`，该改动被上游吸收，无需再保留。 |

逐行差异可直接看本仓库的提交：`git show <本 Fork 的 feat 提交>`；或本地保全包里的 `custom_changes_vs_5.0.patch`。

## 三、配置数据不在仓库里

- **计费标准与四段单价**保存在 HA 的 config entry（`.storage/core.config_entries`），不在组件目录；
- `/config/state_grid_info_<户号>.json` 是电量/电费数据缓存；
- HACS 更新只替换 `custom_components/state_grid_info/` 目录，以上数据都不会丢，也不应该提交到仓库。

## 四、安装 / 切换到本 Fork

### HACS（推荐）

1. HACS → 集成 → 右上角「⋮」→ **自定义存储库**；
2. 填 `https://github.com/shinnaluo/state_grid_info`，类别选 **集成**；
3. 搜索「国家电网辅助信息」，下载版本 `5.1`；
4. 重启 Home Assistant。

> 若此前装的是上游 `xiaoshi930/state_grid_info`：两个仓库的 domain 相同，**不要同时启用**。请先移除上游条目（或将其标为忽略），再添加本 Fork；现有 4 个 config entry 会保留，实体 ID 不变。

### 手动安装

从本仓库 Release `5.1` 下载源码 zip，把 `custom_components/state_grid_info/` 整个目录复制到 HA 的 `/config/custom_components/`，重启 HA。

## 五、以后上游出新版本怎么同步

本地 clone 里已经配好两个 remote：`origin` = 本 Fork，`upstream` = 上游。

```bash
git fetch upstream --tags
git merge upstream/main          # 或 git rebase upstream/main
# 若出现冲突，只会在计费标准相关段落；保留「月纯峰谷」分支与四段单价表单即可
git push origin main

# 发新版（版本号需与 manifest.json 保持一致）
# 1) 把 custom_components/state_grid_info/manifest.json 的 version 改成新版本号
git commit -am "chore: bump version to 5.2"
git tag 5.2
git push origin main 5.2
# 2) 在 GitHub 上基于 tag 5.2 建 Release，HACS 才能看到新版本
```

> 从本 Fork 更新时，HACS 下载的代码**本身已包含定制**，不需要再跑任何「恢复脚本」。
> 只有继续装上游仓库时，上游每次更新才会覆盖这 3 个文件。

## 六、升级后验证清单

- 4 个集成条目均为 `loaded`，实体未变「不可用」；
- 开发者工具 → 状态，主实体（如 `sensor.6_1101_1101`）属性含有 `尖电价 / 峰电价 / 平电价 / 谷电价`；
- 集成选项里的计费标准仍是「月纯峰谷计费」，四段单价未丢；
- 数值基准（户号 3309947664382，2026-11-09 升级前）：余额 **955.68 元**、当月电费 **3.58 元**、当月用电 **6.23 kWh**、累计电费 **56.99 元**、累计用电 **101.01 kWh**；
- `home-assistant.log` 无 `state_grid_info` 相关报错。

## 七、备注

- 官方 `5.0 → 5.1` 经逐字节比对只改了 `manifest.json` 的版本号，没有任何代码变化，因此本 Fork 的 5.1 在功能上等价于「官方 5.1 + 上述定制」。
- 上游项目没有 LICENSE 文件；本 Fork 仅作个人定制留存，改动部分可自由合回上游（若作者愿意收录）。
