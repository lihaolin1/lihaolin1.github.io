<script>
  export let columns;
  export let rows;
  export let size;
  export let displayCLSToken = false;
  export let selectedColumn = Math.round(columns / 2);
  export let selectedRow = Math.round(rows / 2);
  export let onChangeRowColumn;

  let clsShift = displayCLSToken ? 1 : 0;

  let active = true;

  const onMouseOver = (col, row) => {
    if (active) {
      onChangeRowColumn(row, col);
    }
  };

  const onClick = (col, row) => {
    active = !active;
    onMouseOver(col, row);
  };
</script>

<style>

</style>

<main>
  <svg width={(columns + clsShift) * size} height={rows * size}>
    <!-- CLS square -->
    {#if displayCLSToken}
      <rect
        x={0}
        y={0}
        width={size}
        height={size}
        style={selectedColumn == null && selectedRow == null ? 'stroke:black;stroke-width:2;fill:white;fill-opacity:0' : 'fill:grey;fill-opacity:0.4'}
        on:mousemove={() => onMouseOver(null, null)}
        on:click={() => onClick(null, null)} />
    {/if}
    {#each [...Array(columns).keys()] as col}
      {#each [...Array(rows).keys()] as row}
        <rect
          x={(col + clsShift) * size}
          y={row * size}
          width={size}
          height={size}
          style={col == selectedColumn && row == selectedRow ? 'stroke:black;stroke-width:2;fill:white;fill-opacity:0' : 'fill:white;fill-opacity:0.2'}
          on:mousemove={() => onMouseOver(col, row)}
          on:click={() => onClick(col, row)} />
      {/each}
    {/each}
  </svg>
</main>
