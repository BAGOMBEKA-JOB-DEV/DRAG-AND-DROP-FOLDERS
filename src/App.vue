<template>
  <div class="file-system" @contextmenu.prevent="openContextMenu($event, null)">
    <div
      v-for="item in fileSystem"
      :key="item.id"
      class="file-item"
      :class="{ folder: item.type === 'folder' }"
      draggable="true"
      @dragstart="dragStart(item, $event)"
      @dragover.prevent
      @drop="dropItem(item, $event)"
      @contextmenu.prevent="openContextMenu($event, item)"
    >
      <span class="icon">{{ item.type === 'folder' ? '📁' : '📄' }}</span>
      <span class="name">{{ item.name }}</span>
      <div v-if="item.type === 'folder'" class="folder-contents">
        <div
          v-for="child in item.children"
          :key="child.id"
          class="file-item"
          :class="{ folder: child.type === 'folder' }"
          draggable="true"
          @dragstart="dragStart(child, $event)"
          @dragover.prevent
          @drop="dropItem(child, $event)"
          @contextmenu.prevent="openContextMenu($event, child)"
        >
          <span class="icon">{{ child.type === 'folder' ? '📁' : '📄' }}</span>
          <span class="name">{{ child.name }}</span>
        </div>
      </div>
    </div>
    
    <div v-if="contextMenu.visible" class="context-menu" :style="{ top: contextMenu.y + 'px', left: contextMenu.x + 'px' }">
      <ul>
        <li @click="openModal('file', contextMenu.target)">Create File</li>
        <li @click="openModal('folder', contextMenu.target)">Create Folder</li>
      </ul>
    </div>
    
    <div v-if="modal.visible" class="modal">
      <div class="modal-content">
        <h3>Create {{ modal.type }}</h3>
        <input v-model="modal.name" placeholder="Enter name" />
        <button @click="confirmCreate">Create</button>
        <button @click="modal.visible = false">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  setup() {
    const fileSystem = ref([{ id: 1, type: 'folder', name: 'Documents', children: [] }]);
    const contextMenu = ref({ visible: false, x: 0, y: 0, target: null });
    const modal = ref({ visible: false, type: '', name: '', target: null });
    const draggedItem = ref(null);
    
    const openContextMenu = (event, item) => {
      contextMenu.value = { visible: true, x: event.clientX, y: event.clientY, target: item };
    };
    
    document.addEventListener('click', () => contextMenu.value.visible = false);
    
    const openModal = (type, target) => {
      modal.value = { visible: true, type, name: '', target };
      contextMenu.value.visible = false;
    };
    
    const confirmCreate = () => {
      if (!modal.value.name) return;
      const newItem = { id: Date.now(), type: modal.value.type, name: modal.value.name, children: modal.value.type === 'folder' ? [] : undefined };
      if (modal.value.target && modal.value.target.type === 'folder') {
        modal.value.target.children.push(newItem);
      } else {
        fileSystem.value.push(newItem);
      }
      modal.value.visible = false;
    };
    
    const dragStart = (item) => draggedItem.value = item;
    
    const dropItem = (target) => {
      if (draggedItem.value && draggedItem.value.id !== target.id && target.type === 'folder') {
        removeItem(draggedItem.value);
        target.children.push(draggedItem.value);
        draggedItem.value = null;
      }
    };
    
    const removeItem = (item) => {
      const findAndRemove = (arr) => {
        const index = arr.findIndex(i => i.id === item.id);
        if (index > -1) arr.splice(index, 1);
      };
      findAndRemove(fileSystem.value);
      fileSystem.value.forEach(folder => folder.type === 'folder' && findAndRemove(folder.children));
    };
    
    return { fileSystem, contextMenu, modal, openContextMenu, openModal, confirmCreate, dragStart, dropItem };
  }
};
</script>

<style>
.file-system {
  width: 800px;
  height: 800px;
  background: #333;
  color: white;
  position: relative;
  padding: 10px;
  display: flex;
  flex-wrap: wrap;
}
.file-item {
  padding: 10px;
  margin: 5px;
  background: #555;
  border-radius: 5px;
  cursor: pointer;
}
.file-item.folder { background: #777; }
.context-menu {
  position: absolute;
  background: #222;
  color: white;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0 0 5px black;
}
.context-menu ul { margin: 0; padding: 0; list-style: none; }
.context-menu li { padding: 5px; cursor: pointer; }
.context-menu li:hover { background: #444; }
.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 0 10px black;
}
.modal-content { text-align: center; }
.modal input { margin: 10px; }
.modal button { margin: 5px; }
</style>
