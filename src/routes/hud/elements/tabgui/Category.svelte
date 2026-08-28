<script lang="ts">

    export let name: string;
    export let selected: boolean;
</script>

<div class="category" class:selected>
    <div class="name">
        {name}
    </div>
</div>

<style lang="scss">

    .name {
        font-weight: 500;
        color: var(--tabgui-text-color);
        font-size: 14px;
        width: 100%;
        padding: 2px 12px 1px 11px;

        background: linear-gradient(
            to left,
            var(--tabgui-category-background-color) 50%,
            var(--tabgui-category-active-background-color) 50%
        );
        background-size: 200% 100%;
        background-position: right bottom;
        will-change: background-position;
        transition: background-position 0.2s ease-out;
        overflow: hidden;
    }

    .category {
        display: flex;

        &.selected .name {
            background-position: left bottom;
        }
    }

</style>
