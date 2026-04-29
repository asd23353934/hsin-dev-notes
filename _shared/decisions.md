# Architecture Decision Records (ADR)

> 「為什麼選 X 不選 Y」的技術選型決策紀錄。跨 repo / 技術棧 / 專案層級。
> 與各 `conventions.md` 的差別：conventions 記「我們怎麼做」，本檔記「**為什麼**這樣做（含被否決的選項與失效條件）」。
> 最後更新：2026-04-29

---

## 何時用 ADR vs `conventions.md`

- 「driver A、driver B、driver C 都試過，最後選 A」這種**選型決策** → ADR
- 「我們專案內 component 用 PascalCase」這種**慣例規則** → `conventions.md`
- ADR 是**一次性決定**（會失效但不常改），conventions 是**持續遵循的規則**

---

## 紀錄模板

```markdown
### YYYY-MM-DD｜決策標題
- **範圍**：repo 全域 / 技術棧（angular / nextjs / ...） / 特定專案名
- **選擇**：用 X
- **替代**：考慮過 Y、Z（簡述各自取捨）
- **關鍵理由**（最多 3 條）：
  1.
  2.
- **代價**：放棄了什麼（X 沒有但 Y 有的）
- **失效條件**：什麼情況該重新評估（套件 EOL / 新方案出現 / 需求變化）
- **參考**：issue / 文件連結（可選）
```

---

## 紀錄

_尚無紀錄。第一次做技術選型決定時依模板補進來。_
