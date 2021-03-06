<style>
  $header-height: 36px;
  .header {
    font-size: 12px;
    display: flex;
    cursor: default;
    height: $header-height;
    border-bottom: 1px solid #ddd;
    -webkit-app-region: drag;
    &.is-mac {
      padding-left: 80px;
    }
    .tab {
      height: $header-height;
      line-height: $header-height;
      padding: 0 10px;
      padding-right: 0;
      position: relative;
      text-align: center;
      border-left: 1px solid #ddd;
      display: flex;
      -webkit-app-region: no-drag;
      .tab-title {
        color: #999;
        white-space: nowrap;
        -webkit-user-select: none;
        overflow: hidden;
        text-overflow: ellipsis;
        max-width: 220px;
      }
      &:last-child {
        border-right: 1px solid #ddd;
      }
      &.current-tab {
        border-left-color: #1976D2;
        background-color: white;
        border-left-width: 2px;
        .tab-title {
          color: #333;
        }
      }
      &:hover {
        .tab-indicator {
          background-color: rgba(255, 255, 255, 0.84);
          .dot {
            display: none;
          }
          .cross {
            display: block;
          }
          &:hover {
            font-weight: bold;
          }
        }
      }
    }

    &.single-tab {
      .tab {
        .tab-title {
          color: #333;
        }
        &:first-child {
          border-left-color: #1976D2;
          background-color: white;
          border-left-width: 2px;
          .tab-title {
            color: #333;
          }
        }
      }
    }

    .tab-indicator {
      -webkit-user-select: none;
      width: 30px;
      text-align: center;
      height: calc(100% - 1px);
      >span {
        position: relative;
        top: -1px;
      }
      .dot {
        width: 5px;
        height: 5px;
        background-color: #4b89ff;
        border-radius: 50%;
        display: inline-block;
      }
      .cross {
        display: none;
        font-size: 1rem;
        transition: all .3s;
      }
    }
  }
</style>

<template>
  <header class="header"
    :class="{'single-tab': tabs.length === 1, 'is-mac': isMac}"
    @dblclick="createNewTab">
    <div class="tab"
      @click="setCurrentTab($index)"
      data-index="{{ $index }}"
      v-for="tab in tabs"
      :class="{'current-tab': $index === currentTabIndex}"
      drop="handleDragAndDrop"
      v-drag-and-drop>
      <span class="tab-title" v-if="tab && !tab.rename">
        {{ tab.title || 'untitled' }}
      </span>
      <span class="tab-title" v-if="tab && tab.rename">
        <input type="text"
          class="rename-input"
          @dblclick.stop
          @click.stop
          @keyup.enter="renameCurrentFile($event, $index)"
          @keyup.esc="cancelRename($event, $index)"
          :value="tab.title" />
      </span>
      <span class="tab-indicator" @click.stop="closeTab($event, $index)">
        <span class="dot" v-show="!tab.saved"></span>
        <span class="cross">×</span>
      </span>
    </div>
  </header>
</template>

<script>
  import path from 'path'
  import {isMac} from 'utils/os'
  import event from 'utils/event'

  export default {
    vuex: {
      getters: {
        tabs: state => state.editor.tabs.map(tab => {
          return {
            title: path.basename(tab.filePath),
            saved: tab.saved,
            rename: tab.rename
          }
        }),
        currentTabIndex: state => state.editor.currentTabIndex
      },
      actions: {
        setCurrentTab({dispatch}, index) {
          dispatch('SET_CURRENT_TAB', index)
          setTimeout(() => {
            event.emit('focus-current-tab')
          }, 200)
        },
        handleDragAndDrop({dispatch}, draggedElement, droppedOnElement) {
          const newIndex = droppedOnElement.getAttribute('data-index')
          const oldIndex = draggedElement.getAttribute('data-index')
          dispatch('REORDER_TABS', {
            newIndex: Number(newIndex),
            oldIndex: Number(oldIndex)
          })
        }
      }
    },
    data() {
      return {isMac}
    },
    methods: {
      closeTab(e, index) {
        event.emit('close-tab', index)
      },
      createNewTab() {
        event.emit('new-tab')
      },
      renameCurrentFile(e, index) {
        const name = e.target.value
        if (name) {
          event.emit('file-rename', name, index)
        } else {
          this.$store.dispatch('UPDATE_RENAME_STATUS', {
            index,
            rename: false
          })
        }
      },
      cancelRename(e, index) {
        this.$store.dispatch('UPDATE_RENAME_STATUS', {
          index,
          rename: false
        })
      }
    }
  }
</script>
