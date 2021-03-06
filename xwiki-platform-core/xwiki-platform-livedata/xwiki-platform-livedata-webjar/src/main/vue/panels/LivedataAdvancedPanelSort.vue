<!--
* See the NOTICE file distributed with this work for additional
* information regarding copyright ownership.
*
* This is free software; you can redistribute it and/or modify it
* under the terms of the GNU Lesser General Public License as
* published by the Free Software Foundation; either version 2.1 of
* the License, or (at your option) any later version.
*
* This software is distributed in the hope that it will be useful,
* but WITHOUT ANY WARRANTY; without even the implied warranty of
* MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
* Lesser General Public License for more details.
*
* You should have received a copy of the GNU Lesser General Public
* License along with this software; if not, write to the Free
* Software Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA
* 02110-1301 USA, or see the FSF site: http://www.fsf.org.
 -->


<!--
  The LivedataAdvancedPanelSort component is used to provide
  advance sorting, whatever layout is being used.
  It allows:
  - Adding / removing sort levels
  - Reorderin sorts levels
  - Changing sort direction
-->
<template>
  <!--
    Uses the LivedataBaseAdvancedPanel as root element, as it handles for us
    all the Advanced Panels default behavior
  -->
  <LivedataBaseAdvancedPanel
    class="livedata-advanced-panel-sort"
    panel-id="sortPanel"
  >
    <!-- Provide the panel name and icon to the `header` slot -->
    <template #header>
      <span class="fa fa-sort"></span> Sort
    </template>

    <!-- Define panel content inside the `body` slot -->
    <template #body>
      <!--
        The sort entries are wrapped inside a XWikiDraggable component
        in order to allow the user to reorder them easily
      -->
      <XWikiDraggable
        :value="data.query.sort"
        @change="reorderSorts"
      >
        <!--
          Sort entries
          Uses the XWikiDraggableItem component that goes along the
          XWikiDraggable one
        -->
        <XWikiDraggableItem
          class="sort-entry"
          v-for="(sortEntry, level) in data.query.sort"
          :key="level"
        >
          <!-- Property name -->
          <span>{{ logic.getPropertyDescriptor(sortEntry.property).name }}</span>
          <span style="margin: 0 1rem"> - </span>

          <!--
            Direction select
            Allow to select either Ascending or Descending
          -->
          <select
            @change="logic.sort(sortEntry.property, level, $event.target.value === 'true')"
          >
            <option
              value="false"
              :selected="!sortEntry.descending"
            >Ascending</option>
            <option
              value="true"
              :selected="sortEntry.descending"
            >Descending</option>
          </select>

          <!-- Delete sort entry button -->
          <a
            class="delete-sort"
            href="#"
            @click.prevent="logic.removeSort(sortEntry.property)"
            title="Delete Sort"
          >
            <span class="fa fa-trash-o"></span>
          </a>
        </XWikiDraggableItem>
      </XWikiDraggable>


      <!--
        Add Sort
        When a user select an property from this select,
        it directly adds an sort entry in the sort array of the Livedata config.
        If all property are sorting, it is hidden.
      -->
      <select
        class="add-sort-select"
        v-show="unsortedProperties.length > 0"
        @change="addSortLevel($event.target.value)"
      >
        <!--
          This is the default option that is always selected
          When the user change the value of the select,
          it adds the sort corresponging to the option
          then falls back to this default option
        -->
        <option
          value="none"
          ref="selectPropertiesNone"
          selected disabled
        >Add Sort level</option>
        <!--
          Unsorted properties
          Only display in the select properties that are not already sorting
        -->
        <option
          v-for="property in unsortedProperties"
          :key="property.id"
          :value="property.id"
          v-text="property.name"
        ></option>
      </select>

    </template>

  </LivedataBaseAdvancedPanel>
</template>


<script>
import LivedataBaseAdvancedPanel from "./LivedataBaseAdvancedPanel.vue";
import XWikiDraggable from "../utilities/XWikiDraggable.vue";
import XWikiDraggableItem from "../utilities/XWikiDraggableItem.vue";

export default {

  name: "LivedataAdvancedPanelSort",

  components: {
    LivedataBaseAdvancedPanel,
    XWikiDraggable,
    XWikiDraggableItem,
  },

  inject: ["logic"],

  computed: {
    data () { return this.logic.data; },

    // Property descriptors that does not have a sort entry in Livedata config
    unsortedProperties () {
      return this.logic.getSortablePropertyDescriptors()
        .filter(propertyDescriptor => !this.logic.getQuerySort(propertyDescriptor.id));
    },
  },

  methods: {
    // Change event handler called by the add-sort select
    addSortLevel (value) {
      if (value === "none") { return; }
      this.logic.addSort(value);
      this.$refs.selectPropertiesNone.selected = true;
    },
    // Event handler called when sort entries are dragged and dropped
    reorderSorts (e) {
      this.logic.reorderSort(e.moved.element.property, e.moved.newIndex)
      .catch(err => void console.warn(err));
    },
  },

};
</script>


<style>

.livedata-advanced-panel-sort .sort-entry {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: center;
}

.livedata-advanced-panel-sort .draggable-item .handle {
  width: 3rem;
}

.livedata-advanced-panel-sort .delete-sort {
  display: inline-block;
  visibility: hidden;
  margin-left: 5px;
  padding: 6px 10px;
  border-radius: 3px;
  color: currentColor;
}
.livedata-advanced-panel-sort .sort-entry:hover .delete-sort {
  visibility: visible;
}
.livedata-advanced-panel-sort .delete-sort:hover {
  background-color: #ccc4;
}
.livedata-advanced-panel-sort .delete-sort:active {
  background-color: unset;
}

.livedata-advanced-panel-sort .add-sort-select {
  margin-top: 1rem;
}

</style>
