# Ender's Delight — Arcadia Fix Fork

## What

A patched build of [Ender's Delight 1.2.0](https://www.curseforge.com/minecraft/mc-mods/enders-delight) for use in the **Arcadia V2 'Echoes Of Power'** modpack (MC 1.21.1 / NeoForge).

## Issue fixed

`EndstoneStoveBlockEntity.cookingTick` calls `vectorwing.farmersdelight.common.utility.ItemUtils.isInventoryEmpty(IItemHandler)` which **no longer exists** in **Farmer's Delight 1.3.1**. The method was renamed to `doesInventoryHaveItems(IItemHandler)` (with inverted return semantic — returns true when inventory has items, opposite of the old method).

Result: server crash on every endstone_stove tick :

```
java.lang.NoSuchMethodError: 'boolean vectorwing.farmersdelight.common.utility.ItemUtils.isInventoryEmpty(net.neoforged.neoforge.items.IItemHandler)'
  at com.axedgaming.endersdelight.Common.Blocks.entity.EndstoneStoveBlockEntity.cookingTick(EndstoneStoveBlockEntity.java:79)
```

## Fix

Bytecode-level patch of `EndstoneStoveBlockEntity.cookingTick`:

1. **Constant pool Utf8 entry**: `isInventoryEmpty` → `doesInventoryHaveItems` (length prefix updated 16→22 bytes)
2. **Branch opcode inversion**: `ifne` (0x9A) → `ifeq` (0x99) at the matching call site, since the new method has inverted return semantic

Equivalent source change:

```diff
- if (!ItemUtils.isInventoryEmpty(stove.inventory)) {
+ if (ItemUtils.doesInventoryHaveItems(stove.inventory)) {
      ItemUtils.dropItems(level, pos, stove.inventory);
      stove.inventoryChanged();
  }
```

## Files

- `original.jar` — upstream `endersdelight-1.2.0-1.21.1.jar` (unchanged)
- `patched.jar` — fixed jar shipped in the modpack as `endersdelight-1.2.0-1.21.1-arcadia-fix.jar`
- `extracted/` — extracted contents used to repackage

## Install

Replace `endersdelight-1.2.0-1.21.1.jar` in your `mods/` folder with `patched.jar` (renamed to `endersdelight-1.2.0-1.21.1-arcadia-fix.jar`).

## Credits

Author: vyrriox
