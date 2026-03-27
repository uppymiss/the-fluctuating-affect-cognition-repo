# HOW TO: lab.js Cognitive Tasks in SoSci Survey -- Data Storage Guide

This guide documents how to embed lab.js cognitive tasks (Stroop, Flanker) in SoSci Survey and reliably store trial-level data. It covers the full pipeline: lab.js export, SoSci embedding, data transfer via `postMessage`, and variable setup.

This was developed for a longitudinal study on cognitive and emotional fluctuations but applies to any lab.js task embedded in SoSci Survey.

---

## Table of Contents

1. [The Core Problem](#1-the-core-problem)
2. [Architecture Overview](#2-architecture-overview)
3. [Step 1: Prepare lab.js Task for Data Transfer](#step-1-prepare-labjs-task-for-data-transfer)
4. [Step 2: Export from lab.js](#step-2-export-from-labjs)
5. [Step 3: Set Up SoSci Variables](#step-3-set-up-sosci-variables)
6. [Step 4: Upload and Embed in SoSci](#step-4-upload-and-embed-in-sosci)
7. [Step 5: Add the JavaScript Listener on the SoSci Page](#step-5-add-the-javascript-listener-on-the-sosci-page)
8. [Step 6: Auto-Advance After Task Completion](#step-6-auto-advance-after-task-completion)
9. [Troubleshooting](#troubleshooting)
10. [Variable Naming Convention](#variable-naming-convention)

---

## 1. The Core Problem

lab.js tasks run inside an `<iframe>` on the SoSci Survey page. The iframe is sandboxed -- JavaScript inside the iframe **cannot directly access** SoSci's form fields or the `SoSciTools.put()` function. Calls to `SoSciTools.put()` from within the iframe will silently fail.

**Solution:** Use `window.parent.postMessage()` to send data from the iframe (lab.js) to the parent page (SoSci), where a JavaScript listener receives the data and writes it into hidden form fields.

---

## 2. Architecture Overview

```
+--------------------------------------------------+
|  SoSci Survey Page (parent window)               |
|                                                   |
|  +--------------------------------------------+  |
|  |  lab.js Task (iframe)                       |  |
|  |                                             |  |
|  |  1. Task runs, collects trial data          |  |
|  |  2. On End screen: postMessage(data, '*')   |  |
|  |         |                                   |  |
|  +---------|-----------------------------------+  |
|            |                                      |
|            v                                      |
|  3. Event listener receives message               |
|  4. Writes data into hidden SoSci form fields     |
|  5. SoSci saves fields on page submit             |
|                                                   |
+--------------------------------------------------+
```

---

## Step 1: Prepare lab.js Task for Data Transfer

In the lab.js builder, add a **messageHandler** to the **End screen** component. Set:
- **Title:** `Daten an SoSci senden`
- **Message:** `run`
- **Code:** (see examples below)

### Stroop Example (MZP1_1, Variables K103_01 to K103_07)

```javascript
// === Stroop data: structured transfer to SoSci Survey ===
var ds = this.options.datastore;
var allData = ds.data;

// Filter only stimulus screens
// Practice: sender='Practice_Stimulus', Test: sender starts with 'Test_Stimulus'
var trials = allData.filter(function(row) {
  return row.sender === 'Practice_Stimulus'
      || (row.sender && row.sender.indexOf('Test_Stimulus') === 0);
});

// Collect data per column
var blocks = [];
var trialNrs = [];
var trialarten = [];
var accs = [];
var rts = [];
var stimuli = [];
var reaktionen = [];

trials.forEach(function(trial, i) {
  var block = (trial.sender === 'Practice_Stimulus') ? 'practice' : 'test';
  blocks.push(block);
  trialNrs.push(i + 1);
  trialarten.push(trial.congruency || '');
  accs.push(trial.correct === true ? 1 : 0);
  rts.push(Math.round(trial.duration || 0));
  stimuli.push(trial.word || '');           // Stroop: color word
  reaktionen.push(trial.response || '');
});

// Send structured data via postMessage to SoSci page
var msg = {
  type: 'labjs-data',
  task: 'stroop',
  K103_01: blocks.join(','),        // Block (practice/test)
  K103_02: trialNrs.join(','),      // Trial number
  K103_03: trialarten.join(','),    // Congruency
  K103_04: accs.join(','),          // Accuracy (0/1)
  K103_05: rts.join(','),           // Reaction time (ms)
  K103_06: stimuli.join(','),       // Stimulus (color word)
  K103_07: reaktionen.join(',')     // Response (key pressed)
};

try {
  window.parent.postMessage(msg, '*');
  console.log('Stroop: ' + trials.length + ' trials sent via postMessage.');
} catch(e) {
  console.warn('Stroop postMessage failed:', e);
}
```

### Flanker Example (MZP1_1, Variables K105_01 to K105_07)

The Flanker code is identical except for:
- `task: 'flanker'` instead of `'stroop'`
- `trial.orientation` instead of `trial.word` for the stimulus column
- Variable names `K105_01` to `K105_07` instead of `K103_01` to `K103_07`

### Adapting for Different Measurement Points

Each measurement point (MZP) uses different SoSci variable names. Only the variable prefixes in the `msg` object change:

| MZP | Stroop Variables | Flanker Variables |
|-----|-----------------|-------------------|
| MZP1_1 | K103_01 -- K103_07 | K105_01 -- K105_07 |
| MZP1_2 | K203_01 -- K203_07 | K205_01 -- K205_07 |
| MZP1_3 | K303_01 -- K303_07 | K305_01 -- K305_07 |
| MZP2_1 | K403_01 -- K403_07 | K405_01 -- K405_07 |
| MZP2_2 | K503_01 -- K503_07 | K505_01 -- K505_07 |
| MZP2_3 | K603_01 -- K603_07 | K605_01 -- K605_07 |
| MZP3_1 | K703_01 -- K703_07 | K705_01 -- K705_07 |
| MZP3_2 | K803_01 -- K803_07 | K805_01 -- K805_07 |
| MZP3_3 | K903_01 -- K903_07 | K905_01 -- K905_07 |

---

## Step 2: Export from lab.js

1. In the lab.js builder, click **Export** (top right)
2. Choose **Generic survey tools (SoSci, Unipark, ...)**
3. This downloads a `.zip` file containing the task as a self-contained HTML package

---

## Step 3: Set Up SoSci Variables

In SoSci Survey, create **internal variables** (type: internal / hidden text field) for each data column. For each task and MZP, you need 7 variables:

| Variable | Content | Example Values |
|----------|---------|----------------|
| K*xx*_01 | Block | `practice,practice,...,test,test,...` |
| K*xx*_02 | Trial number | `1,2,3,...,72` |
| K*xx*_03 | Congruency | `congruent,incongruent,...` |
| K*xx*_04 | Accuracy | `1,0,1,1,0,...` |
| K*xx*_05 | Reaction time (ms) | `432,567,389,...` |
| K*xx*_06 | Stimulus | Stroop: `rot,blau,...` / Flanker: `left,right,...` |
| K*xx*_07 | Response | Stroop: `red,blue,...` / Flanker: `left,right,...` |

**Important:** Each variable stores a comma-separated string of all trial values for that column. This means a single variable might contain 72 values (12 practice + 60 test trials).

### How to Create Internal Variables in SoSci

1. Go to **List of Questions** in SoSci
2. Create a new question with the appropriate section code (e.g., `K1`)
3. Set type to **Internal Variables**
4. Create 7 items (01 through 07) with sufficiently large text field size
5. The HTML IDs of these fields will match the variable names (e.g., `K103_01`)

---

## Step 4: Upload and Embed in SoSci

1. In SoSci Survey, go to the questionnaire page where the task should appear
2. Add an **HTML code** element
3. Upload the exported `.zip` file via the **Resource Manager**
4. Embed the task using an iframe:

```html
<iframe
  src="RESOURCE_URL_FROM_SOSCI"
  style="width: 100%; min-height: 600px; border: none;"
  allow="fullscreen"
></iframe>
```

Replace `RESOURCE_URL_FROM_SOSCI` with the actual URL that SoSci generates after upload.

**Tip:** Set `min-height` to at least 600px so the task displays without scrolling.

---

## Step 5: Add the JavaScript Listener on the SoSci Page

On the **same SoSci page** where the iframe is embedded, add a JavaScript listener via HTML code. This listener receives the `postMessage` from the lab.js task and writes the data into the hidden form fields.

### Stroop Listener Example (MZP1_1)

```html
<script type="text/javascript">
window.addEventListener('message', function(event) {
  if (event.data && event.data.type === 'labjs-data' && event.data.task === 'stroop') {
    var fields = ['K103_01','K103_02','K103_03','K103_04','K103_05','K103_06','K103_07'];
    fields.forEach(function(id) {
      var el = document.getElementById(id);
      if (el) el.value = event.data[id] || '';
    });
  }
});
</script>
```

### Flanker Listener Example (MZP1_1)

```html
<script type="text/javascript">
window.addEventListener('message', function(event) {
  if (event.data && event.data.type === 'labjs-data' && event.data.task === 'flanker') {
    var fields = ['K105_01','K105_02','K105_03','K105_04','K105_05','K105_06','K105_07'];
    fields.forEach(function(id) {
      var el = document.getElementById(id);
      if (el) el.value = event.data[id] || '';
    });
  }
});
</script>
```

### Key Details

- The `event.data.type === 'labjs-data'` check ensures you only process messages from your lab.js task, not other browser messages.
- The `event.data.task` check distinguishes between Stroop and Flanker if both are on the same page.
- `document.getElementById(id)` works because SoSci renders internal variables as hidden `<input>` elements with `id` matching the variable name.
- The data is written into the fields **before** the page is submitted, so SoSci saves it automatically.

---

## Step 6: Auto-Advance After Task Completion

To automatically move to the next SoSci page after the lab.js task ends, set a **timeout** on the End screen component in lab.js:

- **Timeout:** `15` (milliseconds)

This causes the End screen to display briefly and then the lab.js study ends. Combined with the `postMessage` handler firing on the `run` event of the End screen, the data is sent before the timeout expires.

**Note:** The auto-advance to the next SoSci page requires SoSci's page transition to be triggered. This typically happens when the iframe's `postMessage` is received and the page auto-submits, or you can add additional JavaScript on the SoSci page to submit the form after receiving the data:

```html
<script type="text/javascript">
window.addEventListener('message', function(event) {
  if (event.data && event.data.type === 'labjs-data') {
    // ... (write data to fields as above) ...

    // Auto-submit after a short delay
    setTimeout(function() {
      document.getElementById('pageForm').submit();
    }, 50);
  }
});
</script>
```

---

## Troubleshooting

### Data not arriving in SoSci variables

1. **Check the browser console** (F12 > Console): Look for the `"Stroop: X Trials per postMessage gesendet"` log message. If absent, the messageHandler on the End screen is not running.
2. **Check that the listener is on the same page as the iframe.** The JavaScript listener must be on the same SoSci page, not a different one.
3. **Check variable IDs:** Open the SoSci page source (Ctrl+U) and search for your variable name (e.g., `K103_01`). If the `<input>` element is not found, the variable is not placed on that page.
4. **Check message filtering:** Make sure `event.data.task` matches what the lab.js code sends (`'stroop'` or `'flanker'`).

### SoSciTools.put() does not work

This is expected. `SoSciTools.put()` cannot be called from within an iframe because of browser security restrictions (same-origin policy). Use `postMessage` instead -- that is the entire point of this architecture.

### Only some variables are filled

- Check that all 7 variables are placed on the SoSci page (not just created in the variable list).
- Verify the variable IDs in the listener script match exactly (case-sensitive).

### Trial count is wrong

- Check component naming in lab.js: The messageHandler filters by `sender === 'Practice_Stimulus'` and `sender.indexOf('Test_Stimulus') === 0`. If your stimulus screens have different names, the filter will miss them.
- Expected trial counts: 12 practice + 60 test = 72 trials total (for the standard version with 3 test blocks of 20 trials each).

### Duplicate practice trials (e.g., 8 instead of 4 in Flanker)

This was a known bug caused by inconsistent component naming. Each stimulus screen must have a unique, correctly named `sender` field. Ensure Practice_Stimulus and Test_Stimulus names do not collide.

---

## Variable Naming Convention

The naming scheme follows the pattern `K[Section][Variable]`:

- **Section** (first digit): Corresponds to the MZP number (1-9)
- **Variable type** (second digit): 03 = Stroop, 05 = Flanker (within each section)
- **Item** (after underscore): 01-07, one per data column

Example: `K503_04` = MZP2_2 (section 5), Stroop (03), Accuracy column (04)

---

## Data Columns Reference

| Column | Variable Suffix | Stroop Values | Flanker Values |
|--------|----------------|---------------|----------------|
| Block | _01 | `practice` / `test` | `practice` / `test` |
| Trial Nr | _02 | `1, 2, ..., 72` | `1, 2, ..., 72` |
| Congruency | _03 | `congruent` / `incongruent` | `congruent` / `incongruent` |
| Accuracy | _04 | `0` (wrong) / `1` (correct) | `0` (wrong) / `1` (correct) |
| RT (ms) | _05 | Reaction time in ms | Reaction time in ms |
| Stimulus | _06 | Color word (rot, blau, gruen, orange) | Arrow orientation (left, right) |
| Response | _07 | Response color key | Response arrow key (left, right) |

---

*Last updated: 2026-03-27*
