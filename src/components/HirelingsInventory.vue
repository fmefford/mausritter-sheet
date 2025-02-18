<template>
  <w-flex row justify-center class="mx4 mt2">
    <div class="inventory-cell">
      <draggable :list="mainPaw" group="items" item-key="id" class="h-max" :move="move">
        <template #header><span class="inventory-cell-label">{{ $t('Main paw') }}</span></template>
        <template #item="{ element }">
          <conditions v-if="element.type === TYPE_CONDITION" size="xl" :condition="element" can-delete @delete="deleteItem(mainPaw, element.id)" />
          <items v-else-if="element.type === TYPE_ITEM && !existCondition(mainPaw)" can-delete show-use show-damage :item="element" @delete="deleteItem(mainPaw, element.id)" @toggle-used="toggleUsed" />
        </template>
      </draggable>
    </div>

    <div class="inventory-cell ml4">
      <draggable :list="pack1" group="items" item-key="id" class="h-max" :move="move">
        <template #header><span class="inventory-cell-label">1</span></template>
        <template #item="{ element }">
          <conditions v-if="element.type === TYPE_CONDITION" size="xl" :condition="element" can-delete @delete="deleteItem(pack1, element.id)" />
          <items v-else-if="element.type === TYPE_ITEM && !existCondition(pack1)" can-delete show-use show-damage :item="element" @delete="deleteItem(pack1, element.id)" @toggle-used="toggleUsed" />
        </template>
      </draggable>
    </div>

    <div class="inventory-cell">
      <draggable :list="pack2" group="items" item-key="id" class="h-max" :move="move">
        <template #header><span class="inventory-cell-label">2</span></template>
        <template #item="{ element }">
          <conditions v-if="element.type === TYPE_CONDITION" size="xl" :condition="element" can-delete @delete="deleteItem(pack2, element.id)" />
          <items v-else-if="element.type === TYPE_ITEM && !existCondition(pack2)" can-delete show-use show-damage :item="element" @delete="deleteItem(pack2, element.id)" @toggle-used="toggleUsed" />
        </template>
      </draggable>
    </div>
  </w-flex>

  <w-flex row justify-center class="mx4">
    <div class="inventory-cell">
      <draggable :list="offPaw" group="items" item-key="id" class="h-max" :move="move">
        <template #header><span class="inventory-cell-label">{{ $t('Off paw') }}</span></template>
        <template #item="{ element }">
          <conditions v-if="element.type === TYPE_CONDITION" size="xl" :condition="element" can-delete @delete="deleteItem(offPaw, element.id)" />
          <items v-else-if="element.type === TYPE_ITEM && !existCondition(offPaw)" can-delete show-use show-damage :item="element" @delete="deleteItem(offPaw, element.id)"  @toggle-used="toggleUsed"/>
        </template>
      </draggable>
    </div>

    <div class="inventory-cell ml4">
      <draggable :list="pack3" group="items" item-key="id" class="h-max" :move="move">
        <template #header><span class="inventory-cell-label">3</span></template>
        <template #item="{ element }">
          <conditions v-if="element.type === TYPE_CONDITION" size="xl" :condition="element" can-delete @delete="deleteItem(pack3, element.id)" />
          <items v-else-if="element.type === TYPE_ITEM && !existCondition(pack3)" can-delete show-use show-damage :item="element" @delete="deleteItem(pack3, element.id)" @toggle-used="toggleUsed" />
        </template>
      </draggable>
    </div>

    <div class="inventory-cell">
      <draggable :list="pack4" group="items" item-key="id" class="h-max" :move="move">
        <template #header><span class="inventory-cell-label">4</span></template>
        <template #item="{ element }">
          <conditions v-if="element.type === TYPE_CONDITION" size="xl" :condition="element" can-delete @delete="deleteItem(pack4, element.id)" />
          <items v-else-if="element.type === TYPE_ITEM && !existCondition(pack4)" can-delete show-use show-damage :item="element" @delete="deleteItem(pack4, element.id)" @toggle-used="toggleUsed" />
        </template>
      </draggable>
    </div>
  </w-flex>
  <w-alert v-if="isEncumbered" class="size--sm my0" error plain>
    {{ $t('You\'re encumbered. You cannot run, and makes all saves with disadvantage.') }}
  </w-alert>
</template>

<script>
import { v4 as uuidv4 } from 'uuid'
import draggable from "vuedraggable"
import { TYPE_CONDITION, TYPE_ITEM, getItemEncumbrance, getItem } from "@/services/items-conditions"
import Conditions from "./Conditions.vue"
import Items from "./Items.vue"

const INVENTORY_ID = [ 'mainPaw', 'offPaw', 'pack1', 'pack2', 'pack3', 'pack4' ]

export default {
  name: "Inventory",
  components: { Conditions, draggable, Items },
  data() {
    return {
      TYPE_CONDITION,
      TYPE_ITEM,
      mainPaw: [],
      offPaw: [],
      pack1: [],
      pack2: [],
      pack3: [],
      pack4: []
    }
  },
  computed: {
    stacks () { return [ this.mainPaw, this.offPaw, this.pack1, this.pack2, this.pack3, this.pack4 ] },
    isEncumbered () {
      const result = this.encumbrance(this.mainPaw) + this.encumbrance(this.offPaw) + this.encumbrance(this.pack1) + this.encumbrance(this.pack2) + this.encumbrance(this.pack3) + this.encumbrance(this.pack4)
      return result > this.stacks.length
    }
  },
  methods: {
    encumbrance (list) {
      let result = 0
      list.forEach(item => {
        result += getItemEncumbrance(item)
      })
      return result
    },
    selfList (list) {
      return list === this.mainPaw || list === this.offPaw || list === this.pack1 || list === this.pack2 || list === this.pack3 || list === this.pack4
    },
    canDrop (element, list) {
      if (list && list.canDrop && !this.selfList(list)) {
        return list.canDrop(element, list)
      }

      // Rules to disallow to drop from drawer:
      // - Forbidden to drop anything if a condition already exists in target
      // - Forbidden to drop a condition if target already contains one
      // - Forbidden to drop an item if target already contains one
      if (!list || !list.length) return true // empty list, we can drop
      let i = 0
      let abort = false
      while (!abort && i < list.length) {
        if (list[i].type === element.type) abort = true
        i++
      }

      if (abort) return false // same type of item is in target or a condition is already in target
      return true
    },
    deleteItem(list, id) {
      const i = this.getIndex(list, id)
      if (i >= 0) {
        list.splice(i, 1) // Remove item from list
      }
    },
    existCondition (list) {
      let i = 0
      let found = false
      while (!found && i < list.length) {
        if (list[i].type === TYPE_CONDITION) found = true
        i++
      }
      return found
    },
    getIndex(list, id) {
      let i = 0
      let found = -1
      while (found === -1 && i < list.length) {
        if (list[i].id === id) found = i
        i++
      }
      return found
    },
    getItemByUuid(list, uuid) {
      let i = 0
      let found
      while (!found && i < list.length) {
        if (list[i].uuid === uuid) found = list[i]
        i++
      }
      return found
    },
    findItemUuid (uuid) {
      let s = 0
      let found
      while (!found && s < this.stacks.length) {
        found = this.getItemByUuid(this.stacks[s], uuid)
        s++
      }
      return found
    },
    move(e) {
      if (e.relatedContext.list && e.relatedContext.list.canDrop) return this.canDrop(e.draggedContext.element, e.relatedContext.list)

      // Rules for internal moves:
      // Forbidden to drop anything if a condition already exists in target
      // Forbidden to drop a condition if target already contains one
      // Forbidden to drop an item if target is not empty
      if (!e.relatedContext.list || !e.relatedContext.list.length) return true // empty list, we can drop
      if (e.draggedContext.element.type === TYPE_ITEM) return false // item can drop on empty location only

      let i = 0
      let abort = false
      while (!abort && i < e.relatedContext.list.length) {
        if (e.relatedContext.list[i].type === e.draggedContext.element.type || e.relatedContext.list.type === TYPE_CONDITION) abort = true
        i++
      }  return !abort
    },
    putItem (itemId, inventoryId, { customLabel, desc } = {}) {
      if (inventoryId === '?') {
        // Search for am empty slot
        if (this.pack1.length === 0) inventoryId = 'pack1'
        else if (this.pack2.length === 0) inventoryId = 'pack2'
        else if (this.mainPaw.length === 0) inventoryId = 'mainPaw'
        else if (this.pack3.length === 0) inventoryId = 'pack3'
        else if (this.pack4.length === 0) inventoryId = 'pack4'
        else if (this.offPaw.length === 0) inventoryId = 'offPaw'
      }

      if (!itemId || !inventoryId) return false

      const item = getItem(itemId)
      if (!INVENTORY_ID.includes(inventoryId)) {
        console.error('[error] inventory putItem, unknown slot:', inventoryId)
        return false
      }
      if (!item && this.canDrop(item, this.$data[inventoryId])) {
        console.error('[error] inventory putItem, item not found:', itemId)
        return false
      }
      if (!this.canDrop(item, this.$data[inventoryId])) {
        console.error('[error] inventory putItem, cannot drop in:', inventoryId, item, this.$data[inventoryId])
        return false
      }

      if (customLabel) item.customLabel = this.$t(customLabel)
      if (desc) item.desc = desc
      this.$data[inventoryId].push({ ...item }) // Clone item in inventory
      return true
    },
    reset (data = {}) {
      this.mainPaw = this.sanitizeList(data.mainPaw || [])
      this.offPaw = this.sanitizeList(data.offPaw || [])
      this.pack1 = this.sanitizeList(data.pack1 || [])
      this.pack2 = this.sanitizeList(data.pack2 || [])
      this.pack3 = this.sanitizeList(data.pack3 || [])
      this.pack4 = this.sanitizeList(data.pack4 || [])
      // Export drop testing method
      this.mainPaw.canDrop = this.canDrop
      this.offPaw.canDrop = this.canDrop
      this.pack1.canDrop = this.canDrop
      this.pack2.canDrop = this.canDrop
      this.pack3.canDrop = this.canDrop
      this.pack4.canDrop = this.canDrop
    },
    sanitizeList (list) {
      list.forEach(item => {
        if (item.type === TYPE_ITEM) {
          item.uuid = item.uuid || uuidv4()
          item.used = item.used || 0
        }
      })
      return list
    },
    serialize () {
      return {
        mainPaw: this.mainPaw,
        offPaw: this.offPaw,
        pack1: this.pack1,
        pack2: this.pack2,
        pack3: this.pack3,
        pack4: this.pack4,
      }
    },
    setData (data) {
      this.reset(data)
    },
    toggleUsed ({ uuid, checked }) {
      const item = this.findItemUuid(uuid)
      if (item) {
        if (checked) item.used++
        else item.used--
      }
    }
  },
  created () {
    this.reset()
  }
}
</script>