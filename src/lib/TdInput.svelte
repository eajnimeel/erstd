<script>
    import {Input} from 'svelte-ux';

    export let value = '';
    let isEditable = false;

    const enableEdit = async () => {
        isEditable = true;
    }
    const disableEdit = () => {
        isEditable = false;
    }

</script>
<td on:dblclick={enableEdit}>
    {#if isEditable}
        <Input bind:value on:blur={disableEdit} autofocus/>
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
