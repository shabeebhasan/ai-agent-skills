---
name: Channelling Station Development
description: How to create, implement, and test a new channelling sorting station (PreSort, PostQC, Rework, FinalSort)
---

# Channelling Station Development Skill

This skill documents the complete workflow for building a new channelling sorting station mode in IMSOFT. Follow these phases in order.

## Prerequisites

Before starting, ensure you have:
- The station mode name (e.g., `POST_QC`, `REWORK`, `FINAL_SORT`)
- The T-spec document defining functional requirements
- T12 shared infrastructure is in place:
  - `web/css/channelling_station.css` — shared design tokens + layout classes
  - `web/js/channelling_station_base.js` — shared state machine + error mapping + delivery helpers

## Phase 1: Design (Create Flow)

### 1.1 Define Station Blueprint

Create a station blueprint document answering:
- Which fields to **emphasize** in the route card (channel_decision? QC result? rework cycle?)
- Which **actions** are available (Confirm, Override, Print, QC input?)
- Which **error codes** apply beyond the shared 14
- Whether **strict mode** (print + area scan) is required
- Whether **delivery list** is needed and how items are grouped

### 1.2 Define Endpoints

Map the station's API contract:

```
POST /channelling/{station-slug}-scan       → scan endpoint
POST /channelling/{station-slug}-confirm    → confirm endpoint
GET  /channelling/{station-slug}-delivery-list
GET  /channelling/{station-slug}-delivery-export
POST /channelling/{station-slug}-delivery-note-create
POST /channelling/{station-slug}-delivery-list-clear
```

### 1.3 Define Form Model Fields

Extend or create a form model with station-specific fields beyond the base set:

**Base fields** (always present):
- `scan_code`, `station`, `allow_override`, `override_target_code`
- `include_delivery_list`, `target_code`, `auftragsnr`, `confirm_token`
- `area_scan_code`, `label_printed`

**Mode-specific** examples:
- PostQC: `qc_result`, `qc_grade`, `fail_reason`, `defect_type`
- Rework: `rework_provider`, `rework_cycle`

---

## Phase 2: Implement

### 2.1 Form Model

**File:** `models/view/form/channelling/Form{StationName}.php`

```php
<?php
namespace app\models\view\form\channelling;
use yii\base\Model;

class FormChannelling{StationName} extends Model {
    const STATION_{MODE} = '{MODE}';
    const SCENARIO_SCAN = 'scan';
    const SCENARIO_CONFIRM = 'confirm';
    const SCENARIO_DELIVERY = 'delivery';

    // Base fields
    public $scan_code;
    public $station = self::STATION_{MODE};
    public $allow_override = 0;
    public $override_target_code;
    public $include_delivery_list = 1;
    public $target_code;
    public $auftragsnr;
    public $sorting_target_code;
    public $confirm_token;
    public $area_scan_code;
    public $label_printed = 0;

    // Mode-specific fields here

    public function scenarios() { /* ... */ }
    public function rules() { /* ... */ }
}
```

**Key rules:**
- `scan_code` required on scan scenario
- `station` must be literal `{MODE}`
- `allow_override=1` requires `override_target_code`
- confirm requires `auftragsnr`, `sorting_target_code`, `confirm_token`
- all string inputs must use `trim` filter

### 2.2 Controller Actions

**File:** `controllers/ChannellingController.php`

Add these actions following the PreSort pattern:

```php
// 1. Page render action
public function action{StationName}() {
    $this->ensurePreSortOperatorAccess();
    $bootstrapConfig = [
        'station' => Form::STATION_{MODE},
        'feature_flags' => $this->get{Mode}FeatureFlags(),
        'endpoints' => [
            'scan' => Url::to(['channelling/{slug}-scan']),
            'confirm' => Url::to(['channelling/{slug}-confirm']),
            // delivery endpoints...
        ],
        'permissions' => $this->get{Mode}Permissions(),
        'allowed_override_targets' => $this->get{Mode}AllowedOverrideTargets(),
    ];
    return $this->render('//channelling/channelling_{slug}', [
        'bootstrapConfig' => $bootstrapConfig,
    ]);
}

// 2. Scan action (POST, JSON response)
// 3. Confirm action (POST, JSON response, token validation)
// 4. Delivery list/export/create/clear actions
```

**Critical implementation rules:**
- All actions enforce `ensurePreSortOperatorAccess()`
- Scan always generates `confirm_token` on dry_run
- Confirm validates token, checks override permissions, enforces strict mode
- Status transition uses `ServiceAuftrag::setAuftragStatus()`
- Return deterministic `error_code` on failure (never generic messages)

### 2.3 View

**File:** `views/channelling/channelling_{slug}.php`

Follow this structure exactly:

```php
<?php
// 1. Register shared CSS + JS
$this->registerCssFile('@web/css/channelling_station.css?v=...');
$this->registerJsFile('@web/js/channelling_station_base.js?v=...');
// 2. Register mode-specific JS
$this->registerJsFile('@web/js/channelling_{slug}.js?v=...');
?>

<!-- Station container with bootstrap config -->
<div id="{slug}-station" class="body-content animated fadeIn channelling-station"
     data-bootstrap-config="<?= $bootstrapJson ?>">

  <!-- HEADER: mode + operator + connectivity -->
  <!-- SCAN PANEL (col-md-4): input + submit + alert + strict flow -->
  <!-- MAIN PANEL (col-md-8): route card + details + actions -->
  <!-- DELIVERY LIST: controls + table + select-all -->
  <!-- HISTORY TABLE: recent confirmations -->
</div>
```

**All DOM IDs must follow pattern:** `#{slug}-{field}` (e.g., `#post-qc-scan-code`)

### 2.4 JavaScript

**File:** `web/js/channelling_{slug}.js`

```javascript
(function ($) {
    'use strict';
    var $root = $('#{slug}-station');
    if (!$root.length) return;

    // Parse bootstrap config
    var bootstrap = JSON.parse($root.attr('data-bootstrap-config') || '{}');

    // Create station base (shared infrastructure)
    var station = ChannellingStationBase.create({
        rootSelector: '#{slug}-station',
        stationCode: bootstrap.station || '{MODE}',
        endpoints: bootstrap.endpoints || {},
        permissions: bootstrap.permissions || {},
        featureFlags: bootstrap.feature_flags || {},
        overrideTargets: bootstrap.allowed_override_targets || [],
        onStateChange: function (mode) { renderActionState(); },
        onError: function (code) { /* mode-specific error handling */ }
    });

    // Mode-specific rendering using station.formatValue(), etc.
    function renderResult(result) { /* ... */ }
    function renderActionState() { /* ... */ }

    // Scan submit
    function submitScan() {
        station.clearAlert();
        station.setMode('scan_pending');
        $.ajax({
            url: station.endpoints.scan,
            method: 'POST',
            data: { scan_code: scanCode, station: station.stationCode, dry_run: 1,
                    idempotency_key: station.generateIdempotencyKey('{slug}') },
            dataType: 'json'
        }).done(function (resp) {
            if (!resp || !Number(resp.success)) {
                station.setMode('error');
                station.showMappedError(resp);
                station.focusPrimaryInput();
                return;
            }
            station.state.lastScan = station.normalizeScanResponse(resp);
            renderResult(station.state.lastScan);
            station.setMode('result_ready');
            station.focusPrimaryInput();
        }).fail(function () {
            station.setMode('error');
            station.showAlert('danger', 'Network error', 'Scan request failed.');
            station.focusPrimaryInput();
        });
    }

    // Bind events, init...
})(jQuery);
```

---

## Phase 3: Test

### 3.1 Functional Test (Screen Rendering)

**File:** `tests/functional/controllers/Channelling{StationName}ScreenCest.php`

Minimum test methods (9 scenarios from T12 §18):

```php
class Channelling{StationName}ScreenCest {
    // 1. All station regions rendered
    public function screenRendersAllRegions(\FunctionalTester $I) { }
    // 2. Scan panel complete with autofocus
    public function scanPanelIsComplete(\FunctionalTester $I) { }
    // 3. Strict flow panel rendered
    public function strictFlowPanelRendered(\FunctionalTester $I) { }
    // 4. Routing card has all fields
    public function routingCardHasAllFields(\FunctionalTester $I) { }
    // 5. Action row complete
    public function actionRowIsComplete(\FunctionalTester $I) { }
    // 6. Delivery controls present
    public function deliveryControlsPresent(\FunctionalTester $I) { }
    // 7. History table present
    public function historyTablePresent(\FunctionalTester $I) { }
    // 8. Bootstrap config embedded
    public function bootstrapConfigPresent(\FunctionalTester $I) { }
    // 9. Last result summary shown
    public function lastResultSummaryPresent(\FunctionalTester $I) { }
}
```

### 3.2 Unit Test (Service Logic)

**File:** `tests/unit/models/service/channelling/ServiceChannelling{StationName}Test.php`

Test the backend decision/routing service:
- Each valid input combination returns correct target + metadata
- Invalid inputs return deterministic error codes
- Action normalization handles known variants
- Output contains all required fields

### 3.3 Run Tests

```bash
# Functional tests
// turbo
vendor/bin/codecept run functional tests/functional/controllers/Channelling{StationName}ScreenCest.php

# Unit tests
// turbo
vendor/bin/codecept run unit tests/unit/models/service/channelling/ServiceChannelling{StationName}Test.php
```

---

## File Checklist

For each new station mode, you need these files:

| # | File | Type |
|---|------|------|
| 1 | `models/view/form/channelling/Form{StationName}.php` | Form model |
| 2 | `controllers/ChannellingController.php` (add actions) | Controller |
| 3 | `views/channelling/channelling_{slug}.php` | View |
| 4 | `web/js/channelling_{slug}.js` | JavaScript |
| 5 | `tests/functional/controllers/Channelling{StationName}ScreenCest.php` | Functional test |
| 6 | `tests/unit/models/service/channelling/...Test.php` | Unit test (if new service) |

**Shared files (already exist — do NOT recreate):**
- `web/css/channelling_station.css`
- `web/js/channelling_station_base.js`

---

## Reference Implementations

- **PreSort (complete):** [channelling_pre_sorting.php](file:///views/channelling/channelling_pre_sorting.php) + [channelling_pre_sorting.js](file:///web/js/channelling_pre_sorting.js)
- **Design spec:** [t12_design_specification.md](file:///t12_design_specification.md)
- **Error taxonomy:** See `ChannellingStationBase.ERROR_MAP` in `channelling_station_base.js`
