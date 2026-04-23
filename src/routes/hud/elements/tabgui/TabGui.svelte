<script lang="ts">
    import { onMount } from "svelte";
    import type { GroupedModules, Module as TModule } from "../../../../integration/types";
    import { getModules, setModuleEnabled } from "../../../../integration/rest";
    import { groupByCategory } from "../../../../integration/util";
    import Category from "./Category.svelte";
    import { getTextWidth } from "../../../../integration/text_measurement";
    import { listen } from "../../../../integration/ws";
    import { fly } from "svelte/transition";
    import Module from "./Module.svelte";
    import type { KeyEvent, ModuleToggleEvent } from "../../../../integration/events";

    let modules: TModule[] = [];
    let groupedModules: GroupedModules = {};
    let categories: string[] = [];
    let selectedCategoryIndex = 0;
    let selectedModuleIndex = 0;
    let renderedModules: TModule[] = [];
    let categoriesElement: HTMLElement;

    onMount(async () => {
        modules = (await getModules()).filter((m) => m.category !== "Client");
        groupedModules = groupByCategory(modules);

        categories = Object.keys(groupedModules).sort(
            (a, b) =>
                getTextWidth(b, "Inter 14px") - getTextWidth(a, "Inter 14px"),
        );
    });

    async function handleKeyDown(e: KeyEvent) {
        if (e.action !== 1) return;

        switch (e.key) {
            case "key.keyboard.down":
                if (renderedModules.length === 0) {
                    selectedCategoryIndex =
                        (selectedCategoryIndex + 1) % categories.length;
                } else {
                    selectedModuleIndex =
                        (selectedModuleIndex + 1) % renderedModules.length;
                }
                break;

            case "key.keyboard.up":
                if (renderedModules.length === 0) {
                    selectedCategoryIndex =
                        (selectedCategoryIndex - 1 + categories.length) %
                        categories.length;
                } else {
                    selectedModuleIndex =
                        (selectedModuleIndex - 1 + renderedModules.length) %
                        renderedModules.length;
                }
                break;

            case "key.keyboard.left":
                renderedModules = [];
                selectedModuleIndex = 0;
                break;

            case "key.keyboard.right":
                if (renderedModules.length > 0) return;
                renderedModules = groupedModules[categories[selectedCategoryIndex]];
                break;

            case "key.keyboard.enter":
                if (renderedModules.length === 0) return;

                const mod = renderedModules[selectedModuleIndex];
                await setModuleEnabled(mod.name, !mod.enabled);
                break;
        }
    }

    listen("key", handleKeyDown);

    listen("moduleToggle", (e: ModuleToggleEvent) => {
        const mod = modules.find((m) => m.name === e.moduleName);
        if (!mod) return;

        mod.enabled = e.enabled;
        groupedModules = groupByCategory(modules);

        if (renderedModules.length > 0) {
            renderedModules = groupedModules[categories[selectedCategoryIndex]];
        }
    });
</script>

<div class="tabgui-wrapper">


    <div class="title">
        <span class="title-main">Sigma</span>
        <span class="title-version">v3.9</span>
    </div>


    <div class="tabgui">
        <div class="categories" bind:this={categoriesElement}>
            {#each categories as name, index}
                <Category {name} selected={index === selectedCategoryIndex} />
            {/each}
        </div>

        {#if renderedModules.length > 0}
            <div
                class="modules"
                transition:fly={{ x: -10, duration: 200 }}
                style="height: {categoriesElement?.offsetHeight}px"
            >
                {#each renderedModules as { name, enabled }, index}
                    <Module
                        {name}
                        {enabled}
                        selected={selectedModuleIndex === index}
                    />
                {/each}
            </div>
        {/if}
    </div>
</div>

<style lang="scss">
.tabgui-wrapper {
    display: inline-flex;
    flex-direction: column;
    align-items: stretch;
    width: fit-content;


    text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.75);
}


.title {
    position: fixed;
    top: 5px;
    left: 42px;

    display: flex;
    align-items: flex-start;

    padding: 2px 5px;
    padding-left: 7px;

    background: var(--tabgui-modules-background-color);
    border: 1px solid rgba(255, 255, 255, 0.06);

    box-sizing: border-box;
    user-select: none;

    width: fit-content;
    z-index: 999;
}


.title-main {
    color: #d0d0d0;
    font-size: 22px;
    font-weight: 700;
    letter-spacing: 0.4px;
}

.title-version {
    color: #4da3ff;
    font-size: 14px;
    transform: translateY(-3px);
    font-weight: 600;
}


.tabgui {
    display: flex;
    width: fit-content;
    margin-top: 38px;
}

.categories {
    display: flex;
    flex-direction: column;
    overflow: hidden;
}

.modules {
    background-color: var(--tabgui-modules-background-color);
    margin-left: 6px;

    display: flex;
    flex-direction: column;

    min-width: 100px;
    overflow: auto;

    &::-webkit-scrollbar {
        width: 0;
    }
}
</style>
