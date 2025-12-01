<script setup lang="ts">
import {ref, watch, nextTick, computed} from 'vue';
import {ElMessage} from 'element-plus';
import {
  Document,
  Edit,
  Setting,
  MagicStick,
  Delete,
  Plus,
  RefreshLeft,
  RefreshRight,
  View,
  Download,
  Close,
  Calendar,
  ArrowDown,
  DArrowLeft,
  DArrowRight,
  CircleCheckFilled,
  Checked,
  Filter,
} from '@element-plus/icons-vue';

// Types
interface FormField {
  id: string;
  type: 'input' | 'select' | 'checkbox' | 'textarea' | 'date';
  label: string;
  placeholder?: string;
  required: boolean;
  options?: {label: string; value: string}[];
  value?: any;
}

// State
const components = [
  {type: 'input', label: '单行输入', icon: Edit},
  {type: 'select', label: '下拉选择', icon: ArrowDown},
  {type: 'cascader', label: '级联选择', icon: Filter},
  {type: 'textarea', label: '多行文本', icon: Document},
  {type: 'checkbox', label: '复选框', icon: Checked},
  {type: 'radio', label: '单选框', icon: CircleCheckFilled},
  {type: 'date', label: '日期选择', icon: Calendar},
];

const formSchema = ref<FormField[]>([]);
const selectedField = ref<FormField | null>(null);
const aiQuery = ref('');
const isGenerating = ref(false);
const showAIDialog = ref(false);

// Sidebar State
const isLeftSidebarOpen = ref(true);
const isRightSidebarOpen = ref(true);

const canvasMaxWidth = computed(() => {
  if (!isLeftSidebarOpen.value && !isRightSidebarOpen.value) return 'max-w-6xl';
  if (!isLeftSidebarOpen.value || !isRightSidebarOpen.value) return 'max-w-5xl';
  return 'max-w-4xl';
});

// Actions
const addField = (type: string) => {
  const newField: FormField = {
    id: Math.random().toString(36).substr(2, 9),
    type: type as any,
    label: type === 'input' ? '新单行输入' : type === 'select' ? '新下拉选择' : type === 'checkbox' ? '新复选框' : type === 'textarea' ? '新多行文本' : '新日期选择',
    placeholder: '请输入...',
    required: false,
    options:
      type === 'select'
        ? [
            {label: '选项 1', value: '1'},
            {label: '选项 2', value: '2'},
          ]
        : undefined,
  };
  formSchema.value.push(newField);
  selectedField.value = newField;
};

const selectField = (field: FormField) => {
  selectedField.value = field;
};

const removeField = (index: number) => {
  const field = formSchema.value[index];
  if (field && selectedField.value?.id === field.id) {
    selectedField.value = null;
  }
  formSchema.value.splice(index, 1);
};

// Mock AI Generation
const askAI = async () => {
  if (!aiQuery.value.trim()) return;

  isGenerating.value = true;
  // Simulate network delay
  await new Promise((resolve) => setTimeout(resolve, 1500));

  const query = aiQuery.value.toLowerCase();
  const newSchema: FormField[] = [];

  formSchema.value = [];
  selectedField.value = null;
  isGenerating.value = false;
  showAIDialog.value = false;
  aiQuery.value = '';
  ElMessage.success('表单生成成功！');
};
</script>

<template>
  <div class="flex flex-col h-screen w-full bg-slate-50 text-slate-900 font-sans overflow-hidden selection:bg-indigo-100 selection:text-indigo-700">
    <!-- Header Navigation -->
    <header class="h-16 bg-white border-b border-slate-200 flex items-center justify-between px-6 z-30 shadow-sm shrink-0 relative">
      <!-- Left: Brand -->
      <div class="flex items-center gap-3 w-64">
        <div class="w-9 h-9 bg-indigo-600 rounded-xl flex items-center justify-center text-white shadow-lg shadow-indigo-200">
          <el-icon :size="20"><Document /></el-icon>
        </div>
        <span class="font-bold text-xl text-slate-800 tracking-tight">FormCraft</span>
      </div>

      <!-- Center: Redesigned Toolbar -->
      <div class="absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 flex items-center gap-6">
        <!-- Undo / Redo Group -->
        <div class="flex items-center bg-slate-100/80 p-1 rounded-lg border border-slate-200/50">
          <el-tooltip content="撤销 (Undo)" placement="bottom" :show-after="500">
            <button
              @click="undo"
              class="flex items-center rounded-md text-slate-500 hover:text-indigo-600 hover:bg-white hover:shadow-sm disabled:opacity-30 disabled:hover:bg-transparent disabled:hover:shadow-none transition-all cursor-pointer"
            >
              <el-icon :size="18"><RefreshLeft /></el-icon>
            </button>
          </el-tooltip>
          <div class="w-px h-4 bg-slate-300 mx-1"></div>
          <el-tooltip content="重做 (Redo)" placement="bottom" :show-after="500">
            <button
              @click="redo"
              class="flex items-center rounded-md text-slate-500 hover:text-indigo-600 hover:bg-white hover:shadow-sm disabled:opacity-30 disabled:hover:bg-transparent disabled:hover:shadow-none transition-all cursor-pointer"
            >
              <el-icon :size="18"><RefreshRight /></el-icon>
            </button>
          </el-tooltip>
        </div>

        <!-- Feature Buttons -->
        <div class="flex items-center gap-3">
          <el-button type="primary" color="#4f46e5" round class="!shadow-md !shadow-indigo-100 !border-none hover:!scale-105 transition-transform" @click="showAIDialog = true">
            <el-icon class="mr-1.5"><MagicStick /></el-icon> AI 助手
          </el-button>

          <el-button round plain class="!bg-white hover:!text-indigo-600 hover:!border-indigo-200">
            <el-icon class="mr-1.5"><View /></el-icon> 预览
          </el-button>

          <el-button round plain class="!bg-white hover:!text-indigo-600 hover:!border-indigo-200">
            <el-icon class="mr-1.5"><Download /></el-icon> 导出
          </el-button>
        </div>
      </div>

      <!-- Right Spacer -->
      <div class="w-64 flex justify-end">
        <div class="flex items-center gap-2 text-sm text-slate-400">
          <el-tag size="small" type="info" effect="light" round>v1.0.0</el-tag>
        </div>
      </div>
    </header>

    <div class="flex-1 flex overflow-hidden">
      <!-- Left Sidebar: Component Library -->
      <aside
        class="bg-white border-r border-slate-200 flex flex-col z-20 transition-all duration-300 ease-[cubic-bezier(0.25,0.8,0.25,1)] overflow-hidden"
        :class="[isLeftSidebarOpen ? 'w-72' : 'w-14']"
      >
        <!-- Left Sidebar Header -->
        <div class="h-14 flex items-center justify-between px-4 border-b border-slate-100 shrink-0">
          <span class="font-bold text-slate-800 text-base transition-opacity duration-200 whitespace-nowrap" :class="[isLeftSidebarOpen ? 'opacity-100' : 'opacity-0 w-0 overflow-hidden']">
            组件库
          </span>
          <button
            @click="isLeftSidebarOpen = !isLeftSidebarOpen"
            class="p-1.5 rounded-md text-slate-400 hover:bg-slate-100 hover:text-indigo-600 transition-colors cursor-pointer"
            :title="isLeftSidebarOpen ? '收起' : '展开'"
          >
            <el-icon :size="18"><DArrowLeft v-if="isLeftSidebarOpen" /><DArrowRight v-else /></el-icon>
          </button>
        </div>

        <!-- Left Sidebar Content -->
        <div class="flex-1 overflow-y-auto p-4 min-w-[18rem] transition-opacity duration-300" :class="[isLeftSidebarOpen ? 'opacity-100' : 'opacity-0 pointer-events-none']">
          <div class="grid grid-cols-1 gap-3">
            <div
              v-for="comp in components"
              :key="comp.type"
              @click="addField(comp.type)"
              class="group flex items-center gap-3 p-3 bg-white border border-slate-200 rounded-xl cursor-pointer hover:border-indigo-500 hover:shadow-md hover:-translate-y-0.5 transition-all duration-200"
            >
              <div class="w-8 h-8 rounded-lg bg-slate-50 text-slate-500 flex items-center justify-center group-hover:bg-indigo-50 group-hover:text-indigo-600 transition-colors">
                <component :is="comp.icon" class="w-4 h-4" />
              </div>
              <span class="font-medium text-slate-600 group-hover:text-slate-900 text-sm">{{ comp.label }}</span>
              <el-icon class="ml-auto text-slate-300 group-hover:text-indigo-500"><Plus /></el-icon>
            </div>
          </div>
        </div>
      </aside>

      <!-- Main Area: Canvas -->
      <main class="flex-1 flex flex-col relative bg-slate-100/50 overflow-hidden transition-all duration-300">
        <div class="flex-1 overflow-y-auto p-4 md:p-8 flex justify-center">
          <!-- Dynamic Width Container -->
          <div
            class="w-full overflow-y-auto bg-white rounded-2xl shadow-sm border border-slate-200 min-h-[calc(100vh-10rem)] p-8 md:p-12 transition-all duration-300 relative"
            :class="[canvasMaxWidth]"
          >
            <!-- Watermark / Empty State -->
            <div v-if="formSchema.length === 0" class="absolute inset-0 flex flex-col items-center justify-center text-slate-300 pointer-events-none">
              <el-icon :size="64" class="mb-4 opacity-20"><Document /></el-icon>
              <p class="text-lg font-medium opacity-40">画布为空</p>
              <p class="text-sm opacity-40 mt-2">请从左侧选择组件或点击顶部 AI 助手生成</p>
            </div>

            <el-form v-else label-position="top" class="space-y-6">
              <div class="flex items-center justify-between border-b border-slate-100 pb-6 mb-6">
                <h2 class="text-2xl font-bold text-slate-800">表单预览</h2>
                <div class="flex gap-2">
                  <el-tag type="info" effect="plain" round>{{ formSchema.length }} 个字段</el-tag>
                </div>
              </div>

              <transition-group name="list" tag="div" class="space-y-4">
                <div
                  v-for="(field, index) in formSchema"
                  :key="field.id"
                  class="group relative p-6 rounded-xl border-2 transition-all duration-200 cursor-pointer hover:shadow-md bg-white"
                  :class="[selectedField?.id === field.id ? 'border-indigo-500 ring-4 ring-indigo-500/10 z-10' : 'border-transparent hover:border-indigo-200 hover:bg-indigo-50/30']"
                  @click="selectField(field)"
                >
                  <el-form-item :label="field.label" :required="field.required" class="mb-0 pointer-events-none">
                    <el-input v-if="field.type === 'input'" disabled :placeholder="field.placeholder" />
                    <el-select v-if="field.type === 'select'" disabled :placeholder="field.placeholder || '请选择'" class="w-full" />
                    <el-input v-if="field.type === 'textarea'" type="textarea" disabled :placeholder="field.placeholder" :rows="3" />
                    <el-checkbox v-if="field.type === 'checkbox'" disabled>{{ field.label }}</el-checkbox>
                    <el-date-picker v-if="field.type === 'date'" disabled type="date" :placeholder="field.placeholder || '选择日期'" class="w-full" />
                  </el-form-item>

                  <!-- Floating Actions -->
                  <div class="absolute -top-3 -right-3 opacity-0 group-hover:opacity-100 transition-all duration-200 scale-90 group-hover:scale-100 flex gap-1">
                    <button
                      @click.stop="removeField(index)"
                      class="w-8 h-8 rounded-full bg-red-50 text-red-500 hover:bg-red-500 hover:text-white flex items-center justify-center shadow-sm border border-red-100 transition-colors"
                      title="删除字段"
                    >
                      <el-icon><Delete /></el-icon>
                    </button>
                  </div>
                </div>
              </transition-group>

              <div class="pt-8 border-t border-slate-100 mt-8 flex justify-end">
                <el-button type="primary" size="large" class="!rounded-xl !px-8 !font-medium">提交表单</el-button>
              </div>
            </el-form>
          </div>
        </div>
      </main>

      <!-- Right Sidebar: Properties -->
      <aside
        class="bg-white border-l border-slate-200 flex flex-col shadow-sm z-20 transition-all duration-300 ease-[cubic-bezier(0.25,0.8,0.25,1)] overflow-hidden"
        :class="[isRightSidebarOpen ? 'w-80' : 'w-14']"
      >
        <!-- Right Sidebar Header -->
        <div class="h-14 flex items-center justify-between px-4 border-b border-slate-100 bg-white shrink-0">
          <button
            @click="isRightSidebarOpen = !isRightSidebarOpen"
            class="p-1.5 rounded-md text-slate-400 hover:bg-slate-100 hover:text-indigo-600 transition-colors cursor-pointer"
            :title="isRightSidebarOpen ? '收起' : '展开'"
          >
            <el-icon :size="18"><DArrowRight v-if="isRightSidebarOpen" /><DArrowLeft v-else /></el-icon>
          </button>
          <span class="font-bold text-slate-800 text-base transition-opacity duration-200 whitespace-nowrap" :class="[isRightSidebarOpen ? 'opacity-100' : 'opacity-0 w-0 overflow-hidden']">
            属性配置
          </span>
        </div>

        <div class="flex-1 overflow-y-auto p-6 min-w-[20rem] transition-opacity duration-300" :class="[isRightSidebarOpen ? 'opacity-100' : 'opacity-0 pointer-events-none']">
          <div v-if="selectedField" class="space-y-6 animate-fade-in">
            <div class="bg-indigo-50 rounded-lg p-4 border border-indigo-100 mb-6">
              <div class="text-xs font-semibold text-indigo-500 uppercase tracking-wider mb-1">当前选中</div>
              <div class="font-medium text-indigo-900 capitalize">
                {{
                  selectedField.type === 'input'
                    ? '单行输入'
                    : selectedField.type === 'select'
                    ? '下拉选择'
                    : selectedField.type === 'textarea'
                    ? '多行文本'
                    : selectedField.type === 'checkbox'
                    ? '复选框'
                    : '日期选择'
                }}
              </div>
            </div>

            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-slate-700 mb-1.5">标题 (Label)</label>
                <el-input v-model="selectedField.label" />
              </div>

              <div v-if="['input', 'textarea', 'select'].includes(selectedField.type)">
                <label class="block text-sm font-medium text-slate-700 mb-1.5">占位提示 (Placeholder)</label>
                <el-input v-model="selectedField.placeholder" />
              </div>

              <div class="flex items-center justify-between p-3 bg-slate-50 rounded-lg border border-slate-100">
                <label class="text-sm font-medium text-slate-700">必填项</label>
                <el-switch v-model="selectedField.required" />
              </div>

              <div v-if="selectedField.type === 'select'" class="pt-4 border-t border-slate-100">
                <div class="flex items-center justify-between mb-2">
                  <label class="text-sm font-medium text-slate-700">选项列表</label>
                  <el-button link type="primary" size="small" @click="selectedField.options?.push({label: '新选项', value: 'new'})">添加</el-button>
                </div>
                <div class="space-y-2">
                  <div v-for="(opt, idx) in selectedField.options" :key="idx" class="flex gap-2">
                    <el-input v-model="opt.label" placeholder="标签" size="small" />
                    <el-input v-model="opt.value" placeholder="值" size="small" />
                    <el-button link type="danger" size="small" @click="selectedField.options?.splice(idx, 1)">
                      <el-icon><Delete /></el-icon>
                    </el-button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div v-else class="h-full flex flex-col items-center justify-center text-slate-400 text-center p-8">
            <div class="w-16 h-16 bg-slate-50 rounded-full flex items-center justify-center mb-4">
              <el-icon :size="24"><Setting /></el-icon>
            </div>
            <p class="text-sm font-medium text-slate-500">未选择组件</p>
            <p class="text-xs mt-1">请在画布中点击字段进行配置</p>
          </div>
        </div>
      </aside>
    </div>

    <!-- AI Dialog -->
    <div v-if="showAIDialog" class="fixed inset-0 z-50 flex items-center justify-center p-4">
      <div class="absolute inset-0 bg-slate-900/40 backdrop-blur-sm" @click="showAIDialog = false"></div>
      <div class="relative w-full max-w-lg bg-white rounded-2xl shadow-2xl border border-slate-100 overflow-hidden transform transition-all animate-fade-in-up">
        <!-- Dialog Header -->
        <div class="px-6 py-4 border-b border-slate-100 flex items-center justify-between bg-slate-50/50">
          <div class="flex items-center gap-2 text-indigo-600 font-semibold">
            <el-icon><MagicStick /></el-icon>
            <span>AI 智能生成</span>
          </div>
          <button @click="showAIDialog = false" class="text-slate-400 hover:text-slate-600 transition-colors">
            <el-icon :size="20"><Close /></el-icon>
          </button>
        </div>
        <!-- Dialog Body -->
        <div class="p-6">
          <label class="block text-sm font-medium text-slate-700 mb-2">请描述您想要的表单</label>
          <el-input v-model="aiQuery" type="textarea" :rows="4" placeholder="例如：生成一个用户注册表单，包含用户名、密码、邮箱和同意条款..." class="mb-4" />
          <div class="flex justify-end gap-3">
            <el-button @click="showAIDialog = false">取消</el-button>
            <el-button type="primary" @click="askAI" :loading="isGenerating" color="#4f46e5">
              {{ isGenerating ? '生成中...' : '开始生成' }}
            </el-button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}

.animate-fade-in {
  animation: fadeIn 0.3s ease-out;
}
.animate-fade-in-up {
  animation: fadeInUp 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
