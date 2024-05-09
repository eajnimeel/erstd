<script>
    import {autoHeight, TextField} from 'svelte-ux';
    import {tick} from "svelte";

    export let value = '';
    let isEditable = false;
    let el;

    const enableEdit = async () => {
        isEditable = true;
        await tick(); // 다음 렌더링 사이클을 기다립니다.
        el.focus();
    }
    const disableEdit = () => {
        isEditable = false;
    }

</script>
<td on:dblclick={enableEdit}>
    {#if isEditable}
        <TextField bind:value multiline actions={(node) => [autoHeight(node)]} on:blur={disableEdit} autofocus/>
    {:else}
        {@html value ? value.replace('\n', '<br>') : ''}
    {/if}
</td>
<style>
    td {
        border: 1pt solid black;
        padding: 0.05em 0.25em;
        min-width: 10em;
    }
</style>
