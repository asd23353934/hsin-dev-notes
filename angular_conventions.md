# Angular Conventions

> Angular 專案的寫法慣例。Claude Code 在產出 Angular 程式碼時必須遵守。
> 最後更新：2026-04-27

---

## 元件（Components）

### 一律使用 standalone components
- 不建立 NgModule
- 直接在 component 的 `imports` 陣列引入依賴
- 範例：
  ```typescript
  @Component({
    selector: 'app-user',
    standalone: true,
    imports: [CommonModule, ButtonModule],
    templateUrl: './user.component.html',
  })
  export class UserComponent {}
  ```

### 使用 signal-based API
- 用 `signal()` / `computed()` / `effect()` 管理狀態
- 用 `input()` / `output()` / `model()` 取代 `@Input()` / `@Output()`
- 範例：
  ```typescript
  count = signal(0);
  doubled = computed(() => this.count() * 2);
  
  name = input.required<string>();
  selected = model<boolean>(false);
  ```

### Template 語法
- **必用**新控制流：`@if` / `@for` / `@switch` / `@defer`
- **不用**結構指令：`*ngIf` / `*ngFor` / `*ngSwitch`
- `@for` 一定要寫 `track`：
  ```html
  @for (item of items(); track item.id) {
    <div>{{ item.name }}</div>
  }
  ```

---

## 樣式

- **優先**：Tailwind utility classes
- **次選**：Tailwind 不夠時用 component-scoped SCSS
- **最後手段**：`::ng-deep`（限定改 PrimeNG 等第三方元件樣式）
- 不在全域 styles.scss 寫元件相關樣式

---

## PrimeNG

- 引入用 standalone：`imports: [ButtonModule]`
- 自訂主題用 PrimeNG 的 theming API 或 Tailwind config
- Tree 元件相關踩坑見 `errors.md`

---

## RxJS / Signals

- **新程式碼優先用 signal**，不用 BehaviorSubject
- 仍需 RxJS 時：
  - 訂閱統一用 `takeUntilDestroyed()`，**不再用 `ngOnDestroy + Subject` 模式**
  - 避免 nested subscribe，多用 `switchMap` / `mergeMap`
  - HTTP 錯誤處理在 service 層用 `catchError`，不在元件層

---

## HTTP / API

- 統一用 `HttpClient` + `inject()`
- API 呼叫包在 service 內，元件不直接呼叫
- 錯誤處理範例：
  ```typescript
  getUsers() {
    return this.http.get<User[]>('/api/users').pipe(
      catchError(err => {
        // 統一錯誤處理
        return throwError(() => err);
      })
    );
  }
  ```

---

## 命名

| 對象 | 規則 | 範例 |
|------|------|------|
| 元件檔 | kebab-case | `user-profile.component.ts` |
| 類別 | PascalCase | `UserProfileComponent` |
| 變數 / 函式 | camelCase | `getUserData` |
| Signal | 名詞，不加 `$` | `users`、`isLoading` |
| Observable | 名詞 + `$` | `users$`、`loading$` |
| 常數 | UPPER_SNAKE | `MAX_RETRY_COUNT` |

---

## 檔案結構

```
src/app/
├── core/                 # 全域 service、guard、interceptor
├── shared/               # 共用元件、pipe、directive
├── features/
│   └── user/
│       ├── components/
│       ├── services/
│       ├── models/
│       └── user.routes.ts
└── app.routes.ts
```

---

## 待補項目

以下章節隨開發累積補充：

- [ ] 表單慣例（Reactive Form vs Template-driven）
- [ ] 路由與守衛
- [ ] 測試慣例
- [ ] i18n 處理
