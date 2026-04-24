---
banner: "![[banner_蓝紫.png]]"
---

---
![[待办事项#Todo]]

| **最近访问**<br>                                                                                                                                                               | **快捷导航**<br>                                                            |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `$=dv.list (dv.pages (''). where (f => !f.file.name.startsWith ("Home🏠")&& !f.file.name.startsWith("待办事项")) . sort (f=>f.file. mtime. ts,"desc"). limit (5). file. link)` | [[【西湖大学】强化学习]]<br>[[Bellman Optimality Equation]]<br>[[标注块]]<br>[[Git]] |

---

```contributionGraph
title: Contribution
graphType: default
dateRangeValue: 365
dateRangeType: LATEST_DAYS
startOfWeek: 1
showCellRuleIndicators: true
titleStyle:
  textAlign: center
  fontSize: 15px
  fontWeight: normal
dataSource:
  type: PAGE
  value: ""
  dateField: {}
  filters: []
fillTheScreen: true
enableMainContainerShadow: true
cellStyleRules:
  - id: Ocean_a
    color: "#bec8e6ff"
    min: 1
    max: 2
  - id: Ocean_b
    color: "#8a9cd2ff"
    min: 2
    max: 3
  - id: Ocean_c
    color: "#5670beff"
    min: 3
    max: "4"
  - id: Ocean_d
    color: "#2244aaff"
    min: "4"
    max: 9999

```

