<script>
    import {fly} from "svelte/transition";
    import {createEventDispatcher} from "svelte";
    import VirtualList from "../utils/VirtualList.svelte";
    import LogMessage from "./LogMessage.svelte";
    import ConfigRadioButton from "../config/inputs/ConfigRadioButton.svelte";
    import {invoke} from "@tauri-apps/api";

    export let messages;

    let autoScroll = true;

    const dispatch = createEventDispatcher();

    async function uploadLogs() {
        await invoke("upload_logs", {
            log: messages.join("")
        }).then((result) => {
            console.debug("Received Result", result)
            navigator.clipboard.writeText(result.url)
        }).catch((error) => {
            console.error(error)
        })
    }
</script>

<div class="log" transition:fly={{ y: -10, duration: 200 }}>
    <div class="header">
        <div class="title nes-font">CLIENT LOG</div>
        <div class="title nes-font red-text-clickable" on:click={() => dispatch("hideClientLog")}>X</div>
    </div>

    <div class="output">
        <VirtualList items={messages} let:item {autoScroll}>
            <LogMessage text={item.split("]: ")[1]}/>
        </VirtualList>
    </div>

    <div class="bottom">
        <ConfigRadioButton bind:value={autoScroll} text="Auto Scroll"/>
        <p on:click={uploadLogs}>COPY</p>
    </div>
</div>

<style>
    .log {
        width: calc(100% - 150px);
        height: calc(100% - 150px);
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background-color: var(--background-contrast-color);
        backdrop-filter: blur(10px);
        padding: 25px;
        z-index: 1000;
        display: flex;
        flex-direction: column;
        border: 5px solid black;
    }

    .bottom {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .bottom p {
        font-family: 'Press Start 2P', serif;
        font-size: 20px;
        color: var(--primary-color);
        text-shadow: 2px 2px var(--primary-color-text-shadow);
        user-select: none;
        cursor: pointer;
        transition: transform 0.3s;
    }

    .bottom p:hover {
        transform: scale(1.2);
    }

    .output {
        flex: 1;
        overflow: hidden;
        margin-bottom: 10px;
    }

    .header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 20px;
    }

    .nes-font {
        font-family: 'Press Start 2P', serif;
        font-size: 30px;
        user-select: none;
    }

    .title {
        font-size: 30px;
    }

    .button-hide {
        background-color: transparent;
        border: none;
        cursor: pointer;
    }
</style>

