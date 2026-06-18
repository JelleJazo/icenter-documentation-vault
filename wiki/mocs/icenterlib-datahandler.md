---
type: moc
title: "ICenterLib/DataHandler — GenericQuery + Excel + grid helpers"
status: draft
module: "ICenterLib/DataHandler"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\DataHandler\\"
tags: [moc, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/DataHandler

## What this hub covers

`ICenterLib\DataHandler\` — the **shared data-access plumbing**. Two things matter most:

1. **`GenericQuery`** — the parameterised-SQL wrapper used throughout the modern code path (the "good" way to talk to SQL — as opposed to per-class `SqlCommand` boilerplate).
2. **`OpenXml\Excel.vb` / `OpenXmlSpreadsheet.vb`** — DocumentFormat-OpenXml-based Excel reader/writer.

19 source files (~115 KB) + 2 generated.

## `GenericQuery` — the parameterised-SQL wrapper

```vb
Public Class GenericQuery
    Public Property CommandType As CommandType = CommandType.Text

    Public Function ExecuteSelect(Connection, CommandText, Parameters As Dictionary)        As DataTable
    Public Async Function ExecuteSelectAsync(Connection, CommandText, Parameters)           As Task(Of DataTable)
    Public Sub ExecuteUpdate(Connection, CommandText, Parameters)
End Class
```

`AddParameters` accepts a `Dictionary(Of String, Object)` and auto-prefixes keys with `@` if missing.

**`ExecuteUpdate` rethrows wrapped exceptions** (`Throw New Exception(ex.ToString, ex)`) — losing the inner stack trace via `ex.ToString` instead of just passing `ex` as inner. Q-282.

**`ExecuteSelect` does NOT have a try/catch** — caller must handle. **`ExecuteUpdate` does**. Inconsistent.

## Files

| File | Role |
|------|------|
| `GenericQuery.vb` | the central wrapper |
| `BetterDataGridView.vb` | DataGridView subclass with improvements |
| `DataSetComparer.vb` + `DataSetCompareResult.vb` + `DataSetCompareTolerance.vb` | DataSet diff with tolerances |
| `DataTableColumnSchema.vb` + `DataTableColumnSchemaHandler.vb` + `DataTableColumnSchemaRecord.vb` | column-schema introspection / persistence |
| `ExportExcel.vb` | DataTable → Excel export |
| `FrmDataGridView.vb` + `.Designer.vb` | DataGridView host form |
| `FrmDataViewer.vb` + `.Designer.vb` | DataTable viewer form |
| `IStreamWrapper.vb` | IStream wrapper for OpenXml |
| `OpenXml\Excel.vb` | Excel reader (DocumentFormat.OpenXml) |
| `OpenXml\OpenXmlSpreadsheet.vb` | spreadsheet writer |
| `QrCode.vb` | QR-code generator |
| `Selection.vb` | generic selection helper |
| `Toolbox.vb` | misc DataTable helpers |
| `ZeroCode.vb` | "zero-code" sentinel/helper |
| `JsonHelper.vb` | Newtonsoft.Json helper |

## Open questions

- **Q-282 (new):** `GenericQuery.ExecuteUpdate` rethrows as `Throw New Exception(ex.ToString, ex)` — using `ex.ToString` as the message loses the inner-stack readability. Use `ex.Message` or just `Throw`.
- **Q-283 (new):** `ExecuteSelect` has no try/catch while `ExecuteUpdate` does. Inconsistent. Document caller expectations or align.

Logged in [[../needs-review/_index]].

## Coverage

All 19 source files marked `done` overview-level; the central GenericQuery is documented above. Designer files marked `generated`.
