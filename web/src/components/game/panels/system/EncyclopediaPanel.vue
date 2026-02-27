<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { NSpin, NEmpty, NIcon, NInput, NInputGroup, NButton } from 'naive-ui'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

type EncItem = {
    id: string
    name: string
    desc: string
    type: string
    grade: string
    color: [number, number, number]
    effect_desc?: string
    attribute?: string
}

type EncData = {
    weapons: EncItem[]
    materials: EncItem[]
    elixirs: EncItem[]
    auxiliaries: EncItem[]
    techniques: EncItem[]
}

const activeTab = ref<'weapons' | 'materials' | 'elixirs' | 'auxiliaries' | 'techniques'>('weapons')
const items = ref<EncData | null>(null)
const loading = ref(true)
const searchQuery = ref('')
const selectedItem = ref<EncItem | null>(null)

onMounted(async () => {
    try {
        const res = await fetch('/api/meta/encyclopedia')
        items.value = await res.json()
    } catch (e) {
        console.error('Failed to load encyclopedia data', e)
    } finally {
        loading.value = false
    }
})

const currentList = computed(() => {
    if (!items.value) return []
    return items.value[activeTab.value] || []
})

const filteredList = computed(() => {
    if (!searchQuery.value) return currentList.value
    const query = searchQuery.value.toLowerCase()
    return currentList.value.filter(item =>
        item.name.toLowerCase().includes(query) ||
        item.desc.toLowerCase().includes(query) ||
        (item.effect_desc && item.effect_desc.toLowerCase().includes(query))
    )
})

function getTabName(key: typeof activeTab.value) {
    const map: Record<string, string> = {
        weapons: t('item.type.weapon', '兵器'),
        materials: t('item.type.material', '材料'),
        elixirs: t('item.type.elixir', '丹药'),
        auxiliaries: t('item.type.auxiliary', '法宝'),
        techniques: t('item.type.technique', '功法')
    }
    return map[key] || key
}

function selectItem(item: EncItem) {
    selectedItem.value = item
}

function getRgbStyle(colorArray: [number, number, number] | undefined) {
    if (!colorArray || colorArray.length !== 3) return 'color: #eee'
    return `color: rgb(${colorArray[0]}, ${colorArray[1]}, ${colorArray[2]})`
}

function getRgbBorder(colorArray: [number, number, number] | undefined) {
    if (!colorArray || colorArray.length !== 3) return 'border-color: #333'
    return `border-color: rgba(${colorArray[0]}, ${colorArray[1]}, ${colorArray[2]}, 0.5)`
}
</script>

<template>
    <div class="encyclopedia-panel">

        <!-- Top Filter Tabs -->
        <div class="enc-tabs">
            <button v-for="tab in ['weapons', 'auxiliaries', 'materials', 'elixirs', 'techniques'] as const" :key="tab"
                class="enc-tab-btn" :class="{ active: activeTab === tab }"
                @click="activeTab = tab; selectedItem = null; searchQuery = ''" v-sound:select>
                {{ getTabName(tab) }}
            </button>
        </div>

        <!-- Main Content Area -->
        <div class="enc-content" v-if="!loading && items">
            <!-- Left side: List -->
            <div class="enc-list-section">
                <div class="enc-search">
                    <n-input-group>
                        <n-input v-model:value="searchQuery" :placeholder="t('ui.search', '搜索...')" clearable />
                        <n-button type="primary">
                            <template #icon>
                                <n-icon>
                                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
                                        <path fill="none" stroke="currentColor" stroke-miterlimit="10" stroke-width="32"
                                            d="M221.09 64a157.09 157.09 0 1 0 157.09 157.09A157.1 157.1 0 0 0 221.09 64Z" />
                                        <path fill="none" stroke="currentColor" stroke-linecap="round"
                                            stroke-miterlimit="10" stroke-width="32" d="M338.29 338.29L448 448" />
                                    </svg>
                                </n-icon>
                            </template>
                        </n-button>
                    </n-input-group>
                </div>

                <div class="enc-list-scroller">
                    <div v-if="filteredList.length === 0" class="enc-empty">
                        <n-empty :description="t('ui.no_data', '暂无数据')" />
                    </div>

                    <div v-else v-for="item in filteredList" :key="item.id" class="enc-list-item"
                        :class="{ active: selectedItem?.id === item.id }" :style="getRgbBorder(item.color)"
                        @click="selectItem(item)" v-sound>
                        <div class="item-header">
                            <span class="item-name" :style="getRgbStyle(item.color)">{{ item.name }}</span>
                            <span class="item-grade">{{ item.grade }}</span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Right side: Details -->
            <div class="enc-detail-section">
                <div v-if="selectedItem" class="detail-card">
                    <h2 :style="getRgbStyle(selectedItem.color)">{{ selectedItem.name }}</h2>
                    <div class="detail-meta">
                        <span class="meta-tag" :style="getRgbBorder(selectedItem.color)">{{ selectedItem.grade }}</span>
                        <span class="meta-tag" v-if="selectedItem.attribute">属性: {{ selectedItem.attribute }}</span>
                        <span class="meta-tag">类别: {{ getTabName(activeTab) }}</span>
                    </div>

                    <div class="detail-desc">
                        <h3>{{ t('ui.desc', '描述') }}</h3>
                        <p>{{ selectedItem.desc }}</p>
                    </div>

                    <div class="detail-effect" v-if="selectedItem.effect_desc">
                        <h3>{{ t('ui.effect', '效果') }}</h3>
                        <p>{{ selectedItem.effect_desc }}</p>
                    </div>

                </div>
                <div v-else class="enc-empty detail-empty">
                    <n-empty :description="t('ui.select_item_to_view', '请在左侧选择项目以查看详情')" />
                </div>
            </div>
        </div>

        <div v-else-if="loading" class="enc-loading">
            <n-spin size="large" />
        </div>

    </div>
</template>

<style scoped>
.encyclopedia-panel {
    display: flex;
    flex-direction: column;
    height: 100%;
    width: 100%;
    max-width: 1000px;
    margin: 0 auto;
}

.enc-tabs {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    padding-bottom: 15px;
}

.enc-tab-btn {
    background: rgba(255, 255, 255, 0.05);
    border: 1px solid rgba(255, 255, 255, 0.1);
    color: #aaa;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.2s;
    font-size: 16px;
}

.enc-tab-btn:hover {
    background: rgba(255, 255, 255, 0.1);
    color: #eee;
}

.enc-tab-btn.active {
    background: rgba(74, 158, 255, 0.2);
    border-color: #4a9eff;
    color: #fff;
}

.enc-content {
    display: flex;
    gap: 20px;
    flex: 1;
    min-height: 0;
    overflow: hidden;
}

.enc-list-section {
    width: 40%;
    display: flex;
    flex-direction: column;
    background: rgba(0, 0, 0, 0.2);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 8px;
    overflow: hidden;
    height: 60vh;
}

.enc-search {
    padding: 15px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    flex-shrink: 0;
}

.enc-list-scroller {
    flex: 1;
    overflow-y: auto;
    padding: 10px;
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.enc-list-item {
    padding: 12px 15px;
    background: rgba(255, 255, 255, 0.03);
    border: 1px solid rgba(255, 255, 255, 0.05);
    border-left-width: 4px;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.2s;
    flex-shrink: 0;
}

.enc-list-item:hover {
    background: rgba(255, 255, 255, 0.08);
    transform: translateX(2px);
}

.enc-list-item.active {
    background: rgba(255, 255, 255, 0.15);
}

.item-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.item-name {
    font-size: 16px;
    font-weight: 500;
}

.item-grade {
    font-size: 12px;
    color: #888;
    background: rgba(0, 0, 0, 0.3);
    padding: 2px 6px;
    border-radius: 4px;
}

.enc-detail-section {
    flex: 1;
    background: rgba(0, 0, 0, 0.2);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 8px;
    padding: 25px;
    overflow-y: auto;
    height: 60vh;
}

.detail-card h2 {
    margin: 0 0 15px 0;
    font-size: 28px;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
}

.detail-meta {
    display: flex;
    gap: 10px;
    margin-bottom: 25px;
    flex-wrap: wrap;
}

.meta-tag {
    background: rgba(0, 0, 0, 0.4);
    border: 1px solid;
    padding: 4px 10px;
    border-radius: 4px;
    font-size: 14px;
    color: #ccc;
}

.detail-desc,
.detail-effect {
    margin-bottom: 25px;
    background: rgba(255, 255, 255, 0.03);
    padding: 15px 20px;
    border-radius: 8px;
    border-left: 3px solid rgba(255, 255, 255, 0.2);
}

.detail-desc h3,
.detail-effect h3 {
    margin: 0 0 10px 0;
    font-size: 16px;
    color: #888;
    text-transform: uppercase;
    letter-spacing: 1px;
}

.detail-desc p,
.detail-effect p {
    margin: 0;
    font-size: 16px;
    line-height: 1.6;
    color: #eee;
}

.enc-empty {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
}

.detail-empty {
    opacity: 0.6;
}

.enc-loading {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
}
</style>
