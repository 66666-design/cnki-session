# cnki-session — 知网会话层（cnki-search / cnki-citation 共用）

> Shared CNKI session layer for
> [cnki-search](https://github.com/66666-design/cnki-search) and
> [cnki-citation](https://github.com/66666-design/cnki-citation):
> search/export endpoints + pure-Python AJ-Captcha WAF solver.

不是独立 CLI，是被两个姊妹仓 import 的会话模块。提供：

- `CNKI` 类：带自动过验的知网会话（搜索 16 字段/5 排序/专业检索、引文导出）
- `CaptchaError` / `FIELD_LABEL` / `SORT_CODES`
- WAF(AJ-Captcha blockPuzzle) 纯 Python 求解器

## 协议规格（2026-10-07/08 实测逆向）

- **检索**：`POST kns.cnki.net/kns8s/brief/grid`；第 1 页 `boolSearch=true+CurPage=1`；
  翻页 = `boolSearch=false` + `pageNum` + 第 1 页响应里 `#hidTurnPage` 令牌 +
  `sortField/sortType` + QueryJson 填充 `Products`/`SearchFrom=4` + `aside` 置空。
- **引文**：`POST kns.cnki.net/dm8/API/GetExport`
  （filename=`input.cbItem` 的加密 value；displaymode 必须三 mode 齐传）。
- **专业检索**：同 grid，`SearchType=4 + Field:EXPERT + Value=表达式`。
- **WAF**：AJ-Captcha blockPuzzle。`/verify-api/get` 发底图+块图+token+secretKey；
  `/verify-api/web/check` 收 AES-ECB(secretKey) 的 `{"x":310*缺口x/图宽,"y":5}`；
  成功后 `GET returnUrl?captchaId=` 拿放行 cookie。块图是全高 PNG，先按 alpha
  bbox 裁剪再 FFT SSD 匹配；单次不中换图重试。两种拦法：-403 JSON 挑战、
  软封（200+"暂无数据"）——后者靠调用方自愈重试。
- clickWord 挑战未解，靠刷新抽滑块。

## License

Copyright (C) 2026 66666-design · **AGPL-3.0-or-later**。
