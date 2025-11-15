<template>
  <div class="app-container">
    <h1>OSINT Tools</h1>
    <input
      type="text"
      v-model="searchQuery"
      placeholder="Recherche (min 2 lettres)"
      aria-label="Recherche"
    />
    <div class="tree-container">
      <ul
        class="tree-column"
        v-for="(nodes, level) in displayedColumns"
        :key="level"
      >
        <li
          v-for="node in nodes"
          :key="node.name"
          :class="['node-item', node.type]"
          @click="onNodeClick(node, level)"
          tabindex="0"
        >
          <span>{{ node.name || (node.type === 'folder' ? 'Dossier' : 'Élément') }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

interface TreeNode {
  name?: string
  url?: string
  type: 'folder' | 'url'
  children?: TreeNode[]
}

const preferredFiles = [
  { file: 'json/username.json' },
  { file: 'json/email_address.json' },
  { file: 'json/images_videos_docs.json' },
  { file: 'json/telephone_numbers.json' },
  { file: 'json/social_networks.json' },
  { file: 'json/geolocation_tools_maps.json' },
  { file: 'json/ip_and_mac_address.json' },
  { file: 'json/search_engines.json' },
  { file: 'json/ai_tools.json' },
  { file: 'json/archives.json' },
  { file: 'json/business_records.json' },
  { file: 'json/classifieds.json' },
  { file: 'json/dark_web.json' },
  { file: 'json/dating.json' },
  { file: 'json/digital_currency.json' },
  { file: 'json/documentation_evidence_capture.json' },
  { file: 'json/domain_name.json' },
  { file: 'json/encoding_decoding.json' },
  { file: 'json/exploits_and_advisories.json' },
  { file: 'json/forums_blogs_irc.json' },
  { file: 'json/instant_messaging.json' },
  { file: 'json/language_translation.json' },
  { file: 'json/malicious_file_analysis.json' },
  { file: 'json/metadata.json' },
  { file: 'json/mobile_emulation.json' },
  { file: 'json/opsec.json' },
  { file: 'json/people_search_engines.json' },
  { file: 'json/_records.json' },
  { file: 'json/terrorism.json' },
  { file: 'json/threat_intelligence.json' },
  { file: 'json/tools.json' },
  { file: 'json/training.json' },
  { file: 'json/transportation.json' }
]

const rootNodes = ref<TreeNode[]>([])
const displayedColumns = ref<TreeNode[][]>([])
const searchQuery = ref('')

async function fetchJson(file: string): Promise<TreeNode | null> {
  try {
    const resp = await fetch(file)
    if (!resp.ok) throw new Error()
    return await resp.json()
  } catch {
    return null
  }
}

function filterNodes(nodes: TreeNode[], query: string): TreeNode[] {
  const q = query.toLowerCase()
  return nodes.reduce<TreeNode[]>((results, node) => {
    const name = node.name?.toLowerCase() ?? ''
    if (name.includes(q)) results.push(node)
    else if (node.children) {
      const filtered = filterNodes(node.children, query)
      if (filtered.length) results.push({ ...node, children: filtered })
    }
    return results
  }, [])
}

function onNodeClick(node: TreeNode, level: number) {
  if (node.type === 'url' && node.url) window.open(node.url, '_blank')
  else if (node.type === 'folder' && node.children?.length) {
    displayedColumns.value = displayedColumns.value.slice(0, level + 1)
    displayedColumns.value.push(node.children)
  }
}

async function loadTree() {
  rootNodes.value = []
  for (const { file } of preferredFiles) {
    const data = await fetchJson(file)
    if (data) rootNodes.value.push(data)
  }
  displayedColumns.value = []
  displayedColumns.value.push(rootNodes.value)
}

watch(searchQuery, val => {
  const v = val.trim()
  if (v.length >= 2) displayedColumns.value = [filterNodes(rootNodes.value, v)]
  else displayedColumns.value = [rootNodes.value]
})

onMounted(loadTree)
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');

.app-container {
  font-family: 'Inter', system-ui, Avenir, Helvetica, Arial, sans-serif;
  padding: 2rem;
  min-height: 100vh;
  text-align: left;
}

h1 {
  font-weight: 800;
  font-size: 2.1rem;
  margin-bottom: 1.5rem;
  color: #fff;
  letter-spacing: 0.01em;
}

input[type="text"] {
  display: block;
  margin-bottom: 1.5rem;
  padding: 13px 18px;
  width: 100%;
  max-width: 450px;
  font-size: 1.1rem;
  border: none;
  border-radius: 6px;
  background-color: #434357;
  color: #ececec;
  font-weight: 500;
  outline: none;
  font-family: inherit;
}

input[type="text"]:focus {
  box-shadow: 0 0 10px #77aaff44;
}

.tree-container {
  display: flex;
  gap: 14px;
  background: #282838;
  padding: 18px 10px;
  border-radius: 8px;
  overflow-x: auto;
  min-height: 370px;
  box-sizing: border-box;
  width: 100%;
}

.tree-column {
  list-style: none;
  padding-left: 0.5rem;
  border-left: 1.5px solid #ffffffff;
  min-width: 220px;
  max-width: 280px;
  margin: 0;
}

.tree-column:first-child {
  border-left: none;
}

.node-item {
  cursor: pointer;
  padding: 8px 5px;
  border-radius: 5px;
  color: #dbdbef;
  font-size: 1.09rem;
  font-weight: 500;
  font-family: inherit;
  outline: none;
  user-select: none;
  background: transparent;
  border: none;
  display: flex;
  align-items: center;
  transition: background 0.18s, color 0.18s;
}

.node-item.folder::before { content: "📁 "; margin-right: 2px;}
.node-item.url::before { content: "🔗 "; margin-right: 2px;}

.node-item:hover, .node-item:focus {
  background-color: #466cce;
  color: #ffffff;
}
.node-item:active {
  background-color: #2e497a;
}
</style>
