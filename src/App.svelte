<script lang="ts">
  let jsonInput = $state('');
  let error = $state('');

  function escapeCSV(value: string): string {
    const str = String(value ?? '');
    if (str.includes(',') || str.includes('"') || str.includes('\n') || str.includes('\r')) {
      return '"' + str.replace(/"/g, '""') + '"';
    }
    return str;
  }

  function convert() {
    error = '';
    let parsed: any;

    try {
      parsed = JSON.parse(jsonInput.trim());
    } catch {
      error = 'JSON tidak valid. Periksa kembali format JSON yang dimasukkan.';
      return;
    }

    const items = parsed?.response?.data?.calendar?.items;
    if (!items || typeof items !== 'object') {
      error = 'Struktur JSON tidak sesuai. Pastikan ada path: response.data.calendar.items';
      return;
    }

    const headers = ['id', 'message', 'photo_thumb', 'photo_original', 'channel_network', 'channel_type', 'channel_id', 'channel_date'];
    const rows: string[] = [headers.join(',')];

    for (const key of Object.keys(items)) {
      const item = items[key];
      if (!item.photo?.thumb || !item.photo?.original) continue;
      const row = [
        escapeCSV(item.id ?? ''),
        escapeCSV(item.message ?? ''),
        escapeCSV(item.photo?.thumb ?? ''),
        escapeCSV(item.photo?.original ?? ''),
        escapeCSV(item.channel?.network ?? ''),
        escapeCSV(item.channel?.type ?? ''),
        escapeCSV(item.channel?.id ?? ''),
        escapeCSV(item.channel?.date ?? ''),
      ];
      rows.push(row.join(','));
    }

    const csvContent = rows.join('\n');
    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'calendar_export.csv';
    a.click();
    URL.revokeObjectURL(url);
  }

  function clearInput() {
    jsonInput = '';
    error = '';
  }
</script>

<main>
  <div class="container">
    <header>
      <h1>VBOUT Result Converter</h1>
      <p>Paste JSON response dari VBOUT, lalu klik Convert untuk mengunduh file CSV.</p>
    </header>

    <div class="editor">
      <div class="label-row">
        <label for="json-input">JSON Input</label>
        {#if jsonInput}
          <button class="btn-clear" onclick={clearInput}>Hapus</button>
        {/if}
      </div>
      <textarea
        id="json-input"
        bind:value={jsonInput}
        placeholder='Paste JSON di sini...'
        spellcheck="false"
      ></textarea>
    </div>

    {#if error}
      <div class="error">{error}</div>
    {/if}

    <button class="btn-convert" onclick={convert} disabled={!jsonInput.trim()}>
      Convert & Download CSV
    </button>
  </div>
</main>
