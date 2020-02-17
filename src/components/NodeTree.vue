<template>
  <ul class="tree__item">
    <li class="node">
      <span>{{ node.name }}</span>
      <input
        type="checkbox"
        :checked="node.checked"
        :disabled="!!getChildrenOfNode(node.id).length"
        @click="node.checked = !node.checked"
        ref="status">
      <button @click="add">+</button>
    </li>
    <!-- Keep nodes in div to use v-if and keep html cleaner -->
    <div
      class="node__list"
      v-if="getChildrenOfNode(node.id) && getChildrenOfNode(node.id).length">
      <node
        v-for="child in getChildrenOfNode(node.id)"
        @add-item="$emit('add-item', $event)"
        :key="child.id"
        :tree="tree"
        :node="child">
        <!-- Emit action add-item to parent, until it get to Tree component -->
      </node>
    </div>
  </ul>
</template>

<script lang="ts">
import { Component, Vue, Prop, Emit, Watch } from 'vue-property-decorator';
@Component({
  name: 'node',
})
export default class NodeTree extends Vue {
  @Prop(Object) node!: any
  @Prop(Array) tree!: any

  /* Filter by children */
  getChildrenOfNode( id: number ) {
    return this.tree.filter((element: any) => element.parentID === id)
  }

  /* Filter by selected children */
  getCheckedChildrenOfNode( id: number ) {
    return this.getChildrenOfNode(id).filter((element: any) => element.checked)
  }

  /* Filter by selected children and convert to boolean value */
  hasCheckedChildrenOfNode( id: number) {
    return !!this.getCheckedChildrenOfNode(id).length
  }

  /* Emit actions to parent */
  @Emit('add-item')
  add() {
    return this.node.id
  }

  /* Watch changes from global array */
  @Watch('tree', {deep: true})
  change() {
    /* Break if there is no checkbox */
    if(!this.$refs.status) return false
    let input: any = this.$refs.status
    /* Break if there are no children */
    if(!this.getChildrenOfNode(this.node.id).length) return false
    /* If there are selected children */
    if(this.hasCheckedChildrenOfNode(this.node.id)) {
      /* The number of selected children is equal to the number of children */
      if(this.getChildrenOfNode(this.node.id).length === this.getCheckedChildrenOfNode(this.node.id).length) {
        /* Remove the indeterminate from input and add checked to node */
        input.indeterminate = false
        this.node.checked = true
      }
      /* If the number of selected children is not equal to the total number of children */
      else {
        /* Set the indeterminate to input and remove checked from node */
        input.indeterminate = true
        this.node.checked = false
      }
    }
    /* If there are no selected children */
    else {
      /* Remove the indeterminate from input and remove checked from node */
      input.indeterminate = false
      this.node.checked = false
    }
  }

  mounted() {
    this.change()
  }
}
</script>
/* Simple styles for easy reading  */
<style lang="sass" scoped>
.node
  input[type="checkbox"]
    margin: 0 7px
  button
    padding: 0
    
    width: 20px
    height: 20px

    border-radius: 5px
    border: none

    line-height: 20px

    cursor: pointer
  &__list
    margin: 15px 0 0 0
</style>