<script>
export let json
export let depth = Infinity
export let _lvl = 0
export let _last = true

const collapsedSymbol = '...'
const getType = (i) => {
  if (i === null) return 'null'
  return typeof i
}

let items
let isArray
let openBracket
let closeBracket
$: {
  items = getType(json) === 'object' ? Object.keys(json) : []
  isArray = Array.isArray(json)
  openBracket = isArray ? '[' : '{'
  closeBracket = isArray ? ']' : '}'
}

let collapsed
$: collapsed = depth < _lvl

const format = (i) => {
  switch (getType(i)) {
    case 'string':
      return `"${i}"`
    case 'function':
      return 'f () {...}'
    case 'symbol':
      return i.toString()
    default:
      return i
  }
}
const clicked = () => {
  collapsed = !collapsed
}
</script>

{#if items.length}
  <span class:hidden={collapsed}>
    <span class="bracket" on:click={clicked} tabindex="0">{openBracket}</span>
    <ul>
      {#each items as i, idx}
        <li>
          {#if !isArray}
            <span class="key">"{i}":</span>
          {/if}
          {#if getType(json[i]) === 'object'}
            <svelte:self json={json[i]} {depth} _lvl={_lvl + 1} _last={idx === items.length - 1} />
          {:else}
            <span class="val {getType(json[i])}"
              >{format(json[i])}{#if idx < items.length - 1}<span class="comma">,</span>{/if}</span
            >
          {/if}
        </li>
      {/each}
    </ul>
    <span class="bracket" on:click={clicked} tabindex="0">{closeBracket}</span>{#if !_last}<span
        class="comma">,</span
      >{/if}
  </span>
  <span class="bracket" class:hidden={!collapsed} on:click={clicked} tabindex="0"
    >{openBracket}{collapsedSymbol}{closeBracket}</span
  >{#if !_last && collapsed}<span class="comma">,</span>{/if}
{/if}

<style>
ul {
  list-style: none;
  margin: 0;
  padding: 0;
  padding-left: var(--nodePaddingLeft, 1rem);
  border-left: var(--nodeBorderLeft, 1px dotted #9ca3af);
  color: var(--nodeColor, #374151);
}
.hidden {
  display: none;
}
.bracket {
  cursor: pointer;
}
.bracket:hover {
  background: var(--bracketHoverBackground, #d1d5db);
}
.comma {
  color: var(--nodeColor, #374151);
}
.val {
  color: var(--leafDefaultColor, #9ca3af);
}
.val.string {
  color: var(--leafStringColor, #059669);
}
.val.number {
  color: var(--leafNumberColor, #d97706);
}
.val.boolean {
  color: var(--leafBooleanColor, #2563eb);
}
</style>
