<template>
  <div class="tree">
    <!-- Keep tree in div to use v-if and keep html cleaner -->
    <div v-if="globalParentNodes.length > 0">
      <node-tree
        v-for="item in globalParentNodes"
        :key="item.id"
        :node="item"
        :tree="tree"
        @add-item="addChildrenToItem($event)"
        ></node-tree>
    </div>
    <!-- If generated array has one node with property parentID -->
    <div v-else>Opps. The tree was generated empty. Please, reload the page</div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator';
import NodeTree from './NodeTree.vue';

@Component({
  components: {
    NodeTree
  }
})
export default class Tree extends Vue {
  tree: any = []

  /* Turn an array to a tree */
  get sortedTree() {
    return this.tree.filter((element: any) => {
      element.children = this.tree.filter((item:any) => item.parentID === element.id)

      return element.parentID == null;
    });
  }

  /* Filter by nodes withour property 'parentID' */
  get globalParentNodes() {
    return this.tree.filter((element: any) => element.parentID === null )
  }

  /* Generate an array with random length, but less then 30, and except 0 */
  generateTree = () => {
    const length = this.generateNumberWithMax(30, 0);
    const result = [];

    for (let i = 0; i < length; i += 1) {
      result.push({
        id: i,
        parentID: !this.randomBoolean() ? this.generateNumberWithMax(length, i) : null, // Random parent
        name: 'item',
        checked: this.randomBoolean()
      });
    }
    return result;
  }

  /* Generate a number with max value, and except one number */
  generateNumberWithMax(max: number, except: number) {
    const result = Math.floor(Math.random() * (max - 0)) + 0;
    return (result === except) ? Math.floor(Math.random() * (max - 0)) + 0 : result;
  }

  /* Generate random number and convert to boolean value*/
  randomBoolean() {
    return !!Math.round(Math.random());
  }

  /* Push new node in tree, to cause changes */
  addChildrenToItem(id: number) {
    this.tree.push({
      id: new Date().getTime(),
      parentID: id,
      name: 'item',
      checked: this.randomBoolean()
    })
  }

  created() {
    this.tree = this.generateTree()
  }
}
</script>
/* Simple styles for easy reading  */
<style lang="sass">
.tree
  max-width: 700px
  margin: 0 auto
  &__item
    display: flex

    margin: 5px 0
    padding: 5px
    
    cursor: default

    white-space: nowrap

    list-style: none
    .tree__item
      position: relative
      list-style: circle
      span
        position: relative
        &::before
          content: ''
          width: 170%
          height: 100%
          border-left: 1px solid black
          border-bottom: 1px solid black
          position: absolute
          left: -230%
          top: -8px
</style>
