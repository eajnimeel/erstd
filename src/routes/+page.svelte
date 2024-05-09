<script lang="ts">
    import {Button, Overlay, ProgressCircle, Tooltip} from 'svelte-ux';
    import TdTextField from "$lib/TdTextField.svelte";
    import TdInput from "$lib/TdInput.svelte";

    interface Subject {
        name: string,
        tables: {
            id: string,
            logicalName: string,
            physicalName: string,
            columns: {
                logicalName: string,
                physicalName: string,
                primaryKey: boolean,
                foreignKey: boolean,
                autoIncrement: boolean,
                notNull: boolean,
                dataType: string,
                defaultValue: string | undefined,
                description: string | undefined
            }[]
        }[],
        columnLength: number
    }

    interface Erstd {
        name: string,
        subjects: Subject[]
    }

    let isLoading = false;
    let erd: Erstd = {name: '', subjects: []};
    $: subjects = erd?.subjects;

    let inputImport: HTMLInputElement;
    let inputLoad: HTMLInputElement;

    const startLoading = () => isLoading = true;
    const endLoading = () => isLoading = false;

    const openImport = () => {
        inputImport.click();
    }
    const openLoad = () => {
        inputLoad.click();
    }
    const importErs = (event: Event) => {
        startLoading();
        erd = {name: '', subjects: []};
        const target = event.target;
        const file = target.files[0];
        const reader = new FileReader();
        console.log(file);
        reader.addEventListener('load', () => {
            let text = reader?.result ? reader.result : '';
            doProcess(text)
            endLoading();
        });
        reader.addEventListener('error', function () {
            console.log(reader.error?.message);
            endLoading();
        });
        reader.readAsText(file)
    }

    const doProcess = (text: string) => {
        let json = JSON.parse(text);
        let projectName = json.project.name;
        let entities = json.project.entities;
        let subjects = json.project.subjects.map((subject: { name: string, entityViews: [] }) => {
            let name = subject.name;
            let tables = subject.entityViews.map((entity: { entity_id: string }) => {
                let table = entities.find((e: { id: string }) => e.id === entity.entity_id);
                return {
                    id: table.id,
                    logicalName: table.logicalName,
                    physicalName: table.physicalName,
                    columns: table.attributes
                }
            })
                .sort((a: { logicalName: string }, b: { logicalName: string }) => {
                    return a.logicalName < b.logicalName ? -1 : (a.logicalName > b.logicalName ? 1 : 0)
                });

            return {
                name,
                tables,
                columnLength: tables.reduce((columns: number, table: {
                    columns: []
                }) => columns + table.columns.length, 0)
            }
        })
            .sort();
        console.log(json)

        erd = {
            name: projectName,
            subjects
        }
        console.log(erd)
    }

    const load = (event: Event) => {
        startLoading();
        const target = event.target;
        const file = target.files[0];
        const reader = new FileReader();
        console.log(file);
        reader.addEventListener('load', () => {
            let text = reader?.result ? reader.result : '';
            doLoad(text)
            endLoading();
        });
        reader.addEventListener('error', function () {
            console.log(reader.error?.message);
            endLoading();
        });
        reader.readAsText(file)
    }

    const doLoad = (text: string) => {
        erd = JSON.parse(text);
        console.log(erd)
    }

    const save = () => {
        const blob = new Blob([JSON.stringify(erd)], {type: "text/json;charset=utf-8"});
        const url = window.URL.createObjectURL(blob);

        if (erd.subjects.length === 0) return;

        const a = document.createElement('a');
        a.style.display = 'none';
        a.href = url;
        a.download = erd.name + ".erstd";
        document.body.appendChild(a);

        a.click();

        setTimeout(() => {
            document.body.removeChild(a);
            window.URL.revokeObjectURL(url);
        }, 100);
    }
</script>

<main class="max-w-full m-2">
    <Tooltip title="ers 파일 읽기">
        <Button variant="fill" on:click={openImport}>import ers file</Button>
    </Tooltip>
    <Tooltip title="테이블 정의서 파일(erstd) 읽기">
        <Button variant="fill" on:click={openLoad}>load</Button>
    </Tooltip>
    <Tooltip title="테이블 정의서 파일(erstd) 쓰기">
        <Button variant="fill" on:click={save}>save</Button>
    </Tooltip>
    <div style="display:none">
        <input type="file" name="file" bind:this={inputImport} on:change={importErs} accept=".ers"/>
        <input type="file" name="file" bind:this={inputLoad} on:change={load} accept=".erstd"/>
    </div>

    <div class="mt-1">
        {#if isLoading}
            <Overlay center>
                <ProgressCircle size={100}/>
            </Overlay>
        {/if}
        <table class="w-full">
            <thead>
            <tr>
                <th rowspan="2">subject</th>
                <th rowspan="2">논리 테이블명<br>물리 테이블명</th>
                <th colspan="2">컬럼명</th>
                <th colspan="3">KEY</th>
                <th rowspan="2" class="w-12">Not Null</th>
                <th rowspan="2">데이터 타입</th>
                <th rowspan="2">기본값</th>
                <th rowspan="2">비고</th>
            </tr>
            <tr>
                <th>논리</th>
                <th>물리</th>
                <th class="w-12">PK</th>
                <th class="w-12">FK</th>
                <th class="w-12">AI</th>
            </tr>
            </thead>
            <tbody>
            {#each subjects as subject}
                {#each subject.tables as table, tableIndex}
                    {#each table.columns as column, columnIndex}
                        <tr class="{tableIndex === 0 && columnIndex === 0 ? 'split' : ''}">
                        {#if tableIndex === 0 && columnIndex === 0}
                            <td rowspan="{subject.columnLength}">{subject.name}</td>
                        {/if}
                        {#if columnIndex === 0}
                            <td rowspan="{table.columns.length}">{table.logicalName}<br>({table.physicalName})</td>
                        {/if}
                        <td>{column.logicalName}</td>
                        <td>{column.physicalName}</td>
                        <td class="text-center">{column.primaryKey ? '○' : ''}</td>
                        <td class="text-center">{column.foreignKey ? '○' : ''}</td>
                        <td class="text-center">{column.autoIncrement ? '○' : ''}</td>
                        <td class="text-center">{column.notNull ? '○' : ''}</td>
                        <td>{column.dataType}</td>
                        <TdInput bind:value={column.defaultValue}></TdInput>
                        <TdTextField bind:value={column.description}></TdTextField>
                        </tr>
                    {/each}
                {/each}
            {/each}
            </tbody>
        </table>
    </div>
</main>

<style>
    table, th, td {
        border: 1pt solid black
    }

    th, td {
        padding: 0.05em 0.25em;
    }
    .split {
        border-top: double 0.4em black;
    }
</style>
